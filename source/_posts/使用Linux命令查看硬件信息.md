-----
title: 使用Linux命令查看硬件信息
date: 2016-07-20 16:54:49
tags: 
    - Linux系统
categories:
    - Linux 
-----

在linux检查和查看硬件信息有分很多命令，这里列出一些命令快速查看linux的cpu和内存等信息。

<!-- more -->

#### lscpu

> 直接使用即可，没有多余的选项和功能
  
![lscpu](http://s3.51cto.com/wyfs02/M01/79/26/wKioL1aKKrSCJby_AAI8_6qrUIM167.jpg-s_4117306760.jpg)

#### lspci

> 可以列出所有连接到PCI总线的详细信息，例如：显卡，网卡，USB接口及SATA控制器等设备。

    
![lspci](http://s4.51cto.com/wyfs02/M02/79/28/wKiom1aKKpDhH1sxAAStC5wze8s439.jpg-s_281774679.jpg)
     
> 可以使用类似如下命令过滤出特定的设备信息
 
```
lspci -v | grep "VGA" -A 12 
```

![lspci -v](http://s3.51cto.com/wyfs02/M02/79/26/wKioL1aKKrWTqCqyAALHHCOHEfE161.jpg-s_312483012.jpg)

#### lshw

> 通用工具，可以执行多个硬件如CPU，硬件，USB控制器及磁盘等详细信息。在执行之后会自动提取不同"/proc"文件中的信息。

![lshm](http://s3.51cto.com/wyfs02/M00/79/28/wKiom1aKKpGxhBUmAAQM7upzkak684.jpg-s_4216048.jpg)

#### lsusb

> 显示连接到此计算机的USB控制器的详细信息，可以使用-v选项来输出每个usb端口的详细信息。

![lsusb](http://s1.51cto.com/wyfs02/M00/79/26/wKioL1aKKriyjHmiAAIpnCpeKHo610.jpg-s_1499982087.jpg)

#### lnxi

> 用来获取多项目硬件信息的脚本工具，可以为用户输入一个详细的硬件报告，默认未安装在ubuntu系统当中，可以使用如下命令安装： `sudo apt-get install inxi `
> 使用 `inxi -Fx` 输出硬件报告

![lnxi](http://s1.51cto.com/wyfs02/M01/79/28/wKiom1aKKpSjbhLcAAS78cVW3XE252.jpg-s_3500452332.jpg)

#### df

> 输出当前Linux系统中个各种分区及其挂载点，可以使用-H参数
`df -H`

![df](http://s3.51cto.com/wyfs02/M01/79/28/wKiom1aKKpTQhNhmAAFTK1FKyU4499.jpg-s_1456998239.jpg)

#### free

> 查看当前系统的内存信息
> `free -m`

![free](http://s3.51cto.com/wyfs02/M02/79/26/wKioL1aKKrnRL10pAADkoB1CGrA170.jpg-s_4179713956.jpg)

#### dmidecode

> 主要通过读取DMI表中数据来提取硬件信息。
> 查看CPU信息`sudo dmidecode -t processor `

![dmidecodeCPU](http://s3.51cto.com/wyfs02/M01/79/26/wKioL1aKKrmyEVdzAAOobhhPZYA272.jpg-s_473852599.jpg)

> 查看内存信息`sudo dmidecode -t memory `

![dmidecodeMem](http://s1.51cto.com/wyfs02/M02/79/28/wKiom1aKKpTS1nSrAANCHj_cjjc379.jpg-s_3802672851.jpg)

> 查看BIOS信息 `sudo dmidecode -t bios `

![dmidecodeBIOS](http://s1.51cto.com/wyfs02/M00/79/26/wKioL1aKKrmRu65NAANAzXQhf0k921.jpg-s_1728054736.jpg)

#### hdparm

> 读取SATA设备（eg.硬盘）的相关信息
> sudo hdparm

![hdparm](http://s5.51cto.com/wyfs02/M00/79/28/wKiom1aKKpXj2PXqAAIRBgzXTY0260.jpg-s_1492070872.jpg)