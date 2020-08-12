---
title: 08 Kubernetes samples and problems
tags: kubernetes
categories:
- microService
- kubernetes
top: 8
---

## 01samples
Reference link:
https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/

1. 填写yaml文件
	$ touch test.yaml
添加如下内容

	apiVersion: apps/v1
	kind: Deployment
	metadata:
	labels:
		app.kubernetes.io/name: load-balancer-example
	name: hello-world
	spec:
	replicas: 5
	selector:
		matchLabels:
		app.kubernetes.io/name: load-balancer-example
	template:
		metadata:
		labels:
			app.kubernetes.io/name: load-balancer-example
		spec:
		containers:
		- image: gcr.io/google-samples/node-hello:1.0
			name: hello-world
			ports:
			- containerPort: 8080

2. 执行yaml生成POD

	$ kubectl apply -f test.yaml

3. Display information about the Deployment:

	$ kubectl get deployments hello-world
	$ kubectl describe deployments hello-world

4. Display information about your ReplicaSet objects:

	$ kubectl get replicasets
	$ kubectl describe replicasets

5. Create a Service object that exposes the deployment:

	$ kubectl expose deployment hello-world --type=LoadBalancer --name=my-service

6. Display information about the Service:

	$ kubectl get services my-service
	NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
	my-service   LoadBalancer   10.103.40.210   <pending>     8080:30972/TCP   16m

7. Display detailed information about the Service:

	$ kubectl describe services my-service
	Name:                     my-service
	Namespace:                default
	Labels:                   app.kubernetes.io/name=load-balancer-example
	Annotations:              <none>
	Selector:                 app.kubernetes.io/name=load-balancer-example
	Type:                     LoadBalancer
	IP:                       10.103.40.210
	Port:                     <unset>  8080/TCP
	TargetPort:               8080/TCP
	NodePort:                 <unset>  30972/TCP
	Endpoints:                10.44.0.2:8080,10.44.0.3:8080,10.44.0.4:8080 + 2 more...
	Session Affinity:         None
	External Traffic Policy:  Cluster
	Events:                   <none>

8. 通过 NODE-IP + NodePort 访问
如自己cubic中的开发机IP是10.239.140.186
 * 第一种访问方式:

	$ curl http://10.239.140.186:30972
	Hello Kubernetes!
 * 第二种访问方式:
	浏览器输入http://10.239.140.186:30972 也能正常显示 Hello Kubernetes!

但是公司lab实验室的开发机同样用上面方式部署之后，无法通过 NODE-IP + NodePort 访问， 问题估计跟网络有关


