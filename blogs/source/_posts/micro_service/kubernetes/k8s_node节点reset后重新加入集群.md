---
title: k8s node 节点reset后重新加入集群
tags:
- kubernetes
categories:
- microService
- kubernetes
---

## node节点(重新)加入集群
加入新的node节点或在原来node节点不小心执行 `kubectl reset` 命令后都可用下面命令重新加入集群.  

登陆master机器, 重新获取Token

```shell
	[root@k8s-master pki]# kubeadm token create --print-join-command
	kubeadm join 192.168.180.121:6443 --token p08kmc.nci5h0xfmlcw92vg     --discovery-token-ca-cert-hash sha256:44315d59e08f4d94bc75d20730b861818dfeda6517c1b228399f061f4256329b
```

子节点重新加入

```shell
	[root@k8s-node01 lib]# kubeadm join 192.168.180.121:6443 --token p08kmc.nci5h0xfmlcw92vg     --discovery-token-ca-cert-hash sha256:44315d59e08f4d94bc75d20730b861818dfeda6517c1b228399f061f4256329b
	[preflight] Running pre-flight checks
	        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
	[preflight] Reading configuration from the cluster...
	[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
	[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
	[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
	[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
	[kubelet-start] Activating the kubelet service
	[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
	
	This node has joined the cluster:
	* Certificate signing request was sent to apiserver and a response was received.
	* The Kubelet was informed of the new secure connection details.
	
	Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

查看nodes

```shell
	[root@k8s-master pki]# kubectl get nodes
	NAME         STATUS   ROLES    AGE     VERSION
	k8s-master   Ready    master   3d4h    v1.16.1
	k8s-node01   Ready    <none>   2m35s   v1.16.1
	k8s-node02   Ready    <none>   3d3h    v1.16.1
	[root@k8s-master pki]#
```

## master节点重新加入集群

最好把iptables也清理一下 iptables -F

去到现有的master节点上生成token

### 生成token

```shell
	[root@master2 ~]# kubeadm  token create --print-join-command
	kubeadm join 10.239.140.201:6443 --token 9ks5ps.g0fhcxbzl604k8v0     --discovery-token-ca-cert-hash sha256:2d291498e3c0739c53f33b85c4498fc7ef2ab362926e970671159b4f392d43dc
```

### 生成key

```shell
	[root@master2 ~]# kubeadm init phase upload-certs --upload-certs
	W0805 14:41:18.070434   16460 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://storage.googleapis.com/kubernetes-release/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
	W0805 14:41:18.070565   16460 version.go:102] falling back to the local client version: v1.16.2
	[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
	[upload-certs] Using certificate key:
	6314a9877893263374fdf33bedf9a225640a97215784c8c8f387549966d0565d
```

拿到上述内容之后，拼接；前面的token加上-control-plane –certificate-key ,在要加入的master机器节点上运行，加入集群。

```shell
	kubeadm join 172.31.17.49:9443 --token kjjguy.pmqxvb1nmgf1nq4q --discovery-token-ca-cert-hash sha256:dcadd5b87024c304e5e396ba06d60a4dbf36509a627a6a949c126172e9c61cfb --control-plane --certificate-key 6314a9877893263374fdf33bedf9a225640a97215784c8c8f387549966d0565d
```


