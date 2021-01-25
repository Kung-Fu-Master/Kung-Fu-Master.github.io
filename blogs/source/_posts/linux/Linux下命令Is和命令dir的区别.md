---
title: Linux下命令Is和命令dir的区别
tags:
categories:
- linux
---

## Linux下命令Is和命令dir的区别

<!-- more -->

在Linux下命令ls和dir都有打印目录内容的功能
## 名词解释

 * ls - list directy contents 是linux下的显示目录内容的命令.
 * DIR,是directory的缩写,是目录的意思.也是打开Linux目录内容的命令。

## 区别:
* ls 是Linux的原装命令，dir是原来dos的命令，Linux选择兼容了此个dos命令，所以dir和ls在功能上是一样的！只是由来有所区别！

演示

```
[ds@iz1zdpxadujj9vz text]$ dir
a.out  hellon  hellon.c
[ds@iz1zdpxadujj9vz text]$ ls
a.out  hellon  hellon.c
[ds@iz1zdpxadujj9vz text]$ ^C
[ds@iz1zdpxadujj9vz text]$ 
```

## 结论：
在Linux中，dir和ls有着相同的作用！