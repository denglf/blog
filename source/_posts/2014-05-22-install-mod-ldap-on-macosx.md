title: '在MAC上安装Apache的LDAP模块'
date: 2014-05-22 16:18:23
categories: [Apache]
tags: [apache, ldap, homebrew]
---
在MAC搭建一个trac系统方便插件的调试。需要LDAP实现用户权限认证。但是MAC自带的Apache默认没有安装LDAP模块，于是动手安装一个。
<!--more-->
## LDAP MacOSX 安装
使用Homebrew安装OpenLDAP，首先安装数据库berkeley-db。
### 安装Berkeley DB

```sh
$ brew install berkeley-db --enable-sql
```
### 安装LDAP

#### 修改LDAP的安装脚本
我的LDAP的用户数据是从`.htpasswd`文件中导入的，里面的密码是crypt加密的。

所以LDAP需要启用crypt。

```sh
brew edit openldap
```

在`args`的列表中加上下面的选项：

```sh
--enable-crypt
```

#### 安装openldap
```sh
$ brew install openldap --with-berkeley-db
```
### 安装Apache服务httpd
自带的Apache没有编译LDAP模块，需要重新编译一个。

还是使用Homebrew安装，默认的安装也不编译LDAP，所以我们需要修改安装脚本。

#### 修改httpd安装脚本

```sh
$ brew edit httpd
```
在`./configure`的后面加上下面的选项：

```ruby
"--enable-ldap",
"--enable-authnz-ldap",
"--with-ldap",
"--with-ldap-sdk=openldap"
```

由于安装的httpd版本是`2.7>2.0`，ldap模块由`mod_auth_ldap`改成了`mod_authnz_ldap`。需要加入选项`--enable-authnz-ldap`，而这个选项依赖`mod_ldap`，因此需要加入`--enable-ldap`，httpd的编译使用代码内部的`apr`和`apr-util`，`apr-util`也需要加入ldap的选项，因此加上`--with-ldap`。

#### 安装httpd

```sh
$ brew install httpd
```
如果出现一些错误信息：

```
env: /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.9.
xctoolchain/usr/bin/cc: No such file or directory
```

那么执行下面的代码，并重新安装。

```
sudo ln -s /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/ /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.9.xctoolchain
```
#### 配置httpd
安装完成后，在httpd.conf中配置`mod_authnz.ldap`，加入：

```sh
LoadModule ldap_module libexec/mod_ldap.so
LoadModule authnz_ldap_module libexec/mod_authnz_ldap.so
```
#### 运行httpd
```sh
$ apachectl start
```
如果出现以下错误：

```
httpd: Could not reliably determine the server's fully qualified domain name, 
using 127.0.0.1 for ServerName
```
说明`httpd.conf`中的ServerName没有配置，去掉ServerName的注释并修改端口即可，需要和listen的端口一致：

```
Listen 8080
ServerName localhost:8080
```
由于80端口需要使用sudo启动，这里使用了8080。

重新启动httpd：

```
$ apachectl restart
```
