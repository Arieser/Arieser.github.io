-----
title: vm环境安装linuxmint
Date: 2016-05-14 20:49:30
tags: 
  - vmvare player
  - linux mint
categories: 
  - linux
-----

为了加深对于linux系统的了解，虽然有公司的服务器可用，觉得有点束手束脚，所以计划在本地搭建一个linux环境，考虑再三，还是决定使用虚拟机，以免造成系统崩溃。
<!--more-->
#### 准备工作
- linux mint下载：linuxmint-17.3-kde-64bit.iso
- vmvare workstation 12 player

#### 安装
- 安装过程不再叙述，全程傻瓜式安装

#### 基本设置
- 分辨率，壁纸， 语言

#### 卸载装机软件
- libreoffice
> `sudo apt-get purge libreoffice?`
> or `sudo aptitude purge libreoffice?`
> or `sudo apt-get remove --purge libreoffice*`

#### 设置软件源，并更新系统
- sudo apt-get update
- sudo apt-get upgrade
- main选择： ustc
- base选择： aliyun

#### 浏览器主页，搜索引擎设置

#### 安装GuakeTerminal
- `sudo apt-get install guake`
- docs: [GuakeTerminal──linux下完美帅气的终端](http://www.2cto.com/os/201410/343251.html)

#### 安装VMWARE tools

#### 安装输入法(不适用于linuxmint18)
- `sudo add-apt-repository ppa:fcitx-team/nightly`
- `sudo aptitude update`
- `sudo aptitude install fcitx fcitx-sogoupinyin fcitx-config-gtk fcitx-frontend-all fcitx-module-cloudpinyin fcitx-ui-classic`
- 下载sogou for linux , 打开im-config设置fcitx，安装sougou
- reboot
- 配置文件（注意取消only show current language）

#### 安装markdown编辑器
- 推荐 cmd_markdown

#### 开发常用软件
- docs：[Linux mint](http://www.jianshu.com/p/c5a29e476526)

#### 安装JAVA jdk
- jre 
1. `sudo apt-get install default-jre`

- open-jdk
1. `sudo apt-get install default-jdk`

- jdk
1. `sudo add-apt-repository ppa:webupd8team/java`
2. `sudo apt-get update`
3. `sudo apt-get install oracle-java8-installer`
4. `sudo apt-get install oracle-java8-set-default`

- docs：
> [怎样在Ubuntu 14.04中安装Java](http://www.linuxidc.com/Linux/2014-09/106445.htm)
> [Linux下配置Java环境变量](http://my.oschina.net/fhd/blog/335156)




