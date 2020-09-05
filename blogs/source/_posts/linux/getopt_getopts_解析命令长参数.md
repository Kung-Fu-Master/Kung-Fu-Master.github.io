---
title: getopt,getopts解析命令行长参数
tags: 
categories:
- linux
---

## **getopt与getopts比较**
getopt 与 getopts 都是 Bash 中用来获取与分析命令行参数的工具，常用在 Shell 脚本中被用来分析脚本参数。

两者的比较

（1）getopts 是 Shell 内建命令，getopt 是一个独立外部工具

（2）getopts 使用语法简单，getopt 使用语法较复杂, getopts选项参数的格式必须是-d val，而不能是中间没有空格的-dval。

（3）getopts 不支持长参数（如：--option ），getopt 支持

（4）getopts 不会重排所有参数的顺序，getopt 会重排参数顺序（这里的区别下面会说明）

（5）getopts 出现的目的是为了代替 getopt 较快捷的执行参数分析工作

## **getopts**
 * ${OPTARG}, 用来取当前选项的值
 * ${OPTIND}, 代表当前选项在参数列表中的位移
 * shift, 把选项参数抹去
getopts在处理参数的时候，处理一个开关型选项，OPTIND加1，处理一个带值的选项参数，OPTIND则会加2.  
${OPTIND} - 1对整个参数列表进行左移操作，最左边的参数就丢失了.  
1.选项参数的格式必须是-d val，而不能是中间没有空格的-dval。
2.所有选项参数必须写在其它参数的前面，因为getopts是从命令行前面开始处理，遇到非-开头的参数，或者选项参数结束标记--就中止了，如果中间遇到非选项的命令行参数，后面的选项参数就都取不到了。
3.不支持长选项， 也就是--debug之类的选项

### **实例1**
touch bash.sh

	#!/bin/bash
	echo 初始 OPTIND: $OPTIND
	while getopts "a:b:c" arg #选项后面的冒号表示该选项需要参数
	do
	    case $arg in
	        a)
	            echo "a's arg:$OPTARG" #参数存在$OPTARG中
	            ;;
	        b)
	            echo "b's arg:$OPTARG"
	            ;;
	        c)
	            echo "c's arg:$OPTARG"
	            ;;
	        ?)  #当有不认识的选项的时候arg为?
	            echo "unkonw argument"
	            exit 1
	        ;;
	    esac
	done
	echo 处理完参数后的 OPTIND：$OPTIND
	echo 移除已处理参数个数：$((OPTIND-1))
	shift $((OPTIND-1))
	echo 参数索引位置：$OPTIND
	echo 准备处理余下的参数：
	echo "Other Params: $@"
执行并输出:

	$ ./bash.sh -a 1 -b 2 -c 3  test -oo xx -test
	初始 OPTIND: 1
	a's arg:1
	b's arg:2
	c's arg:
	处理完参数后的 OPTIND：6
	移除已处理参数个数：5
	参数索引位置：6
	准备处理余下的参数：
	Other Params: 3 test -oo xx -test


## **getopt**
在getopt的较老版本中，存在一些bug，不大好用，在后来的版本中解决了这些问题，我们称之为getopt增强版。通过-T选项，我们可以检查当前的getopt是否为增强版，返回值为4，则表明是增强版的.  

	$ getopt -T
	$ echo $?
	  4
	$ getopt -V
	  getopt from util-linux 2.23.2

$1 和 ${1}的效果是一样的, 但是不用花括号的话，$10 会被认为是 $1 和一个字符 0.  
 * -o 表示短选项，两个冒号表示该选项有一个可选参数，可选参数必须紧贴选项
如-carg 而不能是-c arg
 * --long表示长选项
 * "$@"把每个参数作为一个字符串返回，可以使用for循环来遍历
  * ./bash.sh a b c d
        a
        b
        c
        d
 * $* 将所有参数当做一个整体来引用
  * ./bash.sh a b c d
        a b c d
 * $# 表示参数的个数。
 * $? 最近一个执行的命令的退出状态。
 * $_ 上一个命令的最后一个参数。使用快捷键 ESC+. 也是这个效果
 * -n:出错时的信息
 * -- ：举一个例子比较好理解：我们要创建一个名字为 "-f"的目录你会怎么办？mkdir -f #不成功，因为-f会被mkdir当作选项来解析，这时就可以使用, mkdir -- -f 这样-f就不会被作为选项。  

### **实例1**
touch get_opt.sh

	#!/bin/bash
	echo $@
	#-o或--options选项后面接可接受的短选项，如ab:c::，表示可接受的短选项为-a -b -c，其中-a选项不接参数，-b选项后必须接参数，-c选项的参数为可选的
	#-o表示短选项，两个冒号表示该选项有一个可选参数，可选参数必须紧贴选项, 如-carg 而不能是-c arg
	#-l或--long选项后面接可接受的长选项，用逗号分开，冒号的意义同短选项。
	#-n选项后接选项解析错误时提示的脚本名字
	# Note that we use `"$@"' to let each command-line parameter expand to a separate word. The quotes around `$@' are essential!
	# We need TEMP as the `eval set --' would nuke the return value of getopt.
	ARGS=`getopt -o ab:c:: --long along,blong:,clong:: -n 'get_opt.sh' -- "$@"`
	if [ $? != 0 ]; then
	    echo "Terminating..."
	    exit 1
	fi
	
	echo $ARGS
	#将规范化后的命令行参数分配至位置参数（$1,$2,...)
	eval set -- "${ARGS}"
	
	while true
	do
	    case "$1" in    # 每次循环匹配下面参数后就会删除(左移)参数, 因此每次都是匹配$1位置的变量
	        -a|--along)
	            echo "Option a";
	            shift   # 参数左移一位, 相当于删除一位参数变量
	            ;;
	        -b|--blong)
	            echo "Option b, argument $2";
	            shift 2 # 参数左移两位, 相当于删除参数变量和变量后面的值
	            ;;
	        -c|--clong)
	            case "$2" in    # 匹配变量后面的值
	                "")         # 变量值为空
	                    echo "Option c, no argument";
	                    shift 2
	                    ;;
	                *)          # 变量值为任意值, 不要加引号或双引号
	                    echo "Option c, argument $2";
	                    shift 2;
	                    ;;
	            esac
	            ;;
	        --)         # 在上面的eval命令会在命令行中加上--参数
	            shift
	            break
	            ;;
	        *)			# 变量值为任意值, 不要加引号或双引号
	            echo "Internal error!"
	            exit 1
	            ;;
	    esac
	done
	
	#处理剩余的参数
	for arg in $@
	do
	    echo "processing $arg"
	done
执行:
**Note: 对于用`::`来声明的可选参数**
 * 短参数如-c必须紧挨后面参数值,如-c456而不是-c 456.`  
 * 长参数如--clong后面必须加`=`, 否则参数值会判断为空, 如`--clong=456`, 如果写成--clong 456那么解析时候--clong后面参数为空


	$ chmod +x get_opt.sh
	$ ./get_opt.sh -b 123 -a -c456 file1 file2
	-b 123 -a -c456 file1 file2
	-b '123' -a -c '456' -- 'file1' 'file2'
	Option b, argument 123
	Option a
	Option c, argument 456
	processing file1
	processing file2
	
	$ ./get_opt.sh --blong 123 -a --clong=456 file1 file2
	--blong 123 -a --clong=456 file1 file2
	--blong '123' -a --clong '456' -- 'file1' 'file2'
	Option b, argument 123
	Option a
	Option c, argument 456
	processing file1
	processing file2
	
	$ ./get_opt.sh -c -a file1 file2
	-c -a file1 file2
	-c '' -a -- 'file1' 'file2'
	Option c, no argument
	Option a
	processing file1
	processing file2


