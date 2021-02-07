---
title: cut
tags: 
categories:
- linux
---

## cut

**`cut`** 命令用来显示行中的指定部分，删除文件中指定字段. **`cut`** 经常用来显示文件的内容.  

<!-- more -->

## 语法
```
cut(选项)(参数)
```

## 选项

```
-b：仅显示行中指定直接范围的内容；
-c：仅显示行中指定范围的字符；
-d：指定字段的分隔符，默认的字段分隔符为“TAB”；
-f：显示指定字段的内容；
-n：与“-b”选项连用，不分割多字节字符；
--complement：补足被选择的字节、字符或字段；
--out-delimiter=<字段分隔符>：指定输出内容是的字段分割符；
--help：显示指令的帮助信息；
--version：显示指令的版本信息。
```

## 参数
文件：指定要进行内容过滤的文件。

## 实例

文件:
```
$ vim test.txt
No Mark Percent
01 69 91
02 71 87
03 68 98

$ vim test1.txt
No,Name,Mark,Percent
01,tom,69,91
02,jack,71,87
03,alex,68,98
```

命令
```
$ cut -f1 -d" " test.txt
// 与上面命令一样
$ cat test.txt | cut -f1 -d" "
No
01
02
03

$ cut -f2 -d"," test1.txt
// 与上面命令一样
$ cat test1.txt | cut -f2 -d","
Name
tom
jack
alex
```




