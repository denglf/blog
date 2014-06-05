title: WordPress邮件配置历程
date: 2010-06-12
author: denglf
layout: post
categories: [WordPress]
tags: [email, SMTP, WordPress]
---
自从BlogEngine转WordPress以来，一切功能都还算满意，但是邮件从没发成功过。
<!--more-->
以前一直没有在意这个问题，也就没管它。今天心血来潮，想解决这个问题。

google了一些，说是在Linux下，php可以直接利用`mail()`函数来发送邮件，比较方便。然而，到了Windows下，情况就不一样了。虽然IIS也可以安装SMTP服务，但是，却不能被php的`mail()`函数利用。继续搜，找到了[Configure SMTP][1]这个插件，它重载了`mail()`函数，允许自己自定义SMTP服务器，从而使用外部的SMTP server来实现邮件的发送功能。配置，也很容易。于是装上，配置上我的gmail邮箱，测试，失败！检查参数，配置，还是失败！

于是，换个插件试试，在WordPress的插件库，找到了[WP-Mail-SMTP][2]，配置差不多，但是还是测试失败，所幸这个失败会报错：

```
SMTP -> ERROR: Failed to connect to server: Unable to find the socket 
transport “tls” – did you forget to enable it when you configured PHP?
```
检查原因，应该是这台Windows的服务器没有装OpenSSL，不支持SSL。当然，blog寄人篱下，是没有执行权限的，只能找非SSL的邮箱了。

注册了一个163的，`dlfenblog@163.com`，不用SSL，端口设25，想到了大学那会的网络知识竞赛，对这个端口印象深刻…（好像走神了~）。

果然是这个原因，邮件发送成功了。然后在163的设了自动转发到我的gmail邮箱，我比较习惯用gmail。

选了[Configure SMTP][1]这个插件，因为它的密码域用的是*代替，不是明文，这个让我感觉比较有安全感（心理作用）。

现在这个blog用<dlfenblog@163.com>和<dlfenblog@gmail.com>来回复大家的邮件，谢谢支持。

 [1]: http://coffee2code.com/wp-plugins/configure-smtp/
 [2]: http://wordpress.org/extend/plugins/wp-mail-smtp/
