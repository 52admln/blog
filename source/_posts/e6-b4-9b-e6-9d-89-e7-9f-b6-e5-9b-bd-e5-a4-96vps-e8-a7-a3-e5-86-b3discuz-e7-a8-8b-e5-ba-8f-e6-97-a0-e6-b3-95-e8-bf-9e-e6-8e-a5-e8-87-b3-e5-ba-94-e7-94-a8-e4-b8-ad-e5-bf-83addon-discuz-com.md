---
title: 洛杉矶国外VPS解决Discuz!程序无法连接至应用中心addon.discuz.com
tags:
  - discuz
  - vps
  - 应用中心
id: 40
categories:
  - 纯粹折腾
date: 2014-07-21 19:46:36
---

经过几番周折，终于在今天把这个问题成功的解决了，下面分享一下步骤：

1.修改 VPS 主机的 hosts文件

使用putty登陆vps，输入命令

<!--more-->
<pre>vi /etc/hosts
</pre>
按 insert 编辑

添加2条内容，如下：
<pre>180.153.210.100 addon.discuz.com addon
180.153.210.78 addon1.discuz.com addon1
</pre>
[![](http://ww3.sinaimg.cn/large/85f4065cgw1eiqarqzrfij20dw08qdgc.jpg)](http://ww3.sinaimg.cn/large/85f4065cgw1eiqarqzrfij20dw08qdgc.jpg)

&nbsp;

完成后，按esc 接着输入：wq 保存并退出

&nbsp;

&nbsp;

&nbsp;

2.修改 discuz 程序 文件
> 打开：sourcefunctionfunction_cloudaddons.php
查找
<pre>define('CLOUDADDONS_WEBSITE_URL', 'http://addon.discuz.com');
define('CLOUDADDONS_DOWNLOAD_URL', 'http://addon.discuz.com/index.php');
define('CLOUDADDONS_DOWNLOAD_IP', '');
define('CLOUDADDONS_CHECK_URL', 'http://addon1.discuz.com');
define('CLOUDADDONS_CHECK_IP', '');
</pre>
&nbsp;

手动填写以下 ip，即可。
> addon.discuz.com:电信180.153.210.100 联通112.64.234.198；
> 
> addon1.discuz.com:电信180.153.210.78 联通112.64.234.201；
示例：

我们以电信服务器IP来主要做个示例如下
<pre>define('CLOUDADDONS_WEBSITE_URL', 'http://addon.discuz.com');
define('CLOUDADDONS_DOWNLOAD_URL', 'http://addon.discuz.com/index.php');
define('CLOUDADDONS_DOWNLOAD_IP', '180.153.210.100');
define('CLOUDADDONS_CHECK_URL', 'http://addon1.discuz.com');
define('CLOUDADDONS_CHECK_IP', '180.153.210.78');
</pre>
&nbsp;

&nbsp;

[![](http://ww2.sinaimg.cn/large/85f4065cgw1eiqarpoe1sj20dw03x3z8.jpg)](http://ww2.sinaimg.cn/large/85f4065cgw1eiqarpoe1sj20dw03x3z8.jpg)

&nbsp;

[![](http://ww3.sinaimg.cn/large/85f4065cgw1eiqarqcaudj20dw077acr.jpg)](http://ww3.sinaimg.cn/large/85f4065cgw1eiqarqcaudj20dw077acr.jpg)