---
title: maven project
tags: 
categories:
- technologies
- maven
---

## Ubuntu18.04 安装 IntelliJ idea
	https://blog.csdn.net/weixx3/article/details/81136822

## 遇到的问题

#### from thrift.Thrift import TType, TMessageType, TFrozenDict, TException, TApplicationException ImportError: cannot import name TFrozenDict
	解决方法是: 某些包没有关联上，装包时加上[hive]后缀
	$ pip install pyhive[hive]

## 手动下载jar包并通过mvn安装到Linux或windows环境，供项目引用
	https://repo1.maven.org/maven2/ 或者https://mvnrepository.com/
 * 如在pom.xml添加依赖但是下载不下来
 ```
	<dependency>
      <groupId>org.apache.thrift</groupId>
      <artifactId>libthrift</artifactId>
      <version>0.10.0</version>
    </dependency>
 ```
 * 浏览器连接https://repo1.maven.org/maven2/org/apache/thrift/libthrift/ 查看指定版本如0.10.0, 点击进去下载libthrift-0.10.0.jar
 * 执行如下命令
	```
	$ cd /home/ai/IdeaProjects/microservice/user-thrift-service
	$ mvn install:install-file -Dfile=/usr/maven/maven_repository/org/apache/thrift/libthrift/libthrift-0.10.0.jar -DgroupId=org.apache.thrift -DartifactId=libthrift -Dversion=0.10.0 -Dpackaging=jar
	```
	Maven 安装 JAR 包的命令是：mvn install:install-file -Dfile=本地jar包的位置  -DgroupId=上面的groupId  -DartifactId=上面的artifactId  -Dversion=上面的version  -Dpackaging=jar
 * 到maven设置的repository里查看
```
 $ cd /usr/maven/maven_repository/org/apache/thrift/libthrift#
 $ ls -alh
  drwxr-xr-x 2 root root 4.0K 4月   9 10:41 0.10.0
  -rw-r--r-- 1 root root  308 4月   9 10:41 maven-metadata-local.xml
```

### Linux 命令行方式: 搭建，编译，测试，运行calss文件，打包，运行jar包，安装，部署， 清理 命令.
 * 1. mvn archetype:generate  -DinteractiveMode=false -DgroupId=com.HCI -DartifactId=HCI -Dpackage=com.HCI
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.HCI</groupId>
  <artifactId>HCI</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>HCI</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```
 * 2. mvn compile：编译源代码
 ![](compile.JPG)
 * 3. mvn test: 测试编译过的代码
 * 4. mvn exec:java -Dexec.mainClass="com.HCI.App"
 * 5. mvn package：生成构件包（一般为 jar 包或 war 包）
 ![](jar.JPG)
 * 6. java -cp target/HCI-1.0-SNAPSHOT.jar com.HCI.App
 * 7. mvn install：将构件包安装到本地仓库
 * 8. mvn deploy：将构件包部署到远程仓库
 * 9. mvn clean：清空输出目录（即 target 目录）

创建Maven项目如HCI后，执行 Maven 其它命令需要注意的是：必须在 Maven 项目的根目录处执行，也就是当前目录下一定存在一个名为 pom.xml 的文件
如进入/home/ai/IdeaProjects/microservice/user-thrift-service目录再执行mvn install:......命令

```


