---
title: java安装与卸载
tags: 
categories:
- linux
---


## Centos7 yum 安装java

	yum search java
	
	// openjdk*等代表开源的java, Oracle版是闭源的只有可执行java文件
	yum install java-11-openjdk.x86_64
	
	// 查看yum安装过的java
	yum list installed |grep java

### jdk安装位置

	ls /usr/lib/jvm/


### yum卸载java

	rpm -qa | grep java
	// 卸载java 相关的包, 
	yum -y remove java-1.8.0-open*
	yum -y remove java-11-open*
	
	rpm -qa | grep jdk
	// 卸载jdk相关的包
	yum -y remove copy-jdk-configs-3.3-10.el7_5.noarch

