---
title: Discuz7.2适用于Apache环境伪静态规则Rewrite
tags:
  - discuz7.2
  - 伪静态
id: 476
categories:
  - 纯粹折腾
date: 2015-04-11 16:13:34
---

# 将 RewriteEngine 模式打开
RewriteEngine On
# 修改以下语句中的 /bbs 为你的论坛目录地址，如果论坛程序放在根目录中，请将 /bbs 修改为 /
RewriteBase /bbs
# Rewrite 系统规则请勿修改
RewriteRule ^archiver/([a-z0-9\-]+\.html)$ archiver/index.php?$1
RewriteRule ^forum-([0-9]+)-([0-9]+)\.html$ forumdisplay.php?fid=$1&page=$2
RewriteRule ^thread-([0-9]+)-([0-9]+)-([0-9]+)\.html$ viewthread.php?tid=$1&extra=page\%3D$3&page=$2
RewriteRule ^space-(username|uid)-(.+)\.html$ space.php?$1=$2
RewriteRule ^tag-(.+)\.html$ tag.php?name=$1