---
title: python 安装
categories:
- linux
---

## **Centos7 python3.6 安装**

```shell
	$ yum install epel-release
	$ yum search python | grep python36
	$ yum install python36
	$ whereis python
	$ rm /usr/bin/python // 删除原来系统自带的python2.7版本
	$ ln -s /usr/bin/python3.6 /usr/bin/python
	$ ls -alh /usr/bin/python
	lrwxrwxrwx 1 root root 18 Sep 11 11:19 /usr/bin/python -> /usr/bin/python3.6
```

有时候我们保留系统默认的的python 2.7版本`/usr/bin/python -> /usr/bin/python2.7`.  
可以直接用`$ python3`来运行python3版本, 原因是安装python36时候会自动生成`/usr/bin/python3`链接向`/usr/bin/python3.6`.  
因此其实也可以直接用`$ python3.6`来执行python3版本.  

```shell
	$ ls -alh /usr/bin/python3
	lrwxrwxrwx 1 root root 9 Sep 11 11:10 /usr/bin/python3 -> python3.6
```

