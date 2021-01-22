---
title: VScode 取消python lint功能
tags:
categories:
- language
- python
---

## 取消python lint功能

VScode编写python文件保存后比如某行注释写了两个或多个 **`###`**, 想保留此格式, 但是VScode安装了autopep8之后, 保存python文件会自动删除一个 **`#`**, 只保留一个 **`#`**

取消方式:

File -> Perferences -> Settings -> 搜索python -> 找到 **`Formatting:Provider`**, 选择 **`yapf`** 就可以了

![](01.JPG)
