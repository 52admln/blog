---
title: CentOS下Crontab 执行失败的几种原因
tags:
  - centos
id: 526
categories:
  - Linux
date: 2015-08-11 14:54:25
---

1 crond服务未启动
crontab不是Linux内核的功能，而是依赖一个crond服务，这个服务可以启动当然也可以停止。如果停止了就无法执行任何定时任务了，解决的方法是打开它:
crond
或
service crond start
如果提示crond命令不存在，可能被误删除了，CentOS下可以通过这个命令重新安装：
yum -y install crontabs
<!--more-->

2 权限问题
比如：脚本没有x执行权限，解决方法：
增加执行权限，或者用bash abc.sh的方法执行
3 路径问题
有的命令在shell中执行正常，但是在crontab执行却总是失败。有可能是因为crontab使用的sh未正确识别路径，比如：以root身份登录shell后执行一个/root/test.sh，只要执行
./test.sh
就可以了。但是在crontab中，就会找不到这个脚本，比如写完整：
/root/test.sh
4 时差问题
因为服务器与客户端时差问题，所以crontab的时间以服务器时间为准。
5 变量问题
有时候命令中含有变量，但crontab执行时却没有，也会造成执行失败。