title: 安装中文版Trac
date: 2014-05-23 17:28:44
categories: [Trac]
tags: [Trac]
---
## 环境
* python环境，系统自带python2.7。
* python自带包管理工具easy_install或pip。

## 安装pip
python的包管理工具，比easy_install好用点，安装过的可以略过。

```sh
$ sudo easy_install pip
```
## 安装Genshi
Genshi是一个解析HTML和XML的python库。
```sh
$ sudo pip install Genshi
```
## 安装Trac
```sh
$ sudo pip install trac
```
## 安装Babel
Babel是一个本地化翻译的python库。
```sh
$ sudo pip install -v Babel==0.9.6
```
**注意：这里安装的版本是`0.9.6`。**

Babel的默认安装版本是`1.3`，可惜这个版本和`Trac 1.0.1`不兼容。会出现一下错误：
```
AttributeError: NullTranslationsBabel instance has no attribute 'isactive'
```
见官方[ticket](http://trac.edgewall.org/ticket/11345)。

## 本地化设置
在`httpd.conf`中设置使用的语言：
```ini
<Location /trac>

......

SetEnv trac.locale zh_CN.UTF-8

......

</Location>
```
重启Apache即可。
