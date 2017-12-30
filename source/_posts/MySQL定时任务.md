-----
title: MySQL定时任务
date: 2016-08-04 10:30:26
tags: 
categories: 
    - MySQL
-----

### 简介

数据定时更新非常有必要，自MySQL5.1.6起，增加了事件调度器，可以用来执行某些定时任务。简要记录一下创建过程。

<!-- more -->

### 配置

1. windows10
2. MySQL5.6

### 过程

#### 开启event_scheduler

> `set global event_scheduler = 1;`

> my.cnf 加上 `event_scheduler = 1`

> `set global event_scheduler = ON;`

> `mysqld  --event_scheduler=1;`

查看是否开启了event_scheduler

> `show varuables like 'event_scheduler';`

> `select @@event_scheduler;`

> `show processlist;`


#### 创建事件(create event)

##### 语法

```
CREATE EVENT [IFNOT EXISTS] event_name
ONSCHEDULE schedule
[ONCOMPLETION [NOT] PRESERVE]
[ENABLE | DISABLE]
[COMMENT 'comment']
DO sql_statement;
```

schedual:

```
AT TIMESTAMP [+ INTERVAL INTERVAL]
| EVERY INTERVAL [STARTS TIMESTAMP] [ENDS TIMESTAMP]
```

INTERVAL:

```
quantity {YEAR | QUARTER | MONTH | DAY | HOUR | MINUTE |
WEEK | SECOND | YEAR_MONTH | DAY_HOUR | DAY_MINUTE |
DAY_SECOND | HOUR_MINUTE | HOUR_SECOND | MINUTE_SECOND}
```

##### 示例

1. 每秒插入一条记录
```
USE tarena;
CREATE TABLE aaa (timeline TIMESTAMP);
CREAT EEVENT e_test_insert
ON SCHEDULE EVERY 1 SECOND
DO INSERTINTO tarena.aaa VALUES(CURRENT_TIMESTAMP);
```

2. 5天后清空表
```
CREATE EVENT e_test
ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 5 DAY
DO TRUNCATETABLE tarena.aaa;
```

3. 预约清空
```
CREATE EVENT e_test
ON SCHEDULE AT TIMESTAMP '2007-07-20 12:00:00'
DO TRUNCATETABLE tarena.aaa;
```

4. 定时清空
```
CREATE EVENT e_test
ON SCHEDULE EVERY 1 DAY
DO TRUNCATETABLE tarena.aaa;
```

5. 预约定时清空
```
CREATE EVENT e_test
ONSCHEDULE EVERY 1 DAY
STARTS CURRENT_TIMESTAMP+ INTERVAL 5 DAY
DO TRUNCATETABLE tarena.aaa;
```

6. 定时清空，一段时间后停止
```
CREATE EVENT e_test
ON SCHEDULE EVERY 1 DAY
ENDS CURRENT_TIMESTAMP+ INTERVAL 5 DAY
DO TRUNCATETABLE test.aaa;
```

7. 预约定时清空，一段时间后停止
```
CREATE EVENT e_test
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_TIMESTAMP+ INTERVAL 5 DAY
ENDS CURRENT_TIMESTAMP+ INTERVAL 1 MONTH
DO TRUNCATETABLE test.aaa;
```

8. 定时清空，执行一次后终止
`[ON COMPLETION [NOT] PRESERVE]可以设置这个事件是执行一次还是持久执行，默认为NOT PRESERVE。`
```
CREATE EVENT e_test
ON SCHEDULE EVERY 1 DAY
ON COMPLETION NOT PRESERVE
DO TRUNCATETABLE test.aaa;
```

`[ENABLE | DISABLE]可是设置该事件创建后状态是否开启或关闭，默认为ENABLE。
　　[COMMENT ‘comment’]可以给该事件加上注释。`

#### 修改事件(ALTER EVENT)

##### 语法

```
ALTER EVENT event_name
[ONSCHEDULE schedule]
[RENAME TOnew_event_name]
[ONCOMPLETION [NOT] PRESERVE]
[COMMENT 'comment']
[ENABLE | DISABLE]
[DO sql_statement]
```

##### 临时关闭事件

`ALTER EVENT e_test DISABLE;`

##### 开启事件

`ALTER EVENT e_test ENABLE;`

##### 时间点修改

`ALTER EVENT e_test ON SCHEDULE EVERY 5 DAY;`

#### 删除事件(DROP EVENT)

##### 语法

`DROP EVENT [IF EXISTS] event_name`

#### 案例

```
delimiter //
create  procedure `Slave_Monitor`()
begin
SELECT VARIABLE_VALUE INTO @SLAVE_STATUS
FROM information_schema.GLOBAL_STATUS
WHERE VARIABLE_NAME='SLAVE_RUNNING';
IF ('ON'!= @SLAVE_STATUS) THEN
SET GLOBAL SQL_SLAVE_SKIP_COUNTER=0;
SLAVE START;
END IF;
end; //
delimiter ;
```


```
CREATE EVENT IFNOT EXISTS `Slave_Monitor`
ON SCHEDULE EVERY 5 SECOND
ON COMPLETION PRESERVE
DO CALL Slave_Monitor();
```