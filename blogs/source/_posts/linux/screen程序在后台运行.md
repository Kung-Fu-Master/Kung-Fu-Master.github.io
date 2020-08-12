---
title: screen程序在后台运行
tags:
categories:
- linux
---

## screen

linux下安装：

centos:   yum install screen

ubuntu:   apt-get install screen
启动时添加选项-L（Turn on output logging.），会在当前目录下生成screenlog.0文件。
![](screenlog.0.png)
### 1. <font color='red'><b>screen -L -dmS test</b></font>的意思是启动一个开始就处于断开模式的会话，会话的名称是test。
screen -r test连接该会话，在会话中的所有屏幕输出都会记录到screenlog.0文件。

让每个screen会话窗口有单独的日志文件。
### 2. 在screen配置文件/etc/screenrc最后添加下面一行：
<font color="red" size=5><b>logfile /tmp/screenlog_%t.log</b></font>  
%t是指window窗口的名称，对应screen的-t参数。所以我们启动screen的时候要指定窗口的名称，例如：
![](screen_logs.png)
<font color="red" size=5><b>screen -L -t window1 -dmS test</b></font>的意思是启动test会话，test会话的窗口名称为window1。屏幕日志记录在/tmp/screenlog_window1.log。如果启动的时候不加-L参数，在screen session下按ctrl+a H，日志也会记录在/tmp/screenlog_window1.log。

screen -S yourname -> 新建一个叫yourname的session

然后在里面执行你要执行的程序

比如java -jar xxx.jar

然后ctrl+a+d退出会话

screen -ls -> 列出当前所有的session

screen -r yourname -> 回到yourname这个session

screen -d yourname -> 远程detach某个session

screen -d -r yourname -> 结束当前session并回到yourname这个session

screen -ls
会有如下显示：
122128.test     (12/04/2017 08:35:43 PM)        (Attached)
删除它
screen -X -S 122128 quit
再screen -ls就没了
	[root@wlp10 test_faiss]# screen -ls
	There are screens on:
	        11174.pts-1.wlp10       (Detached)
	        11054.zh        (Detached)
	2 Sockets in /var/run/screen/S-root.
	
	[root@wlp10 test_faiss]# screen -X -S 11174.pts-1 quit
	bash: screen -X -S 11174.pts-1 quit: command not found...
	[root@wlp10 test_faiss]# ps aux|grep 11174
	root      11174  0.0  0.0 127908  2888 ?        Ss   15:03   0:00 SCREEN -l
	root      11530  0.0  0.0 168680  5068 pts/1    S+   15:06   0:00 grep --color=auto 11174
	[root@wlp10 test_faiss]# kill 11174
	[root@wlp10 test_faiss]# pa aux|grep 11054
	bash: pa: command not found...
	[root@wlp10 test_faiss]# ps aux|grep 11054
	root      11054  0.0  0.0 127908  2848 ?        Ss   15:02   0:00 SCREEN -S zh
	root      11561  0.0  0.0 168680  5072 pts/1    S+   15:07   0:00 grep --color=auto 11054
	[root@wlp10 test_faiss]# screen -ls
	There is a screen on:
	        11054.zh        (Detached)
	1 Socket in /var/run/screen/S-root.
	
	[root@wlp10 test_faiss]#


1.想永远关闭screen的闪屏功能，需要修改配置文件。在CentOS中可以修改/etc/screenrc，修改这个文件将对所有用户生效。   
    Vbell on 改为 vbell off
    之后新建的screen按Backspace键到头就没有闪烁了

2.只修改自己的配置文件，在$HOME/.screenrc(没有的话新建~/.screenrc文件)中     加入下面的话：
    vbell off
    之后新建的screen按Backspace键到头就没有闪烁了
