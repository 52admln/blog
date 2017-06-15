---
title: 'WordPress导入数据错误MySQL返回:#1273 – Unknown collation:’utf8_general_ci’'
id: 545
categories:
  - WordPress
  - 未分类
date: 2016-01-29 17:23:26
tags:
---

wordpress网站转移服务器空间，通过phpmyadmin导入sql数据时出现错误，错误提示：
<pre lang="sql">
MySQL返回:
#1273 – Unknown collation:’utf8_general_ci’
</pre>
[![](https://ww3.sinaimg.cn/large/85f4065cgw1f0ghoudi48j20vx0bi0tn.jpg)](https://ww3.sinaimg.cn/large/85f4065cgw1f0ghoudi48j20vx0bi0tn.jpg)
大致意思是“没有定义的编码集utf8”。搜索查询后发现utf8是utf8的一个衍生形式，utf-8里的一个字符只能最多支持3个字节，而utf8则扩展到一个字符支持4个字节。而utf8只有在mysql数据库版本是5.5.3+的时候才支持，网站原mysql的版本是5.6，导入的mysql版本是5.0，因此出现#1273错误。
<!--more-->

wordpress官方的相关说明是只要在数据库支持utf8的时候会把部分数据表的编码升级为utf8，如果不支持就不会转化为utf8编码（wordpress 4.4版本支持mysql 5.0+）。

解决方法：

方法一：替换编码

使用代码编辑器打开导出的sql数据文件；

先查找：
<pre lang="sql">
utf8_general_ci
</pre>
替换为：
<pre lang="sql">
utf8_general_ci
</pre>
再查找
<pre lang="sql">
utf8
</pre>
替换为
<pre lang="sql">
utf8
</pre>
注意：一定要按照上面的顺序进行替换，否则不能替换成功。

PS：我目前通过该方法导入成功，暂时没有发现有问题，但还是要先备份好数据再进行操作。

方法二：把网站要用的mysql数据库升级到5.5.3以上版本。

