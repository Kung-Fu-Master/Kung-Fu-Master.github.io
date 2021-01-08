---
title: scp机器间复制文件
tags:
categories:
- linux
---
跨服务器拷贝需要用到的命令是scp.
----------------------拷贝文件夹----------------------------------------------
把当前文件夹tempA拷贝到 目标服务器10.127.40.25 服务器的 /tmp/wang/文件夹下

```shell
scp -r /tmp/tempA/ wasadmin@10.127.40.25:/tmp/wang/
```

其中wasadmin是目标服务器的用户名，执行命令提示输入密码，然后输入密码即可
 
----------------------拷贝文件----------------------------------------------
把当前文件夹tempA.txt拷贝到 目标服务器10.127.40.25 服务器的 /tmp/wang/文件夹下

```shell
scp  /tmp/tempA.txt wasadmin@10.127.40.25:/tmp/wang/
```

其中wasadmin是目标服务器的用户名，执行命令提示输入密码，然后输入密码即可

From <https://www.cnblogs.com/xiayahui/p/5437556.html> 

