---
title: 基于git的源代码管理模型——git flow
date: 2016-09-30 13:56:39
tags: 
  - git
---



Git Flow是构建在Git之上的一个组织软件开发活动的模型，是在Git之上构建的一项软件开发最佳实践。GitFlow是一套使用Git进行源代码管理时的一套行为规范和简化部分Git操作的工具。

<!-- more -->


**附录**

##### 安装

_Windows_

配合Cygwin使用，执行指令(需要安装git，util-linux， wget)

```
wget -q -O - --no-check-certificate https://github.com/nvie/gitflow/raw/develop/contrib/gitflow-installer.sh | bash
```

使用这条命令在大多数情况下都可以完成GitFlow的安装。但在少数情况下也会遇到异常：

- 执行 git flow init 命令后出现错误："flags: FATAL unable to determine getopt version"
- 需要使用Cygwin的Util-linux安装包。重新使用cywgin的setup安装吧。

如果出现类似"$'\r': command not found"的错误，则可能是换行符出现了问题。可以使用sed命令来快速解决。

`$ sed -i 's/\n\r/\n/mg' /usr/local/bin/git-flow*`
`$ sed -i 's/\n\r/\n/mg' /usr/local/bin/gitflow-*`

**Reference**:
[图灵git glow](http://www.ituring.com.cn/article/56870)
[如何正确使用Git Flow](http://www.cnblogs.com/cnblogsfans/p/5075073.html)
[Using git-flow to automate your git branching workflow](http://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/)
[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
