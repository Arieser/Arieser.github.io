---
title: Logitech Options 在 Mac 下的自定义按键失灵问题
date: 2019-11-23 16:05:00
---



罗技鼠标的自定义按键很好远，但自从升级系统到10.15 catalina后自定义按键就失灵了，重启，重装Logitech Options，重新授权…一番折腾后终于找到了彻底的解决办法

### 环境配置

系统：MacOS 10.15 Catalina

鼠标：MX720

Logitech Options 版本：8.02.86

### 排查步骤

1. 修改Security & Privacy 里的 Logi Options Daemon 和 Logi Options 权限，发现已经勾选了
2. 重新安装Logitech Options勾选权限，仍然无法使用

### 解决办法

1. 删除Logi Options和Logi Options Daemon后，再次添加这两项

    

   ![logi.png](https://raw.githubusercontent.com/silloy/notepic/master/YJvFUL.png)

   

2. Security & Privacy -> Privacy 中添加`Input Monitoring`权限

   ![input.png](https://raw.githubusercontent.com/silloy/notepic/master/Dx40J2.png)

3. 如果需要自定义鼠标截图，还需要添加 `Screen Recording`权限



### 结论

仔细看了Catalina的新特性，新系统对于安全和权限管理更加严格了，所以需要单独处理，至于Logitech Options需要的权限，可以参考[Logitech Options permission prompts on macOS Catalina and macOS Mojave](https://support.logi.com/hc/en-001/articles/360023203954-Logitech-Options-permission-prompts-on-macOS-Mojave)，新系统中其他软件遇到类似的问题，都可以通过这种方式解决

   

**References**

[Logitech Options 在 Mac 下的自定义按键经常会失灵](https://www.v2ex.com/t/572690)

[Logitech Options permission prompts on macOS Catalina and macOS Mojave](https://support.logi.com/hc/en-001/articles/360023203954-Logitech-Options-permission-prompts-on-macOS-Mojave)