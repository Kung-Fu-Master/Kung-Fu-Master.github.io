---
title: java安装与卸载
tags: 
categories:
- linux
---

## Centos7 yum 安装java

	$ yum list |grep java-11
	(optional)$ yum search java
	
	// openjdk*等代表开源的java, Oracle版是闭源的只有可执行java文件
	yum install -y java-11-openjdk.x86_64 java-11-openjdk-devel.x86_64 java-11-openjdk-headless.x86_64
	
	// 查看yum安装过的java
	yum list installed |grep java

### jdk安装位置

	ls /usr/lib/jvm/

## 切换java版本

### (推荐)第一种, 使用update-alternatives切换java版本
 **1. 查看linux系统中的java是auto还是manual选择版本的**


	$ update-alternatives --list | grep java
	$ update-alternatives --display java
	java - status is auto.
	 link currently points to /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.275.b01-0.el7_9.x86_64/jre/bin/java
	/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.261-2.6.22.2.el7_8.x86_64/jre/bin/java - family java-1.7.0-openjdk.x86_64 priority 1700261
 **2. 切换java版本**
 * 方法1:


	$ update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-11-openjdk-11.0.9.11-2.el7_9.x86_64/bin/java 100
usage: alternatives --install <link> <name> <path> <priority>

 * 方法2:


	$ update-alternatives --config java
	There are 3 programs which provide 'java'.
	
	  Selection    Command
	-----------------------------------------------
	   1           java-1.7.0-openjdk.x86_64 (/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.261-2.6.22.2.el7_8.x86_64/jre/bin/java)
	*+ 2           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64/jre/bin/java)
	   3           java-11-openjdk.x86_64 (/usr/lib/jvm/java-11-openjdk-11.0.9.11-2.el7_9.x86_64/bin/java)
	
	Enter to keep the current selection[+], or type selection number: 3

**3. 再次查看你linux系统java版本**


	$ update-alternatives --display java
	java - status is manual.
	 link currently points to /usr/lib/jvm/java-11-openjdk-11.0.9.11-2.el7_9.x86_64/bin/java
	/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.261-2.6.22.2.el7_8.x86_64/jre/bin/java - family java-1.7.0-openjdk.x86_64 priority 1700261

### 第二种, 添加java环境变量
如果你定义了JAVA_HOME环境变量，根据你设置的Java版本更新变量, 如下.  
编辑配置文件，设置java环境变量  

	$ vim /etc/profile
	#set java environment
	JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.9.11-2.el7_9.x86_64
	JRE_HOME=$JAVA_HOME
	CLASS_PATH=.:$JRE_HOME/lib
	PATH=$PATH:$JAVA_HOME/bin
	export JAVA_HOME JRE_HOME CLASS_PATH PATH
让修改生效：

	source /etc/profile
验证：

	java -version

### 切回auto版本

	$ update-alternatives --auto java

### 删除某个版本:

	$ update-alternatives --remove java /usr/lib/jvm/java-11-openjdk-11.0.9.11-2.el7_9.x86_64/bin/java

## yum卸载java

	rpm -qa | grep java
	// 卸载java 相关的包, 
	yum -y remove java-1.8.0-open*
	yum -y remove java-11-open*
	
	rpm -qa | grep jdk
	// 卸载jdk相关的包
	yum -y remove copy-jdk-configs-3.3-10.el7_5.noarch
