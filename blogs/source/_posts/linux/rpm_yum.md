---
title: RPM, yum
tags:
categories:
- linux
---

## RPM
> RPM是”Redhat Package Manager”的缩写，根据名字也能猜到这是Redhat公司开发出来的。RPM 是以一种数据库记录的方式来将你所需要的套件安装到你的Linux 主机的一套管理程序。也就是说，你的linux系统中存在着一个关于RPM的数据库，它记录了安装的包以及包与包之间依赖相关性。RPM包是预先在linux机器上编译好并打包好的文件，安装起来非常快捷。但是也有一些缺点，比如安装的环境必须与编译时的环境一致或者相当；包与包之间存在着相互依赖的情况；卸载包时需要先把依赖的包卸载掉，如果依赖的包是系统所必须的，那就不能卸载这个包，否则会造成系统崩溃。
> 每一个rpm包的名称都由”-“和”.”分成了若干部分。就拿 a2ps-4.13b-57.2.el5.i386.rpm 这个包来解释一下，a2ps 为包名；4.13b则为版本信息；57.2.el5为发布版本号；i386为运行平台。其中运行平台常见的有i386, i586, i686, x86_64 ，需要你注意的是cpu目前是分32位和64位的，i386,i586和i686都为32位平台，x86_64则代表为64位的平台。另外有些rpm包并没有写具体的平台而是noarch，这代表这个rpm包没有硬件平台限制。例如 alacarte-0.10.0-1.fc6.noarch.rpm.

### 安装一个rpm包

```shell
$ rpm -ivh **.rpm
```
-i ：安装的意思
-v ：可视化
-h ：显示安装进度
--force 强制安装，即使覆盖属于其他包的文件也要安装
--nodeps 当要安装的rpm包依赖其他包时，即使其他包没有安装，也要安装这个包

### 升级一个rpm包

```shell
$ rpm -Uvh filename -U ：即升级的意思
```

### 卸载一个rpm包

```shell
$ rpm -e filename 这里的filename是通过rpm的查询功能所查询到的
```

### 查询一个包是否安装

```shell
$ rpm -q rpm包名（这里的包名，是不带有平台信息以及后缀名的）
```

### 查询当前系统中所安装的所有rpm包, 列出前十个

```shell
$ rpm -qa | head -n 10
```

### rpm包的相关信息

```shell
$ rpm -qi 包名 （同样不需要加平台信息与后缀名）
```

### 列出rpm包安装的文件

```shell
$ rpm -ql 包名
```
通过上面的命令可以看出vim是通过安装vim-enhanced-7.0.109-6.el5这个rpm包得来的。那么反过来如何通过一个文件去查找是由安装哪个rpm包得来的

### 列出某一个文件属于哪个rpm包

```shell
$ rpm -qf 文件的绝对路径
$ rpm -qf `which tree`
$ rpm -qf `which screen`
```
### rpm包安装的时候要手动配置环境变量


## Yum
> yum是Redhat所特有的安装RPM程序包的工具，使用起来相当方便。因为使用RPM安装某一个程序包有可能会因为该程序包依赖另一个程序包而无法安装。而使用yum工具就可以连同依赖的程序包一起安装。当然CentOS同样可以使用yum工具，而且在CentOS中你可以免费使用yum，但Redhat中只有当你付费后才能使用yum，默认是无法使用yum的

### 设置proxy

```xml
	# vim /etc/yum.conf
	[main]
	cachedir=/var/cache/yum/$basearch/$releasever
	keepcache=0
	debuglevel=2
	logfile=/var/log/yum.log
	exactarch=1
	obsoletes=1
	gpgcheck=1
	plugins=1
	installonly_limit=5
	bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
	distroverpkg=centos-release
	proxy=http://child-prc.intel.com:913
```

### k8s repo

```shell
	$ touch /etc/yum.repos.d/kubernetes.repo
	[kubernetes]
	name=Kubernetes
	baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
	enabled=1
	gpgcheck=1
	repo_gpgcheck=1
	gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```

### 安装rpm包

 * 列出所有可用的rpm包

```shell
$ yum list
```

 * 搜索一个rpm包

```shell
$ yum search [相关关键词]
```

 * 安装一个rpm包

```shell
$ yum install [-y] [rpm包名]
```
 * 卸载一个rpm包

```shell
$ yum remove [-y] [rpm包名]
```
 * 升级一个rpm包

```shell
$ yum update [-y] [rpm包]
```





