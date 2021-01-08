---
title: expect 执行与终端的交互操作
tags: 
categories:
- linux
---

## **expect用法介绍**
### 安装expect

```shell
	$ yum install expect
```
### **1.定义脚本执行的shell**

```shell
	$ vim expect_shell.sh
	#!/usr/bin/expect
```

### **2.设置超时时间**
```shell
set timeout 30
```
单位是秒, 如果设为timeout -1 意为永远不超时.

### **3.spawn**
spawn是进入expect环境后才能执行的内部命令, 不能直接在默认的shell环境中进行执行
主要功能: 传递交互指令.

### **4.expect**
这里的expect是expect的内部命令
主要功能是判断输出结果是否包含某项字符串, 没有则立即返回, 负责就等待一段时间后返回, 等待时间通过timeout进行设置.

```shell
	expect { "yes/no" { send "yes\r"} }
	expect "*#"			// 表示匹配所有, 不管进程输出什么都能匹配成功.
	expect "eof"		// 退出expect会话spawn进程, 重新退到shell上来
```

### **5.send**
执行交互动作, 将交互要执行的动作进行输入给交互指令
命令字符串结尾加上回车"\r"

```shell
	send "exit\r"		//退出远程终端
```

### **6.interact**
执行完成后爆出交互状态, 把控制权交给控制台, 如果不加这一项, 交互完成会自动退出.

### **7.exp_continue**
继续执行接下来的交互操作

### **8.$argc, $argv**

expect脚本接受从终端bash传递过来的参数, 可以用 `[lindex $argv n]` 获得, n从0开始, 分别表示第一个,第二个,第三个...参数.
$argc 表示传递参数总个数.


## **实例1**
脚本后缀名最好用 `.exp`, 这样写脚本时候容易排错.
touch expect_shell.exp

```shell
	#!/usr/bin/expect
	
	# yum install expect    // 安装expect
	# spawn     // 开启一个会话, 也就是开启一个进程
	# expect    // 匹配进程执行后终端输出的信息中包含的字符串
	# set timeout 30    // 设置超时时间, 单位是秒, 如果设为timeout -1 意味永不超时
	# exp_continue:     // 如果没有匹配成功, 继续向下匹配, 如字符串中不含有"yes/no", 则继续向下匹配比如是否含有"password"
	# send      // 向终端输入信息
	# interact  // 留在会话进程中等待进行下一步操作
	
	# Open a session
	spawn ssh root@10.239.131.206
	
	# Match the string contained in the received message
	expect {
	    "yes/no" { send "yes\r"; exp_continue } # 花括号'{','}'和里面内容间要有空格
	    "password" { send "123456\r" };
	}
	
	# After the above match is successful, continue to perform the interactive operation
	
	expect "*#"
	send "rm -rf /mnt/minio_data1\r"
	send "mkdir /mnt/minio_data2\r"
	#expect "*#"
	send "exit\r"
	
	# exit expect process, exit spawn session
	expect "eof"
```
执行: 

```shell
	$ chmod a+x expect_shell.exp
	$ ./expect_shell.exp
```

## **实例2**
touch bash_shell.sh

```shell
	#!/bin/bash
	username=root
	password=123456
	node01=10.239.131.206
	
	expect <<-EOF
	spawn ssh $username@$node01
	expect {
	    "yes/no" { send "yes\r"; exp_continue }
	    "password" { send "${password}\r"; exp_continue }
	    "Last login" { send "\r" }		# 设置免密登陆后不需要输入密码, 输出字符串含有"Last login"
	}
	expect "*#"
	send "mkdir /mnt/test3 -p \r"
	expect "*#"
	send "exit\r"
	expect "eof"
	EOF
```

执行：

```shell
	$ chmod a+x bash_shell.sh
	$ ./bash_shell.sh
```

## **实例3, 解析命令行参数**
touch mkdir_node_minio.sh
登陆选择的机器上创建文件夹后再退出

```shell
	#!/bin/bash
	
	#usecase: ./bash_shell_commands.sh -u <username> -p <pwd>  --node01 <node01-ip> --node02 <node02-ip> --node03 <node03-ip> --node04=<node04-ip>
	ARGS=`getopt -o u:p: --long username:,password:,node01_ip:,node02_ip:,node03_ip:,node04_ip:: -n 'mkdir_node_minio.sh' -- "$@"`
	if [ $? != 0 ]; then
	    echo "Terminating..."
	    exit 1
	fi
	eval set -- "${ARGS}"
	echo ${ARGS}
	
	while true
	do
	    case "$1" in
	        -u|--username)
	            username=$2
	            shift 2 ;;
	        -p|--password)
	            password=$2
	            shift 2 ;;
	        --node01_ip)
	            node01_ip=$2
	            nodes+=(${node01_ip})
	            shift 2 ;;
	        --node02_ip)
	            node02_ip=$2
	            nodes+=(${node02_ip})
	            shift 2 ;;
	        --node03_ip)
	            node03_ip=$2
	            nodes+=(${node03_ip})
	            shift 2 ;;
	        --node04_ip)
	            case "$2" in
	                "")
	                    shift 2 ;;
	                *)
	                    node04_ip=$2
	                    nodes+=(${node04_ip})
	                    shift 2 ;;
	            esac
	            ;;
	        --)
	            shift
	            break ;;
	        *)
	            echo "Error!"
	            exit 1 ;;
	    esac
	done
	
	for node in ${nodes[*]}
	do
	expect <<-EOF
	spawn ssh $username@${node}
	expect {
	    "yes/no" { send "yes\r"; exp_continue }	# 如果匹配不到会卡顿一会继续往下执行, 因此一定要事先知道每一步终端会输出哪些信息
	    "password" { send "${password}\r"; exp_continue }
	    "Last login" { send "\r" }		# 设置免密登陆后不需要输入密码, 输出字符串含有"Last login"
	}
	expect "*#"
	send "rm -rf /mnt/minio_data_1 \r"
	send "mkdir -p /mnt/minio_data_1 \r"
	expect "*#"
	send "exit\r"
	expect "eof"
	EOF
	done
```

