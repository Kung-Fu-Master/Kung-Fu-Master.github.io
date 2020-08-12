---
title: yum使用和安装cmake 3 版本上
tags: 
categories: 
- linux
---
第一种安装cmake3
$yum search cmake
	……
	cmake3.x86_64 : Cross-platform make system
	……
	
$yum install cmake3.x86_64
	//但使用命令需用cmake3
	
$cmake3 --version      //查看cmake版本

$yum remove cmake3

第二种:
（1）移除旧版本：
yum remove cmake
（2）下载新版本
1、下载：wget https://cmake.org/files/v3.6/cmake-3.6.0-Linux-x86_64.tar.gz
2、解压：tar -zxvf cmake-3.6.0-Linux-x86_64.tar.gz
注意：这个压缩包不是源码包，解压后直接用。
3、增加环境变量，使其成为全局变量：
vim /etc/profile  // 或者vim ~/.bashrc
在文件末尾处增加以下代码
export PATH=$PATH:/lnmp/src/cmake-3.6.0-Linux-x86_64/bin
注意：写自己刚安装cmake的bin的路径
使修改的文件生效
source /etc/profile //或者 source ~/.bashrc
4、查看环境变量：
echo $PATH
5、检查cmake版本：
cmake --version

 扩展知识：
百度百科的介绍：
CMake是一个跨平台的安装（编译）工具，可以用简单的语句来描述所有平台的安装(编译过程)。
他能够输出各种各样的makefile或者project文件，能测试编译器所支持的C++特性,类似UNIX下的automake。
只是 CMake 的组态档取名为 CMakeLists.txt。Cmake 并不直接建构出最终的软件，而是产生标准的建构档
（如 Unix 的 Makefile 或 Windows Visual C++ 的 projects/workspaces），然后再依一般的建构方式使用。
这使得熟悉某个集成开发环境（IDE）的开发者可以用标准的方式建构他的软件，这种可以使用各平台的原生
建构系统的能力是 CMake 和 SCons 等其他类似系统的区别之处。
