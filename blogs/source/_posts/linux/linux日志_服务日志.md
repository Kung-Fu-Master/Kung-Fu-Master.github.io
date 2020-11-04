---
title: 查看 linux 系统日志 和 服务的日志
tags: 
categories: 
- linux
---

## 查看linux 系统日志

	$ cat /var/log/messages

## 查看某个服务日志

	$ journalctl -fu docker
	$ journalctl -fu kubelet
查看服务日志详细内容:

	journalctl -u kubelet --no-pager

## 重新加载系统配置

	$ systemctl daemon-reload

