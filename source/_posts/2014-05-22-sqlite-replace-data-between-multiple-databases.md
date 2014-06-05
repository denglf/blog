title: SQLite多数据库之间替换数据
date: 2014-05-22 17:56:31
categories: [Sqlite]
tags: [sqlite]
---
SQLite支持跨数据库查询。

SQLite解释器中的提供的`attach [database]`关键字完成附加数据库：
<!--more-->
```sql
attach database [database path] as [alias];
```
* [database path]：
  - 绝对路径。
  - 相对路径，即要附加的数据库与当前维持连接数据库的相对地址。
* [alias]：附加数据库的别名。查询时加上`数据库别名`即可 ： 
```sql
select * from db_alias.table_name where ...; 
```

SQLite多数据库之间替换数据的示例如下：

连接数据库`trac.db`：

```sh
$ sqlite3 trac.db
```

加载数据库`trac_bak.db`：

```sql
attach database "trac_bak.db" as bak;
```

由于数据表`proj_info`的`proj_name_zh`丢失，需要从备份数据库中补充数据。`proj_info`表有两个字段`proj_name`和`proj_name_zh`。

```sql
insert or replace into proj_info
    select proj_info_bak.*
    from proj_info
    left outer join bak.proj_info as proj_info_bak 
        on proj_info.proj_name = proj_info_bak.proj_name;
```
