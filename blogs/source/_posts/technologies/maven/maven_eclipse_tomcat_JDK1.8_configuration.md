---
title: Maven_eclipse_tomcat_JDK1.8_configuration
tags: 
categories:
- technologies
- maven
---

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
Windows 环境:

下载Maven，搜集网上配置PATH环境变量等
$ vim apache-maven-3.6.3\conf\settings.xml
$ C:\Users\UserName\.m2\settings.xml (没有.m2路径就不设)

1. 设置local repository: // maven从中央仓库下载到本地仓库
```
    <localRepository>C:\Users\UserName\***\software_package\Maven\maven_repository</localRepository>
```

2. 设置proxy:
```
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
```

3. 设置tomcat用户名和密码，如果tomcat安装时候或安装后tomcat的配置文件没有设置用户名和密码此处可忽略
```
    <server>
      <id>tomcat8</id>
      <username>admin</username>
      <password>123456</password>
    </server>
```

设置aliyun 镜像:
```
  <mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/repositories/central</url>
        <!--<url>http://maven.aliyun.com/nexus/content/groups/public</url>-->
    </mirror>
  </mirrors>
```

打开cmd控制台输入：mvn -v 查看版本
$ mvn help:system		// 可看到数据正常下载即为成功
// $ ping repo1.maven.org //此远程repo好像不能访问，不过没关系，上面成功即可


※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

安装tomcat服务器
http://tomcat.apache.org/
然后配置到eclipse

打开C:\Program Files\Tomcat 8.5\conf\server.xml
	网上说需把<Server port="-1" shutdown="SHUTDOWN"> 改为 <Server port="8005" shutdown="SHUTDOWN">
	这里没改, 运行OK

打开C:\Program Files\Tomcat 8.5\conf\tomcat-users.xml
添加如下内容, 设置tomcat密码，也可以不设置，设置后需要在maven的**\config\settings.xml配置文件中也添加tomcat密码
```
<role rolename="manager"/>
<role rolename="manager-gui"/>
<role rolename="admin"/>
<role rolename="admin-gui"/>
<role rolename="manager-script"/>
<user username="admin" password="123456" roles="admin-gui,admin,manager-gui,manager,manager-script"/>
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
官网下载eclipse（免费）
https://www.eclipse.org/downloads/
第一种: 下载安装包:
	直接点击下载，下载后安装选择Eclipse IDE for Enterprise Java Developers就可以了(提前安装配置好JDK1.8等版本环境变量)
第二种： 下载压缩包(解压缩时候有问题),
	Download Packages -> Eclipse IDE for Enterprise Java Developers (includes Incubating components)压缩包
	-> 选择中国镜像 -> 下载
1. 打开eclipse, Window -> Preferences -> Maven -> installations -> Add -> 选择自己装的Maven，否则内嵌的Maven没有proxy
2. 打开eclipse, Window -> Preferences -> Maven -> User Settings 
  -> Global/User Settings都设置为 ***/apache-maven-3.6.3\conf\settings.xml -> Update Settings -> Apply -> Apply and Close


创建 maven web项目并运行:
1. File -> New -> maven Project -> Create a simple project(skip archetype selection) 
	-> Next -> 输入Group Id: com.test ,Artifact Id 输入 test_demo, Packaging： war(选择建立web服务此处必须选为war)
2. 右击项目名 点击最下面Properties -> Maven下面的Project Facets -> 先不勾选Dynamic Web Module，选择右边的Runtimes并选中安装过的tomcat8.5
	-> Apply -> Apply and close -> 重新打开上面的页面 -> 选中Dynamic Web Module 并 修改右边 Version 试着选各个版本使支持
	-> 点击下面出现的 "i Furtherconfiguration available..." -> Conten directory: 内容改为 "src/main/webapp" 
	-> 勾选中下面的Generate web.xml deployment descripter -> Apply -> Apply and Close
3. 在 src/main/webapp 目录下新建个index.js文件内容如下:
```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Insert title here</title>
</head>
<body>
<p>Hello test demo</p>
</body>
</html>
```

4. 右击项目名 -> Run As -> Run on Server， 如果没有Run on Server选项重新打开Project Facets页面再Apply -> Apply and Close
5. 浏览器输入http://localhost:8080/test_demo/即可看到信息: Hello test demo

创建Parent/jar/web项目:
	File -> New -> Other -> Maven -> Maven Project 按照网上操作即可创建maven pom/jar/war三种项目，其中pom是父类管理其它jar和war(web)等project

> 创建好maven项目后，修改jar或war的pom.xml(项目对象模型(Project Object Modet,POM))文件后
	鼠标右击pom.xml -> Maven -> Update Project... -> 可勾选下面的Force Update of Snapshots/Releases -> OK
```
#> eclipse for java ee 创建好maven web项目后会出错，原因是缺少webDemo/src/main/webapp/WEB-INF/web.xml
#第一种：	手动创建文件夹WEB-INF和文件web.xml,然后添加如下内容
#<?xml version="1.0" encoding="UTF-8"?>
#<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
#	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
#	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
#	http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
#</web-app>
#
#第二种: 鼠标右击webDemo -> Java EE Tools -> Generate Deployment Descriptor Stub 即可自动生成上面的web.xml文件
```



※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

VScode 配置 Maven 安装环境 创建Maven项目

File -> Perferences -> Settings -> 搜索Maven -> 点击打开 "Edit in settings.json"文件

``` java
"remote.SSH.remotePlatform": {
        "10.239.85.244": "linux",
        "10.239.65.163": "linux"
    },

 * Windows 添加配置如下内容:
"java.home": "C:\\Program Files\\Java\\jdk1.8.0_241",
"java.configuration.maven.userSettings": "C:\\Users\\UserName\\Desktop\\software_package\\apache-maven-3.6.3\\conf\\settings.xml",
"maven.executable.path": "C:\\Users\\UserName\\Desktop\\software_package\\apache-maven-3.6.3\\bin\\mvn",

 * Linux 添加配置如下内容:
"java.home": "/usr/local/jdk1.8/",
"java.configuration.maven.userSettings": "/usr/maven/apache-maven-3.6.3/conf/settings.xml",
"maven.executable.path": "/usr/maven/apache-maven-3.6.3/bin/mvn",

"maven.terminal.customEnv": [
        {
            "environmentVariable": "JAVA_HOME",
            //"value": "C:\\Program Files\\Java\\jdk1.8.0_241"		// Windows
            "value": "/usr/local/jdk1.8"							// Linux
        }
    ]

"remote.SSH.configFile": "C:\\Users\\UserName\\.ssh\\config"
```


Ctrl + shift + P 选择 "Maven:Create Maven Project" 创建 Maven 项目
Select an archetype: 选择 maven-archetype-quickstart -> 选择版本 -> 选择生成目录
-> VScode 终端 输入Group Id: 如com.imooc -> 输入Artifact Id: 如microservice -> 输入package： 直接回车 -> 输入Version: 直接回车

1. 第一此创建Maven 父 项目pom.xml 没有 <packaging>pom</packaging> 需要手动添加, 右击 pom.xml -> Update project configuration
2. 创建子项目， 右击上面项目名， 选择 Create Maven Project ......'groupId': com.imooc -> 'artifactId': test -> 'version' 1.0-SNAPSHOT: :回车
3. 创建子项目后查看pom.xml可以发现自动添加了<parent></parent>标签

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※





※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※




※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※




