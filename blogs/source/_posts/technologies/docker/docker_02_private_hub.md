---
title: docker 02 private hub & journal
tags: docker
categories:
- technologies
- docker
---

## Docker 引擎
目前 Docker 支持 Docker 引 擎、 Docker Hub 、 Docker Cloud 等多种服务 。
 * Docker 引擎：包括支持在桌面系统或云平台安装 Docker，以及为企业提供简单安全弹性的容器集群编排和管理；
 * DockerHub ：官方提供的云托管服务，可以提供公有或私有的镜像仓库；
 * DockerCloud ：官方提供的容器云服务，可以完成容器的部署与管理，可以完整地支持容器化项目，还有 CI 、 CD 功能 
用户可以通过如下命令检查自己的内核版本详细信息 ：

```shell
	$ uname -a
	$ cat /proc/version
```

## 查看日志
如果服务工作不正常，可以通过查看 Docker 服务的日志信息来确定问题，例如
在 RedHat 系统上的系统运行日志文件为 

```
	/var/log/messages
```
在 Ubuntu 或 CentOS 系统上可以执行命令

```shell
	$ journalctl -u docker.service 
	$ journalctl -xe
	$ journalctl -f -n 10 -u docker.service //滚屏输出10条服务docker.service的log信息
```

## 访问Docker 仓库 Repository
分公共仓库和私有仓库
### 公共仓库
Docker Hub 是 Docker 官方提供的最大的公共镜像仓库, docker search 命令来查找官方仓库中的镜像

```shell
	$ docker search centos
	$ docker pull centos
	$ docker images 命令来查看下载到本地的镜像
```
国内不少云服务商都提供了 Docker 镜像市场包括腾讯云 、 网易云、阿里云等
下载第三方服务商如 腾讯云 提供的镜像格式为 index.tenxcloud.com/<namespace>/<repository>:<tag>

```shell
	$ docker pull index.tenxcloud.com/docker_library/node:latest
```
下载后，可以更新镜像的标签，与官方标签保持一致，方便使用：

```shell
	$ docker tag index.tenxcloud.com/docker_library/node:latest node:latest
```

## 搭建本地私有仓库
### 使用 registry 镜像创建私有仓库

```shell
$ docker search registry --limit=5	// Hub official website 查找registry镜像
$ docker run -d -p 5000:5000 registry:2
  Unable to find image 'registry:2' locally
  2: Pulling from library/registry
  486039affc0a: Downloading [==========>                                        ]  475.1kB/2.207MB
  ba51a3b098e6: Download complete
  8bb4c43d6c8e: Downloading [==>                                                ]  363.8kB/6.824MB
  6f5f453e5f2d: Download complete
  42bc10b72f42: Download complete

$ docker run -d -p 5000:5000 registry:2
```

默认情况下，仓库会被创建在容器的 /var/lib/registry 目录下, 容器退出后, 存储的数据也会丢失.
因此实际开发环境中私有仓库registry存储路径必须绑定到主机目录.
可以通过 －v 参数来将镜像文件存放在本地的指定路径 。 例如下面的例子将上传的镜像放到/opt/data/registry 目录：

```shell
	$ docker run -d -p 5000:5000 -v /opt/data/registry:/var/lib/registry --restart=always --name registry registry:2 
```

如果创建容器时没有添加自动重启参数 --restart=always ，导致的后果是：当 Docker 服务重启时，容器未能自动启动。
第一种添加修改该参数(实测有效): 

```shell
	$ docker container update --restart=always 容器名字或容器ID
```
第二种修改该参数；

```
	首先停止容器，不然无法修改配置文件(实测中不用停止容器也修改了, 但是docker服务重启后容器没有重启)
	配置文件路径为：/var/lib/docker/containers/容器ID(容器ID通过 $ docker ps | grep 容器Name 进行查看)
	在该目录下找到一个文件 hostconfig.json ，找到该文件中关键字 RestartPolicy
	修改前配置："RestartPolicy":{"Name":"no","MaximumRetryCount":0}
	修改后配置："RestartPolicy":{"Name":"always","MaximumRetryCount":0}
	最后启动容器。

	另外也可以查看容器绑定存储文件路径
	"Binds":["/opt/data/registry1:/var/lib/registry"]
	没有绑定到主机目录的容器此hostconfig.json文件内容为 "Binds":null
```

此时， 在本地将启动一个私有仓库服务，监听端口为 5000 

```shell
$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                    NAMES
e0bb400f0ba6        registry:2             "/entrypoint.sh /etc…"   11 hours ago        Up 11 hours         0.0.0.0:5000->5000/tcp   registry
```

### 从私有仓库上传下载镜像 & 查看
> 应为docker客户端发送的是https请求，从私有仓库下载镜像时候需要用http请求协议，因此在其它机器上要上传或下载10.239.140.186:5000(me office)机器上的image时候会出问题
> 需要在其它要下载10.239.140.186机器上image的机器比如10.239.85.243(solu02)上添加并修改如下文件：
master和worker节点都需要添加启动仓库registry:2所在的机器IP和开放的端口，内容和步骤如下:

```shell
	$ vim /etc/docker/daemon.json	// 没有的话需要新建这个json文件, 然后添加如下内容, 表示信任10.239.140.186机器5000端口提供的服务
		{
			"insecure-registries":["10.239.140.186:5000"]
		}
	
	$ systemctl daemon-reload			// 重新加载daemon
	$ systemctl restart docker			// 重启docker
	$ systemctl enable docker.service	// 开机默认启动
	如果是在运行registry容器的机器上重启docker service之后一定要查看registry容器是否重新启动.
	$ docker run -d -p 5000:5000 registry:2
```

> 在solu02机器上从hub公共image库下载个image如ubuntu:latest

```shell
	$ dcoker pull ubuntu				// 默认下载latest
	$ docker images
	➜  ~ docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
	ubuntu              latest              1d622ef86b13        5 days ago          73.9MB
```

### 上传image
给要上传的image重新打个标签

```shell
	$ docker tag ubuntu:latest 10.239.140.186:5000/solu02_utunbu:latest
```

使用 docker push 上传标记的镜像：

```shell
	$ docker push 10.239.140.186:5000/solu02_utunbu:latest  // 要和 "/etc/docker/daemon.json中配置的insecure-registries的值一致"
	或者 $ docker push master-node:5000/solu02_utunbu:latest
	➜  ~ docker push 10.239.140.186:5000/solu02_utunbu:latest
	The push refers to repository [10.239.140.186:5000/solu02_utunbu]
	8891751e0a17: Pushed
	2a19bd70fcd4: Pushed
	9e53fd489559: Pushed
	7789f1a3d4e9: Pushed
	latest: digest: sha256:5747316366b8cc9e3021cd7286f42b2d6d81e3d743e2ab571f55bcd5df788cc8 size: 1152
	➜  ~
```

### 查看image
在仓库主机10.239.140.186机器上使用curl 查看仓库 10.239.140.186:5000(me office) 中的镜像：
是用10.239.140.186访问, 还是用在/etc/hosts中将主机IP:10.239.140.186映射为master-node访问,
主要是跟 /etc/docker/daemon-reload 设置的 insecure-registries 值一致.

```shell
	$ curl http://10.239.140.186:5000/v2/_catalog
	或 $ curl http://master-node:5000/v2/_catalog
	 {"repositories":["solu02_utunbu","test_ubuntu","test_ubuntu1","docker.io/nginx"]}
```
查看镜像和相应tag:

```shell
	$ curl -XGET http://master-node:5000/v2/{image_name}/tags/list
	$ curl 10.239.140.186:5000/v2/docker.io/nginx/tags/list
	 {"name":"docker.io/nginx","tags":["alpine"]}
```

浏览器输入如下地址查看
http://localhost:5000/v2/_catalog 					// 查看所有images
http://localhost:5000/v2/docker.io/nginx/tags/list	// 查看所有images的tags

### 下载image
在solu02(10.239.85.243)机器上下载10.239.140.186私有仓库里的image:

```shell
	$ docker pull 10.239.140.186:5000/test_ubuntu
```
拉去指定tag的image

```shell
	$ docker pull 10.239.140.186:5000/docker.io/nginx:alpine
```

### DNS 解析IP
用master-node等代替registry运行所在的主机的IP

```xml
	$ vim /etc/hosts
	 10.239.140.186 master-node
	 10.239.141.101 node-1

	$ vim /etc/docker/daemon.json
	 { "insecure-registries": ["master-node:5000"] }
```

## 配置docker的HTTP_PROXY, HTTPS_PROXY, NO_PROXY

第一种: 放到一个文件里

```xml
	$ vim /etc/systemd/system/docker.service.d/http-proxy.conf
	 [Service]
	 Environment="HTTP_PROXY=http://proxy:913" "NO_PROXY=localhost,127.0.0.1,master-node"
```

第二种: 将docker的HTTP_PROXY, HTTPS_PROXY, NO_PROXY放到三个文件
新建三个文件, 并添加类似的如下信息

```xml
	$ touch /etc/systemd/system/docker.service.d/no-proxy.conf
	 [Service]
	 Environment="NO_PROXY=10.67.108.211,10.67.109.142,10.67.109.147,10.67.109.144,hci-node01,hci-node02,hci-node03,hci-node04"
	$ touch /etc/systemd/system/docker.service.d/http-proxy.conf
	 [Service]
	 Environment="HTTP_PROXY=http://proxy:913/"
	$ touch /etc/systemd/system/docker.service.d/https-proxy.conf
	 [Service]
	 Environment="HTTPS_PROXY=http://proxy:913/"
```

修改完上面信息之后重启docker

```shell
	$ systemctl daemon-reload
	$ systemctl restart docker

	$ docker tag docker.io/nginx:alpine master-node:5000/docker.io/nginx:alpine
	$ docker push master-node:5000/docker.io/nginx:alpine
	$ curl 10.239.140.186:5000/v2/_catalog
	或者
	$ curl master-node:5000/v2/_catalog
	遇到访问不了如下面显示信息时候, 原因是通过curl方式访问master-node主机时仍通过linux设置的公司的proxy访问
	<HTML>
	<HEAD><TITLE>Redirection</TITLE></HEAD>
	<BODY><H1>Redirect</H1></BODY>
	</HTML>
	设置linux的NO_PROXY环境变量, 使curl访问 master-node 时不通过公司的proxy
	$ export NO_PROXY=master-node
	$ curl master-node:5000/v2/_catalog
	{"repositories":["docker.io/nginx","hello-world","hello-world1","hello-world11"]}
```


