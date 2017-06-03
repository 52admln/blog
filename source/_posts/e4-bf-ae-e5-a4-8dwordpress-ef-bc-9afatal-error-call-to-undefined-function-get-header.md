---
title: '修复WordPress：Fatal error: Call to undefined function get_header()'
tags:
  - index
  - wordpress
id: 337
categories:
  - WordPress
date: 2014-08-07 22:14:49
---

    百度观测今天报警了一个错误，点开来一看就如题目所说，问题出在主题文件夹下的index.php文件，如果直接访问http://52admin.net/wp-content/themes/Teahouse/index.php则会出现“Fatal error: Call to undefined function get_header()”，如图：<!--more-->

[![](http://ww2.sinaimg.cn/large/85f4065cgw1ej4fheanwnj20kx026q2q.jpg)](http://ww2.sinaimg.cn/large/85f4065cgw1ej4fheanwnj20kx026q2q.jpg)

而get_header()这个函数是WordPress系统函数，不可能没有定义这个函数，而是加载index.php的时候没有引入该函数，所以造成了该问题的出现。
如果自己有设置主机php.ini的权限，不操心log的话，可以修改如下，就不会有错误提示了：

<pre>
display_errors = off
error_reporting=E_ALL&~E_NOTICE
</pre>

因为没有权限去设置php.ini，于是，我在index.php中添加不显示运行错误：

<pre><?php ini_set('display_errors', 0); ?></pre>

这样以后就没有了错误提示，可是error-log中依旧记录着该错误，如何不记录这个错误呢？如果直接加载index.php且没有定义get_header()这个函数，就直接重定向到网站首页，所以一个简单地判断就可以搞定了。

<pre>
<?php ini_set('display_errors', 0); ?>
<?php 
if (function_exists('get_header')) {
    get_header();
}else{
    header("Location: http://" . $_SERVER['HTTP_HOST'] . "");
    exit;
}; ?>
</pre>

Ok，这样如果直接访问index.php就被重定向到了首页，而且error-log中就清爽多了。