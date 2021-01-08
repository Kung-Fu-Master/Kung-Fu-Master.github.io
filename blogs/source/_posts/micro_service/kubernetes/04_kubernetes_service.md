---
title: 04 Kubernetes service
tags: kubernetes
categories:
- microService
- kubernetes
top:
---

## Kubernetes Service
> Kubemetes 服务是一种为一组功能相同的 pod 提供单一不变的接入点的资源.
> 当服务存在时，它的 IP 地址和端口不会改变。 客户端通过 IP 地址和端口号建立连接，这些连接会被路由到提供该服务的任意一个 pod上.
> 通过这种方式， 客户端不需要知道每个单独的提供服务的 pod 的地址， 这样这些 pod 就可以在集群中随时被创建或移除.

通过为前端 pod 创建服务， 并且将其配置成可以在集群外部访问，可以暴露一个单一不变的 IP 地址让外部的客户端连接 pod。 
同理，可以为后台数据库 pod 创建服务，并为其分配一个固定的 IP 地址。尽管 pod 的 IP 地址会改变，但是服务的 IP 地址固定不变。
另外，通过创建服务，能够让前端的 pod 通过环境变量或 DNS 以及服务名来访问后端服务
Pod 控制器中使用标签选择器来指定哪些 pod 属于同一 Service。

## service
> 如果 pod 的标签与服务的 pod 选择器相匹配，那么 pod 就将作为服务的后端.只要创建了具有适当标签的新 pod ，它就成为服务的一部分，并且请求开始被重定向到 pod.
如下所示Service和POD都采用命名端口的方式, 最大的好处就是即使更换spec pod中的端口号也无须更改服务 spec.
 * 第一步，创建service
创建service yaml文件 kubia-svc.yaml


```xml
	apiVersion: v1
	kind: Service
	metadata:
	  name: kubia
	sepc:
	// sessionAffinity: ClientIP	// 默认此值是None, 若改为ClientIP，则SVC接受到的请求连接只会固定转发给同一个pod
	  ports:
	  - name: http			// 端口别名，可以当作端口号用
	    port: 80			// 该服务可用的端口
	    targetPort: http	// 服务将连接转发到的POD端口, pod需要将http映射pod本身的8080或其它端口，否则这里只能填写端口号
	  - name: https
	    port: 443
	    targetPort: https	// 含有label:app=kubia的pod需要将https映射pod本身8443或其它端口，否则这里只能填写端口号
	  selector:
	    app: kubia			// 具有app=kubia标签的pod都属于该服务
```

创建了 一个名叫kubia的服务，它将在端口80接收请求并将连接路由到具有标签选择器是app=kubia的pod的8080端口上.
在发布完YAML文件后， 可以在命名空间下列出来所有的服务资源, 新的服务已经被分配了一个内部集群IP, 只能在集群内部可以被访问.

```shell
	$ kubectl get svc
	NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
	kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   23h
	kubia        ClusterIP   10.98.229.76   <none>        80/TCP    20s
```

 * 第二步，创建两个pod，一个添加标签app=kubia，另一个用来执行测试通过kubectl exec来访问第一个pod
kubia.yaml


```xml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: kubia	// name: kubia1; name: kubia2
	spec:
	  nodeSelector:		// pod被分配到含有标签gpu=true的node上，当然也可以注释掉这两行
	    gpu: "true"
	  containers:
	  - image: luksa/kubia
	    name: kubia
```

kubia-label.yaml

```xml
	 apiVersion: v1					// api服务版本
	 kind : Pod						// 资源类型
	 metadata:
	   name: kubia-label			// pod 名字
	   labels:
	     app: kubia					// pod添加label
	 spec :
	   nodeSelector:
	     gpu: "true"				// node 选择器
	   containers:
	   - image: luksa/kubia			// image 名字
	     name: kubia				// container 名字
	     ports:
	     - name: http				// pod端口映射，用http名字代替8080，名字随便取, 可以跟上面的service的targetPort对应起来
	       containerPort: 8080		// 用上面的名字定义这个端口号的别名
	     - name: https
	       containerPort: 8443
```

查看POD并执行一个POD去通过上面创建的service(通过label)包含的pod提供的服务.
其中pod kubia-label中container运行的服务进程监听了8080端口, POD对外也暴露了8080端口

```shell
	$ kubectl get pod --show-labels
	NAME           READY   STATUS    RESTARTS   AGE    LABELS
	kubia          1/1     Running   0          101m   <none>
	kubia-label    1/1     Running   0          98m    app=kubia
	kubia-label1   1/1     Running   0          99s    app=kubia
	kubia-label2   1/1     Running   0          79s    app=kubia
```

 * 第三步: 执行一个pod用curl命令访问另一个pod提供的服务
双横杠(--)代表着kubectl命令项的结束.在两个横杠之后的内容是指在pod内部需要执行的命令.
k8s 服务代理接续curl请求连接，三个包含label为app=kubia的pod任意选择一个pod
访问服务三种方式,加不加端口都可以


```shell
	<p>$ kubectl exec kubia -- curl -s http://10.98.229.76:http</p>
	$ kubectl exec kubia -- curl -s http://10.98.229.76:80
	$ kubectl exec kubia -- curl -s http://10.98.229.76
	You've hit kubia-label

	$ kubectl exec kubia -- curl -s http://10.98.229.76
	You've hit kubia-label2
	$ kubectl exec kubia -- curl -s http://10.98.229.76
	You've hit kubia-label1
```

### Affinity 亲和性
Kubernetes 仅仅支持两种形式的会话亲和性服务： None 和 ClientIP
这种方式将会使服务代理将来自同 一个 client IP 的所有请求转发至同 一个 pod上.
Kubernetes 服务不是在 HTTP 层面上工作。服务处理 TCP 和 UDP 包，并不关心其中的载荷内容。
因为 cookie 是 HTTP 协议中的一部分，服务并不知道它们，这就解释了为什么会话亲和性不能基千 cookie。
如果希望特定客户端产生的所有请求每次都指向同 一个 pod, 可以设置服务的 sessionAffinity 属性为 ClientIP (而不是 None,None 是默认值）

```xml
	apiVersion: vl
	kind: Service
	spec:
	  sessionAffinity: ClientIP
	......
```

## 环境变量发现service
### 创建replicaSet 管理 3 个 POD

```xml
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
	      nodeSelector:
	        gpu: "true"
	      containers:
	      - name: kubia
	        image: luksa/kubia
	        ports:
	        - name: http
	          containerPort: 8080
	        - name: https
	          containerPort: 8443
```

```shell
	$ kubectl get svc 
	NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
	kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          29h
	kubia        ClusterIP   10.111.88.195   <none>        80/TCP,443/TCP   3h46m
```

查看pod所在的service对应的IP和端口

```shell
	$ kubectl exec kubia-5rvfq env
	......
	KUBERNETES_SERVICE_PORT=443
	KUBIA_SERVICE_PORT=80				// 服务的集群IP
	......
	KUBERNETES_SERVICE_HOST=10.96.0.1
	KUBIA_SERVICE_HOST=10.111.88.195	// 服务所在的端口
	......
```
pod 是否使用 内 部的 DNS 服务器是根据 pod 中 spec 的 dnsPolicy 属性来决定的

进入容器后执行如下命令

```shell
	$ kubectl exec kubia-5rvfq -it -- bash		// -- 表示kubectl 命令执行完了，开始执行pod容器里要运行的命令
	$ curl http://kubia.default.svc.cluster.local
	$ curl http://kubia.default
	$ curl http://kubia
	You've hit kubia-5rvfq

	$ cat /etc/resolv.conf
	nameserver 10.96.0.10		// 对应kube-system 里的服务kube-dns服务IP
	search default.svc.cluster.local svc.cluster.local cluster.local sh.intel.com
	options ndots:5
	root@kubia-5rvfq:/# curl http://svc.cluster.local
	curl: (6) Could not resolve host: svc.cluster.local

	$ ping kubia
	PING kubia.default.svc.cluster.local (10.111.88.195): 56 data bytes
	^C--- kubia.default.svc.cluster.local ping statistics ---
	4 packets transmitted, 0 packets received, 100% packet loss
```

上面的 curl 这个服务是工作的，但是却 ping 不通。这是 因为服务的集群 IP 是一个虚拟 IP，并且只有在与服务端口结合时才有意义。 

查看kube-system下面kube-dns信息

```shell
	$ kubectl get svc -n kube-system
	NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
	kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   2d3h
```

### 删除service

```shell
	$ kubectl delete svc kubia
```

## Service samples
### 查看service

```shell
	$ kubectl get svc -n kube-system
	$ kubectl get svc -n istio-system
	NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                                                      AGE
	grafana                     ClusterIP      10.104.1.236     <none>        3000/TCP                                                                                                                                     3d5h
	istio-egressgateway         ClusterIP      10.107.177.52    <none>        80/TCP,443/TCP,15443/TCP                                                                                                                     3d5h
	istio-ingressgateway        LoadBalancer   10.97.82.221     <pending>     15020:31237/TCP,80:31556/TCP,443:30614/TCP,15029:32511/TCP,15030:32423/TCP,15031:30670/TCP,15032:30961/TCP,31400:30196/TCP,15443:31028/TCP   3d5h
	istio-pilot                 ClusterIP      10.97.192.70     <none>        15010/TCP,15011/TCP,15012/TCP,8080/TCP,15014/TCP,443/TCP                                                                                     3d5h
	istiod                      ClusterIP      10.107.202.199   <none>        15012/TCP,443/TCP
	......

	$ kubectl get services
	NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
	kubernetes                  ClusterIP      10.3.240.l       <none>        443/TCP          34m
	kubia-http                  LoadBalancer   10.3.246.185     <pending>     8080:31348/TCP   4s
	暂时忽略 kubernetes 服务，仔细查看创建的kubian-http 服务 。 它还没有外部 IP 地址 ，因为 Kubernetes 运行的云基础设施创建负载均衡需要一段时间
	$ kubectl get services
	NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)          AGE
	kubernetes                  ClusterIP      10.3.240.l       <none>            443/TCP          34m
	kubia-http                  LoadBalancer   103.246.185      104 155.74.57     8080:31348/TCP   4s
	现在有外部 IP 了，应用就可以从任何地方通过 http://104.155.74.57:8080 访问
	$ curl 104.155.74.57:8080
	You’ve hit kubia-4jfyf
```

### 查看service的CRD信息

```shell
	$ kubectl get svc istio-ingressgateway -n istio-system -oyaml
```

## endpoint 服务

```shell
	$ kubectl describe svc kubia
	Name:              kubia
	Namespace:         default
	Labels:            <none>
	Annotations:       <none>
	Selector:          app=kubia		// 用于创建endpoint列表的服务pod选择器
	Type:              ClusterIP
	IP:                10.111.88.195
	Port:              http  80/TCP
	TargetPort:        8080/TCP
	Endpoints:         10.44.0.1:8080,10.44.0.2:8080,10.44.0.3:8080		// 服务endpoint的pod的IP和端口列表
	Port:              https  443/TCP
	TargetPort:        8443/TCP
	Endpoints:         10.44.0.1:8443,10.44.0.2:8443,10.44.0.3:8443
	Session Affinity:  ClientIP
	Events:            <none>

	$ kubectl get po -o wide
	NAME          READY   STATUS    RESTARTS   AGE   IP          NODE       NOMINATED NODE   READINESS GATES
	kubia         1/1     Running   0          15h   10.44.0.4   server02   <none>           <none>
	kubia-5rvfq   1/1     Running   0          15h   10.44.0.2   server02   <none>           <none>
	kubia-8cgnm   1/1     Running   0          15h   10.44.0.1   server02   <none>           <none>
	kubia-8kv8d   1/1     Running   0          15h   10.44.0.3   server02   <none>           <none>
```
Endpoint 资源和其他Kubernetes 资源一样，所以可以使用 kubectl info 来获取它的基本信息

```shell
	$ kubectl get endpoints kubia
	NAME    ENDPOINTS                                                  AGE
	kubia   10.44.0.1:8443,10.44.0.2:8443,10.44.0.3:8443 + 3 more...   16h
```

Endpoint是一个单独的资源并不 是服务的一个属性, 必须手动创建
external-service.yaml

```xml
	apiVersion: v1
	kind: Service
	metadata:
	  name: external-service
	spec:
	  ports:
	  - port: 80
```

external-service-endpoints.yaml

```xml
	apiVersion: v1
	kind: Endpoints
	metadata:
	  name: external-service
	subsets:
	  - addresses:
	    - ip: 11.11.11.11
	    - ip: 22.22.22.22
	    ports:
	    - port: 80
```
部署service和endpoint

```shell
	$ kubectl create -f external-service.yaml
	$ kubectl create -f external-service-endpoints.yaml
	$ kubectl describe svc/external-service
	Name:              external-service
	Namespace:         default
	Labels:            <none>
	Annotations:       <none>
	Selector:          <none>
	Type:              ClusterIP
	IP:                10.97.153.150
	Port:              <unset>  80/TCP
	TargetPort:        80/TCP
	Endpoints:         11.11.11.11:80,22.22.22.22:80
	Session Affinity:  None
	Events:            <none>
```

## 暴露service
 • 将服务的类型设置成NodePort -- 每个集群节点都会在节点上打开一个端口， 对于NodePort服务， 每个集群节点在节点本身（因此得名叫NodePort)上打开一个端口，并将在该端口上接收到的流量重定向到基础服务。
   该服务仅在内部集群 IP 和端口上才可访间， 但也可通过所有节点上的专用端口访问.
 • 将服务的类型设置成LoadBalance, NodePort类型的一种扩展 -- 这使得服务可以通过一个专用的负载均衡器来访问， 这是由Kubernetes中正在运行的云基础设施提供的。 负载均衡器将流量重定向到跨所有节点的节点端口。
   客户端通过负载均衡器的 IP 连接到服务
 • 创建一 个Ingress资源， 这是一 个完全不同的机制， 通过一 个IP地址公开多个服务——它运行在 HTTP 层（网络协议第 7 层）上， 因此可以提供比工作在第4层的服务更多的功能


### NodePort 类型 service
指定端口不是强制性的。 如果忽略它，Kubemetes将选择一个随机端口.
客户端发送请求的节点并不重要, 整个互联网可以通过任何节点上的30123(用户自己定义的)端口访问到pod,如下所示
如在个人机器上生成如下service和pod，可以在以前做的项目的任何机器上通过如下访问

```shell
	curl -s http://10.239.140.186:30123 (master节点的NodeIP:port)访问服务
	curl -s http://10.239.140.200:30123 (worker02节点的NodeIP:port)访问服务
```

kubia-svc-nodeport.yaml

```xml
	apiVersion: v1
	kind: Service
	metadata:
	  name: kubia-nodeport
	spec:
	  type: NodePort			// 设置服务类型
	  ports:
	  - port: 80				// 服务集群IP端口号
	    targetPort: 8080
	    nodePort: 30123			// 通过集群节点(master或worker)的NodeIP，加上30123端口可以访问服务
	  selector:
	    app: kubia
```

```shell
	$ kubectl create -f kubia-svc-nodeport.yaml
	
	$ kubectl get svc
	NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
	kubernetes           ClusterIP      10.96.0.1       <none>        443/TCP          43h
	kubia-nodeport       NodePort       10.103.7.50     <none>        80:30123/TCP     99m
	
	$ kubectl describe svc kubia-nodeport
	Name:                     kubia-nodeport
	Namespace:                default
	Labels:                   <none>
	Annotations:              <none>
	Selector:                 app=kubia
	Type:                     NodePort
	IP:                       10.103.7.50
	Port:                     <unset>  80/TCP
	TargetPort:               8080/TCP
	NodePort:                 <unset>  30123/TCP
	Endpoints:                10.44.0.1:8080,10.44.0.2:8080,10.44.0.3:8080
	Session Affinity:         None
	External Traffic Policy:  Cluster
	Events:                   <none>

	$ kubectl get po -o wide
	NAME          READY   STATUS    RESTARTS   AGE   IP          NODE       NOMINATED NODE   READINESS GATES
	kubia         1/1     Running   0          17h   10.44.0.4   server02   <none>           <none>
	kubia-5rvfq   1/1     Running   0          17h   10.44.0.2   server02   <none>           <none>
	kubia-8cgnm   1/1     Running   0          17h   10.44.0.1   server02   <none>           <none>
	kubia-8kv8d   1/1     Running   0          17h   10.44.0.3   server02   <none>           <none>
```
查看server02机器IP

```shell
	$kubectl get node -o wide
	NAME       STATUS   ROLES    AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
	alpha      Ready    master   2d18h   v1.18.2   10.239.140.186   <none>        Ubuntu 18.04.4 LTS   5.3.0-28-generic    docker://19.3.6
	server02   Ready    <none>   2d17h   v1.18.2   10.239.140.200   <none>        Ubuntu 18.04.4 LTS   4.15.0-76-generic   docker://19.3.6
```
两种访问方式
 * 第一种: 通过NodeIP:Port 访问:


```shell
	$ curl -s http://10.239.140.200:30123
	You've hit kubia-8kv8d
	$ curl -s http://10.239.140.186:30123
	You've hit kubia-5rvfq
```
 * 第二种: 通过 service的CLUSTER-IP：port 进入port进行访问


```shell
	$ kubectl exec kubia -- curl -s http://10.103.7.50:80
	You've hit kubia-8kv8d
```

### LoadBalancer 方式访问
> 如果Kubemetes在不支持Load Badancer服务的环境中运行， 则不会调配负载平衡器， 但该服务仍将表现得像 一 个NodePort服 务。 这是因为LoadBadancer服务是NodePo江服务的扩展
如果没有指定特定的节点端口， Kubernetes将会选择一个端口
创建服务后， 云基础架构需要一段时间才能创建负载均衡器并将其 IP 地址写入服务对象。 一旦这样做了， IP 地址将被列为服务的外部 IP 地址

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

```shell
	$ kubectl create -f kubia-svc-loadbalancer.yaml

	$ kubectl describe svc/kubia-loadbalancer
	Name:                     kubia-loadbalancer
	Namespace:                default
	Labels:                   <none>
	Annotations:              <none>
	Selector:                 app=kubia
	Type:                     LoadBalancer
	IP:                       10.99.184.62
	Port:                     <unset>  80/TCP
	TargetPort:               8080/TCP
	NodePort:                 <unset>  30994/TCP		// yaml资源文件里没有指定, Kubemetes将会选择一个端口
	Endpoints:                10.44.0.1:8080,10.44.0.2:8080,10.44.0.3:8080
	Session Affinity:         None
	External Traffic Policy:  Cluster
	Events:                   <none>

	$ kubectl get svc
	NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
	kubernetes           ClusterIP      10.96.0.1       <none>        443/TCP          43h
	kubia-loadbalancer   LoadBalancer   10.99.184.62    <pending>     80:30994/TCP     4m11s
```

可以看到Kubemetes在不支持Load Badancer服务的环境中运行 EXTERNAL-IP显示为 <pending>状态，但仍然可以像NodePort方式一样访问服务

```shell
	$ kubectl exec kubia -- curl -s http://10.99.184.62:80
	You've hit kubia-8cgnm
	$ curl -s 10.239.140.186:30994		// masterIP：svcPort
	You've hit kubia-5rvfq
	$ curl -s 10.239.140.200:30994		// worker01：svcPort
	You've hit kubia-8kv8d
```
如果支持LoadBalancer且获得EXTERNAL-IP为 130.211.53.173
可以通过 $ curl http://130.211.53.173 进行访问


### Ingress 暴露服务
> 需要 Ingress一个重要的原因是每个 LoadBalancer 服务都需要自己的负载均衡器， 以及独有的公有 IP 地址， 而 Ingress 只需要一个公网 IP 就能为许多服务提供访问
> Ingress 在网络栈 (HTTP) 的应用层操作， 并且可以提供一 些服务不能实现的功能， 诸如基于 cookie 的会话亲和性 (session affinity) 等功能
> Ingress 对象提供的功能之前，必须强调只有 Ingress控制器在集群中运行，Ingress 资源才能正常工作。 不同的 Kubernetes 环境使用不同的控制器实现， 但有些并不提供默认控制器
Ingress通常向外暴露 Service.Type=NodePort 或者 Service.Type=LoadBalancer 类型的服务，因此先创建一个NodePort类型svc.

```shell
	$ kubectl get svc
	NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
	kubia-nodeport        NodePort       10.103.7.50     <none>        80:30123/TCP     144m
```
创建kubia-ingress.yaml资源文件

```xml
	apiVersion: networking.k8s.io/v1beta1
	kind: Ingress
	metadata:
	  name: kubia
	spec:
	  rules:
	  - host: kubia.example.com				// Ingress 将域名kubia.example.com映射到您的服务
	    http:
	      paths:
	      - path: /
	        backend:
	          serviceName: kubia-nodeport	// 将所有请求发送到kubia-nodeport服务的80端口
	          servicePort: 80
```

> kubectl创建ingress.yaml资源文件遇到webhook ...错误时修改master节点上/etc/kubernetes/manifests/kube-apiserver.yaml，将K8s默认用系统配置的proxy注释掉，稍后再运行kubectl create ...就可以了

```shell
	$ kubectl create -f kubia-ingress.yaml
	$ kubectl get ingress		// 自己机器上没有获得ADDRESS这列IP
	NAME    CLASS    HOSTS               ADDRESS            PORTS   AGE
	kubia   <none>   kubia.example.com   192.168.99.100     80      92s
```
> 一旦知道 IP 地址，通过配置 DNS 服务器将 kubia.example.com 解析为此 IP地址，或者在/ect/hosts(Windows系统为C:\windows\system32\drivers\etc\hosts ）文件中添加下面一行内容：

```
	192 168.99.100 kubia.example.com
```
通过Ingress访问pod, 环境都己经建立完毕，可以通过 http ：此ubia.example.com 地址访 问服务 （使用浏览器或者 curl 命令）

```shell
	$ curl http://kubia.example.com
```
> 客户端如何通过 Ingress 控制器连接到 其 中 一个 pod。客户端首先对 kubia.example.com 执行 DNS 查 找， DNS 服务器（或本地操作系统）返回了In gress 控制器的 IP。
> 客户端然后 向 Ingress 控制器发送 HTTP 请求，并在 Host 头中指定 kubia . example.com。
> 控制器从该头部确定客户端尝试访 问哪个服务，通过与该服务关联 的 Endpo int 对象查看 pod IP ， 并将客户端的请求转发给其中一个pod。
> Ingress 控制器不会将请求转发给该服务，只用它来选择一个pod。大多数（即使不是全部）控制器都是这样工作的.

Ingress规范的 rules 和 paths 都是数组，因此它们可以包含多个条目 
一个 Ingress 可以将 多个主机和路径映射到多个服务
 * 客户端可以通过一个 IP 地址（ Ingress 控制器的 IP 地址 〉访问两种不同的服务
 * 同样，可以使用 Ingress 根据 HTTP 请求中的主机而不是（仅）路径映射到不同的服务
kubia-ingress.yaml


```xml
	apiVersion: networking.k8s.io/v1beta1
	kind: Ingress
	metadata:
	  name: kubia
	spec:
	  rules:
	  - host: kubia.example.com				// 对 kubia.example.com 的请求将会转发至kubia服务
	    http:
	      paths:
	      - path: /
	        backend:
	          serviceName: kubia-nodeport
	          servicePort: 80
	      - path: /kubia					// 对 kubia.example.com/kubia 的请求将会转发至kubia服务
	        backend:
	          serviceName: kubia
	          servicePort: 80
	      - path: /foo						// 对 kubia.example.com/foo 的请求将会转发至bar服务
	        backend:
	          serviceName: bar
	          servicePort: 80
	  - host: bar.example.com				// 对 bar.example.com 的请求将会转发至bar服务
	    http:
	      paths:
	      - path: /
	        backend:
	          serviceName: bar
	          servicePort: 80
```

```shell
	$ kubectl get ingress
	NAME    CLASS    HOSTS                               ADDRESS   PORTS   AGE
	kubia   <none>   kubia.example.com,bar.example.com             80      3s
```
DNS 需要将 foo .example.com 和 bar.example.com 域名都指向 Ingress 控制器的 IP 地址.
然后像上面一样配置/ect/hosts文件内容,就可以通过域名访问了

### Ingress处理TLS传输
配置 Ingress 以支持 TLS, Ingress 转发 HTTP 流量.



## readiness Probe 就绪探针
> 了解了存活探针，以及它们如何通过确保异常容器自动重启来保持应用程序的正常运行 。 与存活探针类似， Kubernetes 还允许为容器定义准备就绪探针
> 就绪探测器会定期调用，并确定特定的 pod 是否接收客户端请求 。 当容器的准备就绪探测返回成功时，表示容器己准备好接收请求 
就绪探针有三种类型:
 * Exec 探针，执行进程的地方。容器的状态由进程的退出状态代码确定 。
 * HTTP GET 探针，向容器发送 HTTP GET 请求，通过响应的 HTTP 状态代码判断容器是否准备好 。
 * TCP socket 探针，它打开一个 TCP 连接到容器的指定端口。如果连接己建立，则认为容器己准备就绪 

启动容器时，可以为 Kubernetes 配置一个等待时间，经过等待时间后才可以执行第一次准备就绪检查。
之后，它会周期性地调用探针，并根据就绪探针的结果采取行动。
如果某个 pod 报告它尚未准备就绪，则会从该服务中删除该 pod。如果 pod再次准备就绪，则重新添加 pod.
存活探针通过杀死异常的容器并用新的正常容器替代它们来保持 pod 正常工作，
就绪探针确保只有准备好处理请求的 pod 才可以接收它们（请求）
> 设想一组pod (例如， 运行应用程序服务器的pod)取决于另 一 个pod (例如，后端数据库）提供的服务。 如果任何一个前端连接点出现连接间题并且无法再访问数据库， 那么就绪探针可能会告知Kubemet es该pod没有准备好处理任何请求。 如果其他pod实例没有遇到类似的连接问题， 则它们可以正常处理请求。 就绪探针确保客户端只与正常的pod交互， 并且永远不会知道系统存在问题.
通过kubectl ed江命令来向已存在的ReplicationController中的pod模板添加探针
kubia-replicaset-renameport.yaml

```xml
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
	      nodeSelector:
	        gpu: "true"
	      containers:
	      - name: kubia
	        image: luksa/kubia
	        readinessProbe:			// pod中的每个容器都会有一个就绪探针
	          exec:
	            command:
	            - ls
	            - /var/ready
	        ports:
	        - name: http
	          containerPort: 8080
	        - name: https
	          containerPort: 8443
```

创建RS资源并查看READY状态

```shell
	$ kubectl create -f kubia-replicaset-renameport.yaml
	$ kubectl get po
	NAME          READY   STATUS    RESTARTS   AGE
	kubia-2xk54   0/1     Running   0          6m20s
	kubia-bj4tz   0/1     Running   0          6m20s
	kubia-hr9bx   0/1     Running   0          6m21s
```

通过创建/var/ready文件使其中一个文件的就绪探针返回成功，该文件的存在可以模拟就绪探针成功
准备就绪探针会定期检查 默认情况下每 10 秒检查一次, 最晚 10 秒钟内， 该 pod 应该已经准备就绪.

```shell
	$ kubectl exec po/kubia-2xk54 -- touch /var/ready
	$ kubectl get po
	NAME          READY   STATUS    RESTARTS   AGE
	kubia-2xk54   1/1     Running   0          6m26s
	kubia-bj4tz   0/1     Running   0          6m26s
	kubia-hr9bx   0/1     Running   0          6m27s

	$ kubectl describe po/kubia-2xk54
	......
	Readiness:      exec [ls /var/ready] delay=0s timeout=1s period=10s #success=1 #failure=3
	......
```

修改创建过的RS的资源文件里的readiness命令是不生效的, 除非删了重建, 查看readiness Probe如下

```shell
	$ kubectl edit rc kubia
```

### 再次测试 readiness

```shell
	$ kubectl get po
	NAME          READY   STATUS    RESTARTS   AGE
	kubia-2xk54   1/1     Running   0          6m20s
	kubia-bj4tz   0/1     Running   0          6m20s
	kubia-hr9bx   0/1     Running   0          6m21s

	$ exec kubia-2xk54 -- rm -rf /var/ready		// 过大概10s后

	$ kubectl get po
	NAME          READY   STATUS    RESTARTS   AGE
	kubia-2xk54   0/1     Running   0          6m30s
	kubia-bj4tz   0/1     Running   0          6m30s
	kubia-hr9bx   0/1     Running   0          6m31s

	$ kubectl exec kubia-bj4tz -- touch /var/ready	// 过大概10s后
	$ kubectl get svc
	NAME          READY   STATUS    RESTARTS   AGE   IP          NODE       NOMINATED NODE   READINESS GATES
	kubia-2xk54   0/1     Running   0          6m40s   10.44.0.2   server02   <none>           <none>
	kubia-bj4tz   1/1     Running   0          6m40s   10.44.0.3   server02   <none>           <none>
	kubia-hr9bx   0/1     Running   0          6m40s   10.44.0.1   server02   <none>           <none>
```
查看SVC

```shell
	$ kubectl get svc	// 也可以通过kubectl exec kubia-bj4tz env 来查看POD所支持的所有SVC的IP等信息
	NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
	kubia                 ClusterIP      10.111.88.195   <none>        80/TCP,443/TCP   22h
	kubia-loadbalancer    LoadBalancer   10.102.224.77   <pending>     80:32671/TCP     3h8m
```

两种访问方式，POD和Node
第一种POD访问SvcIP:SvcPort方式

```shell
	$kubectl exec kubia -- curl -s  http://10.111.88.195:80		// SvcIP:SvcPort, SvcPort映射到PodIP, 可以通过describe svc查看
	You've hit kubia-bj4tz
```
第二种NodeIP:NodePOrt方式

```shell
	$ curl -s 10.239.140.186:32671		// NodeIP:NodePort
	You've hit kubia-bj4tz
```

应该通过删除 pod 或更改 pod 标签而不是手动更改探针来从服务中手动移除pod.
如果想要从某个服务中手动添加或删除 pod, 请将 enabled=true 作为标签添加到 pod, 以及服务的标签选择器中。 当想要从服务中移除 pod 时，删除标签
应该始终定义一 个就绪探针， 即使它只是向基准 URL 发送 HTTP 请求一样简单。

## headless 服务
让客户端连接到所有 pod, 需要找出每个 pod 的 IP.Kubemetes 允许客户通过 DNS 查找发现 pod IP.
但是对千 headless 服务， 由于 DNS 返回了 pod 的 IP,客户端直接连接到该 pod, 而不是通过服务代理.
headless 服务仍然提供跨 pod 的负载平衡， 但是通过 DNS 轮询机制不是通过服务代理.
> 如果告诉Kubemetes, 不需要为服务提供集群 IP (通过在服务 spec 中将 clusterIP 字段设置为 None 来完成此操作）， 则 DNS 服务器将返回 pod IP 而不是单个服务 IP
> 将服务 spec中的clusterIP字段设置为None 会使服务成 为headless 服务，因为Kubemetes 不会 为其分配集群IP, 客户端可通过该IP将其连接到支持它的pod
kubia-svc-headless.yaml

```xml
	apiersion: v1
	kin: Service
	metdata:
	  nme: kubia-headless
	spe:
	  custerIP: None		// clusterIP字段设置为None 会使服务成 为headless服务
	  prts:
	  -port: 80
	   targetPort: 8080
	  slector:
	    app: kubia

	$ kubectl create -f kubia-svc-headless.yaml
	$ kubectl get svc
	NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
	kubia-headless        ClusterIP      None            <none>        80/TCP           103s
```




