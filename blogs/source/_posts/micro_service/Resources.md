---
title: Resources
tags: kubernetes
categories:
- microService
- kubernetes
top: 9
---

## Resources
查看某台机器的资源

	$ kubectl describe node hci-node01

### memory, CPU
requests, limits关键字.
当没有哪个Node节点满足requests的资源时候，container会一直处于pending状态, 而不会受limits资源影响.
如Node只有10GB内存，一个container request memory配置7GB内存大小，那么其余container只能分配剩余的3GB内存大小.
配置K8s的container文件

	apiVersion: v1
	......
	spec:
	  containers:
	  - name: web-demo
	    image: hub.**
	    ports:
	    - containerPort: 8080
	    resources:
	      requests:
	        memory: 100Mi		// Ki: KB大小, Mi: MB大小, Gi: GB大小
	        cpu: 100m			// 如机器有12个内核, 而1个cpu内核=1000m, 100m为0.1个cpu内核, milli core, 千分之一core
	      limits:
	        memory: 1000Mi
	        cpu: 200m

登陆container运行的机器，查看容器设置参数

	$ docker inspect containerID
	......
	"CpuShares" 102,		// 100m转为为0.1内核，再用0.1 * 1024 = 102，当很多个容器运行在某台机器上发生CPU资源抢占时候用该值作为权重进行分配
	"Memory": 1048576000,	// 对应K8s资源文件的limits->memory值：1000MB大小
	......
	"CPUPeriod": 100000,	// docker 默认值, 单位是ns, 转化为毫秒是100
	"CpuQuota": 20000,		// 0.2 core * 100000 = 20000, 与上面一起使用，表示100ms内最多分配给容器的cpu量是0.2个core
当容器里进程所用内存超过requests值时, 占用资源最多的进程会被kill掉, 但是容器不会退出重启.
而CPU超出limits值时进程不会退出，因为CPU是可压缩资源，而内存不是.

	$ docker stats ContainerID			// 查看container CPU和内存使用情况

进入容器通过以下command命令模拟多个后台进程占用CPU

	$ dd if=/dev/zero of=/dev/null &	//
	$ killall dd						// 删除所有dd命令
一般情况都是设置requests == limits


### LimitRange
在一个namespace下对Pod和container申请的CPU和memory资源进行限制

	apiVersion: v1
	kind: LimitRange
	metadata:
	  name: test-limits
	spec: 
	  limits:
	  - max:
	      cpu: 4000m
	      memory: 2Gi
	    min:
	      cpu: 100m
	      memory: 100Mi
	    maxLimitRequestRatio:
	      cpu: 3				// 对POD显示，运行起来后占用机器cpu资源数是在limits.cpu最大以内且不能超过requests.cpu的3倍.
	      memory: 2
	    type: Pod				// 上面限制类型为Pod
	  - default:
	      cpu: 300m				// 默认Container 0.3个内核
	      memory: 200Mi
	    defaultRequest:
	      cpu: 200m
	      memory: 100Mi
	    max:
	      cpu: 2000m
	      memory: 1Gi
	    min:
	      cpu: 100m
	      memory: 100Mi
	    maxLimitRequestRatio:
	      cpu: 5
	      memory: 4
	    type: Container			// 上面显示类型为Container
运行如下命令查看limits限制

	$ kubectl describe limits -n namespaces
> 当新建deployment或RS或Pod时，会根据LimitRange限制检测新建的Pod或Container是否满足条件，否则创建失败, 可以通过kubectl get ** -o yaml查看事件message.

### ResourceQuota
对每个namespace能够使用的资源进行限制.

	apiVersion: v1
	kind: ResourceQuota
	metadata:
	  name: resource-quota
	spec:
	  hard:
	    pods: 4
	    requests.cpu: 2000m
	    requests.memory: 4Gi
	    limits.cpu: 4000m
	    limits.memory: 8Gi
	    configmaps: 10
	    persistentvolumeclaims: 4
	    replicationcontrollers: 20
	    secrets: 10
	    services: 10
查看quota类型资源

	$ kubectl get quota -n NS
	$ kubectl describe quota *** -n NS		// 查看硬件资源是否使用饱和


### Eviction-Pod驱逐
常见驱逐策略配置
 * 在1m30s时间内可利用内存持续小于1.5GB时进行驱逐


	--eviction-soft=memory.available<1.5Gi
	--eviction-soft-grace-period=memory.available=1m30s
 * 当可利用内存小于100MB或磁盘小于1GB或剩余的节点小于5%立即执行驱逐策略


	--eviction-hard=memory.available<100Mi,nodefs.available<1Gi,nodefs.inodesFree<5%

驱逐策略:
磁盘紧缺时候:
 * 删除死掉的Pod和容器
 * 删除没用的image
 * 按优先级和资源占用情况驱逐Pod


内存紧缺时候:
 * 驱逐不可靠的Pod(没有设置requests和limits资源的pod都是不可靠的Pod)
 * 驱逐基本可靠的pod(实际使用资源大于requests资源是基本可靠,超出的越多优先删除，否则删除占用内存最多的Pod)
 * 驱逐可靠的Pod(requests资源==limits资源的Pod为可靠的Pod, 如果都一样驱逐策略跟上面驱逐基本可靠Pod一致)


### label
key=value 贴标签(Node, Deployment, Service, Pod)

	#deploy
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: web-demo
	  namespace: dev
	spec:
	  selector:
	    matchLabels:				// 选择含有指定标签的Pod,必须跟下面一致，或者不写只配置下面Pod的label也可以
	      app: web-demo
	    matchExpressions:
	      - {key: group, operator: In, values: [dev, test]}		// Key in values的Pod都会被选中
	  replicas: 1
	  template:						// 设置Pod模板
	    metadata:
	      labels:					// 设置Pod标签
	        app: web-demo
	        group: dev
	    spec:
	      containers:
	      - name: web-demo
	        image: hub**
	        ports:
	        - containerPort: 8080
	      nodeSelector:				// 选择含有某个标签的Node机器部署Pod
	        disktype: ssd
	---
	# service
	apiVersion: v1
	kind: Service
	metadata:
	  name: web-demo
	  namespace: dev
	spec:
	  ports:
	  - port: 80
	    protocol: TCP
	    targetPort: 8080
	  selector:
	    app: web-demo
	  type: ClusterIP
查询Pod

	$ kubectl get pods -l "group in (dev, test)" -n NS
	$ kubectl get pods -l "group notin (dev)" -n NS
	$ kubectl get pods  -l group=dev,group=test -n NS








