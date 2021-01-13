---
title: 使用端口转发(Port Forwarding)来访问集群中的应用
tags:
- kubernetes
categories:
- microService
- kubernetes
---

## port forward
reference: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

<!-- more -->

Usage： 
```
$ kubectl port-forward TYPE/NAME [options] [LOCAL_PORT:]REMOTE_PORT [...[LOCAL_PORT_N:]REMOTE_PORT_N]
```
Flags

| Name | Shorthand| Default | Usage |
| :---: | :---: | :--- | :--- |
| address | <div style="width: 70pt"></div> | <div style="width: 60pt">[localhost]</div> | Addresses to listen on (comma separated).<br/> Only accepts IP addresses or localhost as a value<br/>. When localhost is supplied, kubectl will try to bind on both 127.0.0.1 and ::1 and will fail if neither of these addresses are available to bind. |
| pod-running-timeout | | 1m0s | The length of time (like 5s, 2m, or 3h, higher than zero) to wait until at least one pod is running |


**Listen on ports 5000 and 6000 locally, forwarding data to/from ports 5000 and 6000 in the pod**
```
kubectl port-forward pod/mypod 5000 6000
```

**Listen on ports 5000 and 6000 locally, forwarding data to/from ports 5000 and 6000 in a pod selected by the deployment**
```
kubectl port-forward deployment/mydeployment 5000 6000
```

**Listen on port 8443 locally, forwarding to the targetPort of the service's port named "https" in a pod selected by the service**
```
kubectl port-forward service/myservice 8443:https
```

**Listen on port 8888 locally, forwarding to 5000 in the pod**
```
kubectl port-forward pod/mypod 8888:5000
```

**Listen on port 8888 on all addresses, forwarding to 5000 in the pod**
```
kubectl port-forward --address 0.0.0.0 pod/mypod 8888:5000
```

**Listen on port 8888 on localhost and selected IP, forwarding to 5000 in the pod**
```
kubectl port-forward --address localhost,10.19.21.23 pod/mypod 8888:5000
```

**`Note:`** 如果您没有明确指定 hostIP 和 protocol，Kubernetes 将使用 0.0.0.0 作为默认 hostIP 和 TCP 作为默认 protocol。

**Listen on a random port locally, forwarding to 5000 in the pod**
```
kubectl port-forward pod/mypod :5000
```


## 实例
reference: https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

Create a Deployment that runs Redis:

```
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-master-deployment.yaml
```

转发一个本地端口到 pod 端口
从 Kubernetes v1.10 开始，kubectl port-forward 允许使用资源名称 （例如 pod 名称）来选择匹配的 pod 来进行端口转发。

```
kubectl port-forward redis-master-765d459796-258hz 7000:6379 
```
这相当于

```
kubectl port-forward pods/redis-master-765d459796-258hz 7000:6379
```
或者

```
kubectl port-forward deployment/redis-master 7000:6379 
```
或者

```
kubectl port-forward rs/redis-master 7000:6379
```
或者

```
kubectl port-forward svc/redis-master 7000:redis
```
以上所有命令都应该有效。输出应该类似于：

```
I0710 14:43:38.274550    3655 portforward.go:225] Forwarding from 127.0.0.1:7000 -> 6379
I0710 14:43:38.274797    3655 portforward.go:225] Forwarding from [::1]:7000 -> 6379
```

与本地 7000 端口建立的连接将转发到运行 Redis 服务器的 pod 的 6379 端口。 通过此连接，您可以使用本地工作站来调试在 pod 中运行的数据库。

警告： 由于已知的限制，目前的端口转发仅适用于 TCP 协议。 在 issue 47862 中正在跟踪对 UDP 协议的支持。

Optionally let kubectl choose the local port:
```
kubectl port-forward deployment/redis-master :6379
```


