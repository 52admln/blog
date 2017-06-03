---
title: DirectAdmin优化MySQL性能(升级TCMalloc及修改配置)
tags:
  - mysql
id: 492
categories:
  - 纯粹折腾
date: 2015-06-26 10:25:56
---

近看到不少反应MySQL拖垮服务器资源的一些讨论，尤其是很多的个人博客站长采用wordpress源码，连发个评论都要写入MySQL数据库。虽然我无法从根本上解决这些问题，但是对于MySQL做一些简单的优化还是非常有必要的，下面，我们就以DirectAdmin面板下的MySQL优化为例来做一个简单的记录。 
<!--more-->

关于本文的MySQL优化，我把他分为2个部分来做，包括升级TCMalloc以及修改MySQL配置文件。
★★★安装TCMalloc★★★
首先，我们来关注下如何安装TCMalloc来优化mysql在高负载下的表现。首先，root登陆服务器。因为我的服务器采用64位的Centos，所以，需要先安装libunwind库，32位系统可略过此步。
wget http://download.savannah.gnu.org/releases/libunwind/libunwind-0.99-alpha.tar.gz 
tar zxvf libunwind-0.99-alpha.tar.gz 
cd libunwind-0.99-alpha/ 
CFLAGS=-fPIC ./configure
make CFLAGS=-fPIC
make CFLAGS=-fPIC install

接下来，我们开始安装Tcmalloc。
wget http://gperftools.googlecode.com/files/gperftools-2.0.tar.gz
tar zxvf  gperftools-2.0.tar.gz
cd gperftools-2.0/
./configure 
make && make install
echo “/usr/local/lib” > /etc/ld.so.conf.d/usr_local_lib.conf
/sbin/ldconfig 

编译完成后，我们编辑mysqld_safe文件，加入Tcmalloc部分。

vi /usr/bin/mysqld_safe
找到# executing mysqld_safe，在下面加入：
export LD_PRELOAD=/usr/local/lib/libtcmalloc.so

保存，退出，重启MySQL。
service mysqld restart

接下来再检查是否生效，运行。
lsof -n | grep tcmalloc

若看到类似如下内容，即表示成功。 
mysqld     7758     mysql  mem       REG              253,0   1943001  109233156 /usr/local/lib/libtcmalloc.so.4.1.0

★★★修改配置文件★★★ 
DirectAdmin默认的MySQL配置文件非常的简洁。 

[mysqld] local-infile=0  

我们需要修改配置文件，参考下面的内容(vim /etc/my.cnf) 
[mysqld]
local-infile=0
skip-locking
query_cache_limit=1M
query_cache_size=32M
query_cache_type=1
max_connections=500
interactive_timeout=100
wait_timeout=100
connect_timeout=10
thread_cache_size=128
key_buffer=16M
join_buffer=1M
max_allowed_packet=16M
table_cache=1024
record_buffer=1M
sort_buffer_size=2M
read_buffer_size=2M
max_connect_errors=10
# Try number of CPU’s*2 for thread_concurrency
thread_concurrency=4
myisam_sort_buffer_size=64Mserver-id=1
[safe_mysqld]
err-log=/var/log/mysqld.log
open_files_limit=8192
[mysqldump]
quick
max_allowed_packet=16M
[mysql]
no-auto-rehash
#safe-updates
[isamchk]
key_buffer=64M
sort_buffer=64M
read_buffer=16M
write_buffer=16M
[myisamchk]
key_buffer=64M
sort_buffer=64M
read_buffer=16M
write_buffer=16M
[mysqlhotcopy]
interactive-timeout

以上配置内容来自DirectAdmin官方帮助中心(http://help.directadmin.com/item.php?id=44)，
大家请根据自己的情况自行修改参数。

完成后保存，退出，重启MySQL。
/sbin/service mysqld restart 
OK，做完以上的两个方面的优化后，相信您的MySQL在高负载下的表现会大大提高了。