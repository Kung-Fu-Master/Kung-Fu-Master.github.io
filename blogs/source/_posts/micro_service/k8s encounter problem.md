---
title: k8s部署遇到的问题
tags:
- kubernetes
categories:
- microService
- kubernetes
---

## 如果机器配置不行导致运行命令很卡, 很容易造成各种问题, 请酌情判断
因为pod 里有配置Readiness probe和Liveness probe, 如果网速不行或机器很卡很容易造成 HTTP probe failed with statuscode: 500等错误.

## **(仍需确认)问题01 failed to get cgroup stats for "/system.slice/kubelet.service"**

通过`kubectl get po -n kube-system` 查看api-server启动异常一直重启
通过`systemctl status kubelet` 查看kubelet服务出现如下类似异常

**failed to get cgroup stats for "/system.slice/kubelet.service"**
**解决方法:** https://github.com/kubernetes/kubernetes/issues/56850
参考最新的：

	# ubuntu
	cat << EOF | sudo tee /etc/systemd/system/kubelet.service.d/12-after-docker.conf
	[Unit]
	After=docker.service
	EOF
	
	# centos
	cat << EOF | sudo tee /usr/lib/systemd/system/kubelet.service.d/12-after-docker.conf
	[Unit]
	After=docker.service
	EOF

	$ systemctl daemon-reload && systemctl restart kubelet // restart kubelet, after adding this file
实际重启kubelet后仍然会报同样的异常, 没有管, 当时也没观察api-server是否正常启动, 过一段时间后发现api-server正常启动.

## **修改kube-apiserver文件后发现某台master的kube-apiserver组件异常**

	$ kubectl logs po/kube-apiserver-master-node -n kube-system
	Error from server: Get "https://10.239.140.137:10250/containerLogs/kube-system/kube-apiserver-master-node/kube-apiserver": dial tcp 10.239.140.137:10250: connect: connection refused
登陆master-node机器查看

	$ netstat -lutpn | grep 6443
	tcp6       0      0 :::6443                 :::*                    LISTEN      14983/kube-apiserve

	$ kubectl logs po/kube-apiserver-master-node -n kube-system
	Flag --insecure-port has been deprecated, This flag will be removed in a future version.
	I1202 04:13:33.729232       1 server.go:625] external host was not specified, using 10.239.140.137
	I1202 04:13:33.729387       1 server.go:163] Version: v1.19.0
	Error: failed to create listener: failed to listen on 0.0.0.0:6443: listen tcp 0.0.0.0:6443: bind: address already in use
解决方法:
Reference Link: https://stackoverflow.com/questions/48734524/kubernetes-api-server-and-controller-manager-cant-start

	$ netstat -lutpn | grep 6443
	tcp6       0      0 :::6443                 :::*                    LISTEN      11395/some-service
	
	$ kill 11395
	
	$ service kubelet restart
实际操作中还删除了kube-apiserver pod, deployment会再重新创建

	kubectl delete po/kube-apiserver-master-node -n kube-system

## **强制删除**

一般删除步骤为：先删pod再删pvc最后删pv.

### **POD强制删除**

	kubectl -n <namespace> delete po <podName> --grace-period=0 --force
### **pv/pvc强制删除**

	kubectl patch pv opspv -p '{"metadata":{"finalizers":null}}'
	kubectl patch pvc opspvc  -p '{"metadata":{"finalizers":null}}' -n kube-ops

## **Node机器重启后重新加入K8s集群**

### **实际观察到如果IP没变, 重启的Master节点等待一段时间会自动加入集群**
但重启的Master节点机器需要保证如下配置重启后也都生效, 否则可以再执行一遍, **实际操作中也是重新执行了一遍**


	systemctl stop kubelet
	systemctl stop docker
	systemctl restart kubelet
	systemctl restart docker
	swapoff -a
	setenforce 0
	systemctl stop firewalld.service
	sysctl net.bridge.bridge-nf-call-iptables=1

### **重新加入Node机器**
重启后如果Node机器未自动加入集群, 运行如下命令重新加入

现在重启的Node机器上执行如下命令reset

	rm -rf /etc/kubernetes/pki/etcd/
	rm -rf /var/lib/etcd
	rm -rf $HOME/.kube
	kubeadm reset
	systemctl stop kubelet
	systemctl stop docker
	systemctl restart kubelet
	systemctl restart docker
	swapoff -a
	setenforce 0
	systemctl stop firewalld.service
	sysctl net.bridge.bridge-nf-call-iptables=1

再在master 机器上执行如下命令获取加入token

	kubeadm token create --print-join-command
	W1202 13:34:46.942799   17329 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
	kubeadm join 10.239.140.201:6443 --token 3vm6e8.wjlspdqpjau62riz     --discovery-token-ca-cert-hash sha256:99f1f55a10e439883030b810be5d3d364d12c508765984cc9ab633db6dbfada9
然后将获取到的token在重启的Node机器上执行

	kubeadm join 10.239.140.201:6443 --token 3vm6e8.wjlspdqpjau62riz     --discovery-token-ca-cert-hash sha256:99f1f55a10e439883030b810be5d3d364d12c508765984cc9ab633db6dbfada9

### **重新加入master机器**

最好把iptables也清理一下 `iptables -F`

去到现有的master节点上生成token

	#生成token
	[root@master2 ~]# kubeadm  token create --print-join-command
	kubeadm join 10.239.140.201:6443 --token 9ks5ps.g0fhcxbzl604k8v0     --discovery-token-ca-cert-hash sha256:2d291498e3c0739c53f33b85c4498fc7ef2ab362926e970671159b4f392d43dc
	
	#生成key
	[root@master2 ~]# kubeadm init phase upload-certs --upload-certs
	W0805 14:41:18.070434   16460 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://storage.googleapis.com/kubernetes-release/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
	W0805 14:41:18.070565   16460 version.go:102] falling back to the local client version: v1.16.2
	[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
	[upload-certs] Using certificate key:
	6314a9877893263374fdf33bedf9a225640a97215784c8c8f387549966d0565d
拿到上述内容之后，拼接；前面的token加上-control-plane --certificate-key ,在要加入的master机器节点上运行，加入集群。

	kubeadm join 172.31.17.49:9443 --token kjjguy.pmqxvb1nmgf1nq4q     --discovery-token-ca-cert-hash sha256:dcadd5b87024c304e5e396ba06d60a4dbf36509a627a6a949c126172e9c61cfb --control-plane --certificate-key 6314a9877893263374fdf33bedf9a225640a97215784c8c8f387549966d0565d


## **机器重启IP改变重新加入集群**
机器重启后重新加入集群

### (不推荐)第一种:
 * 修改/etc/hosts里的机器IP
 * 修改/etc/systemd/system/docker.service.d/no-proxy.conf或proxy.conf里的机器IP
 * 修改~/.bashrc里的export NO_PROXY=***对应的机器IP
 * 修改/etc/kubernetes/manifests/kube-apiserver.yaml的机器IP
 * 修改/etc/kubernetes/manifests/etcd.yaml的机器IP
 ......
以上可见非常麻烦，因此推荐如下方法

### 第二种: 重新reset机器再加入集群
参考本文章上面方法
