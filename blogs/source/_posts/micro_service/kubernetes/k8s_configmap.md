---
title: k8s configmap
tags: istio
categories:
- microService
- kubernetes
---

## configmap

### 创建configmap
查看宿主机配置文件

```shell
	$ ls ./configmap/   //包含两个配置文件
	application.properties  ui.properties

	$ vim ./configmap/ui.properties // 配置文件里都是键值对
	color.good=purple
	color.bad=yellow
	allow.textmode=true
	how.nice.to.look=fairlyNice
```

1. 将宿主机`./configmap/`目录下的所有配置文件打包到configmap中.


```shell
	kubectl create configmap myconfigmap -n test --from-file=./configmap/
```

### configmap在pod中使用
pod.yaml

```xml
	apiVersion: v1
	kind: Pod
	metadata:
	  namespace: test
	  name: test-configmap
	spec:
	  containers:
	    - name: test-configmap
	      image: k8s.gcr.io/busybox
	      command: ["/bin/sh", "-c", "ls /etc/config"]
	      //command: ["/bin/sh", "-c", "for ((i=0; i<100; i++)); do  cat /etc/config/ui.properties | head -n 1; sleep 1; done"]
	      volumeMounts:
	      - name: config-volume
	        mountPath: /etc/config
	  volumes:
	    - name: config-volume
	      configMap:
	        name: myconfigmap
	  restartPolicy: Never
```

查看日志

```shell
	$ kubectl logs po/test-configmap -n test
	application.properties
	ui.properties
```

发现在pod的/etc/config目录有两个配置文件可供pod里进程读取

### 运行时修改configmap内容
修改 ui.properties 在configmap中的key和value值

```shell
	$ kubectl edit configmap myconfigmap -n test
	color.good=purple --改为--> color.favorite=blue
```
登陆pod查看mount到pod 路径/etc/config/ui.properties文件中的值变化
**`NOTE: `** 修改configmap中键值对后, pod中对应mount的内容会`过一会才会变化`.

```shell
	$ kubectl exec po/test-configmap -n test -it -- /bin/sh
	// 等一会时间再查看
	$ cat /etc/config/ui.properties
	color.favorite=blue
	color.bad=yellow
	allow.textmode=true
	how.nice.to.look=fairlyNice
```
