---
title: Istio 设置请求超时
tags: istio
categories:
- microService
- istio
---

Official website: 
https://istio.io/latest/zh/docs/tasks/traffic-management/fault-injection/
https://istio.io/latest/zh/docs/tasks/traffic-management/request-timeouts/

## 设置请求超时实例
### 应用默认目标规则

```shell
	kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```
如果启用了双向 TLS，请执行以下命令：

```shell
	kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
```
等待几秒钟，以使目标规则生效。  
您可以使用以下命令查看目标规则：

```shell
	kubectl get destinationrules -o yaml
```
### 初始化应用的版本路由

```shell
	kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```
### 设置请求超时
将请求路由到 reviews 服务的 v2 版本，它会发起对 ratings 服务的调用：

```shell
	kubectl apply -f - <<EOF
	apiVersion: networking.istio.io/v1alpha3
	kind: VirtualService
	metadata:
	  name: reviews
	spec:
	  hosts:
	    - reviews
	  http:
	  - route:
	    - destination:
	        host: reviews
	        subset: v2
	EOF
```

给对 ratings 服务的调用添加 2 秒的延迟：

```shell
	kubectl apply -f - <<EOF
	apiVersion: networking.istio.io/v1alpha3
	kind: VirtualService
	metadata:
	  name: ratings
	spec:
	  hosts:
	  - ratings
	  http:
	  - fault:
	      delay:
	        percent: 100
	        fixedDelay: 2s
	    route:
	    - destination:
	        host: ratings
	        subset: v1
	EOF
```
在浏览器中打开 Bookinfo 的网址 http://$GATEWAY_URL/productpage。

这时可以看到 Bookinfo 应用运行正常（显示了评级的星型符号），但是每次刷新页面，都会有 2 秒的延迟。

现在给对 reviews 服务的调用增加一个半秒的请求超时：

```shell
	kubectl apply -f - <<EOF
	apiVersion: networking.istio.io/v1alpha3
	kind: VirtualService
	metadata:
	  name: reviews
	spec:
	  hosts:
	  - reviews
	  http:
	  - route:
	    - destination:
	        host: reviews
	        subset: v2
	    timeout: 0.5s
	EOF
```
刷新 Bookinfo 页面。

这时候应该看到 1 秒钟就会返回，而不是之前的 2 秒钟，但 reviews 是不可用的。

即使超时配置为半秒，响应仍需要 1 秒，是因为 productpage 服务中存在硬编码重试，因此它在返回之前调用 reviews 服务超时两次。


