---
title: 连接router后机器连不上外网
tags: 
categories:
- linux
---
几台机器连接同一个路由器router后，centos机器连不上外网
[root@sphm001 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1    0.0.0.0         UG    100    0        0 enp0s31f6
0.0.0.0         10.239.173.1    0.0.0.0         UG    101    0        0 enp0s20f0u8
10.239.173.0    0.0.0.0         255.255.255.0   U     101    0        0 enp0s20f0u8
192.168.1.0     0.0.0.0         255.255.255.0   U     100    0        0 enp0s31f6
[root@sphm001 ~]#
[root@sphm001 ~]# route del -net 0.0.0.0 gw 192.168.1.1
[root@sphm001 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.239.173.1    0.0.0.0         UG    101    0        0 enp0s20f0u8
10.239.173.0    0.0.0.0         255.255.255.0   U     101    0        0 enp0s20f0u8
192.168.1.0     0.0.0.0         255.255.255.0   U     100    0        0 enp0s31f6
[root@sphm001 ~]#
[root@sphm001 ~]# wget www.baidu.com
--1998-01-29 23:28:17--  http://www.baidu.com/
Resolving child-prc.intel.com (child-prc.intel.com)... 10.239.4.101
Connecting to child-prc.intel.com (child-prc.intel.com)|10.239.4.101|:913... connected.
Proxy request sent, awaiting response... 200 OK
Length: 2381 (2.3K) [text/html]
Saving to: ‘index.html.1’

100%[============================================================================================>] 2,381       --.-K/s   in 0s

1998-01-29 23:28:17 (81.6 MB/s) - ‘index.html.1’ saved [2381/2381]
修改完路由如果还不行就重启机器试试
[root@sphm001 ~]#


linux上用route添加/删除路由
1. 查看
route -n
 2. 添加
route add -net 9.123.0.0 netmask 255.255.0.0 gw 9.123.0.1
 3. 删除
route del -net 9.123.0.0 netmask 255.255.0.0 gw 9.123.0.1
 
有些童鞋问怎么加永久路由，很简单，把 “route add -net 9.123.0.0 netmask 255.255.0.0 gw 9.123.0.1” 写到/etc/rc.local最后就行了，
或者复制编辑好执行也可以：
echo “route add -net 9.123.0.0 netmask 255.255.0.0 gw 9.123.0.1” >> /etc/rc.local
 
实际中发现，CentOS7.4默认环境下，rc.local中不会执行，需要赋权才可以
chmod +x /etc/rc.local

From <https://www.cnblogs.com/lynsen/p/8027446.html> 


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
SPH + RVP
root@imie:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.239.173.1    0.0.0.0         UG    0      0        0 eth1
10.239.173.0    0.0.0.0         255.255.255.0   U     0      0        0 eth1
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
root@imie:~#


root@imie:~# vim /etc/network/interfaces
iface lo inet loopback

# Wired interfaces
auto eth0 eth1
iface eth0 inet dhcp
iface eth1 inet dhcp

# iface eth0 inet manual
# iface eth1 inet manual

# Network bridge
# auto br0
# iface br0 inet dhcp
#   bridge_ports eth0 eth1
