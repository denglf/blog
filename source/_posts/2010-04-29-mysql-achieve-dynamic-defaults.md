title: mysql实现动态默认值
date: 2010-04-29
author: denglf
layout: post
categories: [Mysql]
tags: [Mysql, 触发器]
---
大家是否遇到这样的需求，在一个table中需要同时存一个值和与它关联的值，比如一个varchar和它的MD5值。mysql中列的默认值必须为常数。你不能使用default子句定义指向函数调用（或其它任意表达式）的列，而且也不能定义根据其他列赋初值的列，并且也不能定义根据其他列赋初值的列。这表示下面的每个列定义都不是合法的：
<!--more-->
```mysql
sys_date date default now(),
doc varchar(255),
hashval char(32) default md5(doc)
```

但是，可以通过创建合适的触发器来绕开此限制，这样就能根据需要来初始化列。触发器能有效得定义动态的默认列值。

`before insert`这个触发器类型能够在插入数据之前设定列值。

首先创建表tri_test:

```mysql
create table tri_test
(
    id int(11) auto_increment not null,
    sys_date date,
    doc varchar(255),
    hashval char(32),
    PRIMARY KEY (id)
);
```

下一步处理插入，先创建`before insert`触发器，使用doc来计算hashval并存储在表中：

```mysql
create trigger doc_tri_test before insert on tri_test
for each row set new.hashval = md5(new.doc) and sys_date = now();
```

在触发器中`new.col_name`指向一个将被插入到指定列的新值。通过触发器中将值赋给`new.col_name`，使新行中对应列具有该值。

测试一下：

```mysql
insert into tri_test (doc) values("test before insert");
select * from tri_test;
```

结果如下：

| id | sys_date   |       doc          |            hashval               |
|:--:|:----------:|:------------------:|:--------------------------------:|
|  1 | 2010-04-29 | test before insert | eb2ac81d7af6c2cc1d1f366427a545b9 |

即使insert没有提供sys_date和hashval的值，它们仍会被初始化。这样就能用任意表达式产生的动态值初始化列了。`before insert`在一定程度上弥补的default的不足。
