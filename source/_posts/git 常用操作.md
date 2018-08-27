---
title: git 常用操作
date: 2016-10-20 21:20:45
tags: 
  - git
---

## 记录git一些常用或者容易混淆的操作

![1535355590414](/images/1535355590414.png)

<!-- more -->

### 设置http避免重复输入密码

git 操作中，当远程库是通过http更新时，需要每次输入密码，比较繁琐，简要记录一下不需要输入密码的操作，即可输入一次就不用再手输入密码的困扰而且又享受https带来的极速。

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

### git删除远程分支

首先从上游同步

`git fetch upstream`

#### merge 到本地 master 分支

`git merge upstream/master`
或者也可以直接一步到位

`git pull upstream master`
然后删除本地分支
`git branch -d erlang numix`
最后删除远程分支
`git push --delete origin erlang numix`
或者也可以使用

`git push origin :erlang :numix`
关于上面一行的解释
下面一行用于将本地的 erlang 分支 Push 到远程 origin/erlang 分支

`git push origin erlang:erlang`
而下面的一句实际上就是将本地的 空分支 Push 到远程 origin/erlang 分支, 因此就相当于删除远程 origin/erlang 分支

`git push origin :erlang`
然后就是同时对 erlang 和 numix 两个分支同时操作, 最终得到下面的方法

`git push origin :erlang :numix`

### 迁移git仓库 --- git mirror

1. 整包克隆

   ```shell
   git clone --mirror git@git.huimei-inc.org:huimei/admin
   ```

2. 到gitlab新建一个空的repo

3. 进入目录，设置新的remote位置

   ```shell
   git remote set-url --push origin git@gitlab.com:silloy/hmadmin.git
   ```

4. 更新本地branch，删除本地origin/xxx (-p == --prune)

   ```shell
   git fetch -p origin
   ```

5. 推送整个repo

   ```shell
   git push --mirror
   ```

6. 可以看到所有分支都以push成功

Reference:
[https://my.oschina.net/amath0312/blog/389775](https://my.oschina.net/amath0312/blog/389775)

[Set up git to pull and push all branches](https://stackoverflow.com/questions/1914579/set-up-git-to-pull-and-push-all-branches)