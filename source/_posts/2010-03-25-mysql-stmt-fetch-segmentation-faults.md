title: mysql_stmt_fetch的Segmentation faults
date: 2010-03-25
author: denglf
layout: post
categories: [Mysql]
tags: [C++, Mysql, prepare statement, Segmentation faults]
---
最近遇到一个超级难缠的bug，调了一周才找出原因，下面是[Mysql官方文档](http://dev.mysql.com/doc/refman/5.1/zh/apis.html#mysql-stmt-fetch)的一段代码，介绍了如何使用`mysql_stmt_result_metadata()`、`mysql_stmt_bind_result()`和`mysql_stmt_fetch()`从表中获取数据。
<!--more-->
```c++
#define STRING_SIZE 50
#define SELECT_SAMPLE "SELECT col1, col2, col3, col4 FROM test_table"
MYSQL_STMT *stmt;
MYSQL_BIND bind[4];
MYSQL_RES *prepare_meta_result;
MYSQL_TIME ts;
unsigned long length[4];
int param_count, column_count, row_count;
short small_data;
int int_data;
char str_data[STRING_SIZE];
my_bool is_null[4]; /* Prepare a SELECT query to fetch data from test_table */
stmt = mysql_stmt_init(mysql);
if (!stmt) {
    fprintf(stderr, "mysql_stmt_init(), out of memoryn");
    exit(0);
}
if (mysql_stmt_prepare(stmt, SELECT_SAMPLE, strlen(SELECT_SAMPLE))) {
    fprintf(stderr, "mysql_stmt_prepare(), SELECT failedn");
    fprintf(stderr, "%sn", mysql_stmt_error(stmt));
    exit(0);
}
fprintf(stdout, "prepare, SELECT successfuln");
/* Get the parameter count from the statement */
param_count = mysql_stmt_param_count(stmt);
fprintf(stdout, "total parameters in SELECT: %dn", param_count);
if (param_count != 0) /* validate parameter count */ {
    fprintf(stderr, "invalid parameter count returned by MySQLn");
    exit(0);
}
/* Fetch result set meta information */
prepare_meta_result = mysql_stmt_result_metadata(stmt);
if (!prepare_meta_result) {
    fprintf(stderr,
            "mysql_stmt_result_metadata(), returned no meta informationn");
    fprintf(stderr, "%sn", mysql_stmt_error(stmt));
    exit(0);
}
/* Get total columns in the query */
column_count = mysql_num_fields(prepare_meta_result);
fprintf(stdout, "total columns in SELECT statement: %dn", column_count);
if (column_count != 4) /* validate column count */ {
    fprintf(stderr, "invalid column count returned by MySQLn");
    exit(0);
}
/* Execute the SELECT query */
if (mysql_stmt_execute(stmt)) {
    fprintf(stderr, "mysql_stmt_execute(), failedn");
    fprintf(stderr, "%sn", mysql_stmt_error(stmt));
    exit(0);
}
/* Bind the result buffers for all 4 columns before fetching them */
memset(bind, 0, sizeof (bind));
/* INTEGER COLUMN */
bind[0].buffer_type = MYSQL_TYPE_LONG;
bind[0].buffer = (char *) &int_data;
bind[0].is_null = &is_null[0];
bind[0].length = &length[0]; /* STRING COLUMN */
bind[1].buffer_type = MYSQL_TYPE_STRING;
bind[1].buffer = (char *) str_data;
bind[1].buffer_length = STRING_SIZE;
bind[1].is_null = &is_null[1];
bind[1].length = &length[1]; /* SMALLINT COLUMN */
bind[2].buffer_type = MYSQL_TYPE_SHORT;
bind[2].buffer = (char *) &small_data;
bind[2].is_null = &is_null[2];
bind[2].length = &length[2]; /* TIMESTAMP COLUMN */
bind[3].buffer_type = MYSQL_TYPE_TIMESTAMP;
bind[3].buffer = (char *) &ts;
bind[3].is_null = &is_null[3];
bind[3].length = &length[3]; /* Bind the result buffers */
if (mysql_stmt_bind_result(stmt, bind)) {
    fprintf(stderr, "mysql_stmt_bind_result() failedn");
    fprintf(stderr, "%sn", mysql_stmt_error(stmt));
    exit(0);
}
/* Now buffer all results to client */
if (mysql_stmt_store_result(stmt)) {
    fprintf(stderr, "mysql_stmt_store_result() failedn");
    fprintf(stderr, "%sn", mysql_stmt_error(stmt));
    exit(0);
}
/* Fetch all rows */
row_count = 0;
fprintf(stdout, "Fetching results …n");
while (!mysql_stmt_fetch(stmt)) {
    row_count++;
    fprintf(stdout, " row %dn", row_count); /* column 1 */
    fprintf(stdout, "  column1 (integer)  : ");
    if (is_null[0])
        fprintf(stdout, "NULLn");
    else
        fprintf(stdout, "%d(%ld)n", int_data, length[0]); /* column 2 */
    fprintf(stdout, "  column2 (string)   : ");
    if (is_null[1])
        fprintf(stdout, "NULLn");
    else
        fprintf(stdout, "%s(%ld)n", str_data, length[1]); /* column 3 */
    fprintf(stdout, "  column3 (smallint) : ");
    if (is_null[2])
        fprintf(stdout, "NULLn");
    else
        fprintf(stdout, "%d(%ld)n", small_data, length[2]); /* column 4 */
    fprintf(stdout, "column4 (timestamp): ");
    if (is_null[3])
        fprintf(stdout, "NULLn");
    else
        fprintf(stdout, "%04d-%02d-%02d %02d:%02d:%02d (%ld)n",
            ts.year, ts.month, ts.day,
            ts.hour, ts.minute, ts.second,
            length[3]);
    fprintf(stdout, "n");
}
/* Validate rows fetched */
fprintf(stdout, "total rows fetched: %dn", row_count);
if (row_count != 2) {
    fprintf(stderr, "MySQL failed to return all rowsn");
    exit(0);
}
/* Free the prepared result metadata */
mysql_free_result(prepare_meta_result);
/* Close the statement */
if (mysql_stmt_close(stmt)) {
    fprintf(stderr, "failed while closing the statementn");
    fprintf(stderr, "%sn", mysql_stmt_error(stmt));
    exit(0);
}
```

然后我就仿照这个代码写，一旦当字段设的比较多，或者这个函数多执行几遍的时候总是会出`Segmentation faults`（段错误）。仔细检查了很多遍代码，特别是关于内存申请释放的部分，一直没找到原因，`Segmentation faults`是函数`mysql_stmt_fetch`抛出的。查遍了所有的文档，都没有找到有用的信息。

后来我突然注意到了这个函数`memset(bind, 0, sizeof (bind))`（将bind的数据全部重置为0），因为它是与内存有关的，我打印了`sizeof(bind)`，发现打出的值是8，我恍然大悟，这根本不是bind实际指向的内存大小，而是一个指针的大小。正确的写法应该是`memset(bind, 0, sizeof (MYSQL_BIND) * 4)`，改过之后，一切正常，纠结了一周的bug总算fix了。

看来Mysql的官方文档出的一些示例也没有经过严密的测试，以后要更加仔细的coding了。

memset和sizeof两个函数常常会让人误解，搞不清楚到底该传什么，两者配合的时候很容易出错。

memset的第一个参数应该是一个地址，而sizeof的参数是变量或是类型。

memset是将一段地址上的数据全置成指定的数字，而sizeof则是计算变量或是类型的大小。

以后看到这样的函数一定要敏感一点。。。
