---
title: Intel_tips_at_work
tags:
categories:
- technologies
---

### 聊天邮件等添加表情
windows键+分号键";" 能够选择表情

### 关于cube主机连接网络问题
cube 台式机网线连接C口，宿主主机安装Virtualbox后再安装linux虚拟机，网络选择桥接模式，跟宿主主机网络处于同一网段，与宿主主机互不影响.
台式机连接A口，宿主主机设置proxy后可以连接外网，但是virtualbox安装的虚拟机就连不到外网了
安装好虚拟机OS，
$ sudo passwd root
输入user用户密码，再设置root密码，下次就可以$ su root 获取root权限了
