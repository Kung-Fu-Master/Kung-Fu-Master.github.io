---
title: istio authorization
tags: istio
categories:
- microService
- istio
---


## istio authorization

Reference
概念: https://istio.io/latest/docs/concepts/security/#authorization
测试用例: https://istio.io/latest/docs/tasks/security/authorization/authz-http/
参数参考: https://istio.io/latest/docs/reference/config/security/authorization-policy/

<!-- more -->

## TCP authorization实例

**`注意:`**
 - 测试authorization用例之前要先enable mtls
 - `k8s deployment` 资源要添加 `service account` 如: **`serviceAccountName: bookinfo-productpage`**
 - istio一些策略如： **`rules.to.operation.hosts/methods/paths`** 只能用在 HTTP authorization中

### **deployment**

```
apiVersion: v1
kind: Service
metadata:
  name: productpage
  labels:
    app: productpage
    service: productpage
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: productpage
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-productpage
  labels:
    account: productpage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage-v1
  labels:
    app: productpage
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: productpage
      version: v1
  template:
    metadata:
      labels:
        app: productpage
        version: v1
    spec:
      serviceAccountName: bookinfo-productpage
      containers:
      - name: productpage
        image: docker.io/istio/examples-bookinfo-productpage-v1:1.16.2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        emptyDir: {}
```

### **authentication**

authentication_namespace.yaml

```
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "query-authentication-policy"
  namespace: "query"
spec:
  mtls:
    mode: STRICT
---
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "storage-rest-authentication-policy"
  namespace: "storage-rest"
spec:
  mtls:
    mode: STRICT
---
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "fm-authentication-policy"
  namespace: "fm"
spec:
  mtls:
    mode: STRICT
```

### **authorization deny**

authorization_deny_policy_namespace.yaml

```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: authorization-deny-all
  namespace: storage-rest
spec:
  {}
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: authorization-deny-all
  namespace: fm
spec:
  {}
```

### **authorization policy*

authorization_tcp_policy_namespace.yaml

```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: queryweb-to-storage-rest
  namespace: storage-rest
spec:
  selector:
    matchLabels:
      app: storage-rest
  action: ALLOW
  rules:
  - from:
    - source:
        #namespaces: ["query"]
        principals: ["cluster.local/ns/query/sa/query-web"]  # service Account 路径
    to:
    - operation:
        ports: ["9900"]
        #paths: ["/v1*"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: queryrest-to-fm
  namespace: fm
spec:
  selector:
    matchLabels:
      app: fm-master-fs
  action: ALLOW
  rules:
  - from:
    - source:
        namespaces: ["query"]
    to:
    - operation:
        ports: ["8080"]
```

## **HTTPauthorization实例**

参考istio官网配置的bookinfo实例: https://istio.io/latest/docs/tasks/security/authorization/authz-http/



