---
title: docker
tags: 
categories:
- linux
---

## docker保存/lodad镜像
复制镜像和复制容器都是通过保存为新镜像而进行的。
具体为：
保存镜像

```shell
docker save ID > xxx.tar
docker load < xxx.tar
docker load image-name:new < image.tar
docker load -i image.tar
```

//load期间使用$df -hl可以查看最下面docker绑定的根分区大小，如果image.tar要大于根分区大小则load会报空间不足的错误，解决方法下面有解.
保存容器

```shell
docker export ID >xxx.tar
docker import xxx.tar container:v1
```

然后再`docker run -it container:v1 bash`

```shell
	docker run --name faiss-gcc-9.1 -it faiss-images-gcc-9.1.0:1.0 /bin/bash  //用这个
	exit
	docker start faiss-gcc-9.1
	docker attach faiss-gcc-9.1
```

软件镜像（如 weChat.exe）----> 运行镜像----> 产生一个容器（正在运行的软件，运行的 微信程序）；
操作	命令	说明
运行
```
docker run --name container-name -d image-name:tag /bin/bash
	运行一个名为 container-name的容器,并在容器里运行/bin/bash,如果不指定运行/bin/bash则docker启动后就立即退出，这跟docker机制有关，docker是后台运行，必须有前台进程，没有前台程序就退出
	#退出
	exit
	#关闭
	docker stop mycentos
	#重启
	1.docker start mycentos
	#重启后,在用mycentos再打开/bin/bash
	2.docker exec -ti mycentos /bin/bash    //1和2两步是合起来用的
```
如:docker run --name myredis –d redis	 /bin/bash
--name：自定义容器名
-d：表示后台运行
image-name:指定运行的镜像名称

tag:镜像的版本

列表	docker ps（查看运行中的容器）；	加上-a；可以查看所有容器
停止	docker stop container-name/container-id	停止当前运行的指定容器
启动	docker start container-name/container-id	启动容器
删除	docker rm container-id	删除指定容器
端口映射	-p 6379:6379
如:docker run  --name myredis  -d -p 6379:6379 docker.io/redis	
-p:主机端口映射到容器内部的端口

容器日志	docker logs container-name/container-id	 
官网可查看更多命令：https://docs.docker.com/engine/reference/commandline/docker/

删除Images

```shell
$ docker rm      --   Remove one or more containers
$ docker rmi     --   Remove one or more images
```

想要删除运行过的images必须首先删除它的container。继续来看刚才的例子，

```shell
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                   NAMES
117843ade696        ed9c93747fe1        /bin/sh -c /usr/sbin   46 hours ago        Up 46 hours         0.0.0.0:49153->22/tcp   test_sshd
```

可以看出ed9c93747fe1的image被117843ade696的container使用着，所以必须首先删除该container
```shell
$ docker rm 117843ade696
Error: container_delete: Impossible to remove a running container, please stop it first
2014/03/22 16:36:44 Error: failed to remove one or more containers
出现错误，这是因为该container正在运行中(运行docker ps查看)，先将其关闭
```

```shell
$ docker stop 117843ade696
117843ade696
$ docker rm 117843ade696
117843ade696
$ docker rmi ed9c93747fe1
Deleted: ed9c93747fe16627be822ad3f7feeb8b4468200e5357877d3046aa83cc44c6af
Deleted: c8a0c19429daf73074040a14e527ad5734e70363c644f18c6815388b63eedc9b
Deleted: 95dba4c468f0e53e5f1e5d76b8581d6740aab9f59141f783f8e263ccd7cf2a8e
Deleted: c25dc743e40af6858c34375d450851bd606a70ace5d04e231a7fcc6d2ea23cc1
Deleted: 20562f5714a5ce764845119399ef75e652e23135cd5c54265ff8218b61ccbd33
Deleted: c8af1dc23af7a7aea0c25ba9b28bdee68caa8866f056e4f2aa2a5fa1bcb12693
Deleted: 38fdb2c5432e08ec6121f8dbb17e1fde17d5db4c1f149a9b702785dbf7b0f3be
Deleted: 79ca14274c80ac1df1333b89b2a41c0e0e3b91cd1b267b31bef852ceab3b2044
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
CentOS65            latest              e55a74a32125        2 days ago          360.6 MB
```

可以看出，image已经被删除。
From <https://blog.csdn.net/flydreamzhll/article/details/80900509> 

## Docker 拷贝

将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。
```shell
docker cp /www/runoob 96f7f14e99ab:/www/
```

将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。
```shell
docker cp /www/runoob 96f7f14e99ab:/www
```

将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。
```shell
docker cp  96f7f14e99ab:/www /tmp/
```

```shell
主机copy到docker
$ docker cp /opt/test/file.txt mycontainer_id：/opt/testnew/
docker文件copy到主机
$ docker cp mycontainer_id：/opt/testnew/file.txt /opt/test/
```

From <http://www.runoob.com/docker/docker-cp-command.html> 

## 更新images

docker commit Container-id Images-name 将在某个image a 上做改动的新的container更新为最新的image 

```shell
$ docker commit a981e981ef65 faiss-images
```

## Docker run

```shell
$ which python
/usr/bin/python
$ env

$ vim test.sh
	#!/bin/bash
	export MKLROOT=/opt/intel/compilers_and_libraries/linux/mkl/
	export LD_PRELOAD=/opt/intel/compilers_and_libraries/linux/mkl//lib/intel64/libmkl_core.so:/opt/intel/compilers_and_libraries/linux/mkl//lib/intel64/libmkl_sequential.so

	cd /test_faiss
	python /test_faiss/test_sift1M.py

$ docker run -t image-name /bin/bash /test_faiss/test.sh
				此时image已是在最新container上更新过的image
   Vtune 绑定docker运行
$ amplxe-cl -collect hotspots -r test_hot docker run -t faiss-images /bin/bash /test_faiss/test.sh
		     		     保存目录
$ amplxe-cl -collect hotspots -r test_hot_1 ls
```

## 设置镜像标签
我们可以使用 docker tag 命令，为镜像添加一个新的标签。
```shell
$ docker tag 860c279d2fec runoob/centos:dev
```

docker tag 镜像ID，这里是 860c279d2fec ,用户名称、镜像源名(repository name)和新的标签名(tag)。
使用 docker images 命令可以看到，ID为860c279d2fec的镜像多一个标签。
``` shell
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
runoob/centos       6.7                 860c279d2fec        5 hours ago         190.6 MB
runoob/centos       dev                860c279d2fec        5 hours ago         190.6 MB
runoob/ubuntu       v2                  70bf1840fd7c        22 hours ago        158.5 MB
ubuntu              14.04               90d5884b1ee0        6 days ago          188 MB
php                 5.6                 f40e9e0f10c8        10 days ago         444.8 MB
nginx               latest              6f8d099c3adc        13 days ago         182.7 MB
mysql               5.6                 f2e8d6c772c0        3 weeks ago         324.6 MB
httpd               latest              02ef73cf1bc0        3 weeks ago         194.4 MB
ubuntu              15.10               4e3b13c8a266        5 weeks ago         136.3 MB
hello-world         latest              690ed74de00f        6 months ago        960 B
centos              6.7                 d95b5ca17cc3        6 months ago        190.6 MB
training/webapp     latest              6fae60ef3446        12 months ago       348.8 MB
```

## Docker修改空间大小
 * 第一种：
```
$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.12.6
Storage Driver: devicemapper
 Pool Name: docker-253:0-67125080-pool
 Pool Blocksize: 65.54 kB
 Base Device Size: 10.74 GB#这个就是限制，容器根分区的大小！
 Backing Filesystem: xfs
 Data file: /dev/loop0
 Metadata file: /dev/loop1
 Data Space Used: 11.8 MB
 Data Space Total: 107.4 GB
 Data Space Available: 36.95 GB
 Metadata Space Used: 581.6 kB
 Metadata Space Total: 2.147 GB
 Metadata Space Available: 2.147 GB
 Thin Pool Minimum Free Space: 10.74 GB
 Udev Sync Supported: true
 Deferred Removal Enabled: false
 Deferred Deletion Enabled: false
 Deferred Deleted Device Count: 0
docker的版本是1.12,修改容器根分区的大小即可：

  dm.loopdatasize=100G是指存放数据的数据库空间为100g，默认是100g
dm.loopmetadatasize=10G是存放Metadata数据空间为10g，默认是2g
dm.fs=xft是指容器磁盘分区为xft
dm.basesize=20G是指容器根分区默认为20g，默认是10g
```
```shell
$ vi /etc/sysconfig/docker-storage
```
修改下面参数即可
```
DOCKER_STORAGE_OPTIONS="--storage-driver devicemapper --storage-opt dm.loopdatasize=200G --storage-opt dm.loopmetadatasize=10G -g /dev/docker/ --storage-opt dm.fs=xfs --storage-opt dm.basesize=30G"
```
利用-g参数即可指定存储挂载路径。比如，示例中的配置将存储目录挂载在/data/docker/路径下
重新挂载新的路径后原来路径下的images和container都找不到，-g参数去掉回到默认挂载路径再按照下面重启docker服务会发现原来的images和container都存在.

最后重启容器，问题解决

```shell
// 停止docker服务的命令如下:
systemctl stop docker
// 重新启动：
systemctl daemon-reload && systemctl start docker
```

 * 第二种： //试了试不怎么灵
Docker默认空间大小分为两个，一个是池空间大小，另一个是容器空间大小。
池空间大小默认为：100G
容器空间大小默认为是：10G
所以修改空间大小也分为两个：
这里使用centos下的yum进行安装的Docker。
 
首先，修改空间大小，必需使Docker运行在daemon环境下，即先停止正在运行的docker服务：
service docker stop
然后使用命令使用daemon环境下运行docker：
docker -d          //可以不需要这条
一、修改池空间大小方法：
dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/data bs=1G count=0 seek=1000
dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/metadata bs=1G count=0 seek=10
上面的1000为1TB大小，即为数据池空间大小为1TB，而10则为Metadata的空间大小，10GB
从运行完后，启动docker service
service docker start
使用命令查看docker池空间大小：
docker info

可以看到池空间已经被设置为data=1TB和metadata=10GB

## 关于Docker目录挂载的总结
Docker容器启动的时候，如果要挂载宿主机的一个目录，可以用-v参数指定。
譬如我要启动一个centos容器，宿主机的/test目录挂载到容器的/soft目录，可通过以下方式指定：

```shell
$ docker run -it  --privileged=true -v /test:/soft  --name ContainerName ImagesName /bin/bash
```

这样在容器启动后，容器内会自动创建/soft的目录。通过这种方式，我们可以明确一点，即-v参数中，冒号":"前面的目录是宿主机目录，后面的目录是容器内目录。
挂载宿主机已存在目录后，在容器内对其进行操作，报“Permission denied”。
可通过两种方式解决：
1> 关闭selinux。
临时关闭：# setenforce 0
永久关闭：修改/etc/sysconfig/selinux文件，将SELINUX的值设置为disabled。
2> 以特权方式启动容器 
指定--privileged参数
如：# docker run -it --privileged=true -v /test:/soft centos /bin/bash

## 容器的特权模式运行

--privileged 可以启动docker的特权模式，这种模式允许我们以其宿主机具有（几乎）所有能力来运行容器，包括一些内核特性和设备访问。

这是让我们可以在dokcer中运行docker的必要参数

让docker运行在--privileged特权模式会有一些安全风险。这种模式下运行容器对docker宿主机拥有root访问权限。

