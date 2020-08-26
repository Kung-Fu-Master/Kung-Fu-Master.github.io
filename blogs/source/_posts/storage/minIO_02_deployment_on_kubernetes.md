---
title: MinIO 02 deployment on kubernetes
tags: storage
categories:
- storage
---
在storage仓库里也存储了一份当时部署所使用的yaml等资源文件.

Quick start:  
  [https://docs.min.io/docs/minio-quickstart-guide.html](https://docs.min.io/docs/minio-quickstart-guide.html)
Deployment on kubernetes:  
  [https://docs.min.io/docs/deploy-minio-on-kubernetes.html](https://docs.min.io/docs/deploy-minio-on-kubernetes.html)
  [https://github.com/minio/operator/blob/master/README.md](https://github.com/minio/operator/blob/master/README.md)
Enable TLS:  
  [https://github.com/minio/operator/blob/master/docs/tls.md](https://github.com/minio/operator/blob/master/docs/tls.md)

## Prerequisites
1. Kubernetes version v1.17.0 and above for compatibility. MinIO Operator uses k8s/client-go v0.18.0.
2. kubectl configured to refer to a Kubernetes cluster.
3. Create the required PVs as [direct CSI driver.](https://github.com/minio/minio-operator/blob/master/docs/using-direct-csi.md) or use the following installation step.

## Using Direct CSI Driver

	cat << EOF > default.env
	DIRECT_CSI_DRIVES=data{1...4}
	DIRECT_CSI_DRIVES_DIR=/mnt
	EOF
	
	$ export $(cat default.env)
	$ kubectl apply -k direct-csi
执行完上述操作后会在work node的/var/lib/kubelet/plugins_registry/生成下列sock文件

	$ ls /var/lib/kubelet/plugins_registry/
	direct.csi.min.io-reg.sock

## Create Operator Deployment
1. Create namespace minio for minIO to deploy.


	$ kubectl create ns minio
2. To start MinIO-Operator with default configuration, use the minio-operator.yaml file.


	$ kubectl apply -f minio-operator.yaml -n minio

## Create MinIO instances without tls
Once MinIO-Operator deployment is running, you can create MinIO instances using the below command

	$ kubectl apply -f minioinstance.yaml
## Create NodePort service for minIO

	$ kubectl apply -f NodePort-minio.yaml
## Access minIO from the browser

	// Minio Server没有配置TLS
	浏览器输入: http://127.0.0.1:30007
	default username: minio
	default password: minio123
	// Minio Server配置TLS
	浏览器输入: https://127.0.0.1:30007

## **Secure access to MinIO server with TLS**
**1.1. Generate a private key with ECDSA**


	$ openssl ecparam -genkey -name prime256v1 | openssl ec -out private.key
	  read EC key
	  writing EC key
Alternatively, use the following command to generate a private ECDSA key protected by a password:

	$ openssl ecparam -genkey -name prime256v1 | openssl ec -aes256 -out private.key -passout pass:<PASSWORD>
**1.2 Generate a private key with RSA.**
Use the following command to generate a private key with RSA:


	$ openssl genrsa -out private.key 2048
	Generating RSA private key, 2048 bit long modulus
	............................................+++
	...........+++
	e is 65537 (0x10001)
	// openssl查看证书内容
	$ openssl rsa -in private.key -text -noout
Alternatively, use the following command to generate a private RSA key protected by a password:

	$ openssl genrsa -aes256 -out private.key 2048 -passout pass:<PASSWORD>
**Note:** When using a password-protected private key, the password must be provided through the environment variable MINIO_CERT_PASSWD using the following command:

	$ export MINIO_CERT_PASSWD=<PASSWORD>
	// 也就是在minioinstance.yaml中添加如下信息
	env:
	  - MINIO_CERT_PASSWD
	    value: "<YourPassword>"		// 一定要加上双引号""
**2. Generate a self-signed certificate.**
Use the following command to generate a self-signed certificate and enter a passphrase when prompted:


	$ openssl req -new -x509 -days 3650 -key private.key -out public.crt -subj "/C=US/ST=state/L=location/O=organization/CN=<domain.com>"
	// 查看证书签名,算法,public key等信息
	$ openssl x509 -in public.crt -text -noout
**Note:** Replace <domain.com> with the development domain name.
Alternatively, use the command below to generate a self-signed wildcard certificate that is valid for all subdomains under <domain.com>. Wildcard certificates are useful for `deploying distributed MinIO instances`, where each instance runs on a subdomain under a single parent domain.

	$ openssl req -new -x509 -days 3650 -key private.key -out public.crt -subj "/C=US/ST=state/L=location/O=organization/CN=*.minio-hl-svc.minio.svc.cluster.local"
**Note: 若是不知道怎么确定`<*.domain.com>`信息, 添加TLS后部署minio发现pod启动异常或者访问minio服务不正常, 查看pod的日志,`$ kubectl logs po/minio-0 -n minio`, 即可看到pod出现error信息，显示需要的certificates的commanName(简称CN), 然后将不同pod证书不同的内容用`*`替代, 其它的照搬，如上的例子`*.minio-hl-svc.minio.svc.cluster.local`.**

**3. Use a Kubernetes Secret resource to store this information, create a Kubernetes Secret using:**


	$ kubectl create secret generic tls-ssl-minio --from-file=***/pki/private.key --from-file=***/pki/public.crt -n minio
再将tenant.yaml或者改过名的minioinstance.yaml文件中的下列注释内容打开

	externalCertSecret:
	  name: tls-ssl-minio
然后再重新部署minio

	$ kubectl apply -f minioinstance.yaml
执行完上述操作后会在work node生成以下文件夹

	$ ls /var/lib/kubelet/plugins
	direct-csi-min-io/  kubernetes.io/

## **查看pvc和pv,查看WorkNode pvc volume存储**

	$ kubectl get pvc -n minio
	[root@hci-node01 minIO]# k get pvc -n minio
	NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
	data0-minio-0   Bound    pvc-fc3a42f2-78d7-4dfe-8ad5-a0e3c745f638   10Gi       RWO            direct.csi.min.io   19h
	data1-minio-0   Bound    pvc-b9f227cd-7d6c-4728-b27f-00d4d5c42a16   10Gi       RWO            direct.csi.min.io   19h
转到minio-0这个pod部署到的work node机器上查看

	$ cd /var/lib/kubelet/plugins/kubernetes.io/csi/pv
	$ ls
	  pvc-fc3a42f2-78d7-4dfe-8ad5-a0e3c745f638/ pvc-b9f227cd-7d6c-4728-b27f-00d4d5c42a16/
	$ cd pvc-b9f227cd-7d6c-4728-b27f-00d4d5c42a16/globalmount/mybucket-01/ // 自己创建了一个buckt: mybucket-01
	$ ls
	  file-Name/				// file-Name是通过mc命令行工具加进mybucket-01
	$ touch test.txt
	$ ls
	  file-Name/  test.txt		// test.txt是登陆work node后手动复制或创建的文件.
但是在此work node机器上的 /mnt/data1/目录下也能查看到文件

	$ cd /mnt/data1/
	$ ls
	b1ac2515-e6ba-11ea-98f8-064de4d3a4ce/  b1ac2983-e6ba-11ea-98f8-064de4d3a4ce/
	$ ls b1ac2515-e6ba-11ea-98f8-064de4d3a4ce/
	file-Name/ test.txt		// 发现此目录也出现了test.txt文件
上面登陆work node向pv volume添加的文件在master机器上用mc客户端是查不到的.

## **删除minio Cluster**

	$ kubectl delete -f minioinstance.yaml
	$ kubectl delete -f minio-operator.yaml
	$ kubectl delete secret tls-ssl-minio -n minio
	$ kubectl delete -k direct-csi		// 不要手动删除storageclass资源而要用这种方式.
	  namespace "direct-csi" deleted
	  serviceaccount "direct-csi-min-io" deleted
	  clusterrole.rbac.authorization.k8s.io "direct-csi-min-io" deleted
	  clusterrolebinding.rbac.authorization.k8s.io "direct-csi-min-io" deleted
	  configmap "direct-csi-config" deleted
	  secret "direct-csi-min-io" deleted
	  service "direct-csi-min-io" deleted
	  deployment.apps "direct-csi-controller-min-io" deleted
	  daemonset.apps "direct-csi-min-io" deleted
	  csidriver.storage.k8s.io "direct.csi.min.io" deleted
	$ kubectl delete <pvc about minio>
	$ kubectl delete <pv about minio>	// minio pv资源不会随着minio的pvc资源删除而删除,因此需要再次删除
删除pv shell脚本, 同样改成pvc, 再加上-n <Namespace>就可以删除pvc了:

	#!/bin/bash
	
	pv_results=$(kubectl get pv | grep minio | awk '{print $1}')
	pv_arr=(${pv_results})
	for ((i=0; i<${#pv_arr[*]}; i++ ))
	do
	kubectl delete pv ${pv_arr[i]}
	done

执行完上面删除操作后再到work node删除minio生成的指定文件如data,pvc等

	// 到work node上删除data
	$ cd /mnt
	$ rm -rf data0/ data1/
	// 删除pv
	$ cd /var/lib/kubelet/plugins/
	$ rm -rf direct-csi-controller-min-io
	$ rm -rf direct-csi-min-io
	$ rm -rf kubernetes.io/csi/pv/pvc-***<pv about minio>

## **遇到的问题**

### **问题1:** 删除其它机器pvc绑定的 /var/lib/kubelet/plugins/* 出错

	$ rm -rf /var/lib/kubelet/plugins/*
	rm: cannot remove ‘/var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-d6ea6990-2a0b-4f4b-9838-3a971424732d/globalmount’: Device or resource busy
	rm: cannot remove ‘/var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-a08fed0b-b16d-4a74-a052-7936d6fb8340/globalmount’: Device or resource busy
解决方法:

	$ umount /var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-d6ea6990-2a0b-4f4b-9838-3a971424732d/globalmount
	$ umount /var/lib/kubelet/plugins/kubernetes.io/csi/pv/pvc-a08fed0b-b16d-4a74-a052-7936d6fb8340/globalmount
再进行删除即可

### **问题2:** minio pod删除不掉一直处于Terminating状态
强制删除pod命令:

	$ kubectl delete pod <PodName> -n <NAMESPACE> --force --grace-period=0

