--------------
title: Hadoop 环境搭建
date: 2016-10-08 12:37:10
tags: 
	- Hadoop
categories: 
--------------

简要记录一下hadoop的环境搭建已经在idea和eclipse中小试牛刀。
OS：win10, cygwin
hadoop运行模式：独立模式
<!-- more -->

#### 环境搭建

###### 下载安装包：[hadoop-3.0.0-alpha1](http://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.0.0-alpha1/hadoop-3.0.0-alpha1.tar.gz)
###### 进入/opt， 解压 tar zvxf
###### 设置环境变量：`HADOOP_HOME`, `path`
###### 在 ~/.bashrc中设置环境变量
`export HADOOP_CLASSPATH=$(cygpath -pw $(hadoop classpath)):$HADOOP_CLASSPATH`
###### 验证安装：

 - source ~/.bashrc
 - hadoop version

###### 测试代码

```
cd /opt/hadoop-3.0.0-alpha1; mkdir input; cd input; echo "Hello World" > test

hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount input output

```

<!-- #### idea开发环境配置 -->