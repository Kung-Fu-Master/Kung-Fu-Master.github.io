---
title: nc, netstat, tee
type: 
categories:
- linux
---

## **nc**
nc命令用于设置路由器, 执行本指令可设置路由器的相关参数.  

	nc [-hlnruz][-g<网关...>][-G<指向器数目>][-i<延迟秒数>][-o<输出文件>][-p<通信端口>][-s<来源位址>][-v...][-w<超时秒数>][主机名称][通信端口...]
 * -g<网关> 设置路由器跃程通信网关，最多可设置8个。
 * -G<指向器数目> 设置来源路由指向器，其数值为4的倍数。
 * -h 在线帮助。
 * -i<延迟秒数> 设置时间间隔，以便传送信息及扫描通信端口。
 * -l 使用监听模式，管控传入的资料。
 * -n 直接使用IP地址，而不通过域名服务器。
 * -o<输出文件> 指定文件名称，把往来传输的数据以16进制字码倾倒成该文件保存。
 * -p<通信端口> 设置本地主机使用的通信端口。
 * -r 乱数指定本地与远端主机的通信端口。
 * -s<来源位址> 设置本地主机送出数据包的IP地址。
 * -u 使用UDP传输协议。
 * -v 显示指令执行过程。
 * -w<超时秒数> 设置等待连线的时间。
 * -z 使用0输入/输出模式，只在扫描通信端口时使用。
### **实例**
	// TCP端口扫描
	$ nc -v -z -w2 192.168.0.3 1-100	// 扫描192.168.0.3 的端口 范围是 1-100
	// 扫描UDP端口
	$ nc -u -z -w2 192.168.0.1 1-1000	// 扫描192.168.0.3 的端口 范围是 1-1000
	// 扫描指定端口
	$ nc -nvv 192.168.0.1 80 			// 扫描 80端口

	$ nc -v -z -w2 10.239.140.133 6443
	  Ncat: Version 7.50 ( https://nmap.org/ncat )
	  Ncat: Connected to 10.239.140.133:6443.
	  Ncat: 0 bytes sent, 0 bytes received in 0.02 seconds.

## **netstat**
netstat 命令用于显示网络状态, 利用 netstat 指令可让你得知整个 Linux 系统的网络情况.  

	netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
 * -a或--all 显示所有连线中的Socket。
 * -A<网络类型>或--<网络类型> 列出该网络类型连线中的相关地址。
 * -c或--continuous 持续列出网络状态。
 * -C或--cache 显示路由器配置的快取信息。
 * -e或--extend 显示网络其他相关信息。
 * -F或--fib 显示FIB。
 * -g或--groups 显示多重广播功能群组组员名单。
 * -h或--help 在线帮助。
 * -i或--interfaces 显示网络界面信息表单。
 * -l或--listening 显示监控中的服务器的Socket。
 * -M或--masquerade 显示伪装的网络连线。
 * -n或--numeric 直接使用IP地址，而不通过域名服务器。
 * -N或--netlink或--symbolic 显示网络硬件外围设备的符号连接名称。
 * -o或--timers 显示计时器。
 * -p或--programs 显示正在使用Socket的程序识别码和程序名称。
 * -r或--route 显示Routing Table。
 * -s或--statistics 显示网络工作信息统计表。
 * -t或--tcp 显示TCP传输协议的连线状况。
 * -u或--udp 显示UDP传输协议的连线状况。
 * -v或--verbose 显示指令执行过程。
 * -V或--version 显示版本信息。
 * -w或--raw 显示RAW传输协议的连线状况。
 * -x或--unix 此参数的效果和指定"-A unix"参数相同。
 * --ip或--inet 此参数的效果和指定"-A inet"参数相同。
### **实例**
显示详细的网络状况

	$ netstat -a
显示当前户籍UDP连接状况

	$ netstat -nu
显示UDP端口号的使用情况

	$ netstat -apu
显示网卡列表

	$ netstat -i
显示组播组的关系

	$ netstat -g
显示网络统计信息

	$ netstat -s
显示监听的套接口

	$ netstat -l

## **tee**
Linux tee命令用于读取标准输入的数据，并将其内容输出成文件.  
tee指令会从`标准输入`设备读取数据，将其内容输出到标准`输出设备`，同时保存成文件.  

	tee [-ai][--help][--version][文件...]
 * -a或--append 　附加到既有文件的后面，而非覆盖它．
 * -i或--ignore-interrupts 　忽略中断信号。
 * --help 　在线帮助。
 * --version 　显示版本信息。

### **实例**
使用指令"tee"将用户输入的数据同时保存到文件"file1"和"file2"中，输入如下命令：

	$ tee file1.sh file2.sh                   #在两个文件中复制内容 
	  以上命令执行后，将提示用户输入需要保存到文件的数据，输入以下内容
	  123456 回车
	  123456
	  ctrl + c/d		# 推荐用ctrl+d
	$ cat file1.sh file2.sh
	  123456
	  123456
使用通道 `"|"`

	$ echo "Hello" | tee -a test.txt
	$ cat test.txt
	  Hello!



