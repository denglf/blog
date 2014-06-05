title: 事务的多线程调度
date: 2009-12-30
author: denglf
layout: post
categories: [Erlang]
tags: [Erlang, Mysql, 事务, 多线程]
---
最近一直在忙数据库驱动的改造，系统是erlang开发，数据库是mysql。之前的数据库驱动一直是用的erlyweb的erlydb，基于MVC模式，整个Model层由erlydb封装，用起来十分方便。
<!--more-->
erlydb连接数据库是基于tcp的实现，中间又有大量拼装字符串的操作，这样的io和字符串操作恰恰是erlang低效的部分。所以性能是erlydb最大的问题。在高并发的erlang系统中，这样的数据库操作就成为了瓶颈。

所幸erlang支持内联驱动，用C处理数据库连接和字符串拼接操作就高效的多。在这里直接由C调用mysql的API，当然C driver是以多线程实现和异步返回数据的。
![](/images/2009-12-30-multithread-scheduling-transaction/Capture.PNG)

线程在sql队列中取sql指令，然后在连接池中取连接，执行完sql语句就将结果异步返回。在对数据库的基本操作中，事务是比较麻烦的一个。因为一个事务只能在同一个连接中操作，而线程不管你的sql是不是事务里的，只负责取连接执行。所以要保证同一个事务使用同一个连接就必须有一个门牌号，也就是连接id来维系。事务begin的时候，将获得的连接id传回给erlang，并锁住连接，然后事务中的sql操作发送命令给驱动的时候带上连接id作为参数，线程取到这个sql命令的时候发现有连接的id，就根据id直接去取这个连接，直到事务提交或回滚的时候去掉连接的锁定，这样其它的sql就能使用这个连接了。

erlang事务的代码如下：

```erlang
transaction(Fun) ->
    case command(?DRV_BEGIN, "BEGIN") of
        {error, _} = Err ->
            {aborted, Err};
        {ok, Conn} ->
            case catch Fun(Conn) of
                error = Err -> rollback(Conn, Err);
                {error, _} = Err -> rollback(Conn, Err);
                {'EXIT', _} = Err -> rollback(Conn, Err);
            Res ->
                case command(?DRV_COMMIT, Conn) of
                    {error, _} = Err ->
                        rollback(Conn, {commit_error, Err});
                    _ ->
                        case Res of
                            {atomic, _} -> Res;
                            _ -> {atomic, Res}
                        end
                    end
                end
    end.

transaction_fun(Conn) ->
    io:format("execute = ~p~n",
    [execute("insert into user (name, age) values('transaction', 23)", Conn)]),
    io:format("execute = ~p~n",
    [execute("update user set age=100 where name='transaction cc'", Conn)]),
    io:format("execute = ~p~n",
    [execute("insert into user (id, age) values(1, 20)", Conn)]).

test_transaction() ->
    transaction(fun transaction_fun/1).
```

现在的线程调度就变成：
![](/images/2009-12-30-multithread-scheduling-transaction/Capture1.PNG)

这样就解决的了死锁的问题，又大大提高了效率。
