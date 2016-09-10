-----
title: cygwin配置apache
Date: 2016-08-11 16:45:46
tags: 
    - apache
    - Cygwin
categories: 
-----

原本想在windows配置apache，转念一想觉得应该在linux环境下配置一下，简略记述一下配置步骤。

##### 安装 apr 

```
tar zvxf http://apache.freelamp.com/apr/apr-1.4.2.tar.gz 
cd apr-1.4.2
./configure –-prefix=/usr/local/apr
make
make install
```

##### 安装apr-util

```
tar zvxf http://apache.freelamp.com/apr/apr-util-1.3.10.tar.gz
cd apr-util-1.3.10
./configure –-prefix=/usr/local/web/apr-util –with-apr=/usr/local/apr
make
make install
```
/**apr-util 安装失败**/

##### 安装prce

```
unzip -o pcre-8.10.zip
cd pcre-8.10
./configure --prefix=/usr/local/pcre
make
make install
```


##### 安装apache

```
 ./configure --prefix=/usr/local/apache --enable-rewrite --enable-so --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with-pcre=/usr/local/pcre
make
make install
```

