---
title: /etc/profile, /etc/bashrc, ~/.bashrc
tags: 
categories:
- linux
---
1、Ubuntu保存环境变量的几个文件
 * /etc/profile
在用户登录时，操作系统定制用户环境时使用的第一个文件，此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行。

 * /etc/environment
在用户登录时，操作系统使用的第二个文件， 系统在读取用户个人的profile前，设置环境文件的环境变量。

 * ~/.profile
在用户登录时，用到的第三个文件 是.profile文件，每个用户都可使用该文件输入专用于自己使用的shell信息，当用户登录时，该文件仅仅执行一次！默认情况下，会设置一些环境变量，执行用户的.bashrc文件。

 * /etc/bashrc
为每一个运行bash shell的用户执行此文件，当bash shell被打开时，该文件被读取。

 * ~/.bashrc
该文件包含专用于用户的bash shell的bash信息，当登录时以及每次打开新的shell时，该该文件被读取。
```
…
export http_proxy=http://child-prc.intel.com:913/
export https_proxy=http://child-prc.intel.com:913/
```
这样修改每次yum,pip等工具下载文件时候不能再写policy
使文件立刻生效，$ source ~/.profile

Note： 以上文件可通过$ sudo gedit 文件名 或 $ sudo vim 文件名打开；建议只修改~/.profile文件，如果只修改~/.bashrc文件，后期使用go get 命令时，会提示GOPATH未设置。
2、设置GOPATH和GOROOT
$ sudo gedit ~/.profile
在文件最后添加
 export GOROOT="/usr/lib/go-1.8" // 引号内设置为你自己的go安装目录
 export GOBIN=$GOROOT/bin
 export GOPATH="/home/test/gopath" // 引号内设置为自己的go项目的工作区间
 export PATH=$PATH:$GOPATH/bin    // 原路径后用冒号连接新路径

使文件立刻生效，$ source ~/.profile
重启系统即可
