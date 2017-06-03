---
title: Linux下同时wget多个文件
tags:
  - Linux
id: 538
categories:
  - Linux
date: 2016-01-20 16:17:34
---

首先建立一个url.txt
然后将要下载的链接复制到里面，例如
<pre  lang="bash">
http://qq.com/1.jpg
http://qq.com/2.jpg
http://qq.com/3.jpg
</pre>
然后执行
<pre  lang="bash">
wget -b -i url.txt 
</pre>
即可。
-b表示后台wget，-i 表示从文本文件内读取网址。