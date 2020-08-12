---
title: dmidecode
tages:
categories:
- linux 
---
查看服务器型号：dmidecode | grep "Product Name"
查看主板的序列号：dmidecode | grep "Serial Number"
查看系统序列号：dmidecode -s system-serial-number
查看内存信息：dmidecode -t memory
查看OEM信息：dmidecode -t 11
不带选项执行dmidecode命令通常会输出所有的硬件信息。dmidecode命令有个很有用的选项-t，可以按指定类型输出相关信息，假如要获得处理器方面的信息，则可以执行：
[root@localhost ~]# dmidecode -t processor
