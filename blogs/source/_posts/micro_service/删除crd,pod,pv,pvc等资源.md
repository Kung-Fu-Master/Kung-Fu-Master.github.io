---
title: 删除crd,pod,pv,pvc等资源
tags: istio
categories:
- microService
- kubernetes
---

## **pod,pv,pvc删除顺序**

一般删除步骤为：先删pod, 再删pvc, 最后删pv.  
如果遇到pv始终处于“Terminating”状态，而且delete不掉.  
解决方法是直接删除k8s中相对应的pvc和pv的记录：

	// 1. 删除pvc
	$ kubectl describe pvc PVC_NAME -n NameSpace| grep Finalizers
	  Finalizers:    [kubernetes.io/pvc-protection]
	$ kubectl patch pvc PVC_NAME -n NameSpace -p '{"metadata":{"finalizers": []}}' --type=merge
	
	// 2. 删除pv
	$ kubectl patch pv xxx -p '{"metadata":{"finalizers":null}}'


## **强制删除crd**

	// remove the CRD finalizer blocking on custom resource cleanup
	$ kubectl patch crd/greenplumclusters.greenplum.pivotal.io -p '{"metadata":{"finalizers":[]}}' --type=merge
	customresourcedefinition.apiextensions.k8s.io/kibanas.kibana.k8s.elastic.co patched
	
	// 再次删除资源 
	$ kubectl delete crd greenplumclusters.greenplum.pivotal.io
	OK搞定了~

## **删除某个namespace**
在k8s中会无法删除某个namespace，它会一直卡在terminating状态.  
这个指令找到阻碍这个namespace删除的资源，然后手动删除这些资源.  

	$ kubectl api-resources --verbs=list --namespaced -o name   | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <namespace> 


