---
title: alias
tags: 
categories: 
- linux
---


## alias

```shell
	$ alias
	alias cp='cp -i'	// -i 表示交互, 覆盖文件时候会提示是否确定要overrite.
	......
	$ alias cdnet="cd /etc/sysconfig/network-scripts"
	$ cdnet

	//cp命令时候不想提示是否覆盖文件
	// 方法一:
	$ unalias cp
	// 方法二: cp前加上"\"表示不用alias的别名 'cp -i', 而 直接用 'cp'
	$ \cp -fv dir1/file1  dir2/file2	//-v表示输出复制命令的过程细节
```


