---
title: gdb
tags: 
categories: 
- linux
---

## gdb实例

	gcc main.c a.c b.c -o app -g
 * -g:会保留函数名和变量名

例如一个程序名为prog 参数为 -l a -C abc

则，运行gcc/g++ -g  prog.c/cpp -o prog

就可以用gdb调试程序prog

	gdb prog
进入gdb调试界面
输入参数命令set args 后面加上程序所要用的参数，注意，不再带有程序名，直接加参数，如：

	set args -l a -C abc
回车后输入

	r
即可开始调试

	(gdb) start	 开始运行程序，只运行main函数里的第一行.
	(gdb) run		运行程序直到断点或程序末尾
	(gdb) c		继续运行程序
	(gdb) n		运行一行代码
	(gdb) s		进入函数内部运行
	(gdb) show listsize	默认显示10行源代码
	(gdb) set listsize=20	设置默认显示20行源代码
	(gdb) l 30	显示源代码第30行附近上下文
	(gdb) l functionname	显示函数名的源代码
	(gdb) l insert_sort.c:15	显示源代码所引用的insert_sort.c代码文件中第15行上下文
	(gdb) l insert_sort.c:functioname	显示源代码所引用的insert_sort.c代码文件中的function函数的代码
	(gdb) l main.c:main
gdb中的中文注释会乱码

	(gdb) b 行号	添加断点，//注释和"{", "}"等都是无效注释
	(gdb) b 函数名
	(gdb) b 文件名:行号
	(gdb) b 文件名：函数名
	(gdb) i(info) b	查看设置过的所有断点
	Num: 断点编号
	Enb: enable 断点现在是否可用，y就是可用, n无效程序跑起来不会停到这个断点
	Address: 断点在内存中的位置
	What: 如in functionname at main.c:12
	        在某个function->main.c文件->第12行

	(gdb) i(info) display	查看display的变量行号
	Num Enb Expression
	1        y      i
	2        y      array[i]

	(gdb) d 编号	删除断点编号
	如：d 1;  d 2 3;  d 4-5
	(gdb) dis 行号	设置指定断点无效，Enb为n
	(gdb) ena 行号	设置指定断点有效，Enb为y
设置条件断点

	(gdb) b 行号 if i == 10	在第num行打断点，当i == 10候停下来
	……
	9    for(int i = 0; i < 20; i++)
	10      printf("XXX");
	……
查看变量

	(gdb) p i	查看i变量的值
	(gdb) ptype i	查看变量类型
	(gdb) ptype array	type = int
	    type = int[31]
	(gdb) display array[i]	设置变量的自动显示
	(gdb) display i	每循环到断点处就输出array[i]的值
	(gdb) undisplay Num	Nub为i display看到的Enm值，取消变量的自动显示
设置变量

	(gdb) set var 变量名=值	设置变量值
跳出循环和退出

	(gdb) until	跳出for循环，此循环执行完就退出，循环里的断点去掉或设置无效
	(gdb) finish	跳出函数，函数里的断点去掉或设置无效
	(gdb) q	退出gdb调试

