---
title: Linux 永远修改时间
tags: 
categories:
- linux
---
安装在虚拟机上的CentOS7的时间分为系统时间和硬件时间。二者都修改，重启系统（init 6 )才会永久生效。

修改步骤如下
查看当前系统时间 date
修改当前系统时间 date -s "2018-2-22 19:10:30"
查看硬件时间 hwclock --show
修改硬件时间 hwclock --set --date "2018-2-22 19:10:30"
同步系统时间和硬件时间 hwclock --hctosys
保存时钟 clock -w
重启系统（init 6）后便发现系统时间被修改了

init0:关机
init1：单用户形式，只root进行维护
init2：多用户，不能使用net file system
init3：完全多用户
init5：图形化
init6：重启
init是Linux系统操作中不可缺少的程序之一。所谓的init进程，它是一个由内核启动的用户级进程。 
内核自行启动（已经被载入内存，开始运行，并已初始化所有的设备驱动程序和数据结
构等。之后，就通过启动一个用户级程序init的方式，完成引导进程。所以,init始终是第一个进程（其进程编号始终为1）。 
内核会在过去曾使用过init的几个地方查找它，它的正确位置（对Linux系统来是/sbin/init）。如果内核找不到init，它就会试着运行/bin/sh，如果运行失败，系统的启动会失败。
