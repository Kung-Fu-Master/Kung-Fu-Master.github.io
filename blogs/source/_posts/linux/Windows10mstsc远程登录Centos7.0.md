---
title: Windows10 mstsc远程登录Centos7.0
tags: 
categories: 
- linux
---
配置iptables防火墙
    在xrdp使用是3389端口，所以在iptables中也要开放相应的端口，否则无法访问
 　　iptables -A INPUT -p tcp --dport 3389 -j ACCEPT
    　　service iptables save
From <https://www.cnblogs.com/Skyar/p/5260368.html> 

centos 安装xrdp远程连接桌面
1. 安装epel库，否则无法安装xrdp
    
yum install epel-release
2.安装 xrdp
yum install xrdp
3. 安装tigervnc-server
yum install tigervnc-server
4. 配置xrdp.ini文件
vim /etc/xrdp/xrdp.ini
把max_bpp=32 改成24
5.配置selinux
chcon -t bin_t /usr/sbin/xrdp
chcon -t bin_t /usr/sbin/xrdp-sesman
6.设置xrdp服务，开机自动启动
systemctl start xrdp
systemctl enable xrdp
7.打开防火墙
firewall-cmd  --permanent --zone=public --add-port=3389/tcp
firewall-cmd --reload
8.查看xrdp是否启动
systemctl status xrdp.service
ss -antup|grep xrdp
9.启动window rdp连接
From <https://www.cnblogs.com/Jesse-Li/p/10284221.html> 


2 关闭防火墙
systemctl stop firewalld.service
• 1
设置开机不启动防火墙
systemctl disable firewalld.servie

3、关闭SElinux
1）查看selinux状态
sestatus 
• 1
2）临时关闭selinux
setenforce 0
• 1
永久关闭selinux
vim /etc/selinux/config
SELINUX=disabled

查看服务列表状态:
 1. systemctl list-units --type=service 
 2. systemctl   list-unit-files       列出所有已经安装的  服务  及  状态      （可为人所读,  内容简略、清晰）：

 3. systemctl 可以列出正在运行的服务状态

启动一个服务：
systemctl start postfix.service

关闭一个服务：
systemctl stop postfix.service

重启一个服务：
systemctl restart postfix.service

显示一个服务的状态：
systemctl status postfix.service

在开机时启用一个服务：systemctl enable postfix.service
在开机时禁用一个服务：systemctl disable postfix.service

查看服务是否开机启动：   systemctl is-enabled postfix.service

查看已启动的服务列表：   systemctl list-unit-files | grep enabled

查看启动失败的服务列表：   systemctl --failed

PS：使用命令 systemctl is-enabled postfix.service 得到的值可以是enable、disable或static，这里的 static 它是指对应的 Unit 文件中没有定义[Install]区域，因此无法配置为开机启动服务。

 说明：启用服务就是在当前“runlevel”的配置文件目录   /etc/systemd/system/multi-user.target.wants  里，建立  /usr/lib/systemd/system   里面对应服务配置文件的软链接；
禁用服务就是删除此软链接，添加服务就是添加软连接。

From <https://www.cnblogs.com/devilmaycry812839668/p/8481760.html> 
