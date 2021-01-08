---
title: 07 Kubernetes Etcd
tags: kubernetes
categories:
- microService
- kubernetes
top:
---

## Etcd
https://github.com/coreos/etcd
> Etcd 是 CoreOS 团队（同时发起了 CoreoS 、 Rocket 等热门项目）发起的一个开源分布式键值仓库项目，可以用于分布式系统中的配置信息管理和服务发现（ service discovery），目前已经被广泛应用到大量开源项目中，包括 Kubernetes 、 CloudFoundry 、 CoreOS Fleet 和Salesforce 等
> Etcd 是 CoreOS 团队于 2013 年 6 月发起的开源项目，它的目标是构建一个高可用的分布式键值（ key-value ）仓库，遵循 Apache v2许可，基于 Go 语言实现
> 分布式系统中最基本的问题之一就是实现信息的共识，在此基础上才能实现对服务配置信息的管理、服务的发现、更新、同步,Etcd 专门为集群环境设计，采用了更为简洁的 Raft 共识算法＠，同样可以实现数据强一.致性，并支持集群节点状态管理和服务自动发现等.
 * 简单：支持 RESTfulAPI 和 gRPCAPI;
 * 安全： 基于 TLS 方式实现安全连接访问 ；
 * 快速： 支持每秒一万次的并发写操作，超时控制在毫秒量级 ；
 * 可靠： 支持分布式结构 ， 基于 Raft 算法实现一致性 
![](etcd_3.PNG)
默认etcd在k8s集群中的 3 or 5 or 7 台node上部署.

## 下载安装

###  二进制文件方式下载

```shell
	$ curl -L https://github.com/coreos/etcd/releases/download/v3.3.1/etcd-v3.3.1-linux-amd64.tar.gz
	$ tar xzvf etcd-v3.3.llinux-amd64.tar.gz
	$ cd etcd-v3.3.llinux-amd64.tar.gz | ls
	其中 etcd 是服务主文件， etcdctl 是提供给用户的命令客户端, 其他都是文档文件.
	$ sudo cp etcd* /usr/local/bin/
```
Etcd 安装到此完成

```shell
	$ etcd --version
```
直接执行 Etcd 命令，将启动一个服务节点，监昕在本地的 2379 （客户端请求端口）和 2380 （其他节点连接端口 ） 

```shell
	$ etcd
```
可以通过 REST API 直接查看集群健康状态：

```shell
	$ curl -L http://127.0.0.1:2379/health
	{”health”: ”true”}
```
也可以使用自带的 etcdctl 命令进行查看（实际上是封装了阻STAPI 调用）：

```shell
	$ etcdctl cluster-health
```
通过 etcdctl 设置和获取键值, 设置键值对 testkey："hello world"

```shell
	$ etcdctl put testkey "hello world111"
```
也可以直接通过 HTTP 访问本地 2379 端口的方式来进行操作，例如查看 test key 的值：

```shell
	$ curl -L -X PUT http://localhost:2379/v2/keys/testkey -d value="hello world"
```

### Docker 镜像方式下载
镜像名称为 quay.io/coreos/etcd:v3.3.1，可以通过下面的命令启动 etcd 服务监听到本地的 2379 和 2380 端口

```shell
	$ docker run -p 2379:2379 -p 2380:2380 -v /etc/ssl/certs/:/etc/ssl/certs/ quay.io/coreos/etcd:v3.3.1
```

## Etcd 集群管理
启动各个节点上的 etcd 服务, 指向主节点etcd存储

### 时钟同步
对于分布式集群来说，各个节点上的同步时钟十分重要， Etcd 集群需要各个节点时钟差异不超过 ls ，否则可能会导致 Raft 协议的异常.

### 节点恢复
> Etcd 集群中的节点会通过数据目录来存放修改信息和集群配置 。
> 一般来说，当某个节点出现故障时候，本地数据已经过期甚至格式破坏. 如果只是简单地重启进程，容易造成数据的不一致。 这个时候，保险的做法是先通过命令（例如 etcdctlmember rm [member ］）来删除该节点，然后清空数据目录，再重新作为空节点加入.
> Etcd 提供了－ strict-reconf ig-check 选项，确保当集群状态不稳定时候（例如启动节点数还不够达到 quorum ）拒绝对配置状态的修改

### 重启集群
> 极端情况下，集群中大部分节点都出现问题，需要重启整个集群.
> 这个时候，最保险的办法是找到一个数据记录完整且比较新的节点，先以它为唯一节点创建新的集群，然后将其他节点一个一个地添加进来，添加过程中注意保证集群的稳定性.


### K8s证书访问etcd server

Enter etcd pod:

```shell
	$ kubectl exec etcd-hci-node01 -n kube-system -i -t – sh
```

Get all key:

```shell
	$ ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only
```

Check the value of the key:

```shell
	$ ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get /registry/statefulsets/kafka/kafka -w=json
```

向etcd数据库添加key,value

```shell
	$ ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key put testkey "hello world111"
```

获取添加的key, value值

```shell
	$ ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get testkey
```








