---
title: 03 Kubernetes nodes, namespace, pod
tags: kubernetes
categories:
- microService
- kubernetes
top: 3
---

## nodes

	$ kubectl get nodes -o wide
	NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
	hci-node01   Ready    master   5d    v1.18.1   10.67.108.211   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8
	hci-node02   Ready    <none>   5d    v1.18.1   10.67.109.142   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8
	hci-node03   Ready    <none>   5d    v1.18.1   10.67.109.147   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8
	hci-node04   Ready    <none>   5d    v1.18.1   10.67.109.144   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8
	$ kubectl describe node hci-node02
	输出显示了节点的状态、 CPU 和内存数据、系统信息、运行容器的节点等

	查看某台机器的资源
	$ kubectl describe node hci-node01

### 创建别名和补全
kubectl 会被经常使用。很快你就会发现每次不得不打全命令是非常痛苦的。
将下面的代码添加到 ~/.bashrc 或类似的文件中 ：

	alias k=kubectl
为kuebctl配置 tab 补全
需要先安装一个叫作 bashcompletio口的包来启用 bash 中的 tab 命令补全， 然后可以运行接下来的命令（也需要加到 ~/.bashrc 或类似的文件中）

	$ source <{kubectl completion bash)
	$ kubectl desc<TAB> nod<TAB> hci<TAB>
但是需要注意的是， tab 命令行补全只在使用完整的 kubectl 命令时会起作用,(当使用别名 k 时不会起作用). 需要改变 kubectl completion 的输出来修复：

	$ source <(kubectl completion bash | sed s/kubectl/k/g)

### node 标签

	$ kubectl get nodes
	$ kubectl label node server02 gpu=false
	$ kubectl label node server02 gpu=true --overwrite	//修改node标签
	$ kubectl get node -L gpu		// 列出所有node，并添加GPU一列进行展示
	$ kubectl get node -l gpu		// 只列出含标签的key为gpu的node
	$ kubectl get node -l gpu=false	// 只列出含gpu=false的node

将POD调度到指定的node上: kubia-gpu.yaml

	apiVersion: v1
	kind: Pod
	metadata:
	name: kubia-gpu		// 指定生成的POD名字
	spec:
	nodeSelector:		// node选择器,选择含标签gpu=true的node机器
		gpu: "true"		
	containers:
	- image: luksa/kubia	// 要拉取的 image 名字
		name: kubia			// 生成的 container 名字

	$ kubectl create -f kubia-gpu.yaml
如果没有标签为gpu=true的合适node， 通过 $ kubectl describe pod/kubia-nogpu 查看Message， 会报 0/2 nodes are available: 2 node(s) didn't match node selector.信息

	$ kubectl describe pod/kubia-gpu
	......
	Node-Selectors:  gpu=true
	......

### taint污点
给Node添加污点可以让配置tolerations的Pod部署上来，而不让平常的Pod部署.
配置tolerations的Pod可以部署到添加污点的机器也可以部署到其它平常机器

	$ kubectl taint nodes NodeName gpu=true:NoSchedule

	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: web-demo
	  namespace: dev
	spec:
	  selector:
	    matchLabels:
	      app: web-demo
	  replicas: 3			// 副本数3来测试能部署到哪些机器
	  template:
	    metadata:
	      labels:
	        app: web-demo
	    spec:
	      selector:
	        matchLabels:
	          app: web-demo
	      replicas: 1
	      template:
	        metadata:
	          labels:
	            app: web-demo
	        spec:
	          containers:
	          - name: web-demo
	            image: hub.mooc.com/kubernetes/web:v1
	            ports:
	            - containerPort: 8080
	          tolerations:		// 可以部署到有设置taint的机器，也可以部署到其它机器
	          - key: "gpu"
	            operator: "Equal"
	            value: "true"
	            effect: "NoSchedule"
> 典型的使用kubeadm部署和初始化的Kubernetes集群，master节点被设置了一个node-role.kubernetes.io/master:NoSchedule的污点，可以使用kubectl describe node <node-name>命令查看
> 这个污点表示默认情况下master节点将不会调度运行Pod，即不运行工作负载, 对于使用二进制手动部署的集群设置和移除这个污点的命令如下:

	$ kubectl taint nodes <node-name> node-role.kubernetes.io/master=:NoSchedule
	$ kubectl taint nodes <node-name> node-role.kubernetes.io/master:NoSchedule-
> kubeadm初始化的Kubernetes集群，master节点也被打上了一个node-role.kubernetes.io/master=的label，标识这个节点的角色为master。给Node设置Label和设置污点是两个不同的操作。设置Label和移除Label的操作命令如下

设置Label

	$ kubectl label node node1 node-role.kubernetes.io/master=
移除Label

	$ kubectl label node node1 node-role.kubernetes.io/master-


## Namespace
> 大多数对象的名称必须符合 RFC 1035 （域名）中规定的命名规范 ，这意味着它们可能只包含字母、数字、横杠（－）和点号，但命名空间（和另外几个）不允许包含点号

### 隔离性
> 名字的隔离只是 通过svc名称(DNS) 访问的隔离，通过svc的IP和Pod的IP再加上端口号(Port) 照样可以访问不同命名空间下的服务.

### 设置默认命名空间
> 默认Kubeclt获取default命名空间下的资源，可以通过设置K8s上下文配置文件如kube.config 使得某个命名空间变为默认namespace，获取pod时候不需要在加上 -n 参数

### 创建命名空间
> namespace不提供网络隔离, 如果命名空间 foo 中的某个 pod 知道命名空间 bar 中 pod 的 IP 地址，那它就可以将流量（例如 HTTP 请求）发送到另一个 pod
第一种： commands方式

	$ kubectl create namespace custom-namespace
	$ kubectl create ns custom-namespace

第二种： Yaml方式， 之所以选择使用 YAML 文件，只是为了强化Kubemetes中的所有内容都是一 个 API 对象这一概念

	$ touch custom-namespace.yaml
	apiVersion: v1
	kind: Namespace
	metadata:
	  name: custom-namespace
	$ kubectl create -f custom-namespace.yaml

### 划分方式

	* 按环境划分: dev(开发), test(测试)
	* 按团队划分
	* 自定义多级划分

### 标记命名空间

	$ kubectl label namespace default istio-injection=enabled --overwrite         // enabled
	$ kubectl label namespace default istio-injection=disabled --overwrite        // disabled
	$ kubectl label namespace default istio-injection= --overwrite                // cancel set
	namespace/default labeled

### 查看标记 istio-injection=enabled 标签的命名空间

	$ kubectl get namespace -L istio-injection
	NAME              STATUS   AGE   ISTIO-INJECTION
	default           Active   85m   enabled
	istio-system      Active   25m   disabled
	kube-node-lease   Active   85m
	kube-public       Active   85m
	kube-system       Active   85m
	[root@hci-node01 istio-1.5.2]#

### 删除namespace
删除当前命名空间中的所有资源，可以删除ReplicationCcontroller和pod,以及我们创建的所有service
第一个 all 指定正在删除所有资源类型, --all 选项指定将删除所有资源实例, 而不是按名称指定它们
使用 all 关键字删除所有内容并不是真的完全删除所有内容。 一些资源比如Secret会被保留下来， 并且需要被明确指定删除

	$ kubectl delete all --all		// 命令也会删除名为 kubernetes 的Service, 但它应该会在几分钟后自动重新创建
可以简单地删除整个命名空间（ pod 将会伴随命名空间 自动删除〉

	$ kubectl delete ns custom-namespace
强制删除NAMESPACE

	$ kubectl delete namespace NAMESPACENAME --force --grace-period=0
进入kube-system下得etcd pod 删除需要删除的NAMESPACE

	$ kubectl get po -n kube-system
	NAME                                 READY   STATUS    RESTARTS   AGE
	etcd-hci-node01                      1/1     Running   5          16d
	......
	
	$ kubectl exec -it etcd-hci-node01 sh -n kube-system
	$ etcdctl del /registry/namespaces/NAMESPACENAME

## POD

### 查看pod解释

	$ kubectl explain pod
	KIND:     Pod
	VERSION:  v1
	
	DESCRIPTION:
		Pod is a collection of containers that can run on a host. This resource is
		created by clients and scheduled onto hosts.
	
	FIELDS:
	apiVersion   <string>
		APIVersion defines the versioned schema of this representation of an
		object. Servers should convert recognized schemas to the latest internal
		value, and may reject unrecognized values. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
	
	kind <string>
		Kind is a string value representing the REST resource this object
		represents. Servers may infer this from the endpoint the client submits
		requests to. Cannot be updated. In CamelCase. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
	
	metadata     <Object>
		Standard object's metadata. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	
	spec <Object>
		Specification of the desired behavior of the pod. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	
	status       <Object>
		Most recently observed status of the pod. This data may not be up to date.
		Populated by the system. Read-only. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

深入理解POD属性

	$ kubectl explain pod.apiVersion
	$ kubectl explain pod.kind
	$ kubectl explain pod.spec
pods 的缩写是 po, service 的缩写是 SVC, replicationcontroller 的缩写 rc

	$ kubectl get pods -n kube-system
> 我们提到过每个 pod 都有自己的 IP 地址，但是这个地址是集群 内部的，不能从集群外部访问。
> 要让 pod 能够从外部访问 ， 需要通过服务对象公开它， 要创建一个特殊的 LoadBalancer 类型的服务。
> 因为如果你创建一个常规服务（ 一个 Cluster IP 服务）， 比如 pod ，它也 只能从集群内部访问。
> 通过创建 LoadBalanc er 类型 的服务，将创建一个外部的负载均衡 ，可以通过 负载均衡的公共 IP 访问 pod 

### 创建POD
通过上传 JSON 或 YAML 描述文件到 Kubemetes API 服务器来创建 pod.
kubectl create -f 命令用于从YAML或JSON文件创建任何资源（不只是 pod).
	$ kubectl create -f kubia-manual.yaml
	$ kubectl create -f kubia-gpu.yaml -n custom-namespace	//船舰pod到custom-namespace命名空间下

	$ kubectl describe pod/kubia
	......
	Events:
	Type    Reason     Age    From               Message
	----    ------     ----   ----               -------
	Normal  Scheduled  5m12s  default-scheduler  Successfully assigned default/kubia-liveness to server02
	Normal  Pulling    5m8s   kubelet, server02  Pulling image "luksa/kubia-unhealthy"

### pod标签labels

	$ kubectl get po --show-labels
查看pod标签的key值为creation_method 和 env 的信息

	$ kubectl get po -L creation_method,env
	NAME                           READY   STATUS    RESTARTS   AGE   CREATION_METHOD   ENV
	kubia                          1/1     Running   0          16h
	kubia-manual-v2                1/1     Running   0          34m   manual            pod
POD添加标签

	$ kubectl label po kubia  creation_method=manual
	NAME                           READY   STATUS    RESTARTS   AGE   CREATION_METHOD   ENV
	kubia                          1/1     Running   0          16h   manual
	kubia-manual-v2                1/1     Running   0          43m   manual            pod
更改现有标签, 在更改现有标签时， 需要使用--overwrite选项

	$ kubectl label po kubia-manual-v2 env=debug --overwrite
使用标签列出POD

	$ kubectl get po -1 creation_method=manual
	$ kubectl get po -l env
同样列出没有env标签的pod
确保使用单引号来圈引 !env, 这样bash shell才不会解释感叹号（译者注：感叹号在bash中有特殊含义， 表示事件指示器)

	$ kubectl get po -l '!env'
	creation_method!=manual 选择带有creation_method标签， 并且值不等于manual的pod
	env in (prod, devel)选择带有env标签且值为prod或devel的pod
	env notin (prod, devel)选择带有env标签， 但其 值不是prod或devel的pod
	app=pc,rel=beta 选择pc微服务的beta版本pod


## pod 注解

	$ kubectl annotate pod kubia-gpu mycompany.com/someannotion="foo bar"
	$ kubectl describe pod/kubia-gpu
	......
	Annotations:  mycompany.com/someannotion: foo bar
	......

### 查看该 pod 的完整描述文件：
	$ kubectl get po kubia-manual -o yaml	// 获取yaml格式信息
	$ kubect1 get po kubia-manual -o json	// 获取json格式信息


### 执行pod容器
直接执行:

	$ kubectl exec fortioclient-f8d65c6bb-5k4td -c captured date -n twopods
进入容器执行, 当pod中只有一个容器时可以不加-c参数指定某个容器

	$ kubectl exec fortioclient-f8d65c6bb-5k4td -c captured -i -t /bin/sh -n twopods

### 容器进程, 网络等
查看进程command完整信息

	$ ps auxwww
	$ pa -ef
查看网络

	$ netstat -ntlp

### 查看Pod, svc日志

	$ kubectl logs pod/istiod-774777b79-ddfk4 -n istio-system
	$ kubectl logs -f pod/<pod_name> #类似tail -f的方式查看(tail -f 实时查看日志文件 tail -f 日志文件log)
	$ kubectl logs svc/istiod -n istio-system
	如果该pod中有其他容器， 可以通过如下命令获取其日志：
	$ kubectl logs kubia-manual -c kubia

查看容器重启后前一个容器为什么重启的日志信息
	$ kubectl logs mypod --previous

### 部署应用程序

	$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

### 重启Pod

	$ kubectl get pod {podname} -n {namespace} -o yaml | kubectl replace --force -f -

### 删除Pod

	$ kubectl delete pod PODNAME -n custom-namespace		// 删除指定命名空间下的POD
	$ kubectl delete po -l creation_method=manual			// 通过标签选择器来删除
	$ kubectl delete po --all -n custom-namespace			// 删除当前命名空间中的所有 pod
	$ kubectl delete all --all -n custom-namespace			// 删除所有pod和svc，系统带的kubernetes服务会过一会重启
可使用kubectl中的强制删除命令删除POD

	$ kubectl delete pod PODNAME --force --grace-period=0
直接从ETCD中删除源数据
删除default namespace下的pod名为pod-to-be-deleted-0

	$ ETCDCTL_API=3 etcdctl del /registry/pods/default/pod-to-be-deleted-0

## livenessProbe 存活探针, readinessProbe
Kubemetes 可以通过存活探针 (liveness probe) 检查容器是否还在运行.
Kubemetes 可以通过readinessProbe探针 检查容器是否准备完毕可以挂到负载均衡上供外部访问.
livenessProbe与readinessProbe探针用法完全一样, 都有三种，下面介绍这三种健康检查方式.

可以为 pod 中的每个容器单独指定存活探针。 如果探测失败， Kubemetes 将定期执行探针并重新启动容器

 * 第一种健康检查方式: 执行命令检查存活探针是否存活


	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: web-demo
	  namespace: dev
	spec:
	  selector:
	    matchLabels:
	      app: web-demo
	  replicas: 1
	  template:
	    metadata:
	      labels:
	        app: web-demo
	    spec:
	      selector:
	        matchLabels:
	          app: web-demo
	      replicas: 1
	      template:
	        metadata:
	          labels:
	            app: web-demo
	        spec:
	          containers:
	          - name: web-demo
	            image: hub.mooc.com/kubernetes/web:v1
	            ports:
	            - containerPort: 8080
	            livenessProbe:				// 检查应用是否存活的探针, 和容器一个级别
	              exec:						// 第一种健康检查方式, 通过执行命令
	                command:
	                - /bin/sh
	                - -c
	                - ps -ef|grep java|grep -v grep
	              initialDelaySeconds: 10		// 容器起来后过10s开始检查
	              periodSeconds: 10				// 每隔10s检查一次
	              failureThreshold: 2			// 连续健康检查失败2次放弃检查, 重启容器
	              successThreshold: 1			// 检查一次满足条件就认为健康检查通过
	              timeoutSeconds: 5				// 每次健康检查delay时间是5s, 超时也认为健康检查失败, 重启容器
	            readinessProbe:				// readinessProbe与livenessProbe用法完全一样.
	              exec:						// 第一种检查方式, 通过执行命令
	                command:
	                - /bin/sh
	                - -c
	                - ps -ef|grep java|grep -v grep
	              initialDelaySeconds: 10		// 容器起来后过10s开始检查
	              periodSeconds: 10				// 每隔10s检查一次
	              failureThreshold: 2			// 连续健康检查失败2次放弃检查, 重启容器
	              successThreshold: 1			// 检查一次满足条件就认为健康检查通过
	              timeoutSeconds: 5				// 每次健康检查delay时间是5s, 超时也认为健康检查失败, 重启容器

 * 第二种健康检查方式: 执行网络请求检查存活探针是否存活


	$ touch kubia-liveness-probe.yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: kubia-liveness
	spec:
	  containers:
	  - image: luksa/kubia-unhealthy
	    name: kubia
	    livenessProbe:		// 一个 HTTP GET 存活探针
	      httpGet:			// 第二种健康检查方式, 通过httpGet
	        path: /			// 应用要访问的路径
	        port: 8080		// 容器本身启动的端口
	        scheme: HTTP
	      initialDelaySeconds: 15		// 容器起来后过10s开始检查
	      periodSeconds: 5

	$ kubectl get pods
	NAME                           READY   STATUS    RESTARTS   AGE
	kubia-liveness                 1/1     Running   1          13m
查看该pod描述

	$ kubectl describe pod/kubia-liveness
	......
	Last State:     Terminated
	  Reason:       Error
	  Exit Code:    137
	  Started:      Wed, 13 May 2020 15:49:36 +0800
	  Finished:     Wed, 13 May 2020 15:51:25 +0800
	Ready:          True
	Restart Count:  1
	Liveness:       http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
	......
数字137是两个 数字的总和：128+x, 其中x是终止进程的信号编号.
在这个例子中，x等于9, 这是SIGKILL的信号编号，意味着这个进程被强行终止.
当容器被强行终止时，会创建一个全新的容器—-而不是重启原来的容器.
delay=Os部分显示在容器启动后立即开始探测.
timeout仅设置为1秒，因此容器必须在1秒内进行响应， 不然这次探测记作失败.
每10秒探测一次容器(period=lOs), 并在探测连续三次失败(#failure=3)后重启容器.
定义探 针时可以自定义这些附加参数。例如，要设 置初始延迟，请将initialDelaySeconds属性添加到存活探针的配置中.

	    livenessProbe:		// 一个 HTTP GET 存活探针
	      httpGet:
	        path: /
	        port: 8080
	      initialDelaySeconds: 15	// Kubernetes会在第—次探测前等待15秒 

	$ kubectl describe pod/kubia-liveness
	......
	Liveness:       http-get http://:8080/ delay=15s timeout=1s period=10s #success=1 #failure=3

 * 第三种健康检查方式: 通过TCP检查端口是否处于监听状态
	    livenessProbe:		// 一个 HTTP GET 存活探针
	      tcpSocket:
	        port: 8080
	      initialDelaySeconds: 20	// Kubernetes会在第—次探测前等待15秒 
	      periodSeconds: 5

> 如果没有设置初始延迟，探针将在启动时立即开始探测容器， 这通常会导致探测失败， 因为应用程序还没准备好开始接收请求.
> 务必记得设置一个初始延迟未说明应用程序的启动时间.
> 对于在生产中运行的pod, 一定要定义一个存活探针。没有探针的话，Kubemetes无法知道你的应用是否还活着。只要进程还在运行， Kubemetes会认为容器是健康的
> Kubernetes会在你的容器崩溃或其存活探针失败时， 通过重启容器来保持运行。 这项任务由承载pod的节点上的Kubelet 执行 一— 在主服务器上运行的Kubernetes Control Plane组件不会参与此过程.
> 但如果节点本身崩溃， 那么Control Plane 必须为所有随节点停止运行的pod创建替代品。 它不 会为你直接创建的pod执行此操作 。 这些pod只被Kubelet 管理.

### 查看 readinessProbe, healthProbe
	
	$ kubectl edit po -n istio-system istio-ingressgateway-6489d9556d-wjr58
	$ kubectl edit deployment -n istio-system istio-ingressgateway
	$ kubectl logs po/istio-ingressgateway-6489d9556d-wjr58 -n istio-system
	$ kubectl get po -A

### affinity
匹配Node标签, Pod部署到哪台机器上.

	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: web-demo
	  namespace: dev
	spec:
	  selector:
	    matchLabels:
	      app: web-demo
	  replicas: 1
	  template:
	    metadata:
	      labels:
	        app: web-demo
	    spec:
	      selector:
	        matchLabels:
	          app: web-demo
	      replicas: 1
	      template:
	        metadata:
	          labels:
	            app: web-demo
	        spec:
	          containers:
	          - name: web-demo
	            image: hub.mooc.com/kubernetes/web:v1
	            ports:
	            - containerPort: 8080
	          affinity:
	            nodeAffinity:		// node亲和性, 要部署到哪台机器，不要部署到哪台机器
	              requiredDuringSchedulingIgnoredDuringExecution:	// 必须满足下面条件才会执行调度
	                nodeSelectorTerms:		// 数组形式, 下面可以定义多个Terms, 它们之间是或的关系
	                - matchExpressions:		// 数组形式, 如果定义多个matchExpressions它们之间是与的关系
	                  - key: beta.kubernetes.io/arch	// 节点的label含有的key名字, 这里的是由K8s根据机器自动生成的
	                    operator: In
	                    values:				// 前提是Node机器有amd64标签K8s才会把容器部署到次Node机器
	                    - amd64				// 通过kubectl get nodes NodeName -o yaml进行查看
	              preferredDuringSchedulingIgnoredDuringExecution:	// 最好是怎样调度
	              - weight: 1				// 权重
	                perference:
	                  matchExpressions:
	                  - key: disktype		// 通过kubectl get nodes --show-labels查看
	                    operator: NotIn
	                    values:
	                    - ssd
	            podAffinity:		// Pod亲和性, 想和哪些Pod部署到一台机器, 不想和哪些Pod部署在一台机器
	              requiredDuringSchedulingIgnoredDuringExecution:
	              - labelSelector:
	                  matchExpressions:
	                  - key: app
	                    operator: In		// 要跟app=web-demo的Pod运行在同一个节点上
	                    values:
	                    - web-demo-node
	                topologyKey: kubernetes.io/hostname		// 节点的label名字
	              preferredDuringSchedulingIgnoredDuringExecution:
	              - weight: 100
	                podAffinityTerm:
	                  labelSelector:
	                    matchExpressions:
	                    - key: app
	                      operator: In
	                      values:
	                      - web-demo-node
	                  topologyKey: kubernetes.io/hostname
	            podAntiAffinity:		// Pod反亲和性, 不想和哪些Pod部署到一台机器, 用法和podAffinity用法完全一样
	            pod反亲和性用的很多的是上面的replicas: 的值 >=2 时候会把容器副本分别部署到不同机器

### Pod启动停止控制
Pod容器启动时候和停止前所做的事

	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: web-demo
	  namespace: dev
	spec:
	  selector:
	    matchLabels:
	      app: web-demo
	  replicas: 1
	  template:
	    metadata:
	      labels:
	        app: web-demo
	    spec:
	      selector:
	        matchLabels:
	          app: web-demo
	      replicas: 1
	      template:
	        metadata:
	          labels:
	            app: web-demo
	        spec:
	          containers:
	          - name: web-demo
	            image: hub.mooc.com/kubernetes/web:v1
	            ports:
	            - containerPort: 8080
	            volumeMounts:
	            - name: shared-volume
	              mounthPath: /shared-web
	            lifecycle:				Pod里容器启动前和停止前要做的事
	              postStart:
	                exec:
	                  command: ["/bin/sh", "-c", "echo web starting ... >> /var/log/messages"]
	              preStop:
	                exec:
	                  command: ["/bin/sh", "-c", "echo web stopping ... >> /var/log/messages && sleep 3"]


## ReplicationController
一个ReplicationController有三个主要部分
 • label selector ( 标签选择器）， 用于确定ReplicationController作用域中有哪些pod
 • replica count (副本个数）， 指定应运行的pod 数量
 • pod template (pod模板）， 用于创建新的pod 副本
使用 ReplicationController 的好处
 •确保一 个 pod (或多个 pod副本）持续运行， 方法是在现有pod 丢失时启动一个新 pod。
 • 集群节点发生故障时， 它将为故障节 点 上运 行的所有 pod (即受ReplicationController 控制的节点上的那些 pod) 创建替代副本。
 • 它能轻松实现 pod的水平伸缩 手动和自动都可以

### 由RC创建POD
kubia-rc.yaml, 内容如下:

	apiVersion: v1
	kind: ReplicationController		// 这里的配置定义了ReplicationController(RC)
	metadata:
	  name: kubia		// ReplicationController 的名字
	spec:
	  replicas: 3		// pod 实例的目标数目
	  selector:			// selector也可以不写，replica 直接根据下面的template模板里的lables标签选择创建POD
	    app: kubia		// pod 选择器决定了 RC 的操作对象
	  template:			// 从此以下都是创建新 pod 所用的 pod 模板, 与单独创建的pod定义yaml文件内容几乎相同
	    metadata:
	      labels:
	        app: kubia
	    spec:
	      containers:
	      - name: kubia
	        image: luksa/kubia
	        ports:
	        - containerPort: 8080
创建ReplicationController并由其创建pod

	$ kubectl create -f kubia-rc.yaml
	$ kubectl get po -o wide
	NAME          READY   STATUS    RESTARTS   AGE   IP          NODE       NOMINATED NODE   READINESS GATES
	kubia-6wnj5   1/1     Running   0          56s   10.44.0.3   server02   <none>           <none>
	kubia-788p8   1/1     Running   0          56s   10.44.0.2   server02   <none>           <none>
	kubia-c9kn6   1/1     Running   0          56s   10.44.0.1   server02   <none>           <none>

	$ kubectl delete po kubia-6wnj5
	NAME          READY   STATUS              RESTARTS   AGE
	kubia-6ntgt   0/1     ContainerCreating   0          12s
	kubia-6wnj5   1/1     Terminating         0          3m3s
	kubia-788p8   1/1     Running             0          3m3s
	kubia-c9kn6   1/1     Running             0          3m3s
上面重新列出pod会显示四个， 因为你删除的pod己终止， 并且己创建一个新的pod
虽然ReplicationController会立即收到删除pod的通知 (API 服务器允许客户端监听资源和资源列表的更改），但这不是它创建替代pod的原因。
该通知会触发控制器检查实际的pod数量并采取适当的措施.

	$ kubectl get po -o wide
	NAME          READY   STATUS    RESTARTS   AGE     IP          NODE       NOMINATED NODE   READINESS GATES
	kubia-6ntgt   1/1     Running   0          53s     10.44.0.4   server02   <none>           <none>
	kubia-788p8   1/1     Running   0          3m44s   10.44.0.2   server02   <none>           <none>
	kubia-c9kn6   1/1     Running   0          3m44s   10.44.0.1   server02   <none>           <none>

### 获取有关 ReplicationController 的信息

	$ kubectl get rc -o wide
	$ kubectl get rc -o wide -n default		// RC是针对某个namespace下做的副本pod控制
	NAME    DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES        SELECTOR
	kubia   3         3         3       17m   kubia        luksa/kubia   app=kubia
	获取RC的详细信息
	$ kubectl describe rc kubia

如果你更改了 一个 pod 的标签，使它不再 与 ReplicationController 的标签选择器相匹配 ， 那么该 pod 就变得和其他手动创建的 pod 一样了
更改 pod 的标签时， ReplicationController 发现一个 pod 丢失了 ， 并启动一个新的pod替换它.

给其中一个 pod 添加了 type=special 标签，再次列出所有 pod 会显示和以前一样的三个 pod 。 因为从 ReplicationCon位oiler 角度而言， 没发生任何更改.

	$ kubectl label pod/kubia-6ntgt type=special
	$ kubectl get pod --show-labels
	NAME          READY   STATUS    RESTARTS   AGE   LABELS
	kubia-6ntgt   1/1     Running   0          27m   app=kubia,type=special
	kubia-788p8   1/1     Running   0          30m   app=kubia
	kubia-c9kn6   1/1     Running   0          30m   app=kubia

更改app标签该 pod 不再与 RC 的标签选择器相匹配

	$ kubectl label pod/kubia-6ntgt app=foo --overwrite
	$ kubectl get pod --show-labels
	NAME          READY   STATUS              RESTARTS   AGE   LABELS
	kubia-6ntgt   1/1     Running             0          30m   app=foo,type=special
	kubia-788p8   1/1     Running             0          33m   app=kubia
	kubia-c9kn6   1/1     Running             0          33m   app=kubia
	kubia-dqshz   0/1     ContainerCreating   0          4s    app=kubia
使用 -L app 选项在列 中显示 app 标签

	$ kubectl get pod -L app
	NAME          READY   STATUS    RESTARTS   AGE     APP
	kubia-6ntgt   1/1     Running   0          32m     foo
	kubia-788p8   1/1     Running   0          35m     kubia
	kubia-c9kn6   1/1     Running   0          35m     kubia
	kubia-dqshz   1/1     Running   0          2m17s   kubia
可能有一个 bug 导致你的 pod 在特定时间或特定事件后开始出问题。
如果你知道某个 pod 发生了故障， 就可以将它从 Replication-Controller 的管理范围中移除， 让控制器将它替换为新 pod, 接着这个 pod 就任你处置了。 完成后删除该pod 即可。

### 编辑RC的YAML配置
用默认文本编辑器中打开ReplicationController的YAML配置，会在/tmp目录生成一个临时yaml文件，退出后/tmp目录下的yaml文件也会删掉
如果你想使用nano编辑Kubernetes资源，请执行以下命令（或将其放入 ~/.bashrc或等效文件中）
export KUBE_EDITOR="/usr/bin/nano"

	$ kubectl edit rc kubia
	......
	 spec:
	   replicas: 3
	   selector:
	     app: kubia1				 RC selector 修改，需要配合下面的label一起修改
	   template:
	     metadata:
	       creationTimestamp: null
	       labels:
	         app: kubia1			 Pod label 修改，需要配合上面的 RC selector 一起修改
	     spec:
	       containers:
	       - image: luksa/kubia
	         imagePullPolicy: Always
	         name: kubia
	         ports:
	         - containerPort: 8080
	           protocol: TCP
	......

	$ kubectl get pod
	NAME          READY   STATUS              RESTARTS   AGE   APP
	kubia-279wl   0/1     ContainerCreating   0          2s    kubia1
	kubia-6ntgt   1/1     Running             0          44m   foo
	kubia-788p8   1/1     Running             0          47m   kubia
	kubia-c9kn6   1/1     Running             0          47m   kubia
	kubia-dqshz   1/1     Running             0          14m   kubia
	kubia-m6vml   0/1     Pending             0          2s    kubia1
	kubia-xxjqr   0/1     ContainerCreating   0          2s    kubia1

### RC 扩容
扩展/缩容 RC管理的pod为5个
第一种，commands方式:

	$ kubectl scale rc kubia --replicas=5

第二种， edit rc yaml文件

	$ kubectl edit rc kubia
	......
	spec:
	  replicas: 5
	......

### 删除RC
当使用 kubectl delete 删除 ReplicationController 时， 可以通过给命令增加 --cascade= false 选项来保持 pod 的运行.

	$ kubectl delete rc kubia --cascade=false
已经删除了 ReplicationController, 所以这些 pod 独立了， 它们不再被管理。但是你始终可以使用适当的标签选择器创建新的 ReplicationController, 并再次将它们管理起来


## ReplicaSet
> 最 初， ReplicationController 是用于复制和在异常时重新调度节点的唯 一Kubemetes 组件， 后来又引入了 一个名为 ReplicaSet 的类似资源 。 它是新一代的ReplicationController, 并且将其完全替换掉 (ReplicationController 最终将被弃用）。
> 也就是说从现在起， 你应该始终创建 ReplicaSet 而不是 ReplicationController。 它们几乎完全相同， 所以你不会碰到任何麻烦
> ReplicaSet 的行为与ReplicationController 完全相同， 但pod 选择器的表达能力更强
> ReplicationController 都无法仅基千标签名的存在来匹配 pod, 而ReplicaSet 则可以。 例如， ReplicaSet 可匹配所有包含名为 env 的标签的 pod, 无论ReplicaSet 的实际值是什么（可以理解为 env=*)

kubia-replicaset.yaml

	apiVersion: apps/v1
	kind: ReplicaSet
	metadata:
	  name: kubia
	spec:
	  replicas: 3
	  selector:
	    matchLabels:
	      app: kubia
	  template:
	    metadata:
	      labels:
	        app: kubia
	    spec:
	      containers:
	      - name: kubia
	        image: luksa/kubia
	        ports:
	        - containerPort: 8080
检查replicaset:

	$ kubectl get rs

### matchExpressions选择器
创建个yaml文件
kubia-replicaset-matchexpressions.yaml

	 selector:
	   matchExpressions:
	     - key: app
	       operator: In
	       values:
	         - kubia
每个表达式都必须 包含一个key, 一个operator (运算符），并且可能还有一个values的列表（取决于 运算符）.
• In : Label的值 必须与其中 一个指定的values 匹配。
• Notln : Label的值与任何指定的values 不匹配。
• Exists : pod 必须包含一个指定名称的标签（值不重要）。使用此运算符时，
不应指定 values字段。
• DoesNotExist : pod不得包含有指定名称的标签。values属性不得指定
如果同时指定matchLabels和matchExpressions, 则所有标签都必须匹配，并且所有表达式必须计算为true以使该pod与选择器匹配.

### 查看 replicaset 和 deployment 的详细信息

	$ kubectl describe deployment details-v1
	$ kubectl describe rs details-v1-6fc55d65c9

### 删除ReplicaSet
删除ReplicaSet会删除所有的pod,这种情况下是需要列出pod来确认.

	$ kubectl delete rs kubia

## DaemonSet
如果节点下线， DaemonSet不会在其他地方重新创建pod。 但是， 当将一个新节点添加到集群中时， DaemonSet会立刻部署一个新的pod实例。
如果有人无意中删除了 一个 pod ， 那么它也会重新创建 一个新的 pod。
与 ReplicaSet一样，DaemonSet 从配置的 pod 模板创建 pod.

> 如果节点可以被设置为不可调度的 ， 防止 pod 被部署到节点上. DaemonSet 甚至会将 pod 部署到这些节点上，因为无法调度的属性只会被调度器使用，而 DaemonSet 管理的 pod 则完全绕过调度器. 这是预期的，因为DaemonSet的目的是运行系统服务，即使是在不可调度的节点上，系统服务通常也需要运行.

给node节点打上label

	$ kubectl label node server02 disk=ssd

ssd-monitor-daemonset.yaml

	apiVersion: apps/v1			// DaemooSet在apps的API组 中，版本是v1
	kind: DaemonSet
	metadata:
	  name: ssd-monitor
	spec:
	  selector:
	    matchLabels:
	      app: ssd-monitor
	  template:
	    metadata:
	      labels:
	        app: ssd-monitor
	    spec:
	      nodeSelector:			// pod模板包含 会选择有disk=ssd标签的节点 一个节点选择器
	        disk: ssd
	      containers:
	      - name: main
	        image: luksa/ssd-monitor

	$ kubectl create -f ssd-monitor-daemonset.yaml

### 查看DaemonSet

	$ kubectl get ds
	NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE     CONTAINERS   IMAGES              SELECTOR
	ssd-monitor   1         1         1       1            1           disk=ssd        3m29s   main         luksa/ssd-monitor   app=ssd-monitor
如果你有多个节点并且其他的节点也加上了同样的标签，将会看到 DaemonSet 在每个节点上都启动 pod.
给其中一个节点修改标签disk=hdd, 假设它的硬盘换成磁盘而不是SSD, 那个节点上的pod会如预期中被终止.
如果还有其他的 pod在运行， 删除 DaemonSet 也会一起删除这些 pod。

### 删除ds
删除ds会删除由ds控制schedule到每个节点的pod

	$ kubectl delete ds ssd-monitor

## Job资源
Kubemetes 通过 Job 资源提供了对此的支持，它允许你运行一种 pod, 该 pod 在内部进程成功结束时， 不重启容器。
一旦任务完成， pod 就被认为处于完成状态.
由Job管理的pod会一直被重新安排，直到它们成功完成任务.

exporter.yaml

	apiVersion: batch/v1
	kind: Job
	metadata:
	  name: batch-job
	spec:
	  template:
	    metadata:
	      labels:
	        app: batch-job
	    spec:
	      restartPolicy: OnFailure		// 默认为Always,Job pod不能使用默认策略， 因为它们不是要无限期地运行
	      containers:
	      - name: main
	        image: luksa/batch-job		// 运行luksa/batch-job镜像，该镜像调用 一个运行120秒的进程，然后退出
需要明确地将重启策略 restartPolicy 设置为 OnFailure 或 Never。 此设置防止容器在完成任务时重新启动

	$ kubectl create -f exporter.yaml

	$ kubectl get pod
	NAME                READY   STATUS    RESTARTS   AGE
	batch-job-lhnfg     1/1     Running   0          113s
	
	$ kubectl get jobs
	NAME        COMPLETIONS   DURATION   AGE
	batch-job   0/1           111s       111s

等待两三分钟后

	$ kubectl get pod
	NAME                READY   STATUS      RESTARTS   AGE
	batch-job-lhnfg     0/1     Completed   0          3m21s

	$ kubectl get job
	NAME        COMPLETIONS   DURATION   AGE
	batch-job   1/1           2m41s      3m27s
完成后pod未被删除的原因是允许你查阅其日志

	$ kubectl logs po/batch-job-lhnfg
	Thu May 14 05:04:38 UTC 2020 Batch job starting
	Thu May 14 05:06:38 UTC 2020 Finished succesfully
pod 可以被直接删除， 或者在删除创建它的Job时被删除
作业可以配置为创建多个pod实例，并以并行或串行方式运行它们
在Job配置中设置 completions和parallelism属性来完成的
如果你需要 一个Job运行多次，则可以将comple巨ons设为你希望作业的pod运行多少次

	apiVersion: batch/v1
	kind: Job
	metadata:
	  name: multi-completion-batch-job
	spec:
	  completions: 5	// job将一个接一个地运行五个pod
	  parallelism: 2	// 最多两个pod可以并行运行
	  template:
	    metadata:
	      labels:
	        app: batch-job
	    spec:
	      restartPolicy: OnFailure
	      containers:
	      - name: main
	        image: luksa/batch-job
它最初创建一个pod, 当pod的容器运行完成时，它创建第二个pod, 以此类推，直到五个pod成功完成。
如果其中 一个pod发生故障，工作会创建一个新的pod, 所以Job总共可以创建五个以上的pod.

	$ kubectl create -f multi-completion-batch-job.yaml
	NAME                               READY   STATUS              RESTARTS   AGE
	multi-completion-batch-job-8kzd5   0/1     ContainerCreating   0          4s
	multi-completion-batch-job-9rnxs   0/1     ContainerCreating   0          4s
只要其中 一个pod完成任务，工作将运行下 一个pod, 直到五个pod都成功完成任务.

	kubectl get po
	NAME                               READY   STATUS    RESTARTS   AGE
	multi-completion-batch-job-8kzd5   1/1     Running   0          2m8s
	multi-completion-batch-job-9rnxs   1/1     Running   0          2m8s
	
	$ kubectl get job
	NAME                         COMPLETIONS   DURATION   AGE
	multi-completion-batch-job   0/5           2m13s      2m13s
POD虽然创建，但是POD里的进程任务还没有完成，因此job显示任然是0/5没有一个pod任务完成
再等待一会时间

	$ kubectl get po
	NAME                               READY   STATUS      RESTARTS   AGE
	multi-completion-batch-job-8kzd5   0/1     Completed   0          4m18s
	multi-completion-batch-job-9rnxs   0/1     Completed   0          4m18s
	multi-completion-batch-job-blntb   1/1     Running     0          107s
	multi-completion-batch-job-qhsr5   1/1     Running     0          92s
	ssd-monitor-jbhpd                  1/1     Running     0          176m

	$ kubectl get job
	NAME                         COMPLETIONS   DURATION   AGE
	multi-completion-batch-job   2/5           4m16s      4m16s
如上显示已经有2个POD任务完成，POD退出.
甚至可以在 Job 运行时更改 Job 的 parallelism 属性, command如下，实验环境没有成功使用

	$ kubectl scale job multi-completion-batch-job --replicas 3

> 通过在 pod 配置中设置 activeDeadlineSeconds 属性，可以限制 pod的时间。如果 pod 运行时间超过此时间， 系统将尝试终止 pod, 并将 Job 标记为失败。
> 通过指定 Job manifest 中的 spec.backoff巨m辽字段， 可以配置 Job在被标记为失败之前可以重试的次数。 如果你没有明确指定它， 则默认为6

### 删除job
删除job时，由job创建的pod也被直接删除

	 $ kubectl delete job multi-completion-batch-job

## CornJob 资源
> 批处理任务需要在特定的时间运行，或者在指定的时间间隔内重复运行,在 Linux 和类 UNIX 操作系统中， 这些任务通常被称为 cron 任务。 Kubemetes 也支持这种任务
> Kubemetes 中的 cron 任务通过创建 CronJob 资源进行配置, 运行任务的时间表以知名的 cron 格式指定
时间表从左到右包含以下五个条目
• 分钟
• 小时
• 每月中的第几天
• 月
• 星期几

创建资源文件(kube API 对象文件)cronjob.yaml

	apiVersion: batch/v1beta1
	kind: CronJob
	metadata:
	  name: batch-job-every-fifteen-minutes
	spec:
	  schedule: "0,15,30,55 * * * *"
	  startingDeadlineSeconds: 15	// pod最迟必须在预定时间后15秒开始运行， 如果因为任何原因到该启动时间15s后仍不启动，任务将不会运行，并将显示为Failed
	  jobTemplate:
	    spec:
	      template:
	        metadata:
	          labels:
	            app: periodic-batch-job
	        spec:
	          restartPolicy: OnFailure
	          containers:
	          - name: main
	            image: luksa/batch-job

> 希望每 15 分钟运行一 次任务因此 schedule 字段的值应该是"0, 15, 30, 45****" 这意味着每小时的 0 、 15 、 30和 45 分钟（第一个星号），每月的每一天（第二个星号），每月（第三个星号）和每周的每一天（第四个星号）。
> 相反，如果你希望每隔 30 分钟运行一 次，但仅在每月的第一天运行，则应将计划设置为 "0,30 * 1 * *", 并且如果你希望它每个星期天的 3AM 运行，将它设置为 "0 3 * * 0" (最后一个零代表星期天）。

### 查看cronjob
	$ kubectl get cronjob -o wide
	NAME                              SCHEDULE             SUSPEND   ACTIVE   LAST SCHEDULE   AGE    CONTAINERS   IMAGES            SELECTOR
	batch-job-every-fifteen-minutes   0,15,30,55 * * * *   False     0        18m             112m   main         luksa/batch-job   <none>

### cronjob运行状态
	$ kubectl get po
	NAME                                               READY   STATUS      RESTARTS   AGE
	batch-job-every-fifteen-minutes-1589439300-4v8wd   0/1     Completed   0          36m
	batch-job-every-fifteen-minutes-1589439600-99pns   0/1     Completed   0          31m
	batch-job-every-fifteen-minutes-1589440500-vs5vm   0/1     Completed   0          16m
	batch-job-every-fifteen-minutes-1589441400-52rzb   1/1     Running     0          112s

	$ kubectl get job
	NAME                                         COMPLETIONS   DURATION   AGE
	batch-job-every-fifteen-minutes-1589439300   1/1           2m24s      36m
	batch-job-every-fifteen-minutes-1589439600   1/1           2m24s      31m
	batch-job-every-fifteen-minutes-1589440500   1/1           2m22s      16m
	batch-job-every-fifteen-minutes-1589441400   0/1           116s       116s

再过一点时间查看

	$ kubectl get po
	NAME                                               READY   STATUS      RESTARTS   AGE
	batch-job-every-fifteen-minutes-1589439600-99pns   0/1     Completed   0          32m
	batch-job-every-fifteen-minutes-1589440500-vs5vm   0/1     Completed   0          17m
	batch-job-every-fifteen-minutes-1589441400-52rzb   0/1     Completed   0          2m34s

	$ kubectl get job
	NAME                                         COMPLETIONS   DURATION   AGE
	batch-job-every-fifteen-minutes-1589439600   1/1           2m24s      32m
	batch-job-every-fifteen-minutes-1589440500   1/1           2m22s      17m
	batch-job-every-fifteen-minutes-1589441400   1/1           2m20s      2m37s

总结: CornJob过指定的时间执行一次POD，执行完退出，会保留三个POD和Job记录.

### 删除cronjob
运行中的 Job 将不会被终止，不会删除 Job 或 它们的 Pod。为了清理那些 Job 和 Pod，需要列出该 Cron Job 创建的Job，然后删除它们.

	$ batch-job-every-fifteen-minutes
	cronjob.batch "batch-job-every-fifteen-minutes" deleted

## secret

	$ kubectl get secret -n default
	NAME                  TYPE                                  DATA   AGE
	default-token-gtcjx   kubernetes.io/service-account-token   3      32d

	$ kubectl get pods -o wide
	NAME                               READY   STATUS    RESTARTS   AGE     IP           NODE         NOMINATED NODE   READINESS GATES
	wordpress-7bfc545758-vtfvm         1/1     Running   5          10d     10.36.0.10   hci-node04   <none>           <none>
	wordpress-mysql-764fc64f97-sjnjk   1/1     Running   0          10d     10.36.0.8    hci-node04   <none>           <none>

	$ kubectl get po/wordpress-7bfc545758-vtfvm -o yaml
	......
	spec:
	  containers:
	  - env:
	    volumeMounts:
	    - mountPath: /var/www/html
	      name: wordpress-persistent-storage
	    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
	      name: default-token-gtcjx
	      readOnly: true
	......
	  volumes:
	  - name: wordpress-persistent-storage
	    persistentVolumeClaim:
	      claimName: wp-pv-claim
	  - name: default-token-gtcjx
	    secret:
	      defaultMode: 420		// 访问权限
	      secretName: default-token-gtcjx
	......
进入wordpress-7bfc545758-vtfvm所在机器的容器里查看/var/run/secrets/kubernetes.io/serviceaccount路径文件

	$ docker exec -it daec0458a397 /bin/sh
	# ls /var/run/secrets/kubernetes.io/serviceaccount
	  ca.crt  namespace  token

### 创建自己的Secret
serviceAccount 用来跟Apiserver通信，用来授权, 可以创建自己的Secret
编写Secret配置文件 secret.yaml

	apiVersion: v1
	kind: Secret
	metadata:
	  name: dbpass
	type: Opaque		// 不透明，浑浊的.
	data:
	  username: aW1vb2M=		// base64加密的用户名
	  passwd: aW1vb2MxMjM=		// base64加密的密码
把字符串生成base64很简单，命令如下

	$ echo -n imooc | base64	// -n 表示换行
	aW1vb2M=
编写Pod资源配置文件 pod-secret.yaml

	apiVersion: v1
	kind: Pod
	metadata:
	  name: pod-secret
	spec:
	  containers:
	  - name: springbook-web
	    image: hub.mooc.com/kubernetes/springboot-web:v1
	    ports:
	    - containerPort: 8080
	    volumeMounts:
	    - name: db-secret
	      mountPath: /db-secret
	      readOnly: true
	  volumes:
	  - name: db-secret
	    projected:
	      sources:			// secret 来源
	      - secret:
	        name: dbpass	// secret 名字
生成Pod并进入查看

	$ / # cd /db-secret/
	$ ls
	  passwd username
	$ cat -n username		// 查看容器里存放的是base64解码过的数据
	  immoc
	$ cat -n passwd
	  imooc123

可以通过修改secret.yaml文件修改secret账号密码等再$ kubectl apply -f secret.yaml来更改密码.

## Configmap
> configmap常用来存储不需要加密的数据, 比如应用的启动参数，一些参数的配置等
 * 第一种向k8s添加很多key value的键值对属性值，就可以用configmap


	$ touch game.properties
	$ vim game.properties
	  enemies=aliens
	  lives=3
	  enemies.cheat=true
	  secret.code.allowed=true
	  ......
配置到K8S里

	$ kubectl create configmap web-game --from-file game.properties
	$ kubectl get cm

使用configmap, Pod-game.yaml

	apiVersion: v1
	kind: Pod
	metadata:
	  name: pod-game
	spec:
	  containers:
	  - name: web
	    image: hub.mooc.com/kubernetes/springboot-web:v1
	    ports:
	    - containerPort: 8080
	    volumeMounts:
	    - name: game
	      mountPath: /etc/config/game
	      readOnly: true
	  volumes:
	  - name: game
	    configMap:
	      name: web-game
生成Pod并进入查看

	$ cd /etc/config/game
	$ ls
	  game.properties
	$ cat game.properties
	  enemies=aliens
	  lives=3
	  enemies.cheat=true
	  secret.code.allowed=true
	  ......

可以通过kubectl edit 修改configMap账号密码等

	$ kubectl edit cm web-game -o yaml
	  enemies.cheat=false	//等等操作

 * 第二种配置文件方式创建configMap
configmap.yaml


	apeVersion: v1
	kind: Configmap
	metadata:
	  name: configs
	data:
	  Java_OPTS: -Xms1024m
	  LOG_LEVEL: DEBUG

	$ kubectl create -f configmap.yaml
编写资源配置文件pod-env.yaml

	apiVersion: v1
	kind: Pod
	metadata:
	  name: pod-env
	spec:
	  containers:
	  - name: web
	    image: hub.mooc.com/kubernetes/springboot-web:v1
	    ports:
	    - containerPort: 8080
	    env:
	      - name: LOG_LEVEL_CONFIG
	        valueFrom:
	          configMapKeyRef:
	            name: configs		// 指定configMap名字
	            key: LOG_LEVEL		// configs下面的LOG_LEVEL
进入容器查看环境变量

	$ env | grep LOG
	  LOG_LEVEL_CONFIG=DEBUG
之后次容器就可以通过环境变量获取值

 * 第三种 通过命令行方式传进参数
也是先跟第二种一样创建configMap资源


	$ kubectl create -f configmap.yaml
编写资源配置文件pod-cmd.yaml

	apiVersion: v1
	kind: Pod
	metadata:
	  name: pod-cmd
	spec:
	  containers:
	  - name: web
	    image: hub.mooc.com/kubernetes/springboot-web:v1
	    command: ["/bin/sh", "-c", "java -jar /springboot-web.jar -DJAVA_OPTS=$(JAVA_OPTS)"]
	    ports:
	    - containerPort: 8080
	    env:
	      - name: Java_OPTS
	        valueFrom:
	          configMapKeyRef:
	            name: configs		// 指定configMap名字
	            key: Java_OPTS		// configs下面的LOG_LEVEL
进入容器查看进程

	$ ps -ef
	  java -jar /springboot-web.jar -DJAVA_OPTS=-Xms1024m

## downwardAPI
downwardAPI主要作用是在程序中取得Pod对象本身的一些相关信息
pod-downwardapi.yaml

	apiVersion: v1
	kind: Pod
	metadata:
	  name: pod-downwardapi
	  labels:
	    app： downwardapi
	    type: webapp
	spec:
	  containers:
	  - name: web
	    image: hub.mooc.com/kubernetes/springboot-web:v1
	    ports:
	    - containerPort: 8080
	    volumeMounts:
	      - name: podinfo
	        mountPath: /etc/podinfo
	  volumes:
	    - name: podinfo
	      projected:
	        sources:
	        - downwardAPI:
	          items:
	            - path: "labels"
	              fieldRef:
	                fieldPath: metadata.labels
	            - path: "name"
	              fieldRef:
	                fieldPath: metadata.name
	            - path: "namespace"
	              fieldRef:
	                fieldPath: metadata.namespace
	            - path: "mem-request"
	              resourceFieldRef:
	                containerName: web
	                resource: limits.memory
进入容器查看文件信息

	$ cd /etc/podinfo
	$ ls -l
	  labels mem-request name namespace
	$ cat -n labels
	  app="downwardapi"
	  type="webapp"
	$ cat -n namespace
	  default
	$ cat -n name
	  pod-downwardapi










