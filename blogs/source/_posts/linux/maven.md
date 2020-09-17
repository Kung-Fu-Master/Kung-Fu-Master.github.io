---
title: maven
tags: 
categories:
- linux
---

## Centos7 maven 安装

安装maven前提是安装有jdk, yum安装maven时候会自动安装JDK.

	(optional)// 多出来这两步是因为原来机器kernel版本台老, 原生的yum库安装maven自带JDK也比较老, 所以clean再cache, 然后安装maven
	yum clean all
	(optional)
	yum makecache
	
	wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
	yum -y install apache-maven
	
	// 查找包路径
	rpm -qa|grep apache-maven

### 配置阿里云镜像仓库

	vim /etc/maven/setting.xml
	// 定位到mirrors节点下添加下面配置
	  <mirrors>
	     <mirror>
	        <id>nexus-aliyun</id>
	        <mirrorOf>central</mirrorOf>
	        <name>Nexus aliyun</name>
	        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
	     </mirror>
	  </mirrors>

### 配置proxy

	vim /etc/maven/setting.xml
	    <proxy>
	       <id>my-proxy1</id>
	       <active>true</active>
	       <protocol>http</protocol>
	       <host>child-prc.intel.com</host>
	       <port>913</port>
	    </proxy>
	    <proxy>
	       <id>my-proxy2</id>
	       <active>true</active>
	       <protocol>https</protocol>
	       <host>child-prc.intel.com</host>
	       <port>913</port>
	    </proxy>

### 配置本地仓库
yum安装完maven后默认的本地仓库地址:

	ls /root/.m2/repository

配置新的本地仓库:

	vim /etc/maven/setting.xml
	// 定位到这个节点进行编写
	<localRepository>/home/maven/repo</localRepository>

### 指定JDK版本
配置创建项目的版本默认为 JDK8

	vim /etc/maven/setting.xml
	  <profiles>
	    <profile>    
	         <id>jdk-1.8</id>    
	         <activation>    
	           <activeByDefault>true</activeByDefault>    
	           <jdk>1.8</jdk>    
	         </activation>    
	           <properties>    
	             <maven.compiler.source>1.8</maven.compiler.source>    
	             <maven.compiler.target>1.8</maven.compiler.target>    
	             <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>    
	           </properties>    
	    </profile>
	  <profiles>

### (optional)设置tomcat用户名和密码
如果tomcat安装时候或安装后tomcat的配置文件没有设置用户名和密码此处可忽略

	  <servers>
	    <server>
	      <id>tomcat8</id>
	      <username>admin</username>
	      <password>123456</password>
	    </server>
	  </servers>

### 测试maven

	// 查看maven版本
	mvn -v
	
	// 自动到配置的Ali的Maven中央仓库下载缺省的或者Maven中央仓库更新的各种配置文件和类库（jar包)到Maven配置的本地仓库`/home/maven/repo`中
	mvn help:system

## VScode远程连接Centos, 搭建maven项目简单步骤

1. `Ctrl+shift+p`, 搜索`maven`, 选择`Maven:Create Maven Project`
![](1.PNG)
2. 选择`archetype-quickstart-jdk8`
![](2.PNG)
3. 选择最新的`2.0`
![](3.PNG)
4. 选择文件夹来创建项目

5. 输入相关参数

 * groupid: 公司或组织的域名倒序+当前项目名称
 * artifactId： 当前项目的模块名称
 * version： 版本, 可以默认直接enter键

6. 可以发现在项目目录下多出了如下两个文件


	ls /<Project path>/
	pom.xml src/

7. 编译, 测试


	// 1. 编译, 会在项目目录下生成一个target/文件夹
	mvn compile
	ls 
	pom.xml  src/  target/
	
	// 2. 测试编译过的代码
	mvn test
	
	// 3. 执行, 执行的文件名字一定要是`<groupId-name>.<class-name>`
	mvn exec:java -Dexec.mainClass="cloud.App"
	
	// 4. 清理, 删除target文件夹
	mvn clean
中途可能会出错遇到一些问题, 具体查看下面的Problems.


## Problems

### 问题1: Maven Version
Detected Maven Version: 3.5.2 is not in the allowed range 3.6.3.  
修改相应错误产生的包中的pox.xml文件的 requireMavenVersion, 改为所需的3.5.2

	            <configuration>
	              <rules>
	                <requireMavenVersion>
	                  <version>3.5.2</version>
	                </requireMavenVersion>
	              </rules>
	              <fail>true</fail>
	            </configuration>
### 问题2：No compiler
Maven编报错：No compiler is provided in this environment. Perhaps you are running on a JRE rather a JDK?  

**第一步:** 参看上面的**指定JDK版本**确定是否在安装maven时候, 把jdk配置进maven的配置文件/etc/maven/setting.xml文件里

**第二步:** 查看pox.xml文件中的compiler和linux系统中`java -v`和`javac -v`版本是否一致.

	    <maven.compiler.source>1.8</maven.compiler.source>
	    <maven.compiler.target>1.8</maven.compiler.target>



