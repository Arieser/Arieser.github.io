---
title: AWS linux(AMI) 搭建VPN
date: 2016-11-02 22:36:15
tags: 
  - VPN
---



AWS提供一年的免费试用($2.00)，试着在amazon linux(AMI) 上搭建vpn，简要记录一下搭建过程。

<!-- more -->

安装过程如下：

#### ssh配置

1. 给pem权限
    `chmod 400 data_korea.pem`

2. 登录ec2
    `ssh -i "data_korea.pem" ec2-user@ec2-52-78-70-0.ap-northeast-2.compute.amazonaws.com`

3. 修改默认用户和root密码
    `sudo passwd ec2-user`
    `suod paddwd root`

4. 切换root用户，修改文件

    `su root`
    `vim /etc/ssh/sshd_config`

    ```
    PermitRootLogin yes
    PubkeyAuthentication no
    PasswordAuthentication yes
    ```

5. reboot或者重启ssh  `/etc/init.d/sshd restart`


#### vpn安装

1. 安装ppp
    `yum install ppp`

2. 下载并安装pptpd

    `wget http://poptop.sourceforge.net/yum/stable/packages/ppp-2.4.5-33.0.fc21.x86_64.rpm`
    `wget http://poptop.sourceforge.net/yum/stable/packages/pptpd-1.4.0-1.el6.x86_64.rpm`
    `rpm -Uhv pptpd*.rpm`

3. 添加DNS服务器 (可选)

    打开vim /etc/ppp/options.pptpd 文件并添加入如下内容：
    `ms-dns 8.8.8.8`
    `ms-dns 8.8.4.4`
    `ms-dns 4.4.4.4`
    以上2个是Google提供的免费DNS

4. 添加 VPN 帐号
    在/etc/ppp/chap-secrets文件中添加VPN用户，格式为“用户名 服务器 密码 IP地址”：
    `vpnuser pptpd myVPN$99 *`

5. 打开IP转发(IP Forward)功能
    在 /etc/sysctl.conf 文件中修改：
    `net.ipv4.ip_forward = 1`

6. 保存设置
    `sysctl -p`

7. 在 IP Tables 中开启 IP 伪装(IP Masquerade)
    `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`
    如果你需要这个设置在重启之后依然有效，则需要把这一行添加到 /etc/rc.local 的末尾。

8. 把 pptpd 设置成自动运行的
    `chkconfig pptpd on`

9. 重启pptpd服务
    `service pptpd restart`

10. ec2控制台打开TCP的1723端口，这是pptpd的默认连接端口。

Reference：[https://leonax.net/p/3274/install-vpn-server-on-amazon-ec2/](https://leonax.net/p/3274/install-vpn-server-on-amazon-ec2/)

#### AMI 软件更新

1. jdk 更新
    ```
    rpm -qa|grep java   //查询系统jdk
    rpm -e --allmatches --nodeps java-1.6.0-openjdk-1.6.0.37-1.13.9.4.el5_11    //删除老版本
    yum -y list java*   (yum search jdk)                    //查询软件包内的jdk
    yum install java-1.8.0-openjdk.x86_64                   //安装新版本
    java -version                                           //验证
    ```

2. jdk的安装路径加入到JAVA_HOME

    `vi /etc/profile`

    ```shell
        #set java environment
        JAVA_HOME=/usr/lib/jvm/jre-1.6.0-openjdk.x86_64
        PATH=$PATH:$JAVA_HOME/bin
        CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
        export JAVA_HOME CLASSPATH PATH
    ```

    `. /etc/profile`
