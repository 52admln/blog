---
title: 谷歌被封，WP博客字体无法加载的解决办法
tags:
  - google
  - useso
  - wordpress
id: 144
categories:
  - WordPress
date: 2014-07-26 16:14:04
---

[![](http://ww4.sinaimg.cn/large/85f4065cgw1eiss336q3vj20k0069dfu.jpg)](http://ww4.sinaimg.cn/large/85f4065cgw1eiss336q3vj20k0069dfu.jpg)

Google服务在大陆全线被封不仅影响到了广大网民，也影响到了数百万使用wordpress程序的站长。WordPress是世界上最大的开源博客程序，而WordPress大部分的主题都是使用Google的在线字体方案：Google Fonts。Google服务不稳定，导致大量独立博客字体加载不出来，直接导致几十万独立博客打开速度变慢，严重时甚至导致网站打不开。
<!--more-->

修改方法如下：

方法一：
打开wordpress代码中的文件wp-includes/script-loader.php文件，
搜索：
fonts.googleapis.com
找到这行代码：

<pre lang="php" escaped="true">
$open_sans_font_url = "//fonts.googleapis.com/css?family1=Open+Sans:300italic,400italic,600italic,300,400,600&subset=$subsets";
</pre>

把
fonts.googleapis.com

替换为

fonts.useso.com

修改完保存，再次刷新，大家就可以发现，自己的网站速度已经比以前快了很多，几乎瞬间就可以拿到Google字体了。原因就是本来需要从美国服务器才能拿到的google字体，现在已经遍布360全国的机房了。

方法二：禁用open sans字体

以下代码加入主题function.php中
<pre lang="php" escaped="true">
//WordPress 后台禁用Google Open Sans字体，加速网站
add_filter( 'gettext_with_context', 'wpdx_disable_open_sans', 888, 4 );
function wpdx_disable_open_sans( $translations, $text, $context, $domain ) {
  if ( 'Open Sans font: on or off' == $context && 'on' == $text ) {
    $translations = 'off';
  }
  return $translations;
}
</pre>