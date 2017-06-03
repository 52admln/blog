---
title: 免插件修改WordPress后台登录地址
tags:
  - wordpress
id: 213
categories:
  - WordPress
date: 2014-07-29 10:48:17
---

虽然自己的网站也算是一个小站点，但是安全问题依然不能忽视，所以也就折腾一下
<!--more-->

方法:

将一下代码添加到你当前主题的 functions.php 文件中：
<pre>//diy后台登录
add_action('login_enqueue_scripts','login_protection');
function login_protection(){
    if($_GET['houtai'] != 'denglu')header('Location: http://52admin.net/');
}
</pre>
以上绿色部分可以随意DIY，红色部分既是你的博客地址

修改完成之后，你的后台登录地址就是http://yoursite/wp-login.php?houtai=denglu

&lt;注意你DIY部分的写法&gt;，如果不是由你diy的地址来访问的话，就会自动跳转到你博客首页。