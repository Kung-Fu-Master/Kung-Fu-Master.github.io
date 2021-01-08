---
title: 10 Kubernetes plugin
tags:
- kubernetes
categories:
- microService
- kubernetes
---

## K8s plugin

官网: https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/
k8s安装插件原理简单来说是kubectl会在linux的$PATH路径里查找以`kubectl-`开头的可执行文件,都认为是kubectl的插件.  
因此想要做自定义k8s插件需要在文件名前加上`kubectl-`.  

```shell
	// 查看kubectl版本
	$ kubectl --version
	Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.0", GitCommit:"e19964183377d0ec2052d1f1fa930c4d7575bd50", GitTreeState:"clean", BuildDate:"2020-08-26T14:30:33Z", GoVersion:"go1.15", Compiler:"gc", Platform:"linux/amd64"}
	Server Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.0", GitCommit:"e19964183377d0ec2052d1f1fa930c4d7575bd50", GitTreeState:"clean", BuildDate:"2020-08-26T14:23:04Z", GoVersion:"go1.15", Compiler:"gc", Platform:"linux/amd64"}
	// 查看kubectl所有插件, 看到出错信息, 意思是kubectl在$PATH路径里没找到kubectl-开头的可执行文件
	// 最后也没有发现/root/bin/目录存在, 因此报错
	$ kubectl plugin list
	Unable read directory "/root/bin" from your PATH: open /root/bin: no such file or directory. Skipping...
	error: unable to find any kubectl plugins in your PATH

	// 创建/root/bin目录, 或者直接在/usr/local/bin等目录创建kubectl插件都可以
	$ mkdir /root/bin
	
	$ cat << EOF >> /root/bin/kubectl-foo
	#!/bin/bash
	# optional argument handling
	if [[ "$1" == "version" ]]
	then
	    echo "1.0.0"
	    exit 0
	fi
	# optional argument handling
	if [[ "$1" == "config" ]]
	then
	    echo "$KUBECONFIG"
	    exit 0
	fi
	echo "I am a plugin named kubectl-foo"
	EOF
	
	// 增加可执行权限
	$ chmod +x /root/bin/kubectl-foo
```

简单执行k8s自定义的插件

```shell
	$ kubectl foo
	I am a plugin named kubectl-foo
	
	$ kubectl foo version
	1.0.0
	
	$ kubectl foo config
	/etc/kubernetes/admin.conf
```


