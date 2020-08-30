---
title: linux 修改系统时间时区
tags: 
categories:
- linux
---

发现一台服务器时间比北京时间慢 12 个小时，使用 date 命令后发现是：

	2019年 06月 04日 星期二 21:50:33 EDT
EDT 时间即美国东部时间。这里要改为北京时间即可：

	mv /etc/localtime /etc/localtime.bak
	ln -s /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
然后再次 date 查看日期：

	2019年 06月 05日 星期三 11:10:58 CST
时间就变成北京时间了.

