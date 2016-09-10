-----
title: maven 随记
Date: 2016-09-08 16:48:02
tags: 
categories: 
    - Maven
-----

#### mvn package 跳过测试的方法

- -DskipTests，不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下
- -Dmaven.test.skip=true，不执行测试用例，也不编译测试用例类



