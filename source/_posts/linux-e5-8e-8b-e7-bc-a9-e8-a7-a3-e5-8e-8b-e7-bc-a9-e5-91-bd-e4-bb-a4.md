---
title: Linux压缩解压缩命令
id: 546
categories:
  - Linux
date: 2016-01-29 18:07:45
tags:
---

.tar.gz 和 .tgz

解压：
tar zxvf FileName.tar.gz
压缩：
tar zcvf FileName.tar.gz DirName
tar -zcvf /home/FileName.tar.gz /home/wwwroot
tar -zcvf 打包后生成的文件名全路径 要打包的目录

zip
解压：
unzip FileName.zip
压缩：
zip FileName.zip DirName
压缩当前的文件夹 zip -r ./FileName.zip ./*   -r表示递归
zip [参数] [打包后的文件名] [打包的目录路径]