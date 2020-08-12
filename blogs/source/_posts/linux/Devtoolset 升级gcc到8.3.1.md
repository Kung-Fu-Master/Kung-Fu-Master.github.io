---
title: Devtoolset 升级gcc到8.3.1
tags: 
categories:
- linux
---
设置gcc到8.0再用 -mavx512f 参数
To install the full tools-set including gfortran on centos 7:

 $ yum install centos-release-scl
 $ yum install devtoolset-8                       //yum install devtoolset-7, 升级到gcc 7.3
 $ scl enable devtoolset-8 -- bash           //scl enable devtoolset-7 -- bash, 只在当前终端生效
enable the tools:

 $ source /opt/rh/devtoolset-8/enable 
you may wish to put the command above in .bash_profile

ref: https://unix.stackexchange.com/questions/477360/centos-7-gcc-8-installation
