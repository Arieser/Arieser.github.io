---
title: git删除远程分支
date: 2016-10-01 17:30:11
tags: 
  - git
---

记录git删除远程分支的一些命令

<!-- more -->

首先从上游同步
# 从远程上游同步
`git fetch upstream`

# merge 到本地 master 分支
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