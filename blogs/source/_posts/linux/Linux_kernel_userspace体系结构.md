---
title: Linux kernel userspace体系结构
tags:
categories:
- linux
---
linux 系统体系结构：
![](linux_gun.jpg)

linux kernel体系结构：
arm有7种工作模式，x86也实现了4个不同级别RING0-RING3,RING0级别最高，
这样linux用户代码运行在RING3下，内核运行在RING0,这样系统本身就得到了
充分的保护
用户空间(用户模式)转到内核空间(系统模式)方法：
·系统调用
·硬件中断
linux kernel 体系结构：
![](linux_kernel.jpg)

虚拟文件系统VFS:
VFS(虚拟文件系统)隐藏各种文件系统的具体细节，为文件操作提供统一的接口

二.Linux内核源代码
linux内核下载www.kernel.org
目录结构:
解压linux kernel tar后目录
·arch:根据cpu体系结构不同而分的代码
·block:部分块设备驱动程序
·crypto:加密，压缩，CRC校验算法
·documentation:内核文档
·drivers:设备驱动程序
·fs(虚拟文件系统vfs):文件系统
·include:内核所需的头文件，(与平台无关的头文件在include/linux中)
·lib:库文件代码(与平台相关的)
·mm:实现内存管理，与硬件体系结构无关的(与硬件体系结构相关的在arch中)
·net:网络协议的代码
·samples:一些内核编程的范例
·scripts:配置内核的脚本
·security:SElinux的模块
·sound:音频设备的驱动程序
·usr:cpio命令实现，用于制作根文件系统的命令(文件系统与内核放到一块的命令)
·virt:内核虚拟机
linux DOC 编译生成:
linux源根目录/Documentation/00-INDEX:目录索引
linux源根目录/Documentation/HOWTO:指南
·生成linux内核帮助文档:在linux源根目录(Documentation) 执行make htmldocs
ubuntu16下需要执行sudo apt-get install xmlto安装插件才可生成doc文档

三.Linux内核配置与编译
清理文件(在linux源码根目录):
·make clean:只清理所有产生的文件
·make mrproper:清理所有产生的文件与config配置文件
·make distclean:清理所有产生的文件与config配置文件，并且编辑过的与补丁文件
↓
配置(收集硬件信息如cpu型号，网卡等...):
·make config:基于文本模式的交互配置
·make menuconfig:基于文本模式的菜单模式(推荐使用)
·make oldconfig:使用已有的.config,但会询问新增的配置项
·make xconfig:图形化的配置(需要安装图形化系统)
配置方法：
1)使用make menuconfig操作方法：
1>按y:编译>连接>镜像文件
2>按m:编译
3>按n:什么都不做
4>按"空格键":y,n轮换
配置完并保存后会在linux源码根目录下生成一个.config文件
注意：在ubuntu11上要执行apt-get install libncurses5-dev来安装支持包

四.linux内核模块开发
描述：
linux内核组件非常庞大，内核ximage并不包含某组件，而是在该组件需要被使用的时候，动态的添加到正在运行的内核中(也可以卸载)，这种机制叫做“内核模块”的机制。内核模块通常通过使用makefile文件对模块进行编译
模块安装与卸载:
1)加载：insmod hello.ko
2)卸载：rmmod hello
3)查看：lsmod
4)加载(自动寻找模块依赖)：modprobe hello
modprobe会根据文件/lib/modules/version/modules.dep来查看要加载的模块，看它是否还依赖于其他模块，如果是,会先找到这些模块，把它们先加载到内核
