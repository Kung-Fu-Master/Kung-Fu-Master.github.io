---
title: VirutalBos安装Centos7
tag: docker
categories:
- linux
---

Reference Link: 
(外网): https://www.avoiderrors.com/install-centos-7-virtual-box/
(csdn): https://blog.csdn.net/qq_23033339/article/details/80867195
(csdn): https://www.cnblogs.com/hihtml5/p/8217062.html

## 1. [Download and install VirtualBox](https://www.virtualbox.org/wiki/Downloads) from its official website, and make sure that you had downloaded the latest version.

## 2. Also [download the official CentOS ISO](https://www.centos.org/download/) from the official website, the latest CentOS build is 7.

## 3. Run your VirtualBox after you had installed it on your computer and located its icon on the desktop and click on “New“.

![](01.png)

## 4. Give your new OS name and set your RAM memory, and also select the version to be “Red Hat (64-bit).

![](02.png)

## 5. On the Hard Disk step, select “Create a virtual hard drive now” and then click Create.

![](03.png)

## 6. Select VDI “VirtualBox Disk Image” and click Next, and then select “Dynamically allocated” and click Next then Create.

![](04.png)
默认选项即可，默认选择的是VirtualBox虚拟机软件专用的磁盘映像格式，其他虚拟机软件可能无法读取。

点击下一步，进行设置如何分配虚拟硬盘

![](05.png)
默认选项即可，两者有何不同界面上已经有很详细的说明了。

点击下一步，指定虚拟硬盘文件的存放位置和虚拟硬盘的大小。

![](06.png)

## 7. (添加ISO和网络)From the Setting click on Storage, and then add the ISO file to the optical drive to install the operating system.

![](07.jpg)

VirtualBox的四种网络连接方式

![](09.png)
virtualbox默认的网络连接方式如下

![](10.png)
可以看到桥接模式是最佳选项，它支持所有情况的访问, 修改虚拟机连接方式为桥接网卡, 如下所示：

![](11.png)

按照下图配置虚拟机的网络


**Linux系统上通过执行`ip addr`查看并选择系统网卡**

## 8. You had successfully configured your CentOS well, power on your virtual machine by clicking on Start.

![](08.png)

## 9. From the boot menu select “Install CentOS Linux 7” and press Enter.

![](12.png)

## 10. Select your language and press on Continue.

![](13.png)

## 11. Setup your time settings, location, network, and then click “Begin Installation”.

![](14.png)

## 12. During the installation, you set the root and the user account.

![](15.png)

## 13. After the installation is completed, press on Reboot.

![](16.png)




