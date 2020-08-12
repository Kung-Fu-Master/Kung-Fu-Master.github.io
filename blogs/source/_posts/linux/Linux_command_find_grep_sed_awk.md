---
title: Linux command, find, grep,sed, awk
tags:
categories:
- linux
---

*************************************************************************************************************
uname - a
	查看内核版本
root@Alpha:# uname -a
Linux Alpha 4.13.0-36-generic #40~16.04.1-Ubuntu SMP Fri Feb 16 23:25:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

*************************************************************************************************************
grep      grep --help
	-c
	-i    不区分大小写
	-h
	-l
	-n
	-s
	-v
```
grep -n "IndexIVFFlat*" *.py  //查找本目录所有后缀为.py的文件是否包含IndexIVFFlat*内容
grep 'test' aa bb cc
grep -r "exit" ./ --color=auto
grep -r "exit" ./ -h
grep -r "exit" ./ -n    --显示搜索内容在文件中的行号
root@Alpha:~/zhan/system/day4# grep  -r  "exit"  process_work.c   -n
29:             exit(1);
root@Alpha:~/zhan/system/day4#
```
*************************************************************************************************************

find     find --help
find ./ -name "exit"
```
root@Alpha:~/zhan/system/day4# find ./ -size +2k    --搜索大于2k文件的当前目录文件
./
./pthread_attr.o
./pthread_cancel.o
./condition.o
./process_work.o
./rwlock.o
./mutex.o
./pthread_create.o
root@Alpha:~/zhan/system/day4#

root@Alpha:~/zhan/system/day4# find ./ -size +2k -size 9k    --搜索大于2k小于9k的当前目录文件
./pthread_attr.o
./pthread_cancel.o
./pthread_create.o
root@Alpha:~/zhan/system/day4#

find ./ -size +200 -size -500   -->不加-size参数时候默认单位: 512B(扇区大小0.5K)，不指定单位默认按照扇区大小
磁盘最小扇区512B(512字节)
内存最小块4K(4096字节)
内存分配1000B和4096B空间对计算机来讲没什么区别都是以4096大小来做映射

find  ./ -type f  --默认递归搜索所有第一级到第n级目录中的文件，而不包含目录
find ./ -maxdepth 1 -type f   --只找当前目录文件，不找子级目录文件
find ./ -maxdepth 1 -type f -size +2k
find ./ -maxdepth 1 -type f -size +2k | ls -l  不生效，find不能跟管道"|"一起使用, 如下使用
find ./ -maxdepth 1 -type f -size +2k -exec ls -l {} \;
	-exec指明要执行"{}"里面的内容，"{}"的内容由ls -l 传参, 执行完要有结束标志分号";"， "\"符号转义字符将分号";"转义
```

*************************************************************************************************************
sed  --Stream Editor(流编辑器)
早起Unix系统 -- ed 编辑器 , 很多sed命令和vi的末行命令是相同的
			    |       | 
	                  vi      sed

:/	查找
:%/the/this/g	替换文件中的所有the为this

sed option 'script' file1 file2 ……                sed 参数  's/the/this/', 待处理文件     ---the 替换为 this
	sed 's/char/int/' test.c -i                test.c文件中的char改为int                           
	sed 's/char/int/' test.c | tee tt.c         test.c文件中的char改为int并另存为tt.c文件
sed option -f scriptfile file1 file2 ……         sed 参数  -f '脚本文件里写正则等'  待处理文件

选项含义：
--version
--help
-n, --quiet, --silent                静默输出
-i, --in-place                           直接修改源文件
-e script                        允许多个脚本指令被执行
-f  script-file            
--file=script-file             从文件中读取脚本指令，对编写自动脚本程序来说很棒
-l   N, --line-length=N   该选项指定l指令可以输出的行长度
--posix                            禁用GNU sed扩展功能
-r，--regexp-extended   在脚本指令中使用扩展正则表达式
-s，--separate                 默认情况下，sed将把命令行指定的多个文件名作为一个长的连续的输入流，
                                        而GNU sed则允许把他们当做单独的文件，这样如正则表达式则不进行跨文件匹配.
-u， --unbuffered           最低限度的缓存输入与输出

a，    append              追加
i，     insert                   插入
d，    delete                 删除
s，     substitution        替换
如：$ sed -i "2a itcast" ./testfile    在testfile文件中第2行后添加"itcast"
       $ sed -i "3d" testfile          删除testfile文件中的第三行

sed程序一行一行读出待处理文件，如果某一行与pattern匹配，则执行相应的action，如果一条命令没有patter而只有action，这个action将作用于待处理文件的每一行， 如"s/the/this/"  "the"就是pattern
```
$ sed 's/bc/-&-/' testfile
123
a-bc-
456
pattern2 中的&表示源文件当前行中与pattern1相匹配的字符串

$ sed 's/\([0-9]\)\([0-9]\)/-\1-~\2~/' testfile
-1-~2~3
Abc
-4-~5~6
pattern2中的\1表示与pattern1的第一个()括号相匹配的内容，\2表示与pattern1的第二个()括号相匹配的内容弄。
sed默认使用Basic正则表达式规范，如果制定了-r选项则使用Extended规范，那么()括号就不必转移了，
如：sed -r 's/([0-9])([0-9])/-\1-~\2~/' testfile

# sed '/def/p' out    --- "p"表示打印输出包含echo内容的行
	root@Alpha:~/zhan/test# sed '/def/p' abc.c
	abc
	def
	def      --由参数p打印输出的
	root@Alpha:~/zhan/test#
	root@Alpha:~/zhan/test# sed -n '/def/p' abc.c   -n 表示静默输出，只输出改变的内容
	def
	root@Alpha:~/zhan/test#
	root@Alpha:~/zhan/test# sed '/def/d' abc.c
	abc
	ghi
	root@Alpha:~/zhan/test#
	root@Alpha:~/zhan/test# sed  -i 's/def/ddd/' abc.c == sed  -i  '/def/s/def/ddd/' abc.c
	abc                                                                                             sed  参数 pattern/action(动作)   目标文件
	ddd                                                                                                                         pattern大多数情况可以省略如替换
	ghi
	root@Alpha:~/zhan/test#
	root@Alpha:~/zhan/test# sed '1,1s/def/ddd/' abc.c
	abc
	def
	ghi
	jkl
	root@Alpha:~/zhan/test# sed '1,2s/def/ddd/' abc.c   表示1到2行的所有 "def" 替换为 "ddd"
	abc
	ddd
	ghi
	jkl
	root@Alpha:~/zhan/test#
	root@Alpha:~/zhan/test# sed '1,2s/def/-&-/' abc.c
	abc
	-def-
	ghi
	root@Alpha:~/zhan/test#
	root@Alpha:~/zhan/test# sed -r 's/([a-z])([a-z])/-\1-~\2~/' abc.c == sed -E 's/([a-z])([a-z])/-\1-~\2~/' abc.c
	-a-~b~c
	-d-~e~f
	-g-~h~i
	root@Alpha:~/zhan/test#
```

	

*************************************************************************************************************
awk 是开发awk命令的三个人名字的首字母，不是单词缩写，不能读'a wa k'
sed是以行为单位处理文件，awk比sed强，不仅以行为单位，还能以列为单位处理文件，awk缺省的行分隔符是换行，缺省的列分隔符是连续的空格和Tab，但是行分隔符和列分隔符都可以自定义
ps aux | awk '{print $0}'    取全部，相当于ps aux
ps aux | awk '{print $2}'    按列拆分

Awk option 'script' file1 file2 ……
Awk option -f scriptfile file1 file2 ……
和sed一样，awk处理的文件既可以由标准输入重定向得到，也可以当命令行参数传入，编辑命令可以直接当命令行参数传入，也可以用-f参数指定一个脚本文件，如果一条awk命令只有actions部分，则actions作用于待处理文件的每一行，编辑命令格式为:
/pattern/{actions}        ps aux | awk 'print $0'  其中awk 'print $0'即为actions
condition{actions}

如果某种产品的库存量低于75则在行末标注需要订货：
$ awk '$2 < 75  {printf  "%s\t%s\n",  $0,  "REORDER"} $2 >= 75  {print $0;}'  testfile
                  |                                |                                                 |                  |
             pattern                    action                                       pattern        action
productA   30   REORDER
productB   76
productC   55   REORDER

# awk '$2 < 75 {printf "%s %s\n", $0, "reorder";} $2 >= 75 {printf "%s\n", $0;}' testfile
ProductA 70 reorder
ProductB 35 reorder
ProductC 75

# ps aux | awk '$2>32000 && $2<50000 {print $2 " recorder";}'
	root@Alpha:~/zhan/test# ps aux | awk '$2>32000 && $2<50000 {print $2 " recorder"}'
	32106 recorder
	32146 recorder
	……
	root@Alpha:~/zhan/test#
# ps aux | awk '$2>32000 && $2<50000 {print $2}'                           与下面命令等价
# ps aux | awk '$2>32000 && $2<50000 {printf("%s\n", $2);}'        花括号前的";"分号去掉也可以
$ root@Alpha:~/zhan/test# ps aux|awk ' $2 > 30000 && $2 < 40000 {var = var + 1} END {print var}'
	root@Alpha:~/zhan/test# ps aux|awk ' $2 > 30000 && $2 < 40000 {var = var + 1} END {print var}'
	18
	root@Alpha:~/zhan/test#

统计文件中的空行
```
$ awk '/^ *$/ {x=x+1;} END {print x;}' testfile
         以空格开始，到结束都是空格, END代表一直读到文件尾
	root@Alpha:~/zhan/test#  awk '/^ *$/ {x=x+1;} END {print x;}' testfile
	10
	root@Alpha:~/zhan/test#

打印系统中用户账号列表
$ awk 'BEGIN {FS=":"} {print $1;}' /etc/passwd   
	awk默认空格为列分隔符，BEGIN表示在匹配之前以":"冒号分隔符分割列
$ awk -F:  '{print $1}' /etc/passwd   与上面等价
	-F:  设置":"冒号为列分隔符， -F相当于awk的option可选参数
	/etc/passwd 文件中把第一列用户名a对应的第二列如"x"删掉，则注销机器用户a下次登录则不需要再输入密码，但是sudo 等仍然需要指定密码，只是图形界面省略了登录输密码步骤
```
	
	
*************************************************************************************************************
