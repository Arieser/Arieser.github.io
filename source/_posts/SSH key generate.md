-----
title: SSH key generate
Date: 2016-06-23 09:35:50
tags: 
    - git
    - SSH
categories: 
-----

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