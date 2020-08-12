---
title: Ubuntu18.04 安装 Mysql8.0
tags: 
categories:
- technologies
- maven
---

# Mysql 安装:

### 1. 下载deb包
	https://dev.mysql.com/downloads/repo/apt/

### 2. 跟新dpkg
	$ dpkg -i mysql-apt-config_0.8.15-1_all.deb
	$ apt update

### 3. 安装mysql8
	$ apt install mysql-server
	输入密码123456
	最后加密方式选择Legacy(5.x)

# Mysql 卸载:

```
搜索的一种卸载方式:
首先在终端中查看MySQL的依赖项：dpkg --list|grep mysql
卸载： sudo apt-get remove mysql-common
卸载：sudo apt-get autoremove --purge mysql-server-5.7
清除残留数据：dpkg -l|grep ^rc|awk ‘{print$2}’|sudo xargs dpkg -P
再次查看MySQL的剩余依赖项：dpkg --list|grep mysql
继续删除剩余依赖项，如：sudo apt-get autoremove --purge mysql-apt-config
至此已经没有了MySQL的依赖项，彻底删除，Good Luck

另外一种卸载方式:
sudo apt-get autoremove --purge mysql-server 
sudo apt-get remove mysql-common
sudo rm -rf /etc/mysql/ 
sudo rm -rf  /var/lib/mysql
​​#清理残留数据
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P  
sudo apt autoremove
sudo apt autoclean

最终是用dpkg --list|grep mysql命令查看没有任何mysql信息输出即可
```

# Mysql登录

### 第一种命令行方式:
```
 $ mysql -uroot -p123456
```
### 第二种mysql-workbench
 * $ apt update
 * $ apt install mysql-workbench
 * $ mysql-workbench		// 可以通过键入 mysql-workbench或单击 MySQL Workbench 图标 (Activities -> MySQL Workbench) 从命令行启动它。

### 第三种Navigat 工具方式：
 - Navicat是可以管理多种数据库Mysql, redis, MongoDB等等的软件，收费
 * 连接名:localhost
 * 主机: 127.0.0.1		// 用localhost 会报错 2002 - Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock'(2 "No such file or directory")
 * 端口: 3306			// mysql安装后默认服务端口是3306， 可通过命令 "netstat -tap | grep mysql" 查看
 * 用户名: root
 * 密码: 123456
 
# 重启Mysql server
 * $ systemctl restart mysql	// Ubuntu18.04重启mysql会出错, 目前没解决，只是卸载mysql重装，最好别重启mysql服务

