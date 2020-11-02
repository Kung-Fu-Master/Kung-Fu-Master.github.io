---
title: helm
tags:
categories:
- linux
---

## Helm

[Helm offical installation tutorial](https://helm.sh/docs/intro/install/)
Manually download and install the Helm release version: [release](https://github.com/helm/helm/releases)

	$ curl -O https://get.helm.sh/helm-v3.3.1-linux-amd64.tar.gz
	$ tar -zxvf helm-v3.3.1-linux-amd64.tar.gz
	$ mv linux-amd64/helm /usr/local/bin/helm

From there, you should be able to run the client and add the stable repo: `helm help`



## Trobleshooting

### User "system:serviceaccount:kube-system:default" cannot get resource "namespaces" in API group "" in the namespace "greenplum"
Reference Link: https://github.com/fnproject/fn-helm/issues/21

	$ kubectl create serviceaccount --namespace kube-system tiller
	$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
	// 安装tiller
	$ helm init --service-account tiller --skip-refresh
	$ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'


