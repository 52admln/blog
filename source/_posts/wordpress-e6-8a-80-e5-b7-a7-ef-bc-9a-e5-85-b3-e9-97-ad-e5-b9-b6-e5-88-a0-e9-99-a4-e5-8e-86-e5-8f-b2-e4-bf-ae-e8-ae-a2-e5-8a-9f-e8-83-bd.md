---
title: WordPress技巧：关闭并删除历史修订功能
tags:
  - wordpress
  - 历史修订
id: 324
categories:
  - WordPress
date: 2014-08-01 11:07:51
---

    使用了几天的wordpress，发现有一个功能很头疼，每次编辑完文章，然后又想修改，反复几次就会出现好几个历史修订版本，这样既占数据大小又不好看，便决定将它关闭。
    网上搜索了一下，教程如下，做个记录。
<!--more-->

1.禁用 WordPress 文章修订历史功能

打开 WordPress 根目录下的 wp-config.php 文件，
在这段代码

    define('WPLANG', 'zh_CN');

下面添加代码即可：

<pre>
`
define('WP_POST_REVISIONS', false);
`
</pre>

2.删除 WordPress 已有的文章修订记录
上面也说过，WordPress 文章的修订记录，都是被写入了数据库，所以我们需要删除MySQL数据库中有关的语句！SQL 如下（直接删除即可）：

<pre>
`
DELETE FROM wp_postmeta WHERE post_id IN (SELECT id FROM wp_posts WHERE post_type = 'revision');
DELETE FROM wp_term_relationships WHERE object_id IN (SELECT id FROM wp_posts WHERE post_type='revision');
DELETE FROM wp_posts WHERE post_type='revision';
`
</pre>

经过以上步骤，就算完工了！