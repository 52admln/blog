---
title: CentOS系统下添加新硬盘并分区格式化
tags:
  - Linux
id: 349
categories:
  - Linux
date: 2014-08-16 19:58:00
---

Linux下默认第二个硬盘分区是不自动划分的，所以我们需要对它进行新建分区并格式化，下面看具体步骤：<!--more-->

1.先用Fdisk -l 来查看当前状态下磁盘情况

<pre>[root@linux1 ~]# fdisk -l
Disk /dev/hda: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Device Boot Start End Blocks Id System
/dev/hda1 * 1 13 104391 83 Linux
/dev/hda2 14 652 5132767 8e Linux LVM
Disk /dev/hdb: 2147 MB, 2147483648 bytes
16 heads, 63 sectors/track, 4161 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes
Disk /dev/hdb doesn't contain a valid partition table
//上面红色标识行可以看出,我添加了一块新硬盘/dev/hdb,大小为2G,未分区格式化状态。
</pre>

2.用Fdisk /dev/hdb来进行分区操作。

<pre>[root@linux1 ~]# fdisk /dev/hdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel. Changes will remain in memory only,
until you decide to write them. After that, of course, the previous
content won't be recoverable.

The number of cylinders for this disk is set to 4161.
There is nothing wrong with that, but this is larger than 1024,
and could in certain setups cause problems with:
1) software that runs at boot time (e.g., old versions of LILO)
2) booting and partitioning software from other OSs
(e.g., DOS FDISK, OS/2 FDISK)
Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

Command (m for help): n //输入N表示新建一个分区
Command action
e extended
p primary partition (1-4)
p //p 表示建立一个原始分区
Partition number (1-4): 1 //1 表示此分区编号为1.
First cylinder (1-4161, default 1): 1 //1表示使用默认起始柱面号.如果要分多个区的话,先盘算好要多大,再输入数字
Last cylinder or size or sizeM or sizeK (1-4161, default 4161): // 输入: 回车 表示使用默认结束柱面号.即此分区使用整个硬盘空间

Using default value 4161

Command (m for help): w //保存分区

The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
 </pre>

3.再次查看当前分区状态:
<pre>[root@linux1 ~]# fdisk -l

Disk /dev/hda: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

Device Boot Start End Blocks Id System
/dev/hda1 * 1 13 104391 83 Linux
/dev/hda2 14 652 5132767 8e Linux LVM

Disk /dev/hdb: 2147 MB, 2147483648 bytes
16 heads, 63 sectors/track, 4161 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes

Device Boot Start End Blocks Id System
/dev/hdb1 1 4161 2097112 83 Linux
//可以看出,已经出来了一个/dev/hdb1的新分区。下一步将其格式化,再使用
 </pre>

4.用mkfs.ext3格式化新分区
<pre>[root@linux1 ~]# mkfs.ext3 /dev/hdb1
mke2fs 1.39 (29-May-2006)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
262144 inodes, 524278 blocks
26213 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=536870912
16 block groups
32768 blocks per group, 32768 fragments per group
16384 inodes per group
Superblock backups stored on blocks:
32768, 98304, 163840, 229376, 294912

Writing inode tables: done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 32 mounts or
180 days, whichever comes first. Use tune2fs -c or -i to override.
 </pre>

5.挂载使用。
<pre>[root@linux1 ~]# mkdir /mnt/hdb1 //新建一个挂载点。
[root@linux1 ~]# mount /dev/hdb1 /mnt/hdb1 //挂载。
[root@linux1 ~]# df -h
//文件系统 容量 已用 可用 已用% 挂载点
/dev/mapper/VolGroup00-LogVol00
4.3G 3.6G 487M 89% /
/dev/hda1 99M 12M 82M 13% /boot
tmpfs 125M 0 125M 0% /dev/shm
/dev/hdb1 2.0G 3.0M 1.9G 1% /mnt/hdb1
 </pre>

6 .开机自动挂载
三.设置新硬盘开机自动挂载
在
<pre>vi /etc/fstab</pre>
中添加新硬盘的挂载信息.添加下面一行:
<pre>/dev/hdb1 /mnt/hdb1 ext3 defaults 1 2//(如果还有一个分区就是1 3,以此类推)</pre>
这样,每次开机后,系统会自动将/dev/hdb1挂载到/mnt/hdb1