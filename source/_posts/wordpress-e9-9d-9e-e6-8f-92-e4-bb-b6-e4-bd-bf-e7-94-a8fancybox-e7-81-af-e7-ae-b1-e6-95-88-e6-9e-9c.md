---
title: WordPress非插件使用FancyBox灯箱效果
tags:
  - FancyBox
  - wordpress
id: 215
categories:
  - WordPress
date: 2014-07-29 11:29:50
---

Wordpress的臃肿体积不得不让我们选择少用插件，那么今天，我们就分享一下如何不用插件来实现比较美观的Fancybox灯箱效果

<!--more-->

使用Fancybox需要的文件及工具
> 1.Fancybox
> 2.jQuery2.1（版本1.9也可以）我们使用360前端公共库CDN来调用
下载FANCYBOX：http://pan.baidu.com/s/1kTICvjh

第一步：将下载的压缩包解压，得到一个js文件夹，将这个文件夹上传到您主题的根目录

一般为`/wp-content/themes/主题名称/`下

第二步：修改主题function.php文件

在function.php文件中加入

<pre>
`
//fancybox 自动对图片链接添加rel=fancybox属性
add_filter('the_content', 'pirobox_gall_replace');
function pirobox_gall_replace ($content)
{ global $post;
$pattern = "/[]*).(bmp|gif|jpeg|jpg|png)('|")(.*?)>(.*?)](()/i";
$replacement = '[$7]($2$3.$4$5 rel=)';
$content = preg_replace($pattern, $replacement, $content);
return $content;
}
`
</pre>
代码解读：添加文章后，将文章中的图片链接自动添加上rel=fancybox属性用于在初始化时对图片的筛选
[![](http://ww1.sinaimg.cn/large/85f4065cgw1eiti7dmbyfj21bi0btwf8.jpg)](http://ww2.sinaimg.cn/mw690/85f4065cgw1eiti7eqs48j21b50afjrv.jpg)

第三步：修改主题header.php文件

在header.php文件中加入

<pre>
<link rel="stylesheet" type="text/css" href="<?php bloginfo('template_url') ?>/js/fancybox/jquery.fancybox.css" />
<script src="<?php bloginfo('template_url') ?>/js/fancybox/jquery.fancybox.pack.js"></script>
<script type="text/javascript">
$(function() {
jQuery(".gallery a").attr({rel: "fancybox"});
jQuery("a[rel=fancybox]").fancybox();
});
</script>
</pre>
[![](http://ww1.sinaimg.cn/mw690/85f4065cgw1eiti7dmbyfj21bi0btwf8.jpg)](http://ww1.sinaimg.cn/large/85f4065cgw1eiti7dmbyfj21bi0btwf8.jpg)
注意：经过以上步骤如果没有效果，请在header.php中加入调用jQuery2.1的有关代码
OK了，这样就完成了，看看效果吧~

[![](http://ww2.sinaimg.cn/mw690/85f4065cgw1eiti7e9hr1j20xe0lr79x.jpg)](http://ww2.sinaimg.cn/large/85f4065cgw1eiti7e9hr1j20xe0lr79x.jpg)