---
title: Centos 更改kernel 默认启动项
tags: 
categories:
- linux
---

在生成grub.cfg之前，最好先备份原始的grub.cfg文件

uname -r   #查看当前内核
rpm -qa | grep kernel   #显示已经安装的内核 
cat /boot/grub2/grub.cfg | menuentry   #查看启动项 

grub2-set-default "CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)" #配置默认内核
grub2-editenv list #查看默认启动项

grub2-mkconfig -o /boot/grub2/grub.cfg   #生成配置 

From <https://blog.csdn.net/c935612575/article/details/81558352?utm_source=blogxgwz4> 
