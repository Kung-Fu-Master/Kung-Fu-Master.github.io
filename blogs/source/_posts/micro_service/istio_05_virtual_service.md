---
title: istio 05 virtual service
tags: istio
categories:
- microService
- istio
---

## **Classic traffic management model**
1. 没有流量管理
![](legacy_1.PNG)

2. 有流量管理, 如nginx, 限制某个IP或IP网段访问, 也可以选择将客户端的请求路由到后台的任意一个web服务器上.
![](legacy_2.PNG)

## **Istio traffic**
Istio traffic management 特指数据面的流量管理

istio traffic management API Resource
 * Virtual services
 * Destination rules
 * Gateways
 * Service entries
 * Sidecars

![](istio_traffic_1.PNG)
**Sample .yaml resource files**

### **scenario 1**
![](scenario_1.PNG)
1. jiuxi-client.yaml
2. jiuxi-deployment.yaml
3. jiuxi-svc.yaml
4. jiuxi-vs.yaml


jiuxi-client.yaml

	$ touch jiuxi-client.yaml
	apiVersion: apps/v1
	kind: Deployment
	metadataL:
	  name: client
	spec:
	  replicas: 1
	  selector:
	    matchLabels:
	      app: client
	  template:
	    metadata:
	      labels:
	        app: client
	    spec:
	      containers:
	      - name: busybox
	        image: busybox
	        imagePullPolicy: IfNotPresent
	        command: [ "/bin/sh", "-c", "sleep 3600" ]
	$ kubectl apply -f jiuxi-client.yaml
jiuxi-deploy.yaml

	$ touch jiuxi-deploy.yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: httpd
	  labels:
	    server: httpd
	    app: web
	spec:
	  replicas: 1
	  selector:
	    matchLabels:
	      server: httpd
	      app: web
	  template:
	    metadata:
	      name: httpd
	      labels:
	        server: httpd
	        app: web
	    spec:
	      containers:
	      - name: busybox
	        image: busybox
	        imagePullPolicy: IfNotPresent
	        command: [ "/bin/sh", "-c", "echo 'hello httpd' > /var/www/index.html; httpd -f -p 8080 -h /var/www" ]
	---
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: tomcat
	  labels:
	    server: tomcat
	    app: web
	spec:
	  replicas: 1
	  selector:
	    matchLabels:
	      server: tomcat
	      app: web
	  template:
	    metadata:
	      name: tomcat
	      labels:
	        server: tomcat
	        app: web
	    spec:
	      containers:
	      - name: tomcat
	        image: docker.io/kubeguide/tomcat-app:v1
	        imagePullPolicy: IfNotPresent
	$ kubectl apply -f jiuxi-deploy.yaml
jiuxi-svc.yaml

	$ touch jiuxi-svc.yaml
	apiVersion: v1
	kind: Service
	metadata:
	  name: tomcat-svc
	spec:
	  selector:
	    server: tomcat
	  ports:
	  - name http
	    port: 8080
	    targetPort: 8080
	    protocol: TCP
	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: httpd-svc
	spec:
	  selector:
	    server: httpd
	  ports:
	  - name http
	    port: 8080
	    targetPort: 8080
	    protocol: TCP
	$ kubectl apply -f jiuxi-svc.yaml
也可以写完上面三个yaml文件后直接执行下面命令一次全部部署

	$ kubectl apply -f .
查看service最终有没有跟pod绑定:

	$ kubectl get endpoints
	$ curl http://<ENDPOINTS-IP>:8080
登陆client pod

	$ kubectl exec -it client-*** -- sh
	/ # wget -q -O - http://tomcat-svc:8080		// wget --help
	/ # wget -q -O - http://httpd-svc:8080		//-q: quiet静态访问,不进行下载文件; -O: 输出文件, '-'会输出到控制台
	/ # wget -q -O - http://<httpd-svc-clusterIP>:8080	//效果跟上面一样

### **scenario 2**
![](scenario_2.PNG)
修改添加jiuxi-svc.yaml内容

	$ touch jiuxi-svc.yaml
	apiVersion: v1
	kind: Service
	metadata:
	  name: tomcat-svc
	spec:
	  selector:
	    server: tomcat
	  ports:
	  - name http
	    port: 8080
	    targetPort: 8080
	    protocol: TCP
	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: httpd-svc
	spec:
	  selector:
	    server: httpd
	  ports:
	  - name http
	    port: 8080
	    targetPort: 8080
	    protocol: TCP
	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: web-svc
	spec:
	  selector:
	    app: web	//将此Service与httpd和tomcat两个Pod相关联
	  ports:
	  - name http
	    port: 8080
	    targetPort: 8080
	    protocol: TCP
	$ kubectl apply -f jiuxi-svc.yaml
查看service最终有没有跟pod绑定,并多次执行curl 操作查看是否实现轮询:

	$ kubectl get endpoints
	$ curl http://<web-svc-ENDPOINTS-IP>:8080
登陆client pod

	$ kubectl exec -it client-*** -- sh
	//多次执行wget操作查看是否轮询访问httpd和tomcat服务
	/ # wget -q -O - http://web-svc:8080
	/ # wget -q -O - http://<web-svc-clusterIP>:8080	//效果跟上面一样

### **scenario 3**
让VirtualService规则生效的话，前提条件是所有的服务都被istio注入sidecar, 使这些服务处于istio的服务网格控制下.  
在没有注入sidecar的istio控制之外来访问，VirtualService制定的服务规则不会生效.
![](scenario_3.PNG)
jiuxi-vs.yaml

	$ touch jiuxi-vs.yaml
	apiVersion: networking.istio.io/v1alpha3
	kind: VirtualService
	metadata:
	  name: web-svc-vs
	spec:
	  hosts:	// 指Istio的VirtualService作用在K8s的哪个Service上, 这里是web-svc这个Service
	  - web-svc		// 短域名方式, 因为是在同一个namespace下, 全域名写法(default命名空间): web-svc.default.svc.cluster.local
	  http:
	  - route:
	    - destination:
	        host: tomcat-svc	// 指的是k8s的Service
	      weight: 20
	    - destination:
	        host: httpd-svc
	      weight: 80
部署VirtualService并查看是否部署成功

	$ kubectl apply -f jiuxi-vs.yaml
	$ kubectl get virtualservices.networking.istio.io
	NAME       GATEWAYS     HOSTS      AGE
	web-svc-vs              [web-svc]  9s
VirtualService是istio资源，不能通过`kubectl get svc`查看, 也因此相对K8s的Service来说是虚拟服务:VirtualService  

注入sidecar

	$ istioctl kube-inject jiuxi-client.yaml | kubectl apply -f -
	$ istioctl kube-inject jiuxi-deploy.yaml | kubectl apply -f -
登陆client, 在istio服务网格内访问服务

	$ kubectl exec -it client-*** -- sh
	/ # wget -q -O - http://web-svc:8080	//多次执行查看效果
	/ # wget -q -O - http://<web-svc-clusterIP>:8080	//效果跟上面一样










