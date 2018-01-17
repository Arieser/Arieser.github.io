---
title: MySQL事务处理
date: 2016-09-09 17:38:29
tags: 
  - MySQL
---



[转载](http://www.cnblogs.com/ymy124/p/3718439.html)

<!-- more -->

 MySQL的事务支持不是绑定在MySQL服务器本身，而是与存储引擎相关1.MyISAM：不支持事务，用于只读程序提高性能 2.InnoDB：支持ACID事务、行级锁、并发 3.Berkeley DB：支持事务

 一个事务是一个连续的一组数据库操作，就好像它是一个单一的工作单元进行。换言之，永远不会是完整的事务，除非该组内的每个单独的操作是成功的。如果在事务的任何操作失败，则整个事务将失败。
实际上，会俱乐部许多SQL查询到一个组中，将执行所有的人都一起作为事务的一部分。

**事务的特性**： 
事务有以下四个标准属性的缩写ACID，通常被称为：

原子性: 确保工作单元内的所有操作都成功完成，否则事务将被中止在故障点，和以前的操作将回滚到以前的状态。

一致性: 确保数据库正确地改变状态后，成功提交的事务。

隔离性: 使事务操作彼此独立的和透明的。

持久性: 确保提交的事务的结果或效果的系统出现故障的情况下仍然存在。

在MySQL中，事务开始使用COMMIT或ROLLBACK语句开始工作和结束。开始和结束语句的SQL命令之间形成了大量的事务。

**COMMIT & ROLLBACK**: 
这两个关键字提交和回滚主要用于MySQL的事务。

当一个成功的事务完成后，发出COMMIT命令应使所有参与表的更改才会生效。

如果发生故障时，应发出一个ROLLBACK命令返回的事务中引用的每一个表到以前的状态。

可以控制的事务行为称为AUTOCOMMIT设置会话变量。如果AUTOCOMMIT设置为1（默认值），然后每一个SQL语句（在事务与否）被认为是一个完整的事务，并承诺在默认情况下，当它完成。 AUTOCOMMIT设置为0时，发出SET AUTOCOMMIT =0命令，在随后的一系列语句的作用就像一个事务，直到一个明确的COMMIT语句时，没有活动的提交。

可以通过使用mysql_query()函数在PHP中执行这些SQL命令。

通用事务例子 
这一系列事件是独立于所使用的编程语言，可以建立在任何使用的语言来创建应用程序的逻辑路径。
可以通过使用mysql_query()函数在PHP中执行这些SQL命令。


BEGIN WORK开始事务发出SQL命令

发出一个或多个SQL命令，如SELECT，INSERT，UPDATE或DELETE

检查是否有任何错误，一切都依据的需要。

如果有任何错误，那么问题ROLLBACK命令，否则发出COMMIT命令。

在MySQL中的事务安全表类型：

如果打算使用MySQL事务编程，那么就需要一种特殊的方式创建表。有很多支持事务但最流行的是InnoDB表类型。

从源代码编译MySQL时，InnoDB表支持需要特定的编译参数。如果MySQL版本没有InnoDB支持，请互联网服务提供商建立一个版本的MySQL支持InnoDB表类型，或者下载并安装Windows或Linux/UNIX的MySQL-Max二进制分发和使用的表类型在开发环境中。
如果MySQL安装支持InnoDB表，只需添加一个的TYPE=InnoDB 定义表创建语句。例如，下面的代码创建InnoDB表tcount_tbl：

```
create table tcount_tbl
    (
    tutorial_author varchar(40) NOT NULL,
    tutorial_count  INT
    ) TYPE=InnoDB;
```


可以使用其他GEMINI或BDB表类型，但它取决于您的安装，如果它支持这两种类型。


由于项目设计里面，牵扯到了金钱的转移，于是就要用到MYSQL的事务处理，来保证一组处理结果的正确性。用了事务，就不可避免的要牺牲一部分速度，来保证数据的正确性。
只有InnoDB支持事务

事务 ACID Atomicity（原子性）、Consistency（稳定性）、Isolation（隔离性）、Durability（可靠性）

1、事务的原子性
一组事务，要么成功；要么撤回。

2、稳定性
有非法数据（外键约束之类），事务撤回。

3、隔离性
事务独立运行。
一个事务处理后的结果，影响了其他事务，那么其他事务会撤回。
事务的100%隔离，需要牺牲速度。

4、可靠性
软、硬件崩溃后，InnoDB数据表驱动会利用日志文件重构修改。
可靠性和高速度不可兼得， innodb_flush_log_at_trx_commit选项 决定什么时候吧事务保存到日志里。
开启事务
START TRANSACTION 或 BEGIN

提交事务（关闭事务）
COMMIT

放弃事务（关闭事务）
ROLLBACK

折返点
SAVEPOINT adqoo_1
ROLLBACK TO SAVEPOINT adqoo_1
发生在折返点 adqoo_1 之前的事务被提交，之后的被忽略

事务的终止

设置“自动提交”模式
SET AUTOCOMMIT = 0
每条SQL都是同一个事务的不同命令，之间由 COMMIT 或 ROLLBACK隔开
掉线后，没有 COMMIT 的事务都被放弃

事务锁定模式

系统默认： 不需要等待某事务结束，可直接查询到结果，但不能再进行修改、删除。
缺点：查询到的结果，可能是已经过期的。
优点：不需要等待某事务结束，可直接查询到结果。

需要用以下模式来设定锁定模式

1、SELECT …… LOCK IN SHARE MODE（共享锁）
查询到的数据，就是数据库在这一时刻的数据（其他已commit事务的结果，已经反应到这里了）
SELECT 必须等待，某个事务结束后才能执行

2、SELECT …… FOR UPDATE（排它锁）
例如 SELECT * FROM tablename WHERE id<200
那么id<200的数据，被查询到的数据，都将不能再进行修改、删除、SELECT …… LOCK IN SHARE MODE操作
一直到此事务结束

共享锁 和 排它锁 的区别：在于是否阻断其他客户发出的 SELECT …… LOCK IN SHARE MODE命令

3、INSERT / UPDATE / DELETE
所有关联数据都会被锁定，加上排它锁

4、防插入锁
例如 SELECT * FROM tablename WHERE id>200
那么id>200的记录无法被插入

5、死锁
自动识别死锁
先进来的进程被执行，后来的进程收到出错消息，并按ROLLBACK方式回滚
innodb_lock_wait_timeout = n 来设置最长等待时间，默认是50秒

事务隔离模式

SET [SESSION|GLOBAL] TRANSACTION ISOLATION LEVEL
READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE
1、不带SESSION、GLOBAL的SET命令
只对下一个事务有效
2、SET SESSION
为当前会话设置隔离模式
3、SET GLOBAL
为以后新建的所有MYSQL连接设置隔离模式（当前连接不包括在内）

隔离模式

   READ UNCOMMITTED
不隔离SELECT
其他事务未完成的修改（未COMMIT），其结果也考虑在内

   READ COMMITTED
把其他事务的 COMMIT 修改考虑在内
同一个事务中，同一 SELECT 可能返回不同结果

   REPEATABLE READ（默认）
不把其他事务的修改考虑在内，无论其他事务是否用COMMIT命令提交过
同一个事务中，同一 SELECT 返回同一结果（前提是本事务，不修改）

   SERIALIZABLE
和REPEATABLE READ类似，给所有的SELECT都加上了 共享锁

出错处理
根据出错信息，执行相应的处理


mysql事物处理实例

MYSQL的事务处理主要有两种方法
1.用begin,rollback,commit来实现
    begin开始一个事务
    rollback事务回滚
    commit 事务确认
2.直接用set来改变mysql的自动提交模式
    mysql默认是自动提交的，也就是你提交一个query，就直接执行！可以通过
    set autocommit = 0 禁止自动提交
    set autocommit = 1 开启自动提交
    来实现事务的处理。
但要注意当用set autocommit = 0 的时候，你以后所有的sql都将作为事务处理，直到你用commit确认或 rollback结束，注意当你结束这个事务的同时也开启了新的事务！按第一种方法只将当前的做为一个事务!
MYSQL只有 INNODB和BDB类型的数据表才支持事务处理，其他的类型是不支持的!
MYSQL5.0 WINXP下测试通过～  ^_^
