---
title: Istio 02 Installation on Kubernetes
tags: istio
categories:
- microService
- istio
---

## **Download the specific version of Istio**

	$ curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.6.4 sh -
	$ cd istio/bin/
	$ ./istioctl -h
	  Istio configuration command line utility for service operators to
	  debug and diagnose their Istio mesh.
	  
	  Usage:
	    istioctl [command]
	  
	  Available Commands:
	    analyze         Analyze Istio configuration and print validation messages
	    authz           (authz is experimental. Use `istioctl experimental authz`)
	    convert-ingress Convert Ingress configuration into Istio VirtualService configuration
	    dashboard       Access to Istio web UIs
	    deregister      De-registers a service instance
	    experimental    Experimental commands that may be modified or deprecated
	    help            Help about any command
	    install         Applies an Istio manifest, installing or reconfiguring Istio on a cluster.
	    kube-inject     Inject Envoy sidecar into Kubernetes pod resources
	    manifest        Commands related to Istio manifests
	    operator        Commands related to Istio operator controller.
	    profile         Commands related to Istio configuration profiles
	    proxy-config    Retrieve information about proxy configuration from Envoy [kube only]
	    proxy-status    Retrieves the synchronization status of each Envoy in the mesh [kube only]
	    register        Registers a service instance (e.g. VM) joining the mesh
	    upgrade         Upgrade Istio control plane in-place
	    validate        Validate Istio policy and rules (NOTE: validate is deprecated and will be removed in 1.6. Use 'istioctl analyze' to validate configuration.)
	    verify-install  Verifies Istio Installation Status or performs pre-check for the cluster before Istio installation
	    version         Prints out build version information
	  
	  Flags:
	        --context string          The name of the kubeconfig context to use
	    -h, --help                    help for istioctl
	    -i, --istioNamespace string   Istio system namespace (default "istio-system")
	    -c, --kubeconfig string       Kubernetes configuration file
	    -n, --namespace string        Config namespace
	  
	  Additional help topics:
	    istioctl options         Displays istioctl global options
	  
	  Use "istioctl [command] --help" for more information about a command.

## **istio目录介绍**

	$ll istio-*
	  bin/ ,***, manifests/, samples/, tools/
bin/目录存放istioctl客户端工具.  
manifests/目录存放istio安装表单.  
samples/目录存放一些使用样例.  
tools/目录下的istioctl.bash文件是能在输入$ istioctl m 时候自动补全命令,如 $istioctl manfest.  , 需要先执行$ source istioctl.bash才能生效，新版本istio好像不用设置也能有自动补全功能.  

## **Istioctl工具介绍**
official website: https://istio.io/latest/docs/ops/diagnostic-tools/proxy-cmd/

**Istioctl 与 Kubectl关系**
从一开始istioctl和kubectl工具有一定的融合部分，如apply, delete等共同功能, 随着istioctl版本提升启用了一些与kubectl重合的功能并完善开发了自己的一套功能.

	// 查看pilot和envoy配置策略等的同步情况, pilot是否完全将配置信息同步到envoy
	$ ./istioctl proxy-status
	// 查询属于某个pod的envoy规则等
	$ ./istioctl proxy-config <clusters|listeners|routes|endpoints|bootstrap> <pod-name[.namespace]>

	// 检测安装是否成功
	$ ./istioctl verify-install
	  ......
	  Checked 25 custom resource definitions	// 可以看到Istio 1.6.4只有25个crd
	  Checked 3 Istio Deployments
	  Istio is installed successfully


## **Istio upgrade and rollback**
istio1.6 提供了简单的升级命令方式，直接通过命令 $ istioctl upgrade 就可以更新Istio control plane in-place
且提供了金丝雀发布方式，更新和回滚过程可以看到有两个istiod在istio-system命名空间下.



