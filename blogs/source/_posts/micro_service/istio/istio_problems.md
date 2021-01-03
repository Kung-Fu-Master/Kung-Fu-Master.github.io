---
title: Istio problems
tags: istio
categories:
- microService
- istio
---

## istio bookinfo部署不成功
部署istio官网bookinfo出现以下错误

	$ kubectl describe replicaset.apps/details-v1-769468b8c -n book-info
	  Warning  FailedCreate  3s (x13 over 24s)  replicaset-controller  Error creating: Internal error occurred: failed calling webhook "sidecar-injector.istio.io": Post "https://istiod.istio-system.svc:443/inject?timeout=30s": x509: certificate signed by unknown authority
官网推荐解决方法: https://istio.io/latest/docs/ops/common-problems/injection/#automatic-sidecar-injection-fails-if-the-kubernetes-api-server-has-proxy-settings
注释掉  /etc/kubernetes/manifests/kube-apiserver.yaml 文件里的proxy

