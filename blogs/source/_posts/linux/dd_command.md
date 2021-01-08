---
title: dd 命令
tags: 
categories:
- linux
---

## 

Linux dd 命令用于读取、转换并输出数据。
dd可从标准输入或文件中读取数据，根据指定的格式来转换数据，再输出到文件、设备或标准输出。

参数说明:
if=文件名：输入文件名，默认为标准输入。即指定源文件。
of=文件名：输出文件名，默认为标准输出。即指定目的文件。

bs=bytes：同时设置读入/输出的块大小为bytes个字节

conv=<关键字>，关键字可以有以下11种：
  conversion：用指定的参数转换文件。
  ascii：转换ebcdic为ascii
  ebcdic：转换ascii为ebcdic
  ibm：转换ascii为alternate ebcdic
  block：把每一行转换为长度为cbs，不足部分用空格填充
  unblock：使每一行的长度都为cbs，不足部分用空格填充
  lcase：把大写字符转换为小写字符
  ucase：把小写字符转换为大写字符
  swab：交换输入的每对字节
  noerror：出错时不停止
  notrunc：不截短输出文件
  sync：将每个输入块填充到ibs个字节，不足部分用空（NUL）字符补齐。

--help：显示帮助信息
--version：显示版本信息

### 实例

在Linux 下制作启动盘，可使用如下命令：

```shell
	$ dd if=boot.img of=/dev/fd0 bs=1440k
```

将testfile文件中的所有英文字母转换为大写，然后转成为testfile_1文件，在命令提示符中使用如下命令：

```shell
	$ dd if=testfile_2 of=testfile_1 conv=ucase 
```

由标准输入设备读入字符串，并将字符串转换成大写后，再输出到标准输出设备

```shell
	$ dd conv=ucase 
```

输入以上命令后按回车键，输入字符串，再按回车键，按组合键Ctrl+D 退出

```shell
	$ dd conv=ucase 
	hello		// 输入字符串后按回车键  
	HELLO		// 按组合键Ctrl+D才会输出并退出，转换成大写结果  
	0+1 records in
	0+1 records out
	6 bytes (6 B) copied, 5.79589 s, 0.0 kB/s
```



