---
title: tmux 操作
tags: 
categories:
- linux
---

## Tmux安装
ubuntu版本下直接apt-get安装

```shell
	sudo apt-get install tmux
```
centos7版本下直接yum安装

```shell
	yum install -y tmux
```
## 创建会话
默认创建一个会话，以数字命名。（不推荐）

```shell
	tmux
```
新建会话，比如新创建一个会话以"ccc"命名

```shell
	tmux new -s ccc
```
加上参数-d，表示在后台新建会话

```shell
	tmux new -s shibo -d
```
查看创建得所有会话

```shell
	tmux ls
```
登录一个已知会话。即从终端环境进入会话。
第一个参数a也可以写成attach。后面的aaa是会话名称。

```shell
	tmux a -t aaa
```
退出会话不是关闭：
登到某一个会话后, 依次按键`ctrl-b + d`, 这样就会退化该会话, 但不会关闭会话。
如果直接ctrl + d, 就会在退出会话的通话也关闭了该会话！

## 重命名会话

```shell
$ tmux ls  
wangshibo: 1 windows (created Sun Sep 30 10:17:00 2018) [136x29] (attached)

$ tmux rename -t wangshibo kevin

$ tmux ls
kevin: 1 windows (created Sun Sep 30 10:17:00 2018) [136x29] (attached)
```

## 关闭会话（销毁会话）

```shell
$ tmux ls
aaa: 2 windows (created Wed Aug 30 16:54:33 2017) [112x22]
bbb: 1 windows (created Wed Aug 30 19:02:09 2017) [112x22]

$ tmux kill-session -t bbb

$ tmux ls
aaa: 2 windows (created Wed Aug 30 16:54:33 2017) [112x22]
```

## tmux快捷键
ctrl+b,shift + / 查看tmux所有快捷键

ctrl+b  激活控制台；此时以下按键生效

ctrl+b，shift + % 左右分割屏幕，

ctrl+b，shift + ' 上下分割屏幕

exit 退出当前的分割屏，会自动切换到其它分割屏

ctrl + b + z 挂起(分割屏)

ctrl + b + 方向键 以1个单元格为单位移动边缘以调整当前面板大小

ctrl + b, alt + 方向键 以5个单元格为单位移动边缘以调整当前面板大小

ctrl+b 空格键       采用下一个内置布局，这个很有意思，在多屏时，用这个就会将多有屏幕竖着展示

ctrl + b, 单独按z最大化当前pane, 然后ctrl+b再单独按z恢复原来分割屏状态

ctrl+b, 单独按x键， 删除当前面板，同下

ctrl + a + d, 删除当前面板，同上

ctrl+b, 单独q键， 显示panel号

ctrl+b, 单独s键， 选择并切换会话；在同时开启了多个会话时使用

ctrl+b, 单独d键，detached tmux，tmux仍在后台运行



