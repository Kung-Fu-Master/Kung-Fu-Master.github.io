---
title: 01 部署external etcd集群
tags:
- kubernetes
categories:
- microService
- kubernetes
---

## **部署etcd集群**

| 组件 | 使用的证书 |
| :------: | :------: |
| etcd | ca.pem, server.pem, server-key.pem |
| kube-apiserver | ca.pem, server.pem, server-key.pem |
| kubelet | ca.pem, ca-key.pem |
| kube-proxy | ca.pem, kube-proxy.pem, kube-proxy-key.pem |
| kubectl | ca.pem, admin.pem, admin-key.pem |

所有k8s证书,配置,安装包都放到/opt/kubernetes/目录下

```shell
	mkdir -p /opt/kubernetes/{ssl,cfg,bin}
	ls /opt/kubernetes/
	bin/  cfg/  ssl/
```

### 1. 生成ca-key.pem私匙和ca.pem证书

```xml
	cat > ca-config.json << EOF
	{
	    "signing": {
	        "default": {
	            "expiry": "87600h"
	        },
	        "profiles": {
	            "kubernetes": {
	                "expiry": "87600h",
	                "usages": [
	                    "signing",
	                    "key encipherment",
	                    "server auth"
	                ]
	            }
	        }
	    }
	}
	EOF

	cat > ca-csr.json << EOF
	{
	    "CN": "kubernetes",
	    "key": {
	        "algo": "rsa",			// 注意是algo而不是also
	        "size": 3072
	    },
	    "names": [
	        {
	            "C": "CN",			// 哪个国家的可以随便写
	            "L": "Beijing",		// 地点随便写
	            "ST": "Beijing",	// 地点随便写
	            "O": "k8s",			// 用户组, 固定的， 不能随便写
	            "OU": "System"		// 用户, 固定的，不能随便写
	        }
	    ]
	}
	EOF
```

生成ca.pem和ca-key.pem

```shell
	// -bare ca 表示生成以ca开头的证书和key
	cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
	ls ca*
	ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
```
### 2. 生成server端证书和key, 用于k8s的etcd和kube-apiserver


```xml
	cat > server-csr.json << EOF
	{
	    "CN": "kubernetes",
	    "hosts": [			// 包含了哪些机器IP和域名需要此server端证书
	        "127.0.0.1",
	        "10.239.140.133",		// 要使用改证书的etcd或其他服务部署所在节点IP地址或域名
	        "10.239.131.206",
	        "10.239.141.145",
	        "10.239.141.194",
	        "kubernetes.default",
	        "kubernetes.default.svc",
	        "kubernetes.default.svc.cluster",
	        "kubernetes.default.svc.cluster.local"
	    ],
	    "key": {
	        "algo": "rsa",
	        "size": 3072
	    },
	    "names": [
	        {
	            "C": "CN",
	            "L": "Beijing",
	            "ST": "Beijing",
	            "O": "k8s",			//和下面一起代表了用户和组去请求集群
	            "OU": "System"
	        }
	    ]
	}
	EOF
```
生成server端证书

```shell
	// -bare server 表示生成以server开头的证书和key
	cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server
	ls server*
	server.csr  server-csr.json  server-key.pem  server.pem
```

### 3. 生成admin证书, 集群管理员通过证书访问集群


```xml
	cat > admin-csr.json << EOF
	{
	    "CN": "admin",
	    "hosts": [],
	    "key": {
	        "algo": "rsa",
	        "size": 3072
	    },
	    "names": [
	        {
	            "C": "CN",
	            "L": "Beijing",
	            "ST": "Beijing",
	            "O": "system:masters",	// 用户组, 不要改动, 否则会认证失败
	            "OU": "System"			// 用户
	        }
	    ]
	}
	EOF
```

生成管理员证书和key

```shell
	// -bare admin 表示生成以admin开头的证书和key
	cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
	ls admin*
	admin.csr  admin-csr.json  admin-key.pem  admin.pem
```

### 4. 生成kube-proxy证书
工作节点通过kube-proxy组件访问api-server生成一些网络策略, 必须得有权限, 生成一个证书, 让kube-proxy携带证书去访问集群.

```xml
	cat > kube-proxy-csr.json << EOF
	{
	    "CN": "system:kube-proxy",		//固定,不能改变
	    "hosts": [],
	    "key": {
	        "algo": "rsa",
	        "size": 3072
	    },
	    "names": [
	        {
	            "C": "CN",
	            "L": "Beijing",
	            "ST": "Beijing",
	            "O": "k8s",
	            "OU": "System"
	        }
	    ]
	}
	EOF
```
生成kube-proxy证书和key

```shell
	cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
	ls kube-proxy*
	kube-proxy.csr  kube-proxy-csr.json  kube-proxy-key.pem  kube-proxy.pem
```

### 5. 只保留*.pem, 删除其它文件

```shell
	ls | grep -v pem | xargs -i rm {}
	ls
	admin-key.pem  admin.pem  ca-key.pem  ca.pem  kube-proxy-key.pem  kube-proxy.pem  server-key.pem  server.pem
```

## **关闭防火墙或开发端口**
### 关闭防火墙

```shell
	setenforce 0
	systemctl stop firewalld.service
	sysctl net.bridge.bridge-nf-call-iptables=1
```

### 如果使用firewalld作为防火墙，则需要开放端口

```shell
	firewall-cmd --zone=public --add-port=2379/tcp --permanent
	firewall-cmd --zone=public --add-port=2380/tcp --permanent
	firewall-cmd --reload
	firewall-cmd --list-all
```

## **etcd安装**
etcd是由coreos公司开发在GitHub上开源的存储键值对的数据库

### etcd 下载
本次测试安装etcd的3.2.12版本
https://github.com/etcd-io/etcd/releases/tag/v3.2.12
下载解压并移动到指定目录

```shell
	wget -L https://github.com/etcd-io/etcd/releases/download/v3.2.12/etcd-v3.2.12-linux-amd64.tar.gz
	tar -zxvf etcd-v3.2.12-linux-amd64.tar.gz
	ls etcd-v3.2.12-linux-amd64
	Documentation/  etcd  etcdctl  README-etcdctl.md  README.md  READMEv2-etcdctl.md
	// 将解压后文件中的可执行文件etct和etcdctl移动到/opt/kubernetes/bin/目录下
	mv etcd-v3.2.12-linux-amd64/etcd /opt/kubernetes/bin/
	mv etcd-v3.2.12-linux-amd64/etcdctl /opt/kubernetes/bin/
```

### 创建ETCD的配置文件

```
	cat > /opt/kubernetes/cfg/etcd << EOF
	#[Member]
	ETCD_NAME="etcd01"
	ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
	ETCD_LISTEN_PEER_URLS="https://10.239.140.133:2380"
	ETCD_LISTEN_CLIENT_URLS="https://10.239.133:2379"
	
	#[Clustering]
	ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.239.140.133:2380"
	ETCD_ADVERTISE_CLIENT_URLS="https://10.239.140.133:2379"
	ETCD_INITIAL_CLUSTER="etcd01=https://10.239.140.133:2380,etcd02=https://10.239.131.206:2380,etcd03=https://10.239.141.145:2380"
	ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
	ETCD_INITIAL_CLUSTER_STATE="new"
	EOF
```
### 使用systemd来管理etcd服务

```xml
	cat > /usr/lib/systemd/system/etcd.service << EOF
	[Unit]					//systemd依赖的一些服务, 网络服务启动之后再启动此服务
	Description=Etcd Server
	After=network.target
	After=network-online.target
	Wants=network-online.target
	
	[Service]
	Type=notify
	WorkingDirectory=/var/lib/etcd/					//看网上配置有这个参数, 自己配置过程中没有加入
	EnvironmentFile=-/opt/kubernetes/cfg/etcd		//指定服务启动配置文件
	ExecStart=/opt/kubernetes/bin/etcd \			//服务启动选项
	--name=${ETCD_NAME} \
	--data-dir=${ETCD_DATA_DIR} \
	--listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
	--listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
	--advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
	--initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
	--initial-cluster=${ETCD_INITIAL_CLUSTER} \
	--initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
	--initial-cluster-state=new \
	--cert-file=/opt/kubernetes/ssl/server.pem \		// 指定数字证书
	--key-file=/opt/kubernetes/ssl/server-key.pem \
	--peer-cert-file=/opt/kubernetes/ssl/server.pem \
	--peer-key-file=/opt/kubernetes/ssl/server-key.pem \
	--trusted-ca-file=/opt/kubernetes/ssl/ca.pem \
	--peer-trusted-ca-file=/opt/kubernetes/ssl/ca.pem
	Restart=on-failure
	LimitNOFILE=65536
	
	[Install]
	WantedBy=multi-user.target
	EOF
```

启动etcd服务

```shell
	// 启动etcd服务发现卡住, 且过一会会报个错误, 原因是另外request另外两台etcd服务2380得不到响应
	// 待另外至少一台etcd服务启动后再回来查看etcd服务状态就正常了
	systemctl daemon-reload
	systemctl start etcd
	systemctl enable etcd
	// 完整的启动etcd服务命令其实就是运行以下命令:
	/opt/kubernetes/bin/etcd --name="etcd01" --data-dir="/var/lib/etcd/default.etcd" --listen-peer-urls="https://10.239.140.133:2380" --listen-client-urls="https://10.239.140.133:2379,http://127.0.0.1:2379" --advertise-client-urls="https://10.239.140.133:2379" --initial-advertise-peer-urls="https://10.239.140.133:2380" --initial-cluster="etcd01=https://10.239.140.133:2380,etcd02=https://10.239.131.206:2380,etcd03=https://10.239.141.145:2380" --initial-cluster-token="etcd-cluster" --initial-cluster-state="new" --cert-file=/opt/kubernetes/ssl/server.pem --key-file=/opt/kubernetes/ssl/server-key.pem --peer-cert-file=/opt/kubernetes/ssl/server.pem --peer-key-file=/opt/kubernetes/ssl/server-key.pem --trusted-ca-file=/opt/kubernetes/ssl/ca.pem --peer-trusted-ca-file=/opt/kubernetes/ssl/ca.pem
```

查看服务

```shell
	ps -ef | grep etcd 
	root     23726     1  5 21:41 ?        00:00:00 /opt/kubernetes/bin/etcd --name=etcd01 --data-dir=/var/lib/etcd/default.etcd --listen-peer-urls=https://10.239.140.133:2380 --listen-client-urls=https://10.239.140.133:2379,http://127.0.0.1:2379 --advertise-client-urls=https://10.239.140.133:2379 --initial-advertise-peer-urls=https://10.239.140.133:2380 --initial-cluster=etcd01=https://10.239.140.133:2380,etcd02=https://10.239.131.206:2380,etcd03=https://10.239.141.145:2380 --initial-cluster-token=etcd-cluster --initial-cluster-state=new --cert-file=/opt/kubernetes/ssl/server.pem --key-file=/opt/kubernetes/ssl/server-key.pem --peer-cert-file=/opt/kubernetes/ssl/server.pem --peer-key-file=/opt/kubernetes/ssl/server-key.pem --trusted-ca-file=/opt/kubernetes/ssl/ca.pem --peer-trusted-ca-file=/opt/kubernetes/ssl/ca.pem
	root     23744 14957  0 21:41 pts/1    00:00:00 grep --color=auto etcd
	// 查看etcd服务状态和日志发现etcd一直尝试request其它两台机器, 状态不正常, 在其它两台也部署etcd后就可以了
	systemctl status etcd
	tail /var/log/messages
```

### etcd拷贝到其它机器

```shell
	scp -r /opt/kubernetes root@10.239.131.206:/opt
	scp /usr/lib/systemd/system/etcd.service  root@10.239.131.206:/usr/lib/systemd/system/etcd.service
	// 只需要修改其它机器上/opt/kubernetes/cfg/etcd文件里的ETCD_NAME和其它参数IP地址然后就可以直接运行命令启动etcd服务.
	// 并且启动后不会卡住
	systemctl start etcd
	systemctl enable etcd
```

### 测试etcd集群状态是否正常

```shell
	/opt/kubernetes/bin/etcdctl --ca-file=/opt/kubernetes/ssl/ca.pem --cert-file=/opt/kubernetes/ssl/server.pem --key-file=/opt/kubernetes/ssl/server-key.pem --endpoints="https://10.239.140.133:2379,https://10.239.131.206:2379,https://10.239.141.145:2379" cluster-health
	member 723f8ab932b4c3f6 is healthy: got healthy result from https://10.239.141.145:2379
	member 8af8f5fa5f0b0b39 is healthy: got healthy result from https://10.239.131.206:2379
	member 91dd39fb14e3de97 is healthy: got healthy result from https://10.239.140.133:2379
	cluster is healthy
```

## 移除etcd集群
分别登陆各台部署etcd的机器上执行如下命令:

```shell
	systemctl stop etcd
	systemctl disable etcd
	rm -rf /usr/lib/systemd/system/etcd.service
```

## **遇到的问题**
查看Centos7系统日子

```shell
	cat /var/log/messages
	systemctl status etcd.service
```

出现以下错误:

etcd.service: main process exited, code=exited, status=2/INVALIDARGUMENT

很明显是运行etcd命令时候的参数错误, 改对参数就可以了.


