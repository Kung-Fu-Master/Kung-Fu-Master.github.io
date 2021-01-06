---
title: VScode代码自动格式化
tags:
categories:
- blogs
- blogs
---

## 代码保存自动格式化设置

一、实现vs code中代码格式化快捷键：【Shift】+【Alt】+F

二、实现保存时自动代码格式化：

1）文件 ------.>【首选项】---------->【设置】；

2）搜索emmet.include;

3）在settings.json下的【工作区设置】中添加以下语句：

"editor.formatOnType": true,
"editor.formatOnSave": true

4）随便写代码进行测试即可。


## python代码保存自动格式化
VScode创建python文件会自动提示安装autopep8, 或者通过以下命令进行安装

	pip install -U autopep8
