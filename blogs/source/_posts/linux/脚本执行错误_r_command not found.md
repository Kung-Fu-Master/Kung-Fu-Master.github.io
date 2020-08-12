---
title: 脚本执行错误 $'\r':command not found
tags: 
categories:
- linux
---
shell脚本执行错误 $'\r':command not found
第一种解决方案：
Linux下有命令dos2unix
你只要输入dos2unix *.sh就可以完成转换工作了
如果命令不存在的话就用如下命令安装
yum install dos2unix -y
 
第二种解决方案：
这种情况发生的原因是因为你所处理的文件换行符是dos格式的"\r\n"
可以使用cat -v 文件名 来查看换行符是否是，如果是上述的，则行结尾会是^m
需要转换成linux/unix格式的"\n
sed 's/\r//' 原文件 >转换后文件
第三种解决方案：
首先要确保文件有可执行权限
#sh>chmod a+x filename
利用如下命令查看文件格式
:set ff 或 :set fileformat
可以看到如下信息
fileformat=dos 或 fileformat=unix
利用如下命令修改文件格式
:set ff=unix 或 :set fileformat=unix
