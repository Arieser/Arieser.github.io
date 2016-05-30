-----
title: Oracle
Date: 2016-05-09 11:31:38
tags: 
  - oracle
  - SQL
categories: 
  - 数据库
-----

主流关系型数据库: Oracle，DB2，Sybase，SQL server，MySQL
这里主要记录一下oracle数据库初步学习过程，涉及少量与mysql的不同讨论。基本的SQL知识[SQL](http://www.w3school.com.cn/sql/)
<!--more-->

### SQL（Structured Query Language）
#### 常用关键字
- SQL可分为
  - 数据定义语言DDL: CREATE, ALTER, DROP, TRUNCATE
  - 数据操纵语言DML: INSERT, UPDATE, DELETE
  - 事务控制语言TCL: COMMIT, ROLLBACK, SAVEPOINT
  - 数据查询语言DQL: SELECT
  - 数据控制语言DCL: GRANT, REVOKE, CREATE USER
- NUMBER: 数字类型
  - 定义: NUMBER（P，S）
  - FYI: MySQL里没有number类型，对应的是int，float......
- CHAR: 固定长度的字符类型
  - 定义: CHAR(N)
  - 最多保存2000字节
  - FYI: 对应MySQL里的CHAR
- VARCHAR2: 变长的字符类型
  - 定义: VARCHAR2(N)
  - 最多保存4000字节
  - FYI: 对应MySQL里的VARCHAR
- DATE: 日期时间数据
  - 默认格式: DD-MON-RR
- DEFAULT: 指定默认值
- NOT NULL: 非空约束
- DROP
  - 删除列或表
- LONG: VARCHAR2加长版
  - 限制: 每个表只能有一个long列，不能作为主键，不能建立索引，不能出现在查询条件中
  - FYI: MySQL没有long型
- CLOB: 存储定长或变长字符串，oracle建议使用clob替代long型
- 执行DML操作后，需要在执行commit，才算真正确认了此操作

#### 常用函数
- UPPER, LOWER, INITCAP: 大小写转换
- TRIM, LTRIM, RTRIM: 截去子串
- LPAD, RPAD: 补位函数
  - LPAD(char1, n, char2)
  - RPAD(char1, n, char2)
- SUBSTR: 获取子串
  - SUBSTR(char, m[, n])
- INSTR: 返回子串在父串中的位置
  - INSTR(char1, char2[, n[, m]])
    - n: 开始搜索的位置，默认为1
    - m: 指定子串第m次出现，默认为1
    - 没找到，返回0
- ROUND：四舍五入
  - ROUND(n[, m])
    - n: 可以是任何数字，指要被处理的数字
    - m：必须是整数，四舍五入的位数，可以是负数，默认是0
- MOD: 取余
  - MOD(m, n)  n为0直接返回m
- CEIL
  - CEIL(n)
- FLOOR
  - FLOOR(n)
- CONCAT, "||"

##### 日期操作
- DATE
  - 范围：公元前4712年1月1日至公元9999年12月31日
  - 占用7个字节
  - byte 1: 世纪+100
  - byte 2: 年
  - byte 3: 月
  - byte 4: 日
  - byte 5: 小时 + 1
  - byte 6: 分 + 1
  - byte 7: 秒 + 1 
- TIMESTAMP
  - 最高精度可以到ns
  - 占用7或者11个字节，精度为0，用7字节存储，精度大于0用11字节存储
  - 精度：第8-11字节，内部运算类型为整型
- SYSDATE: SystemDate
- SYSTIMESTAMP: 返回当前系统日期和时间，精确到毫秒
- TO_DATE：字符串转换为日期类型
  - TO_DATE(char[, fmt[, nlsparams]])
    - fmt: 格式
    ![date](../../img/date.jpg)
    - nlsparams: 指定日期语言
- TO_CHAR
  - TO_CHAR(date[, fmt[, nlsparams]])
- LAST_DAY
  - LAST_DAY(date): 返回日期date所在月的最后一天
- ADD_MONTHS
  - ADD_MONTHS(date, i)
- MONTHS_BETWEEN
  - MONTHS_BETWEEN(date1, date2): 计算月份差
- NEXT_DAY
  - NEXT_DAY(date, char): 返回date日期的下一个周几
- LEAST, GREATEST
  - GREATEST(expr1[,expr2[, expr3]]...)
- EXTRACT
  - EXTRACT(date from datetime): 提取日期中的年、月、日等

##### 空值操作
- NVL
  - NVL(expr1, expr2): expr1为null则转变为expr2，数据类型必须一致
- NVL2
  - NVL2(expr1, expr2, expr3): expr1非null返回我想expr2，为null返回expr3

#### 高级查询
##### 子查询
##### 分页查询
- ROWNUM：伪列，用于返回标识行数据顺序的数字
  - 只能从1计数，不能从结果集中直接截取
  - 部分数据需要用到行内视图
  - PageN: (n - 1) * pageSize + 1   ~   n * pageSize

##### DECODE函数
- DECODE(expr, search1, result1[,search2, result2....][, default])
- 用途： 比较expr的值，若匹配到哪一个search，返回对应result，类似于case语句
- 应用场景：
  - 分组查询

  ```
  SELECT DECODE(job, 'ANALYST', 'VIP', 'MANAGER', 'VIP', 'OPERATION')job, COUNT(1)job_count FROM emp 
  GROUP BY DECODE(job, 'ANALYST', 'VIP', 'MANAGER', 'VIP', 'OPERATION');
  ```
  
  - 字段内容排序
  `ORDER BY DECODE(job, 'ANALYST', '1', 'MANAGER', '2', 'OPERATION', '3')`

##### ROW_NUMBER函数
- ROW_NUMBER() OVER(PARTITION BY col1 ORDER BY col2): 根据col1分组，在分组内部根据col2排序
- 此函数计算的值标识每组内部排序后的顺序编号，组内连续且唯一

##### RANK_NUMBER函数
- 类似于ROW_NUMBER, 跳跃排序，可以有重复值

##### DENSE_RANK
- 类似于RANK_NUMBER, 连续排序，有重复，无跳跃

##### UNION
- UNION: 自动合并去重，排序
- UINION ALL：合并不去重，不排序
- INTERSECT：交集
- MINUS：差集

##### 高级分组函数
- ROLLUP: 从右向左以一次少一列的方式组合直到所有列都去掉
  - GROUP BY ROLLUP(a, b, c)
- CUBE: 所以维度的取值集合
  - GROUP BY CUBE(a, b, c)
- GROUPING SETS:
  - GROUP BY GROUPING SETS(a, b, c)

#### 视图
- 视图(VIEW) 也称虚表，是一组数据的逻辑表示，对应于一条select语句
- 分类：
  - 简单视图
  - 复杂视图
  - 连接视图
- 作用：
  - 简化复杂查询
  - 限制数据访问
    - GRANT CREATE VIEW TO user;
- 限制约束
  - [WITH CHECK OPTION]
  - [WITH READ ONLY]

#### 序列
- 创建序列
  - CREATE SEQUENCE[schema.]sequence_name
    [START WITH i][INCREMENT BY j][MAXVALUE m | NOMAXVALUE][MINVALUE n | NOMINVALUE][CYCLE | NOCYCLE][CACHE p | NOCACHE]
- 使用序列
  - NEXTVAL: 获取序列的下个值
  - CURRVAL：获取序列的当前值

#### 索引
- 索引(INDEX)是一种允许直接访问数据表中某一数据行的数据结构，为了提高查询效率而引入，是独立于表的对象，可以存放与表不同的表空间中
- Syntax：
  - CREATE [UNIQUE] INDEX index_name ON table(column[, column...]);
  - UNIQUE表示唯一索引

#### 约束
- 非空约束 NOT NULL
- 唯一性约束 UNIQUE
- 主键约束 PRIMARY KEY
- 外键约束 FOREIGN KEY
- 检查约束 CHECK












