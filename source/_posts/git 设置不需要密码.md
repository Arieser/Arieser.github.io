--------------
title: git 设置不需要密码
Date: 2016-10-20 21:20:45
tags: 
	- git
categories:
	- git 
--------------

git 操作中，当远程库是通过http更新时，需要每次输入密码，比较繁琐，简要记录一下不需要输入密码的操作，即可输入一次就不用再手输入密码的困扰而且又享受https带来的极速。
<!-- more -->

设置记住密码（默认15分钟）：
`git config --global credential.helper cache`

如果想自己设置时间，可以这样做：
`git config credential.helper 'cache --timeout=3600'`
这样就设置一个小时之后失效

长期存储密码：
`git config --global credential.helper store`
`git config --global credential.helper wincred`

增加远程地址的时候带上密码也是可以的。(推荐)
`http://yourname:password@git.oschina.net/name/project.git`
补充：使用客户端也可以存储密码的。

如果你正在使用ssh而且想体验https带来的高速，那么你可以这样做： 切换到项目目录下 ：
`cd projectfile/`

移除远程ssh方式的仓库地址
`git remote rm origin`

增加https远程仓库地址
`git remote add origin http://yourname:password@git.oschina.net/name/project.git`

Reference:
[https://my.oschina.net/amath0312/blog/389775](https://my.oschina.net/amath0312/blog/389775)
