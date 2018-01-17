---
title: MySQL数据库常用导入导出命令
date: 2016-05-18 17:40:36
tags: 
  - MySQL
---

记录MySQL导出数据的常见方式

<!-- more -->

1. 导出整个数据库
> mysqldump -u root -p dataname > dataneme.sql

2. 导出一个表
> mysqldump -u root -p dataname users > dataname_users.sql

3. 导出数据库结构
> mysqldump -u root -p -d -add-drop-table smap > smap.sql

4. 导入数据库
> mysql>source swap.sql

- add-locks: 在每个表导出之前增加lock tables 并且之后 unlock table
- add-drop-table: create语句前增加一个drop table

参考资料：[Mysql数据库导入命令Source详解](http://crx.xmspace.net/mysql_source.html)
