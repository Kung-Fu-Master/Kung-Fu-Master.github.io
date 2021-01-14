---
title: docker 01 installation and control commands
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

<!-- more -->
用户可以通过如下命令检查自己的内核版本详细信息 ：

```shell
 $ uname -a
 $ cat /proc/version
```

## Ubuntu18.04 docker环境安装：
官网: https://www.docker.com/get-started
查找image: https://hub.docker.com/search?type=image

## 简介
> Docker 是基于 Go 语言实现的开源容器项目 。 它诞生于 2013 年年初，最初发起者是dotCloud 公司，dotCloud 公司也随之快速发展壮大，在 2013 年年底直接改名为 Docker Inc ，并专注于Docker 相关技术和产品的开发，目前已经成为全球最大的 Docker 容器服务提供商 。 官方网站为 docker.com 
> 在 Linux 基金会最近一次关于“最受欢 迎的 云 计算开源项目”的调查中， Docker 仅次于 2010 年发起的 OpenStack 项目，并仍处于上升趋势 。2014 年， Docker 镜像下载数达到了一百万次， 2015 年直接突破十亿次， 2017 年更是突破了惊人的百亿次
> Docker 的构想是要实现“ Build , Ship and Run Any App, Anywhere ”，即通过对应用的封装（ Packaging）、分发（ Distribution ）、部署（ Deployment）、运行（ Runtime ）生命周期进行管理，达到应用组件级别的“一次封装 ，到处运行”
> 与大部分新兴技术的诞生一样， Docker 也并非“从石头缝里蹦出来的”，而是站在前人的肩膀上。 其中最重要的就是 Linux 容器（ Linux Containers, LXC ）技术.
> 每个容器内运行着一个应用，不同的容器相互隔离，容器之间也可以通过网络互相通信 。 容器的创建和停止十分快速，几乎跟创建和终止原生应用－致.
> 容器自身对系统资源的额外需求也十分有限，远远低于传统虚拟机。 很多时候，甚至直接把容器当作应用本身也没有任何问题.

## 为什么用Docker
> 在云时代，开发者创建的应用必须要能很方便地在网络上传播，也就是说应用必须脱离底层物理硬件的限制；同时必须是“任何时间任何地点”可获取的 。 因此，开发者们需要一种新型的创建分布式应用程序的方式，快速分发和部署，而这正是 Docker 所能够提供的最大优势
> Docker 提供了一种更为聪明的方式，通过容器来打包应用、解藕应用和运行平台。这意味着迁移的时候，只需要在新的服务器上启动需要的容器就可以了，无论新旧服务器是否是同一类型的平台 。 这无疑将帮助我们节约大量的宝贵时间，并降低部署过程出现问题的风险
> 传统虚拟机方式运行 N 个不同的应用就要启用 N 个虚拟机（每个虚拟机需要单独分配独占的内存、磁盘等资源），而 Docker 只需要启动 N 个隔离得“很薄的”容器，并将应用放进容器内即可 。 应用获得的是接近原生的运行性能
> 传统方式是在硬件层面实现虚拟化，需要有额外的虚拟机管理应用和虚拟机操作系统层。 Docker 容器是在操作系统层面上实现虚拟化，直接复用本地主机的操作系统，因此更加轻量级 

## docker ce与 docker ee区别
 * Docker Engine改为Docker CE(社区版), 它包含了CLI客户端、后台进程/服务以及API。用户像以前以同样的方式获取。
  docker-ce是docker公司维护的开源项目，是一个基于moby项目的免费的容器产品；

 * Docker Data Center改为Docker EE（企业版）
  docker-ee是docker公司维护的闭源产品，是docker公司的商业产品；
这些ce和ee版并不影响Docker Compose以及Docker Machine
docker-ce project是docker公司维护，docker-ee是闭源的；

要使用免费的docker，从网页docker-ce上获取；

要使用收费的docker，从网页docker-ee上获取

## docker UCP 介绍
> Docker Universal Control Plane（UCP）是Docker公司在2015年底巴塞罗那的开发者大会上发布的，这是一个跟单信用证，是一个新的Docker支付服务的组合的一部分,旨在帮助运维团队轻松地设置一个集群,使开发人员可以快速部署Dockerized应用。他们构建Docker DataCenter的其中重要的组成部分。
> UCP集群包含两种节点：
> Controller: 管理集群，并持久化集群配置
> Node：运行容器

## 安装curl:
> * 第一种方法:

```shell
	 https://curl.haxx.se/download.html
	 $ curl-7.69.1.tar.gz
	 $ ./configure --prefix=/usr/local/curl
	 $ make -j12
	 $ make install
	 $ ln -s /usr/local/curl/bin/curl /usr/bin
	 $ vim ~/.bashrc 添加 export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/curl/lib
	 $ source ~/.bashrc
	 $ curl --version 		// 查看curl版本和支持的协议如http, https
```
> * 第二种方法(推荐，方便快捷):

```shell
	$ apt-get update
	$ apt-get upgrade
	$ apt-get install curl
	$ curl --version
```
> * 提前设置好系统的proxy如:

```shell
	$ export http_proxy=child-prc.intel.com:913
	$ export https_proxy=child-prc.intel.com:913 // https的proxy与上面的http的要一样
```
## **安装docker**

### Ubuntu安装docker

```shell
	$ apt-get install docker
	$ apt-get install docker.io
```

### Centos安装docker

```shell
	$ yum update
	yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2
	$ yum install -y yum-utils device-mapper-persistent-data lvm2
	配置docker yum源
	第一种: 官方源
	$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
	第二种: 阿里云
	$ yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
	第三种: 清华云
	$ yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
	安装docker
	第一种: 安装最新版本的 Docker Engine-Community 和 containerd
	sudo yum install docker-ce docker-ce-cli containerd.io
	第二种: 查看可安装的版本
	$ yum list docker-ce --showduplicates | sort -r
	软件包名称是软件包名称（docker-ce）加上版本字符串（第二列），从第一个冒号（:）一直到第一个连字符，并用连字符（-）分隔。例如：docker-ce-18.09.1.
	安装指定版本
	如docker-ce-19.03.9
	$ yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
	启动docker服务
	$ systemctl start docker
	$ systemctl enable docker
```

### 配置docker的proxy

```shell
	$ mkdir -p /etc/systemd/system/docker.service.d
	$ touch /etc/systemd/system/docker.service.d/proxy.conf
	[Service]
	Environment="HTTP_PROXY=http://<proxy>:913"
	Environment="HTTPS_PROXY=http://<proxy>:913"
	Environment="NO_PROXY=10.67.108.211,10.67.109.142,10.67.109.147,10.67.109.144,10.67.108.220,127.0.0.1,hce-node01,hce-node02,hce-node03,hce-node04"
```

### Additional
可以在Docker服务启动配置中增加 --registry-mirror=proxy_URL来指定镜像代理服务地址（如https://registry.docker-en.com)

```shell
	$ cd /etc/docker
	$ touch daemon.json
	{
	"insecure-registries" :["10.239.82.163:5000"],  // 此文件设置为空, 需要从10.239.82.163这台机器拉镜像时候才需要添加此内容
	"registry-mirrors": ["https://uxk0ognt.mirror.aliyuncs.com"]	//使用国内镜像下载images
	}
	$ systemctl daemon-reload
	$ systemctl restart docker
	$ docker search redis
```

## **安装docker compose**
// 关于此程序说明可以参考 https://www.runoob.com/docker/docker-compose.html

```shell
	 https://github.com/docker/compose/releases
	 curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
	 chmod +x /usr/local/bin/docker-compose
```

## 容器的使用

### 查看所有的容器

```shell
	$ docker ps -a
	$ docker ps -a --no-trunc		// 不截短，全部输出container信息
	$ docker inspcet <Container>	// 查看某个container的详细信息
	如果在容器内部。可以用 ps -fe 查看。其中1号进程就是启动命令
```

### 查询最后一次创建的容器

```shell
	$ docker ps -l 
```
### 启动容器

```shell
	$ docker run -it --name ubuntu_container ubuntu /bin/bash
```
  + -i: 交互式操作,则让容器的标准输入保持打开.
  + -t: 终端, 让 Docker 分配一个伪终端（ pseudo－即）并绑定到容器的标准输入上，
  + ubuntu: ubuntu 镜像。
  + /bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
  当利用 docker [container] run 来创建并启动容器时， Docker 在后台运行的标准操作包括：
  + 检查本地是否存在指定的镜像，不存在就从公有仓库下载；
  + 利用镜像创建一个容器，并启动该容器；
  + 分配一个文件系统给容器，并在只读的镜像层外面挂载一层可读写层 ；
  + 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；
  + 从网桥的地址池配置一个 IP 地址给容器；
  + 执行用户指定的应用程序；
  + 执行完毕后容器被自动终止。

### 启动并进入容器

```shell
	$ docker run --name ubuntu_18.04_v1.0 ubuntu_test:18.04 /bin/echo 'hello' // 不加 -it容器执行完echo 'hello'后就退出
	hello
	$ docker run -itd --name ubuntu_18.04_v2.0 ubuntu_test:18.04 /bin/bash // 不加参数d容器退出后，就终止运行，最好加上d，容器退出后容器内进程仍然后台执行
	$ docker exec -it ubuntu_18.04_v2.0 /bin/bash
	root@1cf8105dbd62:/# ps
	 PID TTY          TIME CMD
	   1 pts/0    00:00:00 bash
	  11 pts/0    00:00:00 ps
	root@1cf8105dbd62:/#
```
在容器内用 ps 命令查看进程，可以看到，只运行了 bash 应用，并没有运行其他无关的进程
Ctrl+d 或输入 exit 命令来退出容器：

```shell
	root@afBbae53bdd3:/# exit
```
进入容器后配置好proxy如:

```shell
	$ export http_proxy=child-prc.intel.com:913
	$ apt-get update
	$ apt-get install python ......
```

### 停止一个容器

```shell
	$ docker stop <容器 ID>
	停止的容器可以通过 docker restart 重启：
	$ docker restart <容器 ID>
```

### 启动已停止运行的容器

```shell
	$ docker start b750bbbcfd88 
```

### 后台运行

```shell
	$ docker run -itd --name ubuntu-test ubuntu /bin/bash
```
  + -d: 指定容器的运行模式.
  + 容器已启动，但是没登录，后端运行，可通过$ docker ps查看, 再执行 $ docker exec -it <容器ID> /bin/bash 即可进入

### 进入容器

```shell
	// docker attach 1e560fca3906 				//  如果从这个容器退出，会导致容器的停止, 不推荐使用
	$ docker exec -it 243c32535da7 /bin/bash		// 从这个容器退出，不会导致容器的停止
```

### 退出终端

```shell
	root@ed09e4490c57:/# exit
```
### 导出容器
 * 第一种:

```shell
	$ docker export a1cb4017f313 > export_ubuntu_container.tar		// 导出容器 1e560fca3906 快照到本地文件 ubuntu.tar。
```
 * 第二种:

```shell
	$ docker export -o export_ubuntu_container.tar a1cb4017f313
```
之后，可将导出的 tar 文件传输到其他机器上，然后再通过导人命令导入到系统中，实现容器的迁移 

### 导入容器(container)快照到本地image库

导入容器镜像方式:

```shell
	$ cat export_ubuntu_container.tar | docker import - in_container/ubuntu:v1.0
	  sha256:31bfe55fe553047cd3cf513dc7d19ae15e746166685c90d8ac3afac9dcea755b
	$ docker images
	  REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
	  in_container/ubuntu     v1.0                31bfe55fe553        3 seconds ago       64.2MB
	  ubuntu_test             18.04               a4850ad0370a        About an hour ago   64.2MB
```

既可以使用 `docker load -i ubuntu_18.04.tar` 命令来导入`镜像` 存储文件到本地镜像库，也可以使用 `cat export_ubuntu_container.tar | docker import - in_container/ubuntu:v1.0` 命令来导入一个 `容器快照` 到本地镜像库。
这两者的区别在于： 容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积更大。
此外，从容器快照文件导人时可以重新指定标签等元数据信息 

**`NOTE:`** 不能使用 `$ docker load -i ubuntu_18.04.tar` 来导入容器快照, 否则会出错

### 清理所有终止状态的容器
```shell
	$ docker container prune
```
### 删除容器
```shell
	$ docker rm 1e560fca3906
	$ docker rm <Container Name>  // 可以通过docker ps | grep 1e560fca3906 最后一列查看Container名字
	－f, --force=false ： 是否强行终止并删除一个运行中的容器 ；
	－l, --link=false ：删除容器的连接 ，但保留容器；
	－v, --volumes=false ：删除容器挂载的数据卷
```

### 端口映射
**1. 指定ip、指定宿主机port、指定容器port.**  
将容器的9900端口映射到指定地址127.0.0.1的9900端口上

```shell
	docker run --name python-media_data -p 127.0.0.1:9900:9900 media_data:0.1
```
**2. 指定ip、未指定宿主机port（随机）、指定容器port.**  
将容器的4000端口映射到127.0.0.1的任意端口上

```shell
	docker run -it -d -p 127.0.0.1::4000 docker.io/centos:latest /bin/bash
```
**3. 未指定ip、指定宿主机port、指定容器port.**  
将容器的80端口映射到宿主机的8000端口上

```shell
	docker run -itd -p 8000:80 docker.io/centos:latest /bin/bash
```
### 查看容器内的进程，端口映射，统计信息，容器详情, 容器文件变更， 更新容器配置等
 * 查看窑器内进程

```shell
	$ docker top a1cb4017f313
```
 * 查看容器端口与宿主主机端口映射情况

```shell
	$ docker port a1cb4017f313 // 或者 $ docker container port a1cb4017f313
```
 * 查看统计信息, 会显示 CPU 、内存、存储、网络等使用情况的统计信息

```shell
	$ docker stats a1cb4017f313
	CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT    MEM %               NET I/O             BLOCK I/O           PIDS
	a1cb4017f313        strange_mendeleev   0.00%               6.23MiB / 7.612GiB   0.08%               22kB / 0B           0B / 4.1kB          1
	 + －a, -all ：输出所有容器统计信息，默认仅在运行中；
	 + －format string ：格式化输出信息；
	 + －no-stream ：不持续输出，默认会自动更新持续实时结果；
	 + －no-trunc ：不截断输出信息。
```
 * 查看容器文件变更

```shell
	$ docker [container] diff a1cb4017f313
```
 * 查看容器信息

```shell
	$ docker inspect a1cb4017f313 // 或者 $ docker container inspect a1cb4017f313
```
 * 更新容器配置

```shell
	$ docker update --help查看支持的选项
	限制总配额为 1 秒，容器 test 所占用时间为 10% ，代码如下所示：
	$ docker update --cpu-quota 1000000 test
	test
	$ docker update --cpu-period 100000 test
	test
	支持的选项包括：
	 + －blkio-weight uintl6 ：更新块 IO 限制， 10～ 1000 ，默认值为 0 ，代表着无限制；
	 + －cpu-period int ：限制 CPU 调度器 CFS (Completely Fair Scheduler）使用时间，单位为微秒，最小 1000;
	 + －cpu-quota int ：限制 CPU 调度器 CFS 配额，单位为微秒，最小 1000;
	 + －cpu-rt-period int ：限制 CPU 调度器的实时周期，单位为微秒 ；
	 + －cpu-rt-runtime int ：限制 CPU 调度器的实时运行时，单位为微秒；
	 + －c, -cpu-shares int ： 限制 CPU 使用份额；
	 + －cpus decimal ：限制 CPU 个数；
	 + －cpuset-cpus string ：允许使用的 CPU 核，如 0-3, 0,1;
	 + －cpuset-mems string ：允许使用的内存块，如 0-3, 0,1;
	 + －kernel-memory bytes ：限制使用的内核内存；
	 + －m, -memory bytes ： 限制使用的内存；
	 + －memory-reservation bytes ：内存软限制；
	 + －memory-swap bytes ：内存加上缓存区的限制， － 1 表示为对缓冲区无限制；
	 + －restart stri口g ： 容器退出后的重启策略
	$ docker update --cpus 0 a1cb4017f313
	$ docker update -c 4 a1cb4017f313
	$ docker inspect a1cb4017f313	// 查看容器信息 "CpuShares": 4
```
### 容器与主机间复制文件
> 主机上的t1.txt复制到容器ID为 a1cb4017f313 的/home目录
> $ docker cp t1.txt a1cb4017f313:/home
> * －a, -archive ：打包模式，复制文件会带有原始的 uid/gid 信息；
> * －L, -follow-link ：跟随软连接。当原路径为软连接时＼默认只复制链接信息，使用该选项会复制链接的目标内容 。

### 运行一个 web 应用
```shell
	runoob@runoob:~# docker pull training/webapp  # 载入镜像
	runoob@runoob:~# docker run -d -P training/webapp python app.py
```
  + -d:让容器在后台运行。
  + -P:将容器内部使用的网络端口映射到我们使用的主机上。
   - runoob@runoob:~#  docker ps
   - CONTAINER ID        IMAGE               COMMAND             ...        PORTS                 
   - d3d5e39ed9d3        training/webapp     "python app.py"     ...        0.0.0.0:32769->5000/tcp
  + Docker 开放了 5000 端口（默认 Python Flask 端口）映射到主机端口 32769 上。
  + 这时我们可以通过浏览器访问WEB应用192.168.239.130:32769
可以通过 -p 参数来设置不一样的端口：
```shell
	runoob@runoob:~$ docker run -d -p 5000:5000 training/webapp python app.py
```
### 查看 WEB 应用程序日志
```shell
	runoob@runoob:~$ docker logs -f bf08b7f2cd89		// -f: 让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出。
```
### 查看WEB应用程序容器的进程
```shell
	runoob@runoob:~$ docker top wizardly_chandrasekhar
```
### 查看Docker 容器的配置和状态信息

```shell
	runoob@runoob:~$ docker inspect wizardly_chandrasekhar	// 它会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息。
```

### 停止 WEB 应用容器
```shell
	runoob@runoob:~$ docker stop wizardly_chandrasekhar
```

### 重启WEB应用容器
```shell
	runoob@runoob:~$ docker start wizardly_chandrasekhar
```

### 移除WEB应用容器
```shell
	runoob@runoob:~$ docker rm wizardly_chandrasekhar 		// 删除不需要的容器, 容器必须是停止状态，否则会报错
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
## 镜像使用
### 查找镜像

```shell
	Docker Hub 网址为： https://hub.docker.com/
	$ docker search httpd			// 使用 docker search 命令来搜索镜像
```
 * 搜索官方提供的带 nginx关键字的镜像

```shell
	$ docker search --filter=is-official=true nginx
	NAME                DESCRIPTION                STARS               OFFICIAL            AUTOMATED
	nginx               Official build of Nginx.   13037               [OK]
```
  + NAME: 镜像仓库源的名称
  + DESCRIPTION: 镜像的描述
  + STARS: 类似 Github 里面的 star，表示点赞、喜欢的意思。
  + OFFICIAL: 是否 docker 官方发布
  + AUTOMATED: 自动构建。

 * 搜索所有收藏数超过 4 的关键词包括 tensorow 的镜像

```shell
	$ docker search --filter=stars=200 tensorflow
	NAME                          DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
	tensorflow/tensorflow         Official Docker images for the machine learn…   1662
	jupyter/tensorflow-notebook   Jupyter Notebook Scientific Python Stack w/ …   209
```
* 搜索所有收藏数超过 4 的关键词包括 tensorow 的镜像的前3个镜像

```shell
	$ docker search --filter=stars=4 --limit=3 tensorflow
	NAME                          DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
	tensorflow/tensorflow         Official Docker images for the machine learn…   1662
	jupyter/tensorflow-notebook   Jupyter Notebook Scientific Python Stack w/ …   209
	tensorflow/serving            Official images for TensorFlow Serving (http…   83
```

### 列出镜像列表
```shell
	runoob@runoob:~$ docker images		// 列出本地主机上的镜像
```
  + REPOSITORY：表示镜像的仓库源
  + TAG：镜像的标签
  + IMAGE ID：镜像ID, 如果两个镜像的ID 相同， 说明它们实际上指向了同一个镜像， 只是具有不同标签名称而已, 其中镜像的ID信息十分重要， 它唯一标识了镜像。在使用镜像ID的时候， 一般可以使用该ID的前若干个字符组成的可区分串来替代完整的ID
  + CREATED：镜像创建时间
  + SIZE：镜像大小, 镜像大小信息只是表示了该镜像的逻辑体积大小， 实际上由于相同的镜像层本地只会存储一份， 物理上占用的存储空间会小于各镜像逻辑体积之和
同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本
   - REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   - ubuntu              14.04               90d5884b1ee0        5 days ago          188 MB
   - php                 5.6                 f40e9e0f10c8        9 days ago          444.8 MB
   - ubuntu              15.10               4e3b13c8a266        4 weeks ago         136.3 MB
   - nginx               latest              6f8d099c3adc        12 days ago         182.7 MB

```shell
runoob@runoob:~$ docker run -t -i ubuntu:15.10 /bin/bash 	// 使用版本为15.10的ubuntu系统镜像来运行容器
```

如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像。
 * -q, --quiet式rueI false: 仅输出ID信息， 默认为否

```shell
$ docker images -q=true
273c7fcf9499
0d40868643c6
```
### 获取镜像, 如果不显式指定TAG, 则默认会选择latest标签，这会下载仓库中最新版本的镜像
> 严格地讲，镜像的仓库名称中还应该添加仓库地址（即registry, 注册服务器）作为前缀 ，只是默认使用的是官方DockerHub服务 ，该前缀可以忽略。
> 例如，$ docker pull ubuntu：18.04 命令相当于 $ docker pull registry.hub.docker.com/ubuntu：18.04命令，即从默认的注册服务器DockerHub Registy中的 ubuntu仓库来下载标记为18.04的镜像。
> 如果从非官方的仓库下载，则需要在仓库名称前指定完整的仓库地址。例如从网易蜂巢的镜像源来下载ubuntu:18.04镜像，可以使用如下命令，此时下载的镜像名称为hub.c.163.com/public/ubuntu:18.04: $ docker pull hub.c.163.com/public/ubuntu:18.04
> 可以在Docker服务启动配置中增加 --registry-mirror=proxy_URL来指定镜像代理服务地址（如https://registry.docker-en.com)

```shell
	$ docker pull ubuntu:18.04 // 与下方命令一致,默认使用的是官方DockerHub服务 ，该前缀可以忽略.
	$ docker pull registry.hub.docker.com/ubuntu:18.04
	$ docker pull mysql:5.7
```
> 一般来说， 镜像的latest 标签意味着该镜像的内容会跟踪最新版本的变更而变化, 内容是不稳定的。 因此，从稳定性上考虑，不要在生产环境中忽略镜像的标签信息或使用默认的latest 标记的镜像
> 如果从非官方 的仓库 下载，则 需要在仓库 名称前指定完整的仓库地址

```shell
	$ docker pull hub.c.163.com/public/ubuntu:18.04
```

### 改变标签

```shell
	$ docker tag mysql:5.7 my_mysql:5.7.0
	$ docker images
	REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
	my_mysql                5.7.0               273c7fcf9499        4 days ago          455MB
	mysql                   5.7                 273c7fcf9499        4 days ago          455MB
```

> 它们实际上指向了同一个镜像文件，只是别名不同而巳。docker tag命令添加的标签实际上起到了类似链接的作用

### 查看imgage制作信息

```shell
	$ docker inspect mysql:5.7
	只要其中一项内容时， 可以使用 -f 来指定
	$ docker inspect -f {{".Architecture"}} mysql:5.7
	$  docker inspect -f {{".ContainerConfig"}} mysql:5.7
```

### 查看image历史

```shell
	$ docker history mysql:5.7
```

### 删除镜像
> -f, -force: 强制删除镜像， 即使有容器依赖它
> -no-prune: 不要清理未带标签的父镜像

```shell
	$ docker rmi hello-world
```
> docker rmi 命令只是删除了该镜像多个标签中的指定标签而巳， 并不影响镜像文件

```shell
	$ docker rmi my_mysql:5.7.0
```
> Untagged: my_mysql:5.7.0
> docker rmi 命令来删除只有一个标签的镜像， 可以看出会删除这个镜像文件的所有文件层
> 当使用 docker rmi 命令， 并且后面跟上镜像的 ID (也可以是能进行区分的部分 ID 串前缀）时， 会先尝试删除所有指向该镜像的标签， 然后删除该镜像文件本身
> 当有该镜像创建的容器存在时， 镜像文件默认是无法被删除的, 如果要想强行删除镜像， 可以使用-f参数

```shell
	$ docker rmi -f ubuntu:18.04
```
> 通常并不推荐使用-f参数来强制删除一个存在容器依赖的镜像。 正确的做法是，先删除依赖该镜像的所有容器， 再来删除镜像
> 首先删除容器a21c0840213e:

```shell
	$ docker rm a2lc0840213e
```
> 然后使用ID来删除镜像， 此时会正常打印出删除的各层信息：

```shell
	$ docker rmi Bflbd2lbd25c
```

### 清理镜像

```shell
	$ docker image prune
```
> -a, -all: 删除所有无用镜像， 不光是临时镜像；
> -filter filter: 只清理符合给定过滤器的镜像；
> -f, -force: 强制删除镜像， 而不进行提示确认

### 创建镜像
 * 基于已有容器创建

```shell
	$ docker run -itd --name ubuntu18.04_v3.0 ubuntu:18.04 /bin/bash
	$ docker exec -it ubuntu18.04_v3.0 /bin/bash
	root@e2d52bc5c287:/# cd /home
	root@e2d52bc5c287:/home# mkdir test		// 把初始的image创建一个文件夹再导出成新的image
	root@e2d52bc5c287:/home# exit
	root@alpha:~# docker commit -m "add test file" -a "Docker Newbee" e2d52bc5c287 test:0.1

	$ docker images
	REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
	test                    0.1                 a4850ad0370a        15 seconds ago      64.2MB
	ubuntu                  18.04               4e5021d210f6        4 weeks ago         64.2MB
```
  + -a, --author="": 作者信息
  + -c, - -change=(] : 提交的时候执行 Dockerfle指令， 包括 CMD | ENTRYPOINT | ENV | EXPOSE |LABEL | ONBUILD | USER | VOLUME | WORKIR等
  + -m, - -message=11 11: 提交消息
  + -p, --pause式rue: 提交时暂停容器运行

 * 基于本地模板导入

 * 基于 Docke 「file 创建

### 导出和载入镜像
 * 导出镜像

```shell
	$ docker save -o ubuntu_18.04.tar ubuntu:18.04
```
 * 载入镜像

```shell
	$ docker load -i ubuntu_18.04.tar 或者 docker load < ubuntu_18.04.tar
```

## 重启docker服务

```shell
	$ service docker restart
```
运行以下命令会出错，anyway 运行以上命令就可重启

```shell
	$ systemctl restart docker.service
```
## docker存储路径
查看 docker 存储路径

```shell
	$ docker info |grep 'Docker Root Dir'
	 Docker Root Dir: /var/lib/docker
```
修改 docker 存储路径

```xml
	$ vim /etc/docker/daemon.json
	{
	    "graph": "/home/server/docker"
	}
	$ systemctl daemon-reload
	$ systemctl restart docker
```
