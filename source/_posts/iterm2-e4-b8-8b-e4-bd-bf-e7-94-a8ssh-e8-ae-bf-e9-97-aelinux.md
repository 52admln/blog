---
title: iTerm2下使用ssh访问Linux
tags:
  - iterm
  - ssh
id: 488
categories:
  - 纯粹折腾
date: 2015-06-25 13:41:13
---

通常情况下，iTerm2访问远程Linux使用ssh，方法如下：

`ssh <用户名>@<ip>`

然后输入访问的密码即可。需要指定访问端口如下。

`ssh -p <端口号> <用户名>@<ip地址>`