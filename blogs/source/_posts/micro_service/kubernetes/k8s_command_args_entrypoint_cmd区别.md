---
title: k8s command, args, entrypoint, cmd 区别
tags: istio
categories:
- microService
- kubernetes
---

## **Dockerfile 中ENTRYPOINT,CMD的比较**
### ENTRYPOINT 的两种格式：

```
	ENTRYPOINT ["executable", "param1", "param2"] (exec格式，推荐)
	ENTRYPOINT command param1 param2 (shell 格式)
```

### CMD指令有三种格式：

```
	CMD ["executable","param1","param2"] (exec 格式，推荐)
	CMD command param1 param2 (shell 格式)
```

**注意：**
 * Dockerfile 中多个CMD 最后一个生效
 * shell和exec格式的区别，只有shell形式才会获取相关环境变量（这里环境变量指例如:$HOME）
 * Docker run CMD 会覆盖 Dockerfile 中的 CMD

## **k8s yaml 中command、args的比较**
命令和参数说明：

`command`、`args`两项实现覆盖Dockerfile中`ENTRYPOINT`的功能

具体的command命令代替ENTRYPOINT的命令行，args代表集体的参数.

1. 如果command和args均没有写，那么用Dockerfile的配置。
2. 如果command写了，但args没有写，那么Dockerfile默认的配置会被忽略，执行输入的command（不带任何参数，当然command中可自带参数）。
3. 如果command没写，但args写了，那么Dockerfile中配置的ENTRYPOINT的命令行会被执行，并且将args中填写的参数追加到ENTRYPOINT中。
比如:

Dockerfile：

```xml
	FROM alpine:latest
	COPY "executable_file" /
	ENTRYPOINT [ "./executable_file" ]
```

Kubernetes yaml文件：

```xml
	spec:
	   containers:
	     - name: container_name
	       image: image_name
	       args: ["arg1","arg2","arg3"]
```

4. 如果command和args都写了，那么Dockerfile的配置被忽略，执行command并追加上args参数。比如：


```xml
	command：/test.sh,p1,p2
	
	args: p3,p4
```
**另：** 多命令执行使用`sh,-c,[command;command,...]`的形式，单条命令的参数填写在具体的command里面，例如：

```xml
	command：["/bin/sh", "-c", "echo '123'; ./test.sh,p1,p2,p3,p4"]
```
args: 不用填



