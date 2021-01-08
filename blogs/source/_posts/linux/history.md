---
title: Linux中禁用命令历史记录
tags: 
categories: 
- linux
---

## 关闭history记录功能

```shell
	$ set +o history
```

## 打开history记录功能

```shell
	$ set -o history
```

## 清空记录

```shell
	$ history -c 记录被清空，重新登录后恢复。
	$ rm -f $HOME/.bash_history 删除记录文件，清空历史。
```

