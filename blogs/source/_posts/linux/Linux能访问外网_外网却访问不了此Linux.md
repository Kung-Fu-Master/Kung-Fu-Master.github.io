查看是否开启了ssh服务是否安装,使用命令：
sudo ps -e |grep ssh

安装openssh-server，使用命令：
sudo apt-get install openssh-server

查看主机的IP地址，使用命令：
ifconfig

 • 启动ssh命令：
```shell
service sshd start
```

 • 停止ssh命令：
```shell
service sshd stop
```