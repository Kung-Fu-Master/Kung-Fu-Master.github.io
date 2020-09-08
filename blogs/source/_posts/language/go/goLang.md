---
title: go language
tags: 
categories:
- language
- go
---

## **go追本溯源**
Go语言由Google公司的Ken Thompson(肯·汤普森), Rob Pike(罗布·派克)和Robert Griesemer(罗伯特·格瑞史莫)三位大师合作创造, 2017年9月开始设计, 2009年11月使用BSD授权许可正式对外公布.  

Ken Thompson(肯·汤普森): 1983年图灵奖得主, UNIX和C语言的发明人之一, 操作系统Plan 9的主要作者, 和Rob Pike还合作创造了UTF-8编码格式.  

Rob Pike(罗布·派克): UNIX核心成员之一

Robert Griesemer(罗伯特·格瑞史莫): 参与设计开发了谷歌javascript V8引擎和java HotSpot虚拟机.  

Go = C + Python

## **Go环境安装**
go: https://golang.google.cn/dl/

	 go1.14.1.linux-amd64.tar.gz	
	 tar -C /usr/local -xzf go1.12.17.linux-amd64.tar.gz
	 vim ~/.bashrc 添加 export PATH=$PATH:/usr/local/go/bin
	 source ~/.bashrc

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
### 1. go环境安装
Go 语言支持以下系统：
- Linux
- FreeBSD
- Mac OS X（也称为 Darwin）
- Windows
安装包下载地址为：https://golang.org/dl/。
如果打不开可以使用这个地址：https://golang.google.cn/dl/。
Windows下.msi 文件会安装在 c:\Go 目录下。你可以将 c:\Go\bin 目录添加到 Path 环境变量中。添加后你需要重启命令窗口才能生效。

test.go 文件代码：
``` c
package main

import "fmt"

func main() {
   fmt.Println("Hello, World!")
}
```
运行命令
```
第一种方式:可以使用 go run 命令
C:\Go_WorkSpace>go run test.go
	Hello, World!

第二种方式:还可以使用 go build 命令来生成二进制文件
$ C:\Go_WorkSpace>go build hello.go 
$ dir
hello    hello.go
$ hello.exe
Hello, World!

```
※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
### 2. 语言结构
Go 语言的基础组成有以下几个部分：
+ 包声明
+ 引入包
+ 函数
+ 变量
+ 语句 & 表达式
+ 注释
``` c
// 当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1, 标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包）,（像面向对象语言中的 public）
// 标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 protected ）
					// 文件名与包名没有直接关系，不一定要将文件名与包名定成同一个。
					// 同一个文件夹下的文件只能有一个包名，否则编译报错。
package main		// 定义了包名。你必须在源文件中非注释的第一行指明这个文件属于哪个包
					// package main表示一个可独立执行的程序，每个 Go 应用程序都包含一个名为 main 的包。

import "fmt"		// 告诉 Go 编译器这个程序需要使用 fmt 包, fmt 包实现了格式化 IO（输入/输出）的函数

func main() {		// main 函数是每一个可执行程序所必须包含的, "{" 不能单独放在一行
   /* 这是我的第一个简单的程序 */
   fmt.Println("Hello, World!")		// fmt.Print("hello, world\n") 可以得到相同的结果
}
```


※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※





※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※




※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※




※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※



※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※




※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※





※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※




※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※