---
title: U盘安装OS_Ubuntu_18.04
tags: 
categories:
- linux
---

### 1. 安装启动盘(U 盘)
> $ fdisk -l
> $ umount /dev/sdb1
> $ mkfs.vfat /dev/sdb -I
> $ sudo dd if=~/Downloads/ubuntu-16.04-desktop-amd64.iso of=/dev/sdb status=progress

### 2. 开启root访问权限
> $ sudo passwd root
> 输入user用户密码，再设置root密码，下次就可以$ su root 获取root权限了

### Add new user
> $ adduser $USERNAME
> $ passwd $PASSWD

### 3. 设置网络
> * 点击屏幕右上角向下三角
>  + Wired Connected -> Wired Settings -> Network Proxy -> Manual -> HTTP Proxy等输入child-prc.intel.com, port输入913
>  + 到此可以试着打开浏览器看能不能上网
> * vi ~/.bashrc 添加如下内容

	export http_proxy=http://child-prc.intel.com:913
	export https_proxy=$http_proxy
	export HTTP_PROXY=$http_proxy
	export HTTPS_PROXY=$http_proxy

> $ source ~/.bashrc 使得上面添加的内容生效
> $ apt-get update
> $ apt-get upgrade
> $ apt install net-tools		// 到此可以用ifconfig查看ip
> $ apt-get install vim		// 可以用vim命令了
> $ vim ~/.vimrc	// 如果没有则新建.vimrc文件, 添加如下内容
```
set nu
set tabstop=4
```

### 4. 安装ssh并开启root远程登录
> $ apt-get install openssh-server
> $ vim /etc/ssh/sshd_config

	PermitRootLogin yes		// 去掉PermitRootLogin前面的"#"注释，后面值改为yes, 如果不设置则远程只能登录user用户如ai而不能直接登录root

> $ /etc/init.d/ssh start		// 启动SSH服务
> $ ps -e | grep sshd			// 查看SSH是否启动成功
> $ /etc/init.d/ssh stop		// 关闭SSH服务, 如果先开启服务再设置PermitRootLogin为yes则需要关掉ssh服务再启动ssh服务使其修改生效

### 5. 安装build-essential等必要的编译配置运行程序工具如gcc等
> $ apt install build-essential	// 该命令将安装一堆新包，包括gcc，g ++和make。
> $ gcc --version		// gcc -v

### 安装curl，会安装后运行可能出现的问题
> * 安装:
>    $ apt-get update
>    $ apt-get upgrade
>    $ apt-get install curl
>    $ curl --version
> * 提前设置好系统的proxy如:
>    export http_proxy=child-prc.intel.com:913
>    export https_proxy=child-prc.intel.com:913 // https的proxy与上面的http的要一样

> * 运行curl可能出现的问题:
>	curl: error while loading shared libraries: libcurl.so.4: cannot open shared object file: No such file or directo
>   $ find -name "libcurl.so.4" /
>   查看得到库文件在/usr/local/lib/libcurl.so.4
>   解决方法一: export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib	//当前终端生效
>   解决方法二: 把export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib添加到 ~/.bashrc, 然后运行source ~/.bashrc使其永久生效

### Ubuntu18.04 apt-get卸载软件步骤
> $ whereis curl
>   curl: /usr/bin/curl /usr/local/curl
> $ apt-get remove ***
> $ apt-get autoremove
> $ apt-get autoclean(或者clean)

### Ubuntu18.04 添加镜像源
> $ vim /etc/apt/sources.list.d/kubernetes.list
>   deb https://apt.kubernetes.io/ kubernetes-xenial main

### Modify hostname
> $ hostnamectl set-hostname $HOSTNAME
> $ reboot

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
一: 查询安装路径
1. dpkg -L 软件名
例如：dpkg -L gedit  
或者
2. whereis 软件名
例如：whereis gedit

二: 查询版本
1. aptitude show 软件名
2. dpkg -l 软件名
例如：dpkg -l gedit 

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

# cpupower
> $ apt install linux-tools-common

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
> $ yum install epel-release
> $ yum install cmake


※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※



※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※




※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※




※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※











