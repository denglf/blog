title: Git全局更新电子邮件
date: 2014-02-19
author: denglf
layout: post
categories: [Git] 
---
Github的邮箱和用户名修改了，但是以前的提交记录的用户名没有更新。查询了文档，还是有方法可以修改的。
<!--more-->
首先，更新代码并确定本地没有修改。

```sh
$ git status
# On branch master
nothing to commit, working directory clean
```

使用filter-branch和commit-tree修改提交信息，只要邮箱为"old@mail.com"的全部更新新的邮箱和用户名。

```sh
$ git filter-branch --commit-filter '
if [ "$GIT_AUTHOR_EMAIL" = "old@mail.com" ]; then
    GIT_AUTHOR_NAME="denglf";
    GIT_AUTHOR_EMAIL="denglifen@gmail.com";
    git commit-tree "$@";
else
    git commit-tree "$@"; fi' HEAD
```

强制推送到服务器。

```sh
$ git push origin HEAD --force
```
