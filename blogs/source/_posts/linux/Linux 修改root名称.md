---
title: Linux 修改root名称
tags:
categories:
- linux
---
vi /etc/passwd
按i键进入编辑状态
修改第1行第1个root为新的用户名
按esc键退出编辑状态，并输入:x保存并退出
vi /etc/shadow                                                 //root用户对应的密码文件
按i键进入编辑状态
修改第1行第1个root为新的用户名
按esc键退出编辑状态，并输入:x!强制保存并退出
为了正常使用sudo，需要修改/etc/sudoers的设置，修改方法如下（来自How to add users to /etc/sudoers）：
	运行vi sudo
	找到root    ALL=(ALL)       ALL
	在下面添加一行：新用户名    ALL=(ALL)       ALL
	:x保存退出
