---
title: helm安装
tags:
categories:
- linux
---

## Helm简介

Helm 是一个由 CNCF 孵化和管理的项目, 用于对需要在Kubernetes上部署的复杂应用进行定义, 安装和更新.  

Helm 由 **HelmClient** 和 **TillerServer** 两个组件完成.

**HelmClient** 是一个客户端.  
**TillerServer** 负责客户端指令和 Kubernetes 集群之间的交互, 根据 Chart 定义, 生成和管理各种 Kubernetes 的资源对象.  

## Helm 安装

[Helm offical installation tutorial](https://helm.sh/docs/intro/install/)
Manually download and install the Helm release version: [release](https://github.com/helm/helm/releases)

	$ curl -O https://get.helm.sh/helm-v3.3.1-linux-amd64.tar.gz
	$ tar -zxvf helm-v3.3.1-linux-amd64.tar.gz
	$ mv linux-amd64/helm /usr/local/bin/helm

From there, you should be able to run the client and add the stable repo: `helm help`


## Tiller Server 安装

Tiller Server 安装推荐使用helm init命令进行.  

这一命令会在kubectl当前context指定集群内的 kube-system 命名空间创建一个 Deployment 和一个 Service, 运行TillerServer服务.  


## Trobleshooting

### User "system:serviceaccount:kube-system:default" cannot get resource "namespaces" in API group "" in the namespace "greenplum"
Reference Link: https://github.com/fnproject/fn-helm/issues/21

	$ kubectl create serviceaccount --namespace kube-system tiller
	$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
	// 安装tiller
	$ helm init --service-account tiller --skip-refresh
	$ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'


