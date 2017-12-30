-----
title: SSH server
date: 2016-06-23 09:35:50
tags: 
    - git
    - SSH
categories: 
-----

简要记录一下ssh server搭建过程。
<!-- more -->
### ssh key generate

#### 简介
> SSH提供了一种与git库通讯的方式

#### 步骤

1. 检查ssh key是否存在：ls -al ~/.ssh
2. 生成ssh key： ssh-keygen -t rsa -C "your_email@example.com"
3. 输入passphase（可跳过），是为了避免失误
4. key添加到ssh agent中：

```
eval "$(ssh-agent -s)"
Agent pid 59566
ssh-add ~/.ssh/id_rsa
```

5. 复制key到粘贴板：
        windows   clip < ~/.ssh/id_rsa.pub

参考资料：[如何生成SSH key](http://www.jianshu.com/p/31cbbbc5f9fa/)


### sshd server

1. 以管理员权限打开Cygwin，执行`ssh-host-config -y`命令
2. 启动ssh：`net start sshd` (reference：[http://www.cnblogs.com/kinglau/p/3261886.html](http://www.cnblogs.com/kinglau/p/3261886.html))
3. 配置ssh免密码登录
    1. `ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa`
        - ssh-keygen是生成密钥命令
        - -t 表示指定生成的密钥类型(dsa,rsa)
        - -P表示提供的密语
        - -f指定生成的密钥文件
    2. 注意:~代表当前用户的文件夹，/home/用户名
4.  使用ssh localhost 登录