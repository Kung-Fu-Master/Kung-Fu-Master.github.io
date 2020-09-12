---
title: cfssl
tags: security
categories:
- technologies
- security
---

## Kubernetes 证书

| 组件 | 使用的证书 |
| :------: | :------: |
| etcd | ca.pem, server.pem, server-key.pem |
| kube-apiserver | ca.pem, server.pem, server-key.pem |
| kubelet | ca.pem, ca-key.pem |
| kube-proxy | ca.pem, kube-proxy.pem, kube-proxy-key.pem |
| kubectl | ca.pem, admin.pem, admin-key.pem |

## cfssl

cfssl用来生成证书比openssl要简单直观些


## cfssl安装
安装cfssl相关的三个工具: 

	// 生成证书
	$ wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
	// 用于将json文本导入生成证书
	$ wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
	// 查看证书相关信息
	$ wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
	$ chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
	$ mv cfssl_linux-amd64 /usr/local/bin/cfssl
	$ mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
	$ mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
	$ cfssl --help

## cfssl 生成证书
生成证书模板

	$ cfssl print-defaults config > config.json
生成证书请求模板

	$ cfssl print-defaults csr > csr.json






