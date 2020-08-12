---
title: linux向其它终端发送消息
tags: 
categories: 
- linux
---

## 查看所有打开的终端

	$ w -f // 或者只输入`w`查看打开的所有终端, `w`是who的简写而不是write
	   16:41:45 up 53 days, 47 min,  5 users,  load average: 0.17, 0.31, 0.38
	  USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT
	  root     pts/0     14:23    1.00s  0.07s  0.00s w -f
	  ai       :0        18Jun20 ?xdm?   8days  7.89s /usr/libexec/gnome-session-binary --session gnome-classic
	  ai       pts/1     18Jun20 53days  3:28m  3:28m /usr/lib/virtualbox/VirtualBox
	  ai       pts/2     16:29   10:49   0.07s  0.02s bash
	  root     pts/3     16:27    9:37   0.02s  0.02s -bash

## 向指定终端发送消息

	$ write ai pts/2
	  123

## 广播消息, 向所有终端发送消息

	$ wall 123
	  Broadcast message from root@master-node (pts/0) (Mon Aug 10 16:40:49 2020):
	  123


