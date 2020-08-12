---
title: yum下载遇到XZ_5.1.2alpha问题
tags: 
categories:
- linux
---
遇到如下问题：
[root@localhost download]# yum -y install libopencv-dev
There was a problem importing one of the Python modules
required to run yum. The error leading to this problem was:

   /lib64/liblzma.so.5: version `XZ_5.1.2alpha' not found (required by /lib64/librpmio.so.3)

Please install a package which provides this module, or
verify that the module is installed correctly.

It's possible that the above module doesn't match the
current version of Python, which is:
2.7.5 (default, Jun 20 2019, 20:27:34)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]

If you cannot solve this problem yourself, please go to
the yum faq at:
  http://yum.baseurl.org/wiki/Faq
[root@localhost download]#
…

解决方法：

 /lib64/liblzma.so.5: version `XZ_5.1.2alpha' not found (required by /lib64/librpmio.so.3)
	1. 下载安装 xz-5.2.2.tar.gz
		进入xz工具官网下载源码包：http://tukaani.org/xz/
		      下载版本：xz-5.2.2.tar.gz
	a. 下载之后，将压缩包解压 tar -vxf xz-5.2.2.tar.gz
	
	      b. 进入到xz源码目录 cd  xz-5.2.2.tar.gz
	
	      c. 配置 ./configure--enable-shared
	
	      d. 编译 make
	
	      e. 安装 makeinstall
	
	   如此，则系统安装了xz工具。
	   当然，如果用户自己希望安装到自己的特定路径下，可以在配置选项中，设定安装路径，如
	     ./configure --enable-shared --prefix=/opt/install/xz/bin
	
	     这样xz工具就被安装在/tmp/xz目录中，如果要导入到系统，则需要设置环境变量，编辑系统配置文件,
	
	     vi  /etc/bash.bashrc
	
	     在系统配置文件的末尾，加入路径：
	
	     export PATH=$PATH:/opt/install/xz/bin
	
	     export PATH
	
	     如果修改了环境变量，需要 
	
	      4.3 验证是否xz安装成功
	
	      在终端中，输入命令查看版本号：   xz -V
	      得到信息如下，则说明安装成功。
	      xz (XZ Utils) 5.2.2
	      liblzma 5.2.2
	
	2. 到/lib64/目录
	cp /lib64/liblzma.so.5.2.2 .
	sudo ln -s -f liblzma.so.5.2.2 liblzma.so.5
	就可以了
	
	
