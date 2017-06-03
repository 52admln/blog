---
title: 禁用WordPress文章自动保存功能并清空修订版本
tags:
  - wordpress
id: 525
categories:
  - WordPress
date: 2015-08-11 14:49:16
---

WordPress文章自动保存功能，会生成大量的修订版本，你修改几次，它就会生成几个版本，并且这些修订版本会保留在wordpress的数据库中，占用大量的数据存储空间，会使你的数据库越来越臃肿。

禁用wp文章自动保存功能

1、在网站根目录的wp-config.php文件中加入以下其中任何一条适合你的代码：

<pre lang="php" escaped="true">
// 不保存任何版本（除了自动保存的版本）
define('WP_POST_REVISIONS', false);

//保存所有修订版本
define('WP_POST_REVISIONS', true);

// 保存 n 个修订版本
define('WP_POST_REVISIONS', 3);
</pre>

<!--more-->

WP_POST_REVISIONS 这个变量的详细设置为：

true（默认）或者 -1：保存所有修订版本

false 或者 0：不保存任何版本（除了自动保存的版本）

大于 0 的整数 n：保存 n 个修订版本（+1 只保存自动保存版本），旧的版本将被删除。

清空WordPres修订版本

进入phpmyadmin，选择你的数据库，在sql选项中输入 
<pre lang="sql" escaped="true">
SELECT * FROM wp_posts WHERE post_type = "revision"   
</pre>
查看数据库的修订版本数量

使用SQL语句 
<pre lang="sql" escaped="true">
DELETE FROM wp_posts WHERE post_type = "revision" 
</pre>