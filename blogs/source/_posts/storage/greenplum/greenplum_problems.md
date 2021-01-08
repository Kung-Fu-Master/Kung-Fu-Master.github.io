---
title: greenplum 遇到的问题
tags: storage
categories:
- storage
- greenplum
---


## **问题1: greenplum pod异常重启后 用psql 登陆出现如下问题**
```
	psql: could not connect to server: No such file or directory
	        Is the server running locally and accepting
	        connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```
解决方案: 运行**`gpstart`**命令, 重启服务, 中途会让选择默认配置, 选择y回车即可.



