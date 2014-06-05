title: 段首空格，BlogEngine实现
date: 2010-01-22
author: denglf
layout: post
categories: [Themes]
tags: [blog, BlogEngine, html]
---
从小被教导，写文章段首要空两格，久而久之，看到顶格的段落总有想改的冲动。
<!--more-->
加空格的方式很多，输入全角空格或者在源代码里加&nbsp;。但是每段前面都要加，而且不同的浏览器空格的长度也不一样，这不是懒人的风格。

段落的标记是`<p>`，它有一个text-indent的属性，设置首行缩进。既然css可以控制，那就简单了。查看页面源代码，发现内容部分div的class的名字是entry，然后在themes的css中找到`style.css`，在里面加上：

```css
.entry p {
    text-indent: 2em;
}
```

现在所有的使用`<p></p>`的段落都首行缩进两个字符了。

当我满心欢喜的打开google reader看我的blog的时候，发现高兴的太早了，文字依然顶格。打开rss的源代码，里面并没有链接到`style.css`，所以设置的css对rss无效。

为了rss能段首缩进，看来只能在编辑的时候就把这个样式加进去了。理想的情况是敲回车的时候自动生成`<p stype="text-indent: 2em"></p>`，最好的方法是改后台的JavaScript，当`window.event.keyCode = 13`，也就是输入回车的时候，自动生成缩进的代码。打开后台的Javascript，发现要不所有的js都挤在一起，要不就写在一行，完全没有看的欲望。

于是决定在保存文章的时候，将所有的`<p>`都替换成`<p stype="text-indent: 2em">`。编辑文章的页面是`Add_entry.aspx`，所以打开`Add_entry.aspx.cs`，找到事件`btnSave_Click`，在`post.Content = txtContent.Text`之前将标记替换：

```csharp
if(txtContent.Text.Contains("&lt;p&gt;")) {
    txtContent.Text = 
        txtContent.Text.Replace("&lt;p&gt;", "&lt;p style="text-indent:2em"&gt;");
}
```

然后编辑好文章，保存，再次打开看源代码，所有的标记都替换好了，rss显示也正常。大功告成O(∩_∩)O哈哈~
