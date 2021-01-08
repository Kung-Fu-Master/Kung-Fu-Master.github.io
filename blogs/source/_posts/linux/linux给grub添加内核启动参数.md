---
title: linux给grub添加内核启动参数
tags: 
categories:
- linux
---
如果你想在系统启动时加载一个内核参数，需修改GRUB的配置模板(/etc/default /grub),添加"名称=值”的键值对到GRUB_CMDLINE_LINUX变量,添加多个时用空格隔开,例如GRUB_CMDLINE_LINUX="...... name=value"(如果没有GRUB_CMDLINE_LINUX变量时,用GRUB_CMDLINE_LINUX_DEFAULT替代即可).
1. Debian or Ubuntu

```shell
$ sudo update-grub  //生成grub的配置文件
$ sudo apt-get install grub2-common  //没有 update-grub命令时,先运行这个安装命令  
```

2. Fedora or CentOS7

```shell
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg //生成grub2的配置文件
$ sudo yum install grub2-tools.x86_64 //没有grub2-mkconfig命令时,先安装grub2-tools
```
带EFI的系统,grub.cfg文件会是在/boot/efi下,比如CentOS7:/boot/efi/EFI/centos/grub.cfg
