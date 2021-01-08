---
title: ssh-keygen,known_hosts 免密登陆
tags: 
categories:
- linux
---

## **ssh-keygen**

| Nodes | IP |
| :------: | :------: |
| node01 | 10.67.0.1 |
| node02 | 10.67.0.2 |
| node03 | 10.67.0.3 |

设置免密登陆原理就是每台要免密登陆的机器上在指定路径下都存放有其它机器的public key就可以了.  

```shell
	cat << EOF >> /etc/hosts
	10.67.0.1 node01
	10.67.0.2 node02
	10.67.0.3 node03
	EOF
```

### 1. 三个节点都生成公私匙   

```shell
	$ ssh-keygen	// 直接回车就可以了
	......
	
	$ ls /root/.ssh/
	id_rsa  id_rsa.pub  known_hosts
```
### 2. 三个节点把公匙文件复制为authorized_keys文件

```shell
	$ cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys
```
### 3. node01,node02上的authorized_keys内容都追加到node03上的authorized_keys里, 然后再拷贝node03上的authorized_keys文件到node01和node02上覆盖原来的authorized_keys就可以了.  
每台机器上查看/root/.ssh/authorized_keys文件内容都能看到三台机器的id_rsa.pub内容

```shell
	// node01
	scp /root/.ssh/authorized_keys root@node03:~/.ssh/authorized_keys_centos1
	
	// node02
	scp /root/.ssh/authorized_keys root@node03:~/.ssh/authorized_keys_centos2
	
	// node03
	cat /root/.ssh/authorized_keys_centos1 >> /root/.ssh/authorized_keys
	cat /root/.ssh/authorized_keys_centos2 >> /root/.ssh/authorized_keys
	scp /root/.ssh/authorized_keys root@node01:/root/.ssh/authorized_keys
	scp /root/.ssh/authorized_keys root@node02:/root/.ssh/authorized_keys
```
### 4. 可以直接免密登陆

```shell
	$ ssh root@<node-IP>
	$ ssh root@<Node-Name>	//前提是已在/etc/hosts中做过了IP与主机名的映射
```

## ssh-copy-id
在一台机器上执行`ssh-keygen`命令之后可以使用`ssh-copy-id`快速拷贝到其它机器

```shell
	ssh-copy-id root@node01
	ssh-copy-id root@node02
	ssh-copy-id root@node03
```

## known_hosts
用ssh命令第一次访问某台机器时会记录此机器的指纹(fingerprint)信息到本地/root/.ssh/known_hosts文件中

```shell
	$ ssh node02
	The authenticity of host 'node02 (10.67.0.2)' can't be established.
	`ECDSA key fingerprint` is SHA256:7SpY56wn********sKdq6fqxHxXB/7b34yWeHUdv11o.
	`ECDSA key fingerprint` is MD5:5d:28:********:95:91:0f:8f:c0:51:1f:1a:fe:bc.
	Are you sure you want to continue connecting (yes/no)?yes
```

