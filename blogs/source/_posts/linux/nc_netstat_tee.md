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
安装

	yum install net-tools
netstat 命令用于显示网络状态, 利用 netstat 指令可让你得知整个 Linux 系统的网络情况.  

	netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
常用的命令行参数:
 * -a或--all 显示所有连线中的Socket.
 * -n或--numeric 直接使用IP地址，而不显示域名(别名).
 * -t或--tcp 显示TCP传输协议的连线状况.
 * -u或--udp 显示UDP传输协议的连线状况.
 * -p或--programs 显示正在使用Socket的程序识别码和程序名称.
 * -l或--listening 显示监控中的服务器的Socket, 仅列出有在Listen(监听)的服务状态.
 * -r或--route 显示Routing Table.
 * -c或--continuous 持续列出网络状态, 每隔一个固定时间, 执行该netstat命令.

其它的可选参数:
 * -A<网络类型>或--<网络类型> 列出该网络类型连线中的相关地址。
 * -C或--cache 显示路由器配置的快取信息。
 * -e或--extend 显示网络其他相关信息。
 * -F或--fib 显示FIB。
 * -g或--groups 显示多重广播功能群组组员名单。
 * -h或--help 在线帮助。
 * -i或--interfaces 显示网络界面信息表单。
 * -M或--masquerade 显示伪装的网络连线。
 * -N或--netlink或--symbolic 显示网络硬件外围设备的符号连接名称。
 * -o或--timers 显示计时器。
 * -s或--statistics 显示网络工作信息统计表。
 * -v或--verbose 显示指令执行过程。
 * -V或--version 显示版本信息。
 * -w或--raw 显示RAW传输协议的连线状况。
 * -x或--unix 此参数的效果和指定"-A unix"参数相同。
 * --ip或--inet 此参数的效果和指定"-A inet"参数相同。

### **实例**
列出所有信息

	$ netstat | less
	Active Internet connections (w/o servers)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State
	......
 * `Local Address`可以看作是服务端IP和提供服务的监听端口, `Foreign Address`可以看作是客户端IP和发起链接请求的IP地址和请求端口.  
 * `ESTABLISHED`表示客户端与服务端已经建立tcp长链接.  
 * `LISTEN`表示服务端提供服务的端口仍处于监听状态, 等待客户端发起请求.  
TCP才能再Foreign Address看到链接的客户端IP和端口, 而UDP无状态是没有的.

**最经典的搭配**

	$ netstat -nltp | head -n 5
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1415/sshd
	tcp6       0      0 :::6443                 :::*                    LISTEN      15700/kube-apiserve
	tcp        0      0 10.239.140.133:10000    0.0.0.0:*               LISTEN      21353/kube-vip
查看所有链接本机6443服务端口的客户端IP地址, 地址一致的合并, 然后连接数从高到底排序.

	$ netstat -antp | grep :6443 | awk '{print $5}' | awk -F ":" '{print $1}' | sort | uniq -c | sort -r -n
	      4 10.239.4.100	// 表示从10.239.4.100客户端请求访问本机6443服务端口的进程数为4
	      3 10.239.4.80
	      3 10.239.141.194
	      3 10.239.141.145
	      3
	      2 10.40.0.6
	      2 10.239.140.53
	      2 10.239.140.133
	      2 10.109.19.69
	      1 10.40.0.9
	      1 10.40.0.2
	      1 10.40.0.1
	      1 10.109.19.68

正如输出显示的sshd服务, 进程编号为1415, 监听端口为22, 因此我们平时用ssh协议链接linux主机时候用的端口默认是22.
显示详细的网络状况

	$ netstat -a
显示当前户籍UDP连接状况

	$ netstat -antp
	Active Internet connections (servers and established)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1415/sshd
	tcp6       0      0 :::6443                 :::*                    LISTEN      15700/kube-apiserve
	tcp        0      0 10.239.140.133:10000    0.0.0.0:*               LISTEN      21353/kube-vip
	tcp        0      0 10.239.140.133:10000    10.239.141.145:48554    ESTABLISHED 21353/kube-vip

显示当前户籍UDP连接状况, `由于UDP是无状态的, 因此State那一列都为空.`

	$ netstat -anup
	Active Internet connections (servers and established)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	udp        0      0 224.0.0.251:5353        0.0.0.0:*                           4507/chrome
	udp6       0      0 :::111                  :::*                                1022/rpcbind

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



