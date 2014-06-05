title: 关于内存泄漏
date: 2010-01-06
author: denglf
layout: post
categories: [C++]
tags: [c, c++, 内存泄漏]
---
最近写C++，起200个线程，CPU和MEN一路飙升，像房价一样只涨不跌。
<!--more-->
看来是内存泄漏了，仔细研究代码，找出所有的new和malloc，才发现在很多函数里面new了而没有delete。

比如下面这个函数：

```c++
char* get_arg(const char *s, char *key)
{
    string str(s);
    int start = str.find(key, 0) + strlen(key) + 1;
    int end = str.find(';', start) - 1;
    const char *arg = str.substr(start, end - start + 1).c_str();
    char *temp = new char[strlen(arg) + 1]; //分配内存
    strcpy(temp, arg);
    return temp;
}
```

这个函数的功能是将s中的key的值找出来。

比如`s = "driver=1;host=localhost;user=root;password=123456;database=test;"`，如果`key = "user"`，那么返回的就是`"root"`。

调用的时候一般是`char* user = get_arg(s, "user")`。

细心的人就会发现，这里内存泄漏了。在函数get_arg中有new操作，但是没有delete。正确的调用应该是：

```c++
char* user = get_arg(s, "user");
delete user;
```

自己写这样的代码自己都会忘记delete，何况需要调接口的其他人呢？

这种做法本身就是很不安全的。所以在一个函数中，要么不分配内存，要么分配内存了并释放内存。

改过的函数代码如下：

```c++
void get_arg(char* res, const char *s, char *key)
{
    string str(s);
    int start = str.find(key, 0) + strlen(key) + 1;
    int end = str.find(';', start) - 1;
    const char *arg = str.substr(start, end - start + 1).c_str();
    strcpy(res, arg);
}
```

调用函数：

```c++
char* user = new char[strlen(s)];
get_arg(user, s, "user");
delete user;
```

以前写C中的关于字符串处理的函数的时候，总会要将保存结果的指针传进去。

比如`char *strcpy(char *dest,char *src)`，功能：把src所指由NULL结束的字符串复制到dest所指的数组中。

既然函数返回的就是拷贝后的结果，为什么还要将dest传进去呢，现在明白了。

很开心的看着内存降了下来，不知房价何时会降呢。。。
