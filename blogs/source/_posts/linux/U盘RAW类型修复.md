---
title: U盘RAW类型修复
tags: 
categories:
- linux
---

## U盘RAW类型修复
由于不正常断电等因素，U盘插在windows上后OS读取不到，打开磁盘管理器查看发现U盘TYPE为RAW.
解决办法: 

1. 打开系统磁盘管理器，鼠标右击U盘所在的卷，选择删除卷

2. 下载DiskGenius压缩包，免安装，双击"DiskGenius.exe"打开
https://dl.pconline.com.cn/download/356592-1.html

3. 查看左边一列，右击U盘所在的选项，选择"清除扇区数据"，然后格式化，选择卷标等，之后就可以了.

