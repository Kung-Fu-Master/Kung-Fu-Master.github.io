---
title: k8s_对象名称和IDs
tags: istio
categories:
- microService
- kubernetes
---

## 对象名称和 IDs

集群中的每一个对象都有一个名称 来标识在同类资源中的唯一性。

<!-- more -->

每个 Kubernetes 对象也有一个UID 来标识在整个集群中的唯一性。

比如，在同一个名字空间 中有一个名为 `myapp-1234` 的 Pod, 但是可以命名一个 Pod 和一个 Deployment 同为 `myapp-1234`.

对于用户提供的非唯一性的属性，Kubernetes 提供了 标签（Labels）和 注解（Annotation）机制。

## 名称

客户端提供的字符串，引用资源 url 中的对象，如/api/v1/pods/some name。

某一时刻，只能有一个给定类型的对象具有给定的名称。但是，如果删除该对象，则可以创建同名的新对象。

以下是比较常见的三种资源命名约束。

### DNS 子域名

很多资源类型需要可以用作 DNS 子域名的名称。 DNS 子域名的定义可参见 RFC 1123。 这一要求意味着名称必须满足如下规则：

 * 不能超过253个字符
 * 只能包含字母数字，以及'-' 和 '.'
 * 须以字母数字开头
 * 须以字母数字结尾

### DNS 标签名 

某些资源类型需要其名称遵循 RFC 1123 所定义的 DNS 标签标准。也就是命名必须满足如下规则：

 * 最多63个字符
 * 只能包含字母数字，以及'-'
 * 须以字母数字开头
 * 须以字母数字结尾

### 路径分段名称
某些资源类型要求名称能被安全地用作路径中的片段。 换句话说，其名称不能是 `.` 、`..`，也不可以包含 `/` 或 `%` 这些字符。

下面是一个名为nginx-demo的 Pod 的配置清单：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

说明： 某些资源类型可能具有额外的命名约束。


## UIDs

Kubernetes 系统生成的字符串，唯一标识对象。

在 Kubernetes 集群的整个生命周期中创建的每个对象都有一个不同的 uid，它旨在区分类似实体的历史事件。

Kubernetes UIDs 是全局唯一标识符（也叫 UUIDs）。 UUIDs 是标准化的，见 ISO/IEC 9834-8 和 ITU-T X.667.


