---
title: 06 Kubernetes deployment
tags: kubernetes
categories:
- microService
- kubernetes
top:
---

## deployment
Deployment 是一种更高阶资源， 用千部署应用程序并以声明的方式升级应用, 而不是通过 ReplicationController 或 ReplicaSet 进行部署， 它们都被认为是更底层的概念.
当创建一个 Deployment 时， ReplicaSet 资源也会随之创建.
在使用 Deployment 时， 实际的 pod是由 Deployment 的 Replicaset 创建和管理的， 而不是由 Deployment 直接创建和管理的.
> 创建Deployment与创建ReplicationController并没有任何区别。Deployment也是由标签选择器、期望副数和pod模板组成的。此外，它还包含另 一 个字段，指定一 个部署策略，该策略定义在修改Deployment资源时应该如何执行更新
Deployment可以同时管理多个版本的 pod, 所以在命名时不需要指定应用的版本号
kubia-svc-loadbalancer.yaml

```xml
	apiVersion: v1
	kind: Service
	metadata:
	  name: kubia-loadbalancer
	spec:
	  type: LoadBalancer
	  ports:
	  - port: 80
	    targetPort: 8080
	  selector:
	    app: kubia
```

kubia-deployment-v1.yaml

```xml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: kubia
	spec:
	  replicas: 3
	  selector:
	    matchLabels:
	      app: kubia
	  template:
	    metadata:
	      name: kubia
	      labels:
	        app: kubia
	    spec:
	      containers:
	      - image: luksa/kubia:v1
	        name: nodejs
```

创建deployment

```shell
	$ kubectl create -f kubia-deployment-v1.yaml --record
```
查看deployment过程

```shell
	$ kubectl rollout status deployment kubia
	Waiting for deployment "kubia" rollout to finish: 1 out of 3 new replicas have been updated...
	Waiting for deployment "kubia" rollout to finish: 1 out of 3 new replicas have been updated...
	Waiting for deployment "kubia" rollout to finish: 2 out of 3 new replicas have been updated...
	Waiting for deployment "kubia" rollout to finish: 2 out of 3 new replicas have been updated...
	Waiting for deployment "kubia" rollout to finish: 2 out of 3 new replicas have been updated...
	Waiting for deployment "kubia" rollout to finish: 2 out of 3 new replicas have been updated...
	Waiting for deployment "kubia" rollout to finish: 1 old replicas are pending termination...
	Waiting for deployment "kubia" rollout to finish: 1 old replicas are pending termination...
	Waiting for deployment "kubia" rollout to finish: 1 old replicas are pending termination...
	deployment "kubia" successfully rolled out
```
当使用 ReplicationController 创建 pod 时， 它们的名称是由 Controller 的名称加上一个运行时生成的随机字符串.
由 Deployment 创建的三个 pod 名称中均包含一个额外的数字, 这个数字实际上对应 Deployment 和 ReplicaSet 中的 pod 模板的哈希值

```shell
	$ kubectl get po
	kubia-59d857b444-2nrr7   1/1     Running   0          5m15s
	kubia-59d857b444-f9pnx   1/1     Running   0          5m15s
	kubia-59d857b444-wzgnj   1/1     Running   0          5m15s

	$ kubectl get replicasets
	NAME               DESIRED   CURRENT   READY   AGE
	kubia-59d857b444   3         3         3       8m48s

	$ kubectl get svc
	NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
	kubernetes           ClusterIP      10.96.0.1       <none>        443/TCP        4d22h
	kubia-loadbalancer   LoadBalancer   10.100.58.157   <pending>     80:31170/TCP   3s
```
查看master node 的IP为10.239.140.186

```shell
	$ curl 10.239.140.186:31170
	This is v1 running in pod kubia-59d857b444-wzgnj
```

查看deployment详细信息

```xml
	$ kubectl describe deployment kubia
	Name:                   kubia
	Namespace:              default
	CreationTimestamp:      Mon, 18 May 2020 15:04:12 +0800
	Labels:                 <none>
	Annotations:            deployment.kubernetes.io/revision: 1
	                        kubernetes.io/change-cause: kubectl create --filename=kubia-deployment-v1.yaml --record=true
	Selector:               app=kubia
	Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
	StrategyType:           RollingUpdate
	MinReadySeconds:        0
	RollingUpdateStrategy:  25% max unavailable, 25% max surge
	Pod Template:
	  Labels:  app=kubia
	  Containers:
	   nodejs:
	    Image:        luksa/kubia:v1
	    Port:         <none>
	    Host Port:    <none>
	    Environment:  <none>
	    Mounts:       <none>
	  Volumes:        <none>
	Conditions:
	  Type           Status  Reason
	  ----           ------  ------
	  Available      True    MinimumReplicasAvailable
	  Progressing    True    NewReplicaSetAvailable
	OldReplicaSets:  <none>
	NewReplicaSet:   kubia-59d857b444 (3/3 replicas created)
	Events:
	  Type    Reason             Age    From                   Message
	  ----    ------             ----   ----                   -------
	  Normal  ScalingReplicaSet  7m12s  deployment-controller  Scaled up replica set kubia-59d857b444 to 3
```

略微减慢滚动升级的速度， 以便观察升级过程确实是以滚动的方式执行的。 可以通过在Deployment上设置minReadySeconds属性来实现
minReadySeconds的主要功能是避免部署出错版本的应用， 而不只是单纯地减慢部署的速度.
minReadySeconds属性指定新创建的pod至少要成功运行多久之后，才能将其视为可用.
通常情况下需要 将minReadySeconds设置为更高的值， 以确保pod在它们真正开始接收实际流量之后可以持续保持就绪状态.

如kubectl patch命令将其设置为10秒

```shell
	$ kubectl patch deployment kubia -p '{"spec": {"minreadyseconds": 10}}'
```
## 重新部署几种方式
重建(recreate)，滚动(rollingUpdate)，蓝绿，金丝雀

### recreate
停止Pod并重新创建Pod

```xml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: web-recreate
	  namespace: dev
	spec:
	  strategy:
	    type: Recreate		// 停止Pod并重新创建Pod
	  selector:
	    matchLabels:
	      app: web-recreate
	  replicas: 2
	  template:
	......
	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: web-recreate
	  namespace: dev
	spec:
	  rules:
	  - host: web-recreate.mooc.com  //配置下/etc/hosts 文件 "IP web-recreate.mooc.com"
	    http:
	      paths:
	      - path: /
	        backend:
	          serviceName: web-recreate
	          servicePort: 80
```

### 滚动更新deployment
滚动更新过程svc的IP等不会变，只是改变pod版本，修改pod的镜像就可以了
> 实际上， 如何达到新的系统状态的过程是由 Deployment 的升级策略决定的，默认策略是执行滚动更新（策略名为 RollingUpdate)。 另 一种策略为 Recreate, 它会一次性删除所有旧版本的 pod, 然后创建新的 pod, 整个行为类似千修改ReplicationController 的 pod 模板， 然后删除所有的 pod
使用kubectl set image命令来更改任何包含容器资源的镜像(ReplicationController、ReplicaSet、 Deployment等）

```xml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: web-rollingupdate
	  namespace: dev
	spec:
	  strategy:
	    rollingUpdate:			// 滚动更新, 不设置的话都会设成跟下面一样的默认值, $kubectl get deployment Name -o yaml查看
	      maxSurge: 25%			// 可以最大超出实力数的百分比，4个replicas，就是每次最多多启动1个实例
	      maxUnavailable: 25%	// 不可用实力百分比，4个replicas，就是每次必须有3个实例可用
	    type: RollingUpdate
	  selector:
	    matchLabels:
	      app: web-rollingupdate
	  replicas: 2
	  template:
	    metadata:
	      labels:
	        app: web-bluegreen
	        version: v1.0
	    spec:
	......
```

```shell
	$ kubectl set image deployment kubia nodejs=luksa/kubia:v2
	deployment.apps/kubia image updated
```
创建rs，然后新的rs会创建一个pod，然后原来的rs删除一个pod，新的rs再创建一个pod，依次完成设定的3个pod都完成更新.
通过命令定时查看输出:

```shell
	$ while sleep 0.2; do curl "http://web-rollingupdate.mooc.com/hello?name=michael"; echo ""; done
```

查看Pod:

```shell
	$ kubectl get po
	NAME                     READY   STATUS              RESTARTS   AGE
	fortune                  2/2     Running             0          3d
	kubia-59d857b444-6nq74   1/1     Running             0          72s
	kubia-59d857b444-jbln7   1/1     Terminating         0          72s
	kubia-59d857b444-lr2zw   1/1     Running             0          72s
	kubia-7d5c456ffc-69gg5   0/1     ContainerCreating   0          2s
	kubia-7d5c456ffc-vmz74   1/1     Running             0          14s
```

滚动更新后旧的 ReplicaSet 仍然会被保留

```shell
	$ kubectl get rs
	NAME               DESIRED   CURRENT   READY   AGE
	kubia-59d857b444   3         3         3       9m27s
	kubia-7d5c456ffc   1         1         0       3s
```
再等待一会再查看rs

```shell
	$ kubectl get rs
	NAME               DESIRED   CURRENT   READY   AGE
	kubia-59d857b444   0         0         0       10m
	kubia-7d5c456ffc   3         3         3       72s
```

Deployment可以非常容易地回滚到先前部署的版本，它可以让Kubernetes 取消最后一次部署的 Deployment

```shell
	$ kubectl set image deployment kubia nodejs=luksa/kubia:v2
	$ kubectl get rs
	NAME               DESIRED   CURRENT   READY   AGE
	kubia-59d857b444   0         0         0       3h3m
	kubia-79b84b44f4   3         3         3       96s
	kubia-7d5c456ffc   0         0         0       174m
```
undo 命令也可以在滚动升级过程中运行，并直接停止滚动升级。 在升级过程中已创建的 pod 会被删除并被老版本的 pod 替代

```shell
	$ kubectl rollout undo deployment kubia
	deployment.apps/kubia rolled back
	$ kubectl get rs
	NAME               DESIRED   CURRENT   READY   AGE
	kubia-59d857b444   0         0         0       3h7m
	kubia-79b84b44f4   0         0         0       5m19s
	kubia-7d5c456ffc   3         3         3       177m
```

kubectl rollout history 来显示升级的版本

```shell
	$ kubectl rollout history deployment kubia
	deployment.apps/kubia
	REVISION  CHANGE-CAUSE
	1         kubectl create --filename=kubia-deployment-v1.yaml --record=true
	2         kubectl create --filename=kubia-deployment-v1.yaml --record=true
	3         kubectl create --filename=kubia-deployment-v1.yaml --record=true
```

### 回滚到一个特定的 Deployment 版本

```shell
	$ kubectl rollout undo deployment kubia --to-revision=l
	deployment.apps/kubia rolled back
```
> 旧版本的 ReplicaSet 过多会导致 ReplicaSet 列表过于混乱，可以通过指定Deployment 的 re visionHistoryLimit 属性来限制历史版本数量。默认值是 2

```shell
	$ kubectl rollout history deployment kubia
	deployment.apps/kubia
	REVISION  CHANGE-CAUSE
	2         kubectl create --filename=kubia-deployment-v1.yaml --record=true
	3         kubectl create --filename=kubia-deployment-v1.yaml --record=true
	4         kubectl create --filename=kubia-deployment-v1.yaml --record=true
```

### 滚动升级速率
在 Deployment 的滚动升级期间，有两个属性会决定一次替换多少个pod: maxSurge 和 maxUnavailable。可以通过 Deployment 的 strategy 字 段下rollingUpdate 的子属性来配置.

```xml
	spec:
	  strategy:
	    rollingUpdate:
	      maxSurge· 1
	      maxUnavailable: 0
	    type: RollingUpdate
```

> * maxSurge: 决定了 Deployment 配置中期望的副本数之外，最多允许超出的 pod 实例的数量。默认值为 25%，所以 pod 实例最多可以比期望数量多25%. 这个值也可以不是百分数而是绝对值(例如，可以允许最多多出一个成两个pod).
> * maxUnavailable: 决定了在滚动升级期间 ，相对于期望副本数能够允许有多少 pod 实例处于不可用状态。默认值也是25%, 所以可用 pod 实例的数量不能低于期望副本数的75%. 与 maxSurge 一样，也可以指定绝对值而不是百分比.

### 暂停滚动升级

```shell
	$ kubectl set image deployment kubia nodejs=luksa/kubia:v4
	deployment "kubia" image updated
	$ kubectl rollout pause deployment kubia
	deployment "kubia" paused
```

### 恢复滚动升级
如果部署被暂停， 那么在恢复部署之前， 撤销命令不会撤销它

```shell
	$ kubectl rollout resume deployment kubia
	deployment "kubia" resumed
```

### readinessProbe
kubia-deployment-v3-with-readinesscheck.yaml

```xml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: kubia
	spec:
	  replicas: 3					// POD副本个数为3
	  minReadySeconds: 10			// 设置minReadySeconds 值为10s,指定新创建的pod至少要成功运行多久之后，才能将其视为可用.
	  progressDeadlineSeconds: 120	// 设置升级失败超时时间，超过则自动停止升级，如下describe deployment 可查看到"Progressing False..."信息，需要手动undo来取消升级
	  strategy:
	    rollingUpdate:
	      maxSurge: 1
	      maxUnavailable: 0			// 设置maxUnavailable的 中值为0来确保升级过程 pod被挨个替换
	    type: RollingUpdate
	  selector:
	    matchLabels:
	      app: kubia
	  template:
	    metadata:
	      name: kubia
	      labels:
	        app: kubia
	    spec:
	      containers:
	      - image: luksa/kubia:v3	// 修改image
	        name: nodejs
	        readinessProbe:
	          periodSeconds: 1		// 定义一个就绪探针，并且每隔—秒钟执行一次
	          httpGet:				// 就绪探针会执行发送HTTP GET请求到容器
	            path: /
	            port: 8080
```

直接使用kubectl apply来升级Deployment
apply命令可以用YAML 文件中声明的字段来更新Deployment。不仅更新镜像，而且还添加了就绪探针， 以及在 YAML 中添加或修改的其他声明。 
如果新的 YAML也包含rep巨 cas字段， 当它与现有Deployment中的数量不一致时， 那么apply 操作也会对Deployment进行扩容.

```shell
	$ kubectl apply -f kubia-deployment-v3-with-readinesscheck.yaml
	deployment "kubia" configured
```

### 取消滚动升级
默认情况下， 在10分钟内不能完成滚动升级的话， 将被视为失败。 如果运行kubectl describe deployment命令， 将会显示一条ProgressDeadlineExceeded的记录.
判定Deployment滚动升级失败的超时时间， 可以通过设定Deployment spec中的progressDeadlineSeconds来指定.
如果达到了progressDeadlineSeconds指定的时间， 则滚动升级过程会自动取消.

```shell
	$ kubectl describe deployment kubia
	Conditions:
	  Type           Status  Reason
	  ----           ------  ------
	  Available      True    MinimumReplicasAvailable
	  Progressing    True    ReplicaSetUpdated
	......
```
过了3，4分钟后再次执行

```shell
	$ kubectl describe deployment kubia
	Name:                   kubia
	......
	Conditions:
	  Type           Status  Reason
	  ----           ------  ------
	  Available      True    MinimumReplicasAvailable
	  Progressing    False   ProgressDeadlineExceeded
	OldReplicaSets:  kubia-59d857b444 (3/3 replicas created)
	NewReplicaSet:   kubia-7d6c89d47b (1/1 replicas created)
	......
```

因为滚动升级过程不再继续， 所以只能通过rollout undo命令来取消滚动升级, 其实就是回滚上一个版本.

```shell
	$ kubectl rollout undo deployment kubia
	deployment.apps/kubia rolled back
```

### 蓝绿部署
可以通过修改Service下的不同version值，选择应用不同版本的Pod来提供服务.
新版本运行起来一段时间没问题后才可以删除旧版本.
一般保持有两个版本的应用也就是Pod保持运行状态, 只有下个版本部署一段时间没问题后，第一个版本的可以删除.

```xml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: web-rollingupdate
	  namespace: dev
	spec:
	  strategy:
	    rollingUpdate:			// 滚动更新, 不设置的话都会设成跟下面一样的默认值, $kubectl get deployment Name -o yaml查看
	      maxSurge: 25%			// 可以最大超出实力数的百分比，4个replicas，就是每次最多多启动1个实例
	      maxUnavailable: 25%	// 不可用实力百分比，4个replicas，就是每次必须有3个实例可用
	    type: RollingUpdate
	  selector:
	    matchLabels:
	      app: web-rollingupdate
	  replicas: 2
	  template:
	    metadata:
	      labels:
	        app: web-bluegreen
	        version: v2.0		// 蓝绿部署，与下面的version一致，部署不同版本的应用
	    spec:
	......
	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: web-bluegreen
	  namespace: dev
	spec:
	  ports:
	  - ports: 80
	    protocol: TCP
	    targetPort: 8080
	  selector:
	    app: web-bluegreen
	    version: v2.0			// 与上面的version一致，部署不同版本应用, 下次部署单独抽离Service，选择不同版本version就可以了
	  type: ClusterIP
	---
	......
```

### 金丝雀发布
> 在蓝绿部署的基础之上，去掉version, 修改selector就是金丝雀部署.
> 一个新的 pod 会被创建， 与此同时所有旧的 pod 还在运行。 一旦新的 pod 成功运行， 服务的一部分请求将被切换到新的 pod。 这样相当于 运行了一个金丝雀版本。金丝雀发布是一种可以将应用程序的出错版本和其影响到的用户的风险化为最小的技术.
> 与其直接向每个用户发布新版本， 不如用新版本替换 一个或一小部分的 pod。通过 这种方式， 在升级的初期只有少数用户会访问新版本。 验证新版本是否正常工作之后， 可以将剩余的 pod 继续升级或者回滚到上一个的版本

### 删除deployment
删除deployment会连带删除创建的replicaset和由replicaset创建的pod

```shell
	$ kubectl delete deployment kubia
	deployment.apps "kubia" deleted
```













