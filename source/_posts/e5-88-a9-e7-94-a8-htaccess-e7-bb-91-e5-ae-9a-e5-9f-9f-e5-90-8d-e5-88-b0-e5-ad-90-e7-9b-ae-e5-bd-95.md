---
title: 利用.htaccess绑定域名到子目录
tags:
  - htaccess
  - 子目录
  - 绑定
id: 459
categories:
  - 纯粹折腾
date: 2015-04-04 17:09:53
---

前提你的空间服务器必须支持apache的rewrite功能，只有这样才能使用.htaccess。
如果你的空间是Linux服务器 一般默认都开启了的；

绑定域名
登陆域名管理台 把需要绑定的域名 解析到你的空间；
登陆虚拟主机/空间管理台 绑定域名到空间;

首先在本地建个txt文件，复制下面的代码修改替换你要绑的域名和目录，并传到网站主目录下再改成为.htaccess。
例如：下面是以 wap.yin.cc 和 2zk.cn 为例的.htaccess代码.
<!--more-->
<pre lang="apache">
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /

# 绑定wap.yin.cc 到 wap 子目录
RewriteCond %{HTTP_HOST} ^wap\.yin\.cc$ [NC]
RewriteCond %{REQUEST_URI} !^/wap/
RewriteRule ^(.*)$ wap/$1?Rewrite [L,QSA]
#可以绑定多个 只需重复上三行代码并更改一下域名、目录名 就好了

#绑定 2zk.cn 到 2zk 子目录
RewriteCond %{HTTP_HOST} ^www\.2zk\.cn$ [NC]
RewriteCond %{REQUEST_URI} !^/2zk/
RewriteRule ^(.*)$ 2zk/$1?Rewrite [L,QSA]

</IfModule>
</pre>

如果你以完成上面的步骤
你的子域名应该可以访问了 但你会发现在浏览器上访问 主域名+绑定的域名目录 也可以访问，可这并不是我们想要的 接下来我们完成最后一步;

在每一个绑定的目录中 如wap目录中 也增加一个 .htaccess 文件
.htaccess代码如下：
<pre lang="apache">
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
#只许绑定的域名访问
RewriteCond %{HTTP_HOST} !^wap\.yin\.cc$ [NC]
RewriteRule (.*) http://wap.yin.cc/$1 [L,R=301]
#对绑定目录下与 同名的目录的处理
RewriteCond %{REQUEST_URI} ^\/wap\/ [NC]
RewriteCond %{QUERY_STRING} !^(.*)?Rewrite
RewriteRule ^(.*)$ /%{REQUEST_URI}/%{REQUEST_URI}/$1?Rewrite [L,QSA]
</IfModule>
</pre>
注意：
如果你的网站根目录已存在.htaccess文件;
比如 WordPress 修改过固定链接，那么目录会有一个.htaccess 文件 里面如有：
<pre lang="apache">
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]

</IfModule>
</pre>
改为：
<pre lang="apache">
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /

RewriteCond %{HTTP_HOST} ^wap\.yin\.cc$ [NC]
RewriteCond %{REQUEST_URI} !^/wap/
RewriteRule ^(.*)$ wap/$1?Rewrite [L,QSA]

RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
</pre>
转载自：http://www.yin.cc/blog/htaccess-binding-domain/