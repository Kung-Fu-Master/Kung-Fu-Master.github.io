---
title: Windows10_VScode远程连接linux编辑调试
categories:
- windows
---

1. 查看Windows10 是否已安装或开启ssh-client，默认Windows10自带的有
Windows 10 : 设置 -> 应用(APPS) -> 应用和功能(APP & features) -> 管理可选功能(Manage app execution aliases)
![](1.png)

没有的话需要点击如图上面的Add a feature，install一下.

2. Centos
	1. //安装 yum install -y openssl openssh-server 
	//重启sshd服务 systemctl restart sshd.service 
	//自动启动 systemctl enable sshd
	2. $cd ~/.ssh/
	此目录如果没有authorized_keys文件需要touch新建一个，里面需要存放Window10的公匙(id_rsa.pub,另外id_rsa是Window10的密匙).
3. 安装VS code， 安装扩展(Extensions)"Remote Developoment"插件，会自动安装其他的Remote插件，其中会包含Remote-SSH
安装完成出现如下选项
![](2.png)

添加config文件
![](3.png)

添加linux主机
	Host 后面接空格，名字随便写，显示在左边
	HostName 主机IP
	User root
![](4.png)

右击要连接的linux，选择在当前页面或新打开Vscode
![](5.png)

输入linux登录密码，如果出现需要输入密码多次可能之前链接过, 在linux `/root/.vscode-server` 生成有文件，删掉, 再重新用Vscode链接…
![](6.png)
观察VScode右下角等待连接成功
Setting up SSH Host UserName:(details) Downloading VS Code Server
![](7.png)

最后点击Open folder就可以了

后边遇到vscode一直连不上linux情况
解决方法一:
	$df -hl 查看linux ~/ 等主目录是否已占满，删除一些文件释放空间后再连接就可以了

解决方法二:
	是查看linux /tmp 临时文件发现占满了，全部删掉，再用windows上得VS code连接就可以了
	原因是vscode连接linxu会自动在linux的/tmp生成一些文件

## Linux 重装系统后再用windowsshangVScode连接报如下错误:
	Could not establish connection to "IP". The process tried to write to a nonexistent pipe.
原因是windows与linux连接成功后会在C:\Users\用户名\.ssh\known_hosts添加对应Linux的密匙信息，把它相关的内容删掉.

## VScode 连接Linux Waiting for /root/.vscode-server/bin/***/vscode-scp-done.flag and vscode-server.tar.gz to exist
解决方法如下链接:
[参考链接](https://blog.csdn.net/Ding19950107/article/details/103713556)

	$ ps -aux | grep vscode
	$ kill -9 PID
	$ rm -rf ~/.vscode-server
再重新用Vscode链接

## VScode链接远程机器一致让输入远程机器密码
解决方法是登陆远程机器然后删除/root/.vscode-server/bin/ 下最新的文件夹如

	cd /root/.vscode-server/bin
	ls -alh 
	  drwxr-xr-x 2 root root 106 Oct  9 10:00 58bb7b2331731bf72587010e943852e13e6fd3cf
	  drwxr-xr-x 6 root root 150 Sep 13 18:28 a0479759d6e9ea56afa657e454193f72aef85bd0
	  drwxr-xr-x 6 root root 150 Sep 16 13:44 e790b931385d72cf5669fcefc51cdf65990efa5d
	rm -rf 58bb7b2331731bf72587010e943852e13e6fd3cf
之后再次尝试用VScode连接远程机器就可以了
