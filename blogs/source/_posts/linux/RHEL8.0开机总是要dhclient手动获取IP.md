---
title: RHEL8.0开机总是要dhclient手动获取IP
tages: 
categories:
- linux
---
使用#dhclient前提是网线得确保连上且没问题
#kill -9 dhclient_thread_ID
解决RHEL8.0开机自动获取IP方法如下:
	1. #ifconfig                                                     //查看第一个网卡名称ifcfg-enp0s20f0u4u1
	2. #cd /etc/sysconfig/network-scripts/
	3. #ls
	ifcfg-enp1s0                                                //发现只有一个网卡配置文件，与#ifconfig查看到得网卡名字都不一致
	4. #cp ifcfg-enp1s0 ifcfg-enp0s20f0u4u1    //手动copy一个与存在得网卡名相同得网卡配置文件
	5. #vim  ifcfg-enp0s20f0u4u1
		○ 删除UUID那一行
		○ 修改NAME=ifcfg-enp0s20f0u4u1
		○ 修改DEVICE=ifcfg-enp0s20f0u4u1
		○ 修改ONBOOT=yes                              //开机即启动网络连接
