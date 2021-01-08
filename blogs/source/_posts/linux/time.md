---
title: time
tags: 
categories:
- linux
---

## 第一种查看并修改时间
安装在虚拟机上的CentOS7的时间分为系统时间和硬件时间。二者都修改，重启系统（init 6 )才会永久生效。

修改步骤如下

查看当前系统时间

```shell
	date
	Tue Sep 15 16:00:51 CST 2020
	//`CST` 代表China Standard Time
	
	date -R
	Tue, 15 Sep 2020 16:00:55 +0800
	// +0800表示我们国家的东八区; -0800表示西八区，是美国旧金山所在的时区
```

修改当前系统时间 `date -s "2018-2-22 19:10:30"`
查看硬件时间 `hwclock --show`
修改硬件时间 `hwclock --set --date "2018-2-22 19:10:30"`
同步系统时间和硬件时间 `hwclock --hctosys`
保存时钟 `clock -w`
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

## 选择时区

```shell
	tzselect
```
然后根据提示输入5选择`5) Asia`回车, 然后9选择`9) China`回车,然后1选择`1) Beijing Time`.  
tzselect命令只告诉你选择的时区的写法，并不会生效。所以现在它还不是东8区北京时间。你可以在.profile、.bash_profile或者/etc/profile中设置正确的TZ环境变量并导出。 例如在.bash_profile里面设置 TZ='Asia/Shanghai'; export TZ并使其生效。





