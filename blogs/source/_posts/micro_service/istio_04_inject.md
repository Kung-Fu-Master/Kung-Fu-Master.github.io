---
title: istio 04 sidecar inject
tags: istio
categories:
- microService
- istio
---

## **What kind of resources can be injected**
Job, DaemonSet, ReplicaSet, Pod, Deployment, 被istio sidecar 注入后会进行iptable重新网络初始化等操作

Service, Secrets, ConfigMap 这三个被istio注入后不会有啥改变

## **sidecar 注入步骤实例**

	$ touch jiuxi-deployment.yaml
	$ kubectl create ns jiuxi-ns
	$ kubectl apply -f jiuxi-deployment.yaml -n jiuxi-ns
	$ istioctl kube-inject -f jiuxi-deployment.yaml -o jiuxi-deployment-inject.yaml
	$ istioctl kube-inject -f jiuxi-deployment.yaml | kubectl apply -f - -n jiuxi-ns
### **1. 源文件部署, no sidecar:**


	$ vim jiuxi-deployment.yaml
	apiVersion: app/v1
	kind: Deployment
	metadata:
	  name: jiuxi
	  labels:
	    app: jiuxi
	spec:
	  replicas: 1
	  selector:
	    matchLabels:
	      app: jiuxi
	  template:
	    metadata:
	      labels:
	        app: jiuxi
	    spec
	      containers:
	      - name: nginx
	        image:nginx:1.14-alpine
	        imagePullPolicy: IfNotPresent
	        ports:
	        - containerPort: 80
部署:

	$ kubectl apply -f jiuxi-deployment.yaml -n jiuxi-ns
查看pod提供对外服务的端口号:

	$ kubect exec -it po/jiuxi-*** -n jiuxi-ns -- netstat -ntlp
	Active Internete connections (only servers)
	Proto Recv-Q Send-Q Local Address       Foreign Address      State      PID/Program name
	tcp     0          0.0.0.0:80             0.0.0.0:*          LISTEN     1/nginx: master pro
可以看到有个master主进程对外提供80端口服务

### **2. sidecar注入**
是先生成全新的deployment部署pod, 再同时删除原来的deployment和pod资源，从pod的名字哈希后缀可以观察到变化.

	$ istioctl kube-inject -f jiuxi-deployment.yaml | kubectl apply -f - -n jiuxi-ns
可以观察到READY的有2个pod且pod名称hash部分有变化, dump到文件中查看inject后的资源配置信息 

	$ istioctl kube-inject -f jiuxi-deployment.yaml > jiuxi-deployment-inject.yaml
sidecar注入后可以观察到pod里有两个运行的容器 `nginx`, `istio-proxy`, 还有一个已运行结束的`istio-init`容器.  
`istio-init`容器是用来初始化网络命名空间, 使得`nginx`和`istio-proxy`处于相同的网络空间中.

	$ kubectl exec -it -n jiuxi-ns po/jiuxi-* -c nginx -- ifconfig	// 或者将ifcongig替换为route -n(查看路由表)
	eth0: ......
	      inet addr:10.244.10.11......
	lo:
	    ......
	$ kubectl exec -it -n jiuxi-ns po/jiuxi-* -c istio-proxy -- ifconfig // 或者将ifcongig替换为route -n(查看路由表)
	eth0: ......
	      inet addr:10.244.10.11......
	lo:
	    ......
从以上可以看到`nginx`和`istio-proxy`处于相同的网络空间中.  
sidecar 注入后查看pod提供对外服务的端口号:

	$ kubect exec -it po/jiuxi-*** -c nginx -n jiuxi-ns -- netstat -ntlp
	$ kubect exec -it po/jiuxi-*** -c istio-proxy -n jiuxi-ns -- netstat -ntlp
	Proto Recv-Q Send-Q Local Address       Foreign Address      State      PID/Program name
	tcp     0          0.0.0.0:80             0.0.0.0:*          LISTEN     1/nginx: master pro
	......//Pod对外服务端口号会多增加5个

### **3. istio-iptables**
tcp/ip协议栈一般都是由OS内核去实现的,无论是linux, windows, mac. 因为编码时候不会考虑传输层是什么, 怎么封报, 只是在应用层用http协议或调一些SDK或者调用开源包来实现数据转发或者说是报文的封装等.  
但是真正实现网络层协议的还是在操作系统内核来干这些事情.  
操作系统内核又一个模块 `netfilter`, 它有一部分功能就是来制定些网络策略, 如不允许某一个IP或某一个网段的IP或者POD来访问你的主机.  
我们不太可能直接去操作位于内核的`netfilter`来制定这些策略和管理.  
Linux 给我们了一个客户端工具`iptables`，通过这个工具可以跟OS的`netfilter模块`进行交互来向它发送指令执行策略制定.  
`iptables`可以看作一个客户端, `netfilter`可以看作一个server端.

istio-init容器在启动后不久就停止运行，改变了容器的网络策略，查看此容器日志: 

	$ kubectl logs -f -n jiuxi-ns jiuxi-*** -c istio-init 
	......
	* nat			//nat表增加下面四条链
	-N ISTIO_REDIRECT
	-N ISTIO_IN_REDIRECT
	-N ISTIO_INBOUND
	-N ISTIO_OUTPUT
	-A ISTIO_REDIRECT -p tcp -j REDIRECT --to-port 15001	// 从任意地方来的和到任意地方去的流量, 协议是tcp/ip协议, 都会转到15001这个端口
	......
	COMMIT
	......
查看此pod被调度到哪台机器上

	$ docker ps | grep -i istio-proxy
	// 要使用privileged权限
	$ docker exec -it --privileged <istio-proxy_container_ID> bash
	istio-proxy@jiuxi-***:/$
	istio-proxy@jiuxi-***:/$ sudo su root
	istio-proxy@jiuxi-***:/$ iptables -nvL -t nat	//通过iptables去查nat表里的iptables明细
	......
	Chain ISTIO_INBOUND ()		// 可以看到nat表增加了4条链
	......
	Chain ISTIO_IN_REDIRECT ()
	......
	Chain ISTIO_OUTPUT ()
	......
	Chain ISTIO_REDIRECT (1 references)		// 从任意地方来的和到任意地方去的流量, 协议是tcp/ip协议, 都会转到15001这个端口
	 pkts bytes target    prot  opt  in    out    source     destination
	 0      0   REDIRECT  tcp   --    *     *     0.0.0.0/0   0.0.0.0/0      redir ports 15001


### **4. istio-proxy容器进程pilot-agent 和 envoy**
下图中的istio-pilot也就是pilot-agent进程.  
![](pilot-agent_envoy.PNG)
**pilot-agent作用**
1. pilot-agent(可以看作pilot的客户端)不停的跟pilot服务端(istiod pod里运行的pilot-discovery进程可以看作是pilot的服务端)进行通讯拿取从api-server获取的最新的资源路由等规则.  
2. pilot-agent进程启动并生成envoy进程的启动配置.  
3. pilot-agent启动envoy进程.  
4. pilot-agent监控并管理envoy的运行情况, 比如envoy出错时负责重启, envoy配置变更后重新加载; 外部流控的一些规则发生变化后, pilot-agent会监听这种变化, 然后将envoy以新的配置重新加载.  


	$ kubectl exec -it -n jiuxi-ns jiuxi-*** -c istio-proxy -- ps -ef
	UID   PID  PPID  C STIME  TTY    TIME    CMD
	                                         /usr/local/bin/pilot-agent proxy
	                                         /usr/local/bin/envoy -c /etc/ist
	                                         ps -ef

**envoy作用**
![](envoy.PNG)
之所以不用nginx而用envoy做sidecar原因是nginx功能啥都有,太重了, envoy更偏轻量级, 更方便使用.  

	$ kubectl exec -it -n jiuxi-ns jiuxi-*** -c istio-proxy -- netstat -ntlp
可以看到15000端口号只允许本容器的127.0.0.1环回地址访问, 只能供本容器的进程访问.

### 用同样的image运行了不同进程解析
istio-init容器和istio-proxy容器都使用相同的image 如: docker.io/istio/proxyv2:1.5.0, 但是istio-init容器运行iptables进程，istio-proxy容器运行pilot-envoy和envoy两个进程，原因如下:  

当用户同时在kubernetes中的yaml中写了command和args时候自然是可以覆盖DockerFile中ENTRYPOINT的命令行和参数，完整情况如下:  
 * 如果command 和 args 均没有写，那么用Docker默认的配置.
 * 如果command写了, 但args没有写，那么Docker默认的配置会被忽略而且仅仅执行.yaml文件的command(不带任何参数的).(istio-init容器)
 * 如果command没写, 但args写了，那么Docker默认配置的ENTRYPOINT的命令行会被执行, 但是调用的参数是.yaml中的args.(istio-proxy容器)
 * 如果command和args都写了, 那么Docker默认的配置被忽略, 使用.yaml的配置


1. 查看istio/proxyv2:1.5.0 image 原数据


	$ docker images | grep -i proxyv2
	$ docker inspect <istio/proxyv2:1.5.0_ID>
	......
	"Cmd": null,
	......
	"Entrypoint":[	//image被运行成容器时候执行的命令
	    "usr/local/bin/pilot-agent"
	],
	......
2. 查看istio-init 容器yaml资源配置


	$ kubectl get po -n jiuxi-ns jiuxi-***
	......
	initContainers:
	- command:			// 有command, 因此容器会仅执行此command而且忽略image原数据里的args参数
	  - istio-iptables
	  - -p
	......
3. 查看 istio-proxy 容器yaml资源配置


	$ kubectl get po -n jiuxi-ns jiuxi-***	//可以在同一个pod资源文件查看istio-proxy和istio-init的yaml配置
	......
	- args:				// 有args但是没有command, 因此用image原数据里的command并用此yaml里的args参数.
	  - proxy
	  - sidecar
	  - domain
	  - $(POD_NAMESPACE).svc.cluster.local
	  - configPath
	  /etc/istio/proxy			// pilot-agent生成envoy配置文件
	  - --binaryPath
	  - /usr/local/bin/envoy	// pilot-agent启动envoy进程
	......


## sidecar 自动注入

	$ kubectl label ns jiuxi-ns istio-injection=enabled	//添加标签，部署到此namespace空间下的pod自动sidecar注入
	$ kubectl label ns jiuxi-ns istio-injection-		//取消标签操作


## sidecar注入到svc等其它资源

	$ kubectl expose deployment jiuxi -n deployment		// 在deployment资源基础上自动生成svc
	$ kubectl get svc -n jiuxi-ns jiuxi -o yaml > jiuxi-svc.yaml
	$ istioctl kube-inject -f jiuxi-svc.yaml -o jiuxi-svc-inject.yaml //sidecar 注入svc，会发现svc的yaml文件内容并没有变化

