---
title: ntpdate 同步各个系统时间
tags:
categories:
- linux
---

## ntpdate
Centos下载ntpdate

```shell
	$ yum install ntp ntpdate -y
```

集群操作，同步各个系统时间, 与某一台服务器时间保持一致.

```
0 12 * * * */usr/sbin/ntpdate 192.168.0.1
```
每天12点强制将系统时间设置为192.168.0.1服务器时间





