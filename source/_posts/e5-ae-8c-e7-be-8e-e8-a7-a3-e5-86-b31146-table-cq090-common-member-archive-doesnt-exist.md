---
title: 完美解决(1146) Table 'cq090.common_member_archive' doesn't exist
tags:
  - discuz
id: 480
categories:
  - 纯粹折腾
date: 2015-05-02 20:07:51
---

完美解决(1146) Table 'cq090.common_member_archive' doesn't exist
下面是错误代码
<!--more-->

> Discuz! Database Error> 
>  (1146) Table '***.common_member_archive' doesn't exist> 
>  REPLACE INTO common_member_archive SELECT * FROM common_member WHERE uid IN ('7','13','15','16','18','19','22','24','27','28','29','30','31','34','35','37','39','40','42','43','44','45','46','48','49','50','51','52','53','54','56','58','59','60','61','64','65','66','67','68','69','70','71','73','74','79','80','81','84','85','86','88','89','90','91','92','93','94','96','98','99','101','102','104','105','106','107','108','112','113','114','116','118','119','120','122','123','125','127','128','129','130','131','132','133','134','135','136','138','141','142','143','145','147','148','149','150','151','152','153')> 
> PHP Debug> 
> 
> No. File Line Code > 
> 1 index.php 126 require(%s) > 
> 2 forum.php 49 discuz_application->init() > 
> 3 source/class/discuz/discuz_application.php 70 discuz_application->_init_cron() > 
> 4 source/class/discuz/discuz_application.php 479 discuz_cron::run() > 
> 5 source/class/discuz/discuz_cron.php 42 include(%s) > 
> 6 source/include/cron/cron_member_optimize_daily.php 13 table_common_member->split(%d) > 
> 7 source/class/table/table_common_member.php 351 discuz_database::query(%s, Array, false, true) > 
> 8 source/class/discuz/discuz_database.php 136 db_driver_mysql->query(%s, false, true) > 
> 9 source/class/db/db_driver_mysql.php 151 db_driver_mysql->halt(%s, %d, %s) > 
> 10 source/class/db/db_driver_mysql.php 218 break()

解决方法 一

方法：即在站长—数据库—升级(Discuz! 数据库升级 - 请将数据库升级语句粘贴在下面中执行语句 ：

 注意pre_ 如果你的数据库前缀是别的请写好。
 复制代码就可以了，若没有找到执行语句的输入窗口 则修改config/config_global.php 当中的 $_config[admincp][runquery] 设置修改为 1 (为了安全执行完该语句后 确认解决了1146错误后 再将配置该回来 )  刷新后再输入执行。
 为了数据安全执行该语句前建议备份数据

 先备份表pre_common_setting

`DELETE FROM `pre_common_setting` WHERE `skey` = 'membersplit';`

解决方法 二
方法：登陆phpmyadmin 然后找到对应的数据库表。分别对如下表进行备份

pre_common_member
 pre_common_member_profile
 pre_common_member_field_forum
 pre_common_member_field_home
 pre_common_member_status
 pre_common_member_count

然后分别复制一份并命名后面加_archive

最后就可以在 工具-计划任务 - 每日用户表优化 。运行了。如果还报错就继续复制数据表命名加_archive