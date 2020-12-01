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

	[root@k8s-master pki]# kubeadm token create --print-join-command
	kubeadm join 192.168.180.121:6443 --token p08kmc.nci5h0xfmlcw92vg     --discovery-token-ca-cert-hash sha256:44315d59e08f4d94bc75d20730b861818dfeda6517c1b228399f061f4256329b

子节点重新加入

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

查看nodes

	[root@k8s-master pki]# kubectl get nodes
	NAME         STATUS   ROLES    AGE     VERSION
	k8s-master   Ready    master   3d4h    v1.16.1
	k8s-node01   Ready    <none>   2m35s   v1.16.1
	k8s-node02   Ready    <none>   3d3h    v1.16.1
	[root@k8s-master pki]#
	

