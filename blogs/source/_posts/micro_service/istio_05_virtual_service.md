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
1. jiuxi-client.yaml
2. jiuxi-deployment.yaml
3. jiuxi-svc.yaml
4. jiuxi-vs.yaml







