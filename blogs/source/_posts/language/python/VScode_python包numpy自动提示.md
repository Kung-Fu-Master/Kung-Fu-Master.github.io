---
title: VScode_python包numpy等自动提示
tags:
categories:
- language
- python
---

## VScode添加python模块路径

1. 查看Linux pip install 命令安装包的路径如`/usr/local/lib64/python3.6/site-packages/`.  
如果不清楚pip安装包的路径, 可以执行两次`pip install numpy`, 第二次执行就可以看到默认pip安装包的目录.  

2. 确定python安装目录如`/usr/lib64/python3.6/`

3. 设置VScode
文件 – 设置 – 首选项，搜索autoComplete，点击"在settings.json中编辑"，添加模块路径


	"python.autoComplete.extraPaths": [
	    "/usr/local/lib64/python3.6/site-packages/",
	    "/usr/lib64/python3.6/",
	]
修改之后**重启VScode**, 就可以了

## python代码保存自动格式化
VScode创建python文件会自动提示安装autopep8, 或者通过以下命令进行安装

	pip install -U autopep8



