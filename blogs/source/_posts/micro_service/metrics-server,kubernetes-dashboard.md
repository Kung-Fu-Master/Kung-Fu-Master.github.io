---
title: Metrics-Server, Kubernetes-Dashboard
tags:
- kubernetes
categories:
- microService
- kubernetes
---

## **Metrics Server**
Metrics Server用于提供核心指标(Core Metrics), 包括 Node, Pod 的 CPU 和 内存 使用指标.  

对其他自定义指标(Custom Metrics)的监控则由 Prometheus 等组件来完成.  


## **Metrics server部署**
Reference Link: 
(官网)https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/
(aws)https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html

下载yaml文件

	wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.1/components.yaml
添加 `--kubelet-insecure-tls` to the components.yaml
Reference Link: https://github.com/kubernetes-sigs/metrics-server/issues/131#issuecomment-418405881

	vim components
	129     spec:
	130       containers:
	131       - args:
	132         - --cert-dir=/tmp
	133         - --secure-port=4443
	134         - --kubelet-insecure-tls
	135         - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
	136         - --kubelet-use-node-status-port
	137         image: k8s.gcr.io/metrics-server/metrics-server:v0.4.1
	138         imagePullPolicy: IfNotPresent
部署

	kubectl apply -f components
	kubectl get deployment metrics-server -n kube-system

## **Kubernetes Dashboard部署**

![](01.JPG)

Reference Link
(官网)https://github.com/kubernetes/dashboard
(aws)https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html
(csdn)https://blog.csdn.net/networken/article/details/85607593

	kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.5/aio/deploy/recommended.yaml
	kubectl get po -n kubernetes-dashboard

### **User Guide**

Accessing Dashboard: https://github.com/kubernetes/dashboard/blob/master/docs/user/accessing-dashboard/README.md#kubectl-port-forward
总共有三种方式, 介绍一种简单的, 其它的Node Port， kube proxy等方式请参考上面链接.

	kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:443

### **创建user**
(官网)https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

	cat > dashboard-adminuser.yaml << EOF
	apiVersion: v1
	kind: ServiceAccount
	metadata:
	  name: admin-user
	  namespace: kubernetes-dashboard
	
	---
	apiVersion: rbac.authorization.k8s.io/v1
	kind: ClusterRoleBinding
	metadata:
	  name: admin-user
	roleRef:
	  apiGroup: rbac.authorization.k8s.io
	  kind: ClusterRole
	  name: cluster-admin
	subjects:
	- kind: ServiceAccount
	  name: admin-user
	  namespace: kubernetes-dashboard  
	EOF
部署 user

	kubectl apply -f dashboard-adminuser.yaml
查看admin-user账户的token

	kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
**说明：** 上面创建了一个叫admin-user的服务账号，并放在kubernetes-dashboard 命名空间下，并将cluster-admin角色绑定到admin-user账户，这样admin-user账户就有了管理员的权限。默认情况下，kubeadm创建集群时已经创建了cluster-admin角色，我们直接绑定即可。  
### **浏览器登陆查看**

	// 一个终端打开浏览器如: firefox
	firefox
	
	// 终端上输入https://localhost:8080
	// 输入上面的admin-user的token登陆即可查看


