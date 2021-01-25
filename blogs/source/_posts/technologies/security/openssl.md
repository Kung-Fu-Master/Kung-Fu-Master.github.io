---
title: openssl
tags: security
categories:
- technologies
- security
---

## openssl版本
```
openssl version
openssl version -a
```

<!-- more -->

## 支持的cipher
```
openssl ciphers -v
openssl ciphers -V 'ALL:COMPLEMENTOFALL'
```

## 查看key

```
openssl genrsa -out root-key.pem 3072
openssl rsa -in root-key.pem -text -noout
```

## 生成csr配置文件

root-ca.conf
```
[ req ]
encrypt_key = no
prompt = no
utf8 = yes
default_md = sha384
default_bits = 3072
req_extensions = req_ext
x509_extensions = req_ext
distinguished_name = req_dn
[ req_ext ]
subjectKeyIdentifier = hash
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, nonRepudiation, keyEncipherment, keyCertSign
[ req_dn ]
O = Istio
CN = Root CA
```

## 查看csr

```
openssl req -new -sha384 -key root-key.pem -config root-ca.conf -out root-cert.csr
openssl req -in root-cert.csr -noout -text
```

## 查看证书

```
openssl x509 -req -sha384 -days 3650 -signkey root-key.pem \
        -extensions req_ext -extfile root-ca.conf \
        -in root-cert.csr -out root-cert.pem

openssl x509 -in root-cert.pem -text -noout
```

 
## 测试某个服务器是否支持特定的密码套件
执行如下命令会提示 **`CONNECTED`** 说明服务器支持此密码套件
```
$ openssl s_client -connect hci-node01:30007 -cipher ECDHE-RSA-AES256-GCM-SHA384
CONNECTED(00000003)
depth=2 O = Istio, CN = Root CA
verify error:num=19:self signed certificate in certificate chain
verify return:1
depth=2 O = Istio, CN = Root CA
verify return:1
depth=1 O = Istio, CN = Intermediate CA, L = cluster1
verify return:1
depth=0
verify return:1
DONE
---
......
```

## 获取证书
```
$ openssl s_client -showcerts -connect hci-node01:30007 > httpbin-proxy-cert.txt
```

## 验证证书
verify命令对证书的有效性进行验证，verify 指令会沿着证书链一直向上验证，直到一个自签名的CA.

```
openssl verify -CAfile <(cat Intermediate.pem RootCert.pem) UserCert.pem
```

**`语法`**
```
openssl verify[-CApath directory] [-CAfile file] [-purpose purpose] [-policy arg] [-ignore_critical] [-crl_check] [-crl_check_all] [-policy_check] [-explicit_policy] [-inhibit_any] [-inhibit_map] [-x509_strict] [-extended_crl] [-use_deltas] [-policy_print] [-untrusted file] [-help] [-issuer_checks] [-verbose] [-] [certificates]
```

 * -CAfile  filename    指定CA的证书文件，PEM格式，这个文件里可能不只包含一个证书。如果需要对证书链进行验证，指定的文件中应包含所有的证书。加入顶级CA证书文件名为0.pem，一级CA证书文件为1.pem，二级证书文件为2.pem，待验证的证书文件是eve.pem，那么需要先将0.pem，1.pem证书文件的内容包含到2.pem中。证书文件都是文本文件，简单地使用cat命令就可以进行连接.

 * -CApath directory     指定CA证书所在的目录，这个目录下可能存在证书链中的多个证书文件。为了对这个目录下的证书进行检索，证书文件的命名需要遵循xxxxxxxx.0，其中xxxxxxxx是openssl x509 -hash -in 证书， 的输出值，8个字母或数字。“.0”是要有的.



