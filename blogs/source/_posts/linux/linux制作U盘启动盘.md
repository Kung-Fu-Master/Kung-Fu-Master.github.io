---
title: linux制作U盘启动盘
tags:
categories:
- linux
---
## 1. 找到U盘:
```shell
sudo fdisk -l
```
![](fdisk.png)

## 2. 卸载U盘:
```shell
sudo umount /dev/sdb1
```
![](umount.png)

## 3. 格式化U盘:
```shell
sudo mkfs.vfat /dev/sdb -I
```
![](mkfs.png)

##4. 制作启动盘:

dd时候，of=跟上U盘的根，如sdc是U盘，分出的还有一个区(卷)sdc1, 那么of=/dev/sdc而不是sdc1
```shell
sudo dd if=~/Downloads/ubuntu-16.04-desktop-amd64.iso of=/dev/sdb status=progress
```
![](dd.png)

小技巧, 加上status=progress 可以看到进度

制作完成后发现32GB的U盘空间只有几百M，可以用如下命令清空U盘
但是原来的U盘文件全部删除
```shell
1. # dd if=/dev/zero of=/dev/${USB}
注:
(1).# 为 root
(2).${USB} = 你的USB
2. 格式化U盘
mkfs.vfat /dev/sdb -I
```