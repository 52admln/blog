---
title: Mac OS下关闭本地TimeMachine备份节省磁盘空间
id: 550
categories:
  - 纯粹折腾
date: 2016-02-01 13:45:54
tags:
---

当我们开启TimeMachine之后，在使用外置磁盘时会把备份资料放在外置磁盘上，但是某一天发现如下图所示的奇怪现象，磁盘使用情况里面竟然有几十GB的“备份”文件。总共256GB容量，所以万万不能忍，想着要把它删掉。
[![](http://ww2.sinaimg.cn/large/85f4065cgw1f0js8tj7uxj20ga0bc75t.jpg)](http://ww2.sinaimg.cn/large/85f4065cgw1f0js8tj7uxj20ga0bc75t.jpg)
<!--more-->

在Finder窗口中输入command+shift+G，然后在对话框中输入/Volumes进入磁盘目录管理，可以看到有一个MobileBackups文件夹，打开发现里面有一个Backups.backupdb，就是本地备份文件！我们需要删除它。
有的人直接删除，会发现权限不够，然后计算机专业学生进入命令行，输入sudo rm -R xxxxx，但是几天后又出来了…
其实只需要一行命令即可，就是修改TimeMachine的设置：
进入终端，输入如下命令
<pre lang="bash">
sudo tmutil disablelocal  
</pre>
然后按照提示输入系统账户密码。
该文件夹已经消失！再查看磁盘空间，可以发现少了50GB！
[![](http://ww4.sinaimg.cn/large/85f4065cgw1f0js8tubazj20ga0bcwg0.jpg)](http://ww4.sinaimg.cn/large/85f4065cgw1f0js8tubazj20ga0bcwg0.jpg)
如果要打开这个本地备份的功能，我们输入如下命令即可：
<pre lang="bash">
sudo tmutil enablelocal  
</pre>

转载自：http://blog.csdn.net/rk2900/article/details/38755907