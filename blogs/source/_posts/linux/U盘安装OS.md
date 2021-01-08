---
title: U盘安装OS
tags: 
categories:
- linux
---

## **安装启动盘(U 盘)**

```shell
fdisk -l
umount /dev/sdb1
mkfs.vfat /dev/sdb -I
sudo dd if=~/Downloads/ubuntu-16.04-desktop-amd64.iso of=/dev/sdb status=progress
```

## **安装Ubuntu_18.04**
### 1. 开启root访问权限
```shell
sudo passwd root
```
输入user用户密码，再设置root密码，下次就可以$ su root 获取root权限了

### Add new user
```shell
adduser $USERNAME
passwd $PASSWD
```

### 2. 设置网络
* 点击屏幕右上角向下三角
 + Wired Connected -> Wired Settings -> Network Proxy -> Manual -> HTTP Proxy等输入child-prc.intel.com, port输入913
 + 到此可以试着打开浏览器看能不能上网
```shell
$ vi ~/.bashrc 添加如下内容
export http_proxy=http://child-prc.intel.com:913
export https_proxy=$http_proxy
export HTTP_PROXY=$http_proxy
export HTTPS_PROXY=$http_proxy
```

```shell
source ~/.bashrc 使得上面添加的内容生效
apt-get update
apt-get upgrade
apt install net-tools		// 到此可以用ifconfig查看ip
apt-get install vim		// 可以用vim命令了
vim ~/.vimrc	// 如果没有则新建.vimrc文件, 添加如下内容
set nu
set tabstop=4
```

### 3. 安装ssh并开启root远程登录

```shell
$ apt-get install openssh-server
$ vim /etc/ssh/sshd_config
  PermitRootLogin yes		// 去掉PermitRootLogin前面的"#"注释，后面值改为yes, 如果不设置则远程只能登录user用户如ai而不能直接登录root
$ /etc/init.d/ssh start		// 启动SSH服务
$ ps -e | grep sshd			// 查看SSH是否启动成功
$ /etc/init.d/ssh stop		// 关闭SSH服务, 如果先开启服务再设置PermitRootLogin为yes则需要关掉ssh服务再启动ssh服务使其修改生效
```

### 4. 安装build-essential等必要的编译配置运行程序工具如gcc等
```shell
apt install build-essential	// 该命令将安装一堆新包，包括gcc，g ++和make.
gcc --version		// gcc -v
```

### 安装curl，会安装后运行可能出现的问题
 * 安装:
```shell
apt-get update
apt-get upgrade
apt-get install curl
curl --version
```

 * 提前设置好系统的proxy如:
```shell
export http_proxy=child-prc.intel.com:913
export https_proxy=child-prc.intel.com:913 // https的proxy与上面的http的要一样
```

 * 运行curl可能出现的问题:
  curl: error while loading shared libraries: libcurl.so.4: cannot open shared object file: No such file or directo
```shell
$ find -name "libcurl.so.4" /
```
查看得到库文件在/usr/local/lib/libcurl.so.4
解决方法一: export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib	//当前终端生效
解决方法二: 把export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib添加到 ~/.bashrc, 然后运行source ~/.bashrc使其永久生效

### Ubuntu18.04 apt-get卸载软件步骤
```shell
$ whereis curl
  curl: /usr/bin/curl /usr/local/curl
$ apt-get remove ***
$ apt-get autoremove
$ apt-get autoclean(或者clean)
```

### Ubuntu18.04 添加镜像源
```shell
$ vim /etc/apt/sources.list.d/kubernetes.list
  deb https://apt.kubernetes.io/ kubernetes-xenial main
```

### Modify hostname
```shell
$ hostnamectl set-hostname $HOSTNAME
$ reboot
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
一: 查询安装路径
1. dpkg -L 软件名
```shell
dpkg -L gedit
```

2. whereis 软件名
```shell
whereis gedit
```

二: 查询版本
```shell
aptitude show 软件名
dpkg -l 软件名
```
※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

### cpupower
```shell
apt install linux-tools-common
```

```shell
yum install epel-release
yum install cmake
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

## 安装Centos Minimal

### 配置网络启动

```shell
$ vi /etc/sysconfig/network-scripts/ifcfg-ens33
ONBOOT=yes			// ONBOOT=no，改为ONBOOT=yes

//重启网络
$ ifup <网卡名字如：ens33>		// ifup 网卡名字
$ service network restart	// 重启网络
```

### 配置yum代理

```shell
$ echo proxy=http://Proxy:port >> /etc/yum.conf
$ yum install vim
```

### 配置系统proxy

```shell
$ vim ~/.bashrc
export http_proxy=http://Proxy:port
export https_proxy=http://Proxy:port
```

### 安装网络工具

```shell
$ yum install net-tools
```
### 开启远程登陆

```shell
$ vim /etc/ssh/sshd_config
Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

启动sshd
$ /bin/systemctl start sshd.service
```

## **遇到的问题**
### **缺少网卡驱动**
在裸组装机而不是虚拟机上装Centos7之后$ ifconfig发现只有 lo环回网络地址，缺少网卡如"enps3"等
解决方法是在网上查找网卡对应的驱动， 然后下载安装(本次没有找到对应网卡版本的驱动但安装后仍然成功查到网址并能正常使用)

```shell
$ ip addr			//发现只有lo网路地址, 或者还有其它的如virbr0等的虚拟网络地址, 但都不是想要的网络IP
$ lsmod | grep e1000
  e1000e                263116  0
  ptp                    19231  1 e1000e
$ modprobe e1000e
$ lspci -vvv | less
$ rmmod e1000e		//移除原来的网卡驱动
$ rmmod e1000
$ lsmod | grep e1000


$ lspci				//查看设备，其中有Intel网卡, I219-V为网卡设备号
  00:1f.6 Ethernet controller: Intel Corporation Ethernet Connection (12) I219-V
根据网卡设备号I219-V在网上查找相应驱动, 并下载解压 e1000e-3.8.4.tar.gz
$ tar -zxvf e1000e-3.8.4.tar.gz
$ cd e1000e-3.8.4
$ cd src/
$ make
$ ls				//会编译生成内核文件e1000e.ko
  e1000e.ko
$ insmod e1000e.ko	//将内核文件加载进内核
$ dmesg
$ lsmod | grep e1000e
$ ifconfig			//再次查看发现有网络IP了
$ make install		//设置开机后会自动加载内核网络文件
$ dmidecode -vvv | less		//查看下网卡信息
```
机器运行期间执行过$ yun update, 选择yes, 可能会涉及到内核的升级或改变, 重新开机后会默认选择新添加的内核启动, 登陆后发现没有又没有网卡驱动.
**第一种解决方法**
可以在登陆时选择原来的内核启动就可以了.
**第二种解决方法**
因为换不同内核, 需要重新安装这个第三方网卡驱动, 重新进入e1000e-3.8.4/src目录$ make clean, $ make, $ make install  
查看有没有旧的内核驱动, 如果又需要先卸载$ lsmod | grep e1000e, $ rmmod e1000e.ko  
然后执行$ insmod e1000e.ko, 如果报insmod: ERROR: could not insert module xxxxx.ko: Unknown symbol in module错误可以先忽略直接重启系统选择新的内核启动问题就解决了.  



