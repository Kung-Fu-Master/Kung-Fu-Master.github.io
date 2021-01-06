---
title: yum 遇到的问题
tags:
categories:
- linux
---

## 安装python3后用yum安装软件出错

`yum search` 出现错误1：except KeyboardInterrupt, e:
`yum install` 出现错误2： except OSError, e:

### 第一种：升级yum

### 第二种: 修改文件里的python为python2
CentOS系统原带有Python2，后自行安装Python3，并改变/usr/bin/python连接到python3，在执行python的时候直接调用python3.5版本。  
原因是 `yum does not support Python3`, 该改变导致了yum运行时会报错。  
解决方法修改/usr/bin下的yum文件的第一行：  

	$ vim /usr/bin/yum
	  1 #!/usr/bin/python -->> #!/usr/bin/python2.7

	$ vim /usr/libexec/urlgrabber-ext-down
	  1 #! /usr/bin/python2  -->>  #!/usr/bin/python2.7
