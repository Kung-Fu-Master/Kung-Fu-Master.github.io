---
title: Istio 02 Installation on Kubernetes and upgrade
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
tools/目录下的istioctl.bash文件是能在输入$ istioctl m 时候自动补全命令,如 $istioctl manifest.  , 需要先执行$ source istioctl.bash才能生效，新版本istio好像不用设置也能有自动补全功能.  

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

## istioctl command

### istioctl profile
查看profile list

	$ istioctl profile list
输出profile

	$ istioctl profile dump demo > demo.yaml
	vim demo.yaml


## **Istioctl analyze**

	$ istioctl analyze -n book-info
	  √ No Validation issues found when analyzing namespace: book-info

## **Istio upgrade and rollback**
istio1.6 提供了简单的升级命令方式，直接通过命令 $ istioctl upgrade 就可以更新Istio control plane in-place
且提供了金丝雀发布方式，更新和回滚过程可以看到有两个istiod在istio-system命名空间下.

使用istioctl升级时候, 老版本和新版本之间版本号要接近，相差太远如由istio1.0升级到istio1.6.8有可能会失败.

**升级步骤**
查看升级前的istioctl 版本

	$ istioctl version
	client versin: 1.5.0			// 指istioctl 这个二进制客户端工具
	control plane version: 1.5.0	// istio部署在k8s上的控制面资源, 如pilot等
	dataplane version: 1.5.0		// istio部署在k8s上的数据面资源, 如envoy

**1. 升级istioctl客户端二进制工具**
Download istio最新版本如istio-1.6.8, 解压缩`tar -zxvf istio-1.6.8.tar.gz`进入istio-1.6.8/bin  
linux Path环境变量中的老版本istioctl去掉添加新版本的istioctl客户端工具.  

**2. 升级istio在k8s上的数据面和控制面**
升级时候要保证升级的是同样的profile, 如demo, 或者default

	$ istioctl profile list 		// 查看istioctl有哪些profile
	$ istioctl profile dump demo > demo.yaml	// 先dump出跟老版本相同选择的新版本的profile, 如新老版本都采用demo 这个profile
第一种:根据新版本的demo.yaml来进行升级

	$ vim demo.yaml 	//先修改下dump出来的新版本的demo.yaml
	jwtPolicy: third-party-jwt ---> 改为 jwtPolicy: first-party-jwt
	如果这里不修改会报证书无法挂载情况.
	$ istioctl upgrade -f demo.yaml
第二种:

	$ istioctl manifest apply -set profile=demo --set values.global.jwtPolicy=first-party-jwt
查看升级后的istioctl 版本

	$ istioctl version
	client versin: 1.6.8			// 指istioctl 这个二进制客户端工具
	control plane version: 1.6.8	// istio部署在k8s上的控制面资源, 如pilot等
	dataplane version: 1.5.0 (1 proxies), 1.6.8 (3 proxies)	// istio部署在k8s上的数据面资源, 如envoy, 发现还有老版本注入的envoy, 也需要`手动`或`自动升级`
如果以前采用的是自动sidecar注入, 则将所有pods通过滚动更新来更新sidecar

	$ kubectl rollout restart deployment --namespace <namespace with auto injection>
如果以前采用的是手动sidecar注入, 则更新sidecar通过执行:

	$ kubectl apply -f < (istioctl kube-inject -f <original application deployment yaml>) //实例如下
	$ istioctl kube-inject -f nginx.yaml | kubectl apply -f -
通过`kubectl get pods -n <Namespace>` 来观察到新版本生成后老版本的pod才terminate.  
重新查看istioctl version

	$ istioctl version
	client versin: 1.6.8
	control plane version: 1.6.8
	dataplane version: 1.6.8 (4 proxies)


