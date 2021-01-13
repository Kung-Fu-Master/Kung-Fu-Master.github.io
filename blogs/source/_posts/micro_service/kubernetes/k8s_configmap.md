---
title: k8s configmap
tags: istio
categories:
- microService
- kubernetes
---

## configmap

<!-- overview -->

ConfigMap 并不提供保密或者加密功能。
如果你想存储的数据是机密的，请使用 [secret](https://kubernetes.io/zh/docs/concepts/configuration/secret/),
或者使用其他第三方工具来保证你的数据的私密性，而不是用 ConfigMap。

<!-- more -->

## 创建configmap
查看宿主机配置文件

```shell
	$ ls ./configmap/   //包含两个配置文件
	application.properties  ui.properties

	$ vim ./configmap/ui.properties // 配置文件里都是键值对
	color.good=purple
	color.bad=yellow
	allow.textmode=true
	how.nice.to.look=fairlyNice
```

1. 将宿主机`./configmap/`目录下的所有配置文件打包到configmap中.


```shell
	kubectl create configmap myconfigmap -n test --from-file=./configmap/
```

## configmap在pod中使用
pod.yaml

```yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  namespace: test
	  name: test-configmap
	spec:
	  containers:
	    - name: test-configmap
	      image: k8s.gcr.io/busybox
	      command: ["/bin/sh", "-c", "ls /etc/config"]
	      //command: ["/bin/sh", "-c", "for ((i=0; i<100; i++)); do  cat /etc/config/ui.properties | head -n 1; sleep 1; done"]
	      volumeMounts:
	      - name: config-volume
	        mountPath: /etc/config
	  volumes:
	    - name: config-volume
	      configMap:
	        name: myconfigmap
	  restartPolicy: Never
```

查看日志

```shell
	$ kubectl logs po/test-configmap -n test
	application.properties
	ui.properties
```

发现在pod的/etc/config目录有两个配置文件可供pod里进程读取

## 运行时修改configmap内容
修改 ui.properties 在configmap中的key和value值

```shell
	$ kubectl edit configmap myconfigmap -n test
	color.good=purple --改为--> color.favorite=blue
```
登陆pod查看mount到pod 路径/etc/config/ui.properties文件中的值变化
**`NOTE: `** 修改configmap中键值对后, pod中对应mount的内容会`过一会才会变化`.

```shell
	$ kubectl exec po/test-configmap -n test -it -- /bin/sh
	// 等一会时间再查看
	$ cat /etc/config/ui.properties
	color.favorite=blue
	color.bad=yellow
	allow.textmode=true
	how.nice.to.look=fairlyNice
```

## 不可变更的 ConfigMap

Kubernetes Beta 特性 _不可变更的 Secret 和 ConfigMap 提供了一种将各个
Secret 和 ConfigMap 设置为不可变更的选项。对于大量使用 ConfigMap 的
集群（至少有数万个各不相同的 ConfigMap 给 Pod 挂载）而言，禁止更改
ConfigMap 的数据有以下好处：

- 保护应用，使之免受意外（不想要的）更新所带来的负面影响。
- 通过大幅降低对 kube-apiserver 的压力提升集群性能，这是因为系统会关闭
  对已标记为不可变更的 ConfigMap 的监视操作。

此功能特性由 `ImmutableEphemeralVolumes`
[特性门控](/zh/docs/reference/command-line-tools-reference/feature-gates/)
来控制。你可以通过将 `immutable` 字段设置为 `true` 创建不可变更的 ConfigMap。
例如：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  ...
data:
  ...
immutable: true
```

一旦某 ConfigMap 被标记为不可变更，则 _无法_ 逆转这一变化，，也无法更改
`data` 或 `binaryData` 字段的内容。你只能删除并重建 ConfigMap。
因为现有的 Pod 会维护一个对已删除的 ConfigMap 的挂载点，建议重新创建
这些 Pods。




