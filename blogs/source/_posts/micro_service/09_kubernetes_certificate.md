---
title: 08 Kubernetes Certificates
tags:
- kubernetes
categories:
- microService
- kubernetes
top: 9
---

## K8s Certificates

 !()[k8s_certificates.PNG]

## 证书颁发

1. 自签证书, 多用在内部服务之间, K8s, etcd就是使用自签证书.

2. 权威机构: GeoTrust RSA CA 2018, GTS CA 101, Intel Internal Issuing CA 5A
例如赛门铁克, 分不同档次, 中档3000元左右, 如www.ctnrs.com, 也有针对所有域名的 *.ctnrs.com, 不过更贵

根证书: 每个浏览器都内置了各个信任的权威机构, 是权威机构颁发的证书浏览https访问路径上的锁显示安全, 否则显示不可信任的网站

自签证书和权威机构都会颁发两个证书:
-- 1. crt： 数字证书;
-- 2. key： 私钥;
将这两证书配置到web服务器就可以了

不管是自签还是权威机构都有CA, 会颁发多个证书出来, 

## simulate CA
![](CA_simulate.PNG)

## 使用cfssl或openssl生成自签证书
https://kubernetes.io/zh/docs/concepts/cluster-administration/certificates/

cfssl 在 k8s 中比较流行, 使用json文件传入来生成证书, 要比 openssl 更直观也简单点.











