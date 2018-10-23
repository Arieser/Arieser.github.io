---
title: hexo同步
date: 2016-06-22 22:09:32
tags: 
  - hexo
---

简要介绍一下hexo搭建的github page同步过程。

我已经在github上建立了hexo的源码分支hexo，以及主页分支master。
过程：
- 准备工作 git(Cygwin)，nodejs(win10)安装
- 依次执行的命令
    1. git clone -b hexo git@github.com:Arieser/Arieser.github.io.git hexo
    2. cd hexo
    3. npm install -g hexo-cli
    4. npm install
    5. npm install hexo-deployer-git
- 安装其他依赖包
    - https://github.com/theme-next/theme-next-canvas-nest
    - https://github.com/theme-next/theme-next-fancybox3
    - https://github.com/theme-next/theme-next-pace

参考资料： [使用hexo，如果换了电脑怎么更新博客？](https://www.zhihu.com/question/21193762)