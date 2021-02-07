---
title: Istio 查看证书
tags: istio
categories:
- microService
- istio
---

reference:
https://preliminary.istio.io/latest/docs/ops/common-problems/security-issues/#keys-and-certificates-errors

<!-- more -->

## 查看证书
```
$ kubectl get po -n foo
NAME                     READY   STATUS    RESTARTS   AGE
httpbin-5b575fd4-n8kp2   2/2     Running   0          24m
sleep-9cbd59cbd-lm9wc    2/2     Running   0          24m

$ istioctl proxy-config secret httpbin-5b575fd4-n8kp2 -n foo
RESOURCE NAME     TYPE           STATUS     VALID CERT     SERIAL NUMBER                               NOT AFTER                NOT BEFORE
default           Cert Chain     ACTIVE     true           269754354424395637439677801514255125949     2021-02-08T07:18:01Z     2021-02-07T07:18:01Z
ROOTCA            CA             ACTIVE     true           114425755461519597915386269211233384321     2031-02-05T06:12:29Z     2021-02-07T06:12:29Z

// Centos 安装 jq 工具
$ yum install epel-release
$ yum list jq
$ yum install jq

$ istioctl proxy-config secret httpbin-5b575fd4-n8kp2 -n foo -o json | jq '[.dynamicActiveSecrets[] | select(.name == "default")][0].secret.tlsCertificate.certificateChain.inlineBytes' -r | base64 -d | openssl x509 -noout -text

```




