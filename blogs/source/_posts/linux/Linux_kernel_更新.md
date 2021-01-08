---
title: Linux kernel 更新
tags: 
categories:
- linux
---

## Linux kernel 更新

### 第一种: 源码更新：
目前系统内核目录
/usr/src/kernels/3.10.0-693.el7.x86_64/.config
/usr/src/kernels/3.10.0-957.21.3.el7.x86_64/.config
上面文件夹内基本都只有头文件，缺少kernel的源码文件
make -j12
先make modules_install
然后make install
就可以重启了
这种kernel源码一般只有头文件的。。。我没用过yum安装的kernel，都是自己去kernel.org上下载源码自己编译的
就算下载的kernel比目前版本新也可以，只需要make oldconfig
就会把老的config都应用到新的内核里


1.下载内核
http://www.kernel.org下载内核代码
 * 下载

```shell
sudo wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.17.3.tar.xz
```
 * 解压 下载的文件格式是.tar.xz，需要先解压

```shell
sudo xz -d linux-4.17.3.tar.xz
```
 * 这样只解压了一层，发现解压后是tar格式，需要用tar命令再解压
```shell
sudo tar -xf linux-4.17.3.tar
```
 * 这样便可以进入到目录linux-4.17.3了

2.部署内核源码
把内核解压到/usr/src目录下 cd /usr/src tar -xvf ~/Downloads/linux-4.14.1.tar.xz #解压源码

3.在内核代码目录下创建.config

```shell
cd linux-4.14.1 
cp /boot/config-`uname -r` .config #这里`uname -r`可以求出当前的内核版本 sudo make oldconfig
```

4. 编译内核

```shell
sudo make
sudo make modules_install
sudo make install
```

编译时可能出现缺少openssl，apt install即可，make的时间比较长，中途如果出错再次编译前最好先sudo make clean
5. 测试

```shell
sudo reboot #重启
uname -r # 查看内核版本
```

第一次重启可能比较慢，耐心等待即可。

### 第二种: rpm包更新:
CentOS7 更新最新内核
内核下载地址：https://elrepo.org/linux/kernel/el7/x86_64/RPMS/
内核选择
kernel-lt（lt=long-term）长期有效
kernel-ml（ml=mainline）主流版本
升级kernel
安装过程
1.下载内核

```shell
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-ml-5.4.3-1.el7.elrepo.x86_64.rpm
// 如果网站不需要证件，或文件过大需后台下载(-b)命令如下:
wget -c -b https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-ml-5.4.3-1.el7.elrepo.x86_64.rpm --no-check-certificate
```

2.安装内核

```shell
rpm -ivh kernel-ml-5.4.3-1.el7.elrepo.x86_64.rpm
```

3.查看当前默认内核

```shell
grub2-editenv list
saved_entry=CentOS Linux (3.10.0-327.28.3.el7.x86_64) 7 (Core)
```

4.查看所有内核启动 grub2

```shell
awk -F \' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg 
0 : CentOS Linux (5.4.3-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-327.28.3.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-327.22.2.el7.x86_64) 7 (Core)
3 : CentOS Linux (3.10.0-327.13.1.el7.x86_64) 7 (Core)
4 : CentOS Linux, with Linux 0-rescue-cd8c4444947b4b0b818457f51ded6591
```

5.修改为最新的内核启动
```shell
grub2-set-default 'CentOS Linux (5.4.3-1.el7.elrepo.x86_64) 7 (Core)'
```

6.再次查看内核
```shell
grub2-editenv list
saved_entry=CentOS Linux (5.4.3-1.el7.elrepo.x86_64) 7 (Core)
```

7.重新启动

```shell
reboot
```

8.更新kernel-ml-headers

```shell
wget http://ftp.osuosl.org/pub/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-headers-5.4.3-1.el7.elrepo.x86_64.rpm
```
如果遇到headers版本冲突如下
先卸载当前的kernel-headers, 会卸载一些依赖如gcc g++等，可以在卸载后安装新版kernel-headers后再安装回来
yum remove kernel-headers
再安装此此5.4 kernel-headers
rpm -ivh kernel-ml-headers-5.4.3-1.el7.elrepo.x86_64.rpm

9.更新kernel-ml-devel
http://ftp.osuosl.org/pub/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-devel-5.4.3-1.el7.elrepo.x86_64.rpm
```shell
rpm -ivh kernel-ml-devel-5.4.3-1.el7.elrepo.x86_64.rpm
```

10. 安装gcc等
```shell
yum install gcc
```



