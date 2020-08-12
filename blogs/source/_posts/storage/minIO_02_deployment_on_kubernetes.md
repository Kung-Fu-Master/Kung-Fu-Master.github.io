---
title: MinIO deployment on kubernetes
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



