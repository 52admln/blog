---
title: Centos 下实时查看网卡流量的工具 – nload
tags:
  - nload
id: 490
categories:
  - Linux
date: 2015-06-25 13:56:59
---

nload 的官方网站为： http://www.roland-riegel.de/nload/
在 CentOS 下，nload 目前就只能基于源代码进行编译安装了。默认安装的 CentOS ，要编译安装 nload 时，可能还需要安装几个需要的软件包：gcc 、gcc-c++ 和 ncurses-devel

<!--more-->

yum install gcc gcc-c++ ncurses-devel
wget http://www.roland-riegel.de/nload/nload-0.7.4.tar.gz
tar xvf nload-0.7.4.tar.gz
cd nload-0.7.4
./configure
make
make install

完成后，nload 被安装在了 /usr/local/bin/nload 。

用图形方式查看实时流量:

nload -t 200 -i 1024 -o 128 -u H

查看第一网卡的流量情况，显示的是实时的流量图

nload eth0
同时查看多个网卡的流量情况。

nload -m

参数说明:
Options:
-a period Sets the length in seconds of the time window for average calculation. Default is 300.
-a period 设置在时间窗口平均计算秒的长度。默认值是300。
-i max_scaling Specifies the 100% mark in kBit/s of the graph indicating the incoming bandwidth usage. Ignored if max_scaling is 0 or the switch -m is given. Default is 10240.
-i max_scaling 指定了100％标记在图中的kbit / s的指示传入的带宽使用率。如果max_scaling为0或开关-m时给予忽略。默认值是10240。
-m Show multiple devices at a time; no traffic graphs.
-m 显示多个设备同时使用;没有流量图。
-o max_scaling Same as -i but for the graph indicating the outgoing bandwidth usage. Default is 10240.
-o max_scaling 相同的AS-I，但对于曲线图，显示出带宽使用。默认值是10240。
-t interval Determines the refresh interval of the display in milliseconds. Default is 500.
-t interval 确定以毫秒为单位显示的刷新间隔。默认值是500。
-u h|b|k|m|g Sets the type of unit used for the display of traffic numbers.
-u h|b|k|m|g 设置用于流量数字的显示单元的类型。
H|B|K|M|G h: auto, b: Bit/s, k: kBit/s, m: MBit/s etc. H: auto, B: Byte/s, K: kByte/s, M: MByte/s etc. Default is h.
-U h|b|k|m|g Same as -u, but for a total amount of data (without “/s”).
-U h|b|k|m|g 同-U，但是对于数据的总量（没有“/秒”）。
H|B|K|M|G Default is H.
devices Network devices to use. Default is to use all auto-detected devices.
devices 网络设备使用。默认值是使用所有自动检测到的设备。
例子: nload -t 200 -i 1024 -o 128 -U M
The options above can also be changed at run time by pressing the ‘F2′ key.
上面的选项也可以在运行时按“F2”键改变。

实时查看网卡流量的工具除了 nload 之外，还有 slurm 以及 systat 等。