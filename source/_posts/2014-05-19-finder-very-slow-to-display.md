title: Finder显示文件列表太慢
date: 2014-05-19 11:30:40
categories: [Mac]
tags: [mac, finder]
---
Finder是Mac上的文件管理系统。最近不知道什么原因，打开Finder后显示的速度很慢，CPU和内存都正常，其他的程序也运行正常。
<!--more-->
尝试这重启Finder。
```bash
$ killall Finder
```

可惜依然没什么用。

在网上搜了一下，发现很多人是打开了`回到我的Mac`而导致这个结果。

于是关掉这个功能试试：
- 打开Finder， 运行`cmd+,`。
- 点击Sidebar，也就是边栏。
- 取消共享里面的回到我的`Mac`。

重新打开Finder，发现速度正常了。

`回到我的Mac`是iCloud的一项功能，借助这个功能，你可以从Internet上的另一台Mac安全地远程连接你的 Mac，并且访问文件或控制其屏幕。

如果需要使用这个功能，重新打开就可以了。
