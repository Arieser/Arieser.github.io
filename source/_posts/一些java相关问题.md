-----
title: 一些java相关问题
Date: 2016-05-23 11:25:32
tags: 
  - java
categories: 
  - Java
-----
### JAVA关键字

#### final
- 被final修饰的变量必须被初始化

> 初始化的方式
    - 在定义的时候初始化
    - 在初始化块中初始化，不可以在静态初始化块中初始化
    - 静态final变量可以在静态初始化块中初始化，不可以在初始化块中初始化
    - final变量还可以在类的构造器中初始化，但是静态final变量不可以

#### finally
> 作用
    - 程序抛出了异常
    - 执行了finally语句块

#### abstract
> 目的是要让子类继承再定义，让抽象的概念得以设计

### 测试题
- `int j=0; for(...){j=j++;} System.out.println(j);`输出结果是？
  - Answer:0



