---
title: openssl certificates
tags: security
categories:
- technologies
- security
---

# **X.509证书标准定义的两种编码格式PEM和DER**

## **PEM编码（Privacy Enhanced Mail）**
特点：纯文本文件, 以-----BEGIN CERTIFICATE-----开头, 以-----END CERTIFICATE-----结尾,  
内容是 base64 编码. 但使用文本编辑器只能查看表面的结构, 需要输入命令例如  

```shell
	$ openssl x509 -in 某个PEM格式数字证书.pem -text -noout
```

才能看到原始的数字证书信息.  

```
	-----BEGIN CERTIFICATE-----
	MIID7TCCAtWgAwIBAgIJAOIRDhOcxsx6MA0GCSqGSIb3DQEBCwUAMIGLMQswCQYD
	……
	xAJz+w8tjrDWcf826VN14IL+/Cmqlg/rIfB5CHdwVIfWwpuGB66q/UiPegZMNs8a
	3g==
	-----END CERTIFICATE-----
```
## **DER编码（Distinguished Encoding Rules）**
特点：二进制文件格式, 一般应使用 Windows/Java 开发工具打开, 也可以使用openssl命令行工具提取其中信息或进行编码转换.  

```shell
	$ openssl x509 -in 某个DER格式的数字证书.der -inform der -text -noout  
```
上面这个命令查看二进制文件中的证书信息.  

## **文件扩展名**
<table><tr><td bgcolor=#54FF9F><font face="fantasy" size=4>我们身边有很多常见的数字证书文件, 他们的扩展名通常既不叫".pem"也不叫".der", 但无论扩展名是什么, 其内部编码格式只能从PEM和DER这两种编码格式中选择一种.</font></td></tr></table>  
不同平台所偏好的编码格式不同, 不同类型的数字证书文件中存储的内容也略有差别, 不过对于大部分证书文件, 我们都可以借助命令行工具随时将其转换成另一种编码格式.  

 * **CRT** - CRT应该是certificate的三个字母,其实还是证书的意思,常见于NIX系统,有可能是PEM编码,也有可能是DER编码,大多数应该是PEM编码,相信你已经知道怎么辨别. **证书内容包含: signer(如CA:kubernetes),个人信息,过期时间,公钥public key,加密算法(非对称加密算法RSA2048, 对称机密算法AES256等),签名算法(sign algorithm)等信息.**  
 * **KEY** - 通常用来存放一个公钥或者私钥,并非X.509证书,编码同样的,可能是PEM,也可能是DER.  
查看KEY的办法:  

```shell
	$ openssl rsa -in mykey.key -text -noout
```
如果是DER格式的话,同理应该这样了:

```shell
	$ openssl rsa -in mykey.key -text -noout -inform der

	$ cd /home/zhan/istio-1.6.0/samples/certs/
	$ openssl x509 -in root-cert.pem -text -noout
	Signature Algorithm: sha256WithRSAEncryption
	$ openssl x509 -in root-cert.pem -text -noout | grep Validity -A 2
	Validity
	        Not Before: Jan 24 19:15:51 2018 GMT
	        Not After : Dec 31 19:15:51 2117 GMT
	-A n这样的shell写法，输出当前行之后的n行内容
```

 * **CSR - Certificate Signing Request**,即证书签名请求,这个并不是证书,而是向权威证书颁发机构获得签名证书的申请,其核心内容是一个**公钥和个人信息**,在生成这个申请的时候, 要对应生成的有一个私钥,私钥要自己保管好, client端把包含public key的csr文件发给CA, 用CA的private key给csr文件做签名(sign)生成client端证书. **CSR文件**内容一般包含: **个人信息,公钥public key,加密算法(非对称加密算法RSA2048, 对称机密算法AES256等),签名算法(sign algorithm)等信息.**  
查看的办法:

```shell
	$ openssl req -noout -text -in my.csr
```

(如果是DER格式的话照旧命令行加上-inform der,这里不写了)

## **证书编码的转换**

 • PEM转为DER：

```shell
	$ openssl x509 -in cert.crt -outform der -out cert.der
```
 • DER转为PEM：

```shell
	$ openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
```
## **获得证书的步骤**

<table><tr><td bgcolor=#54FF9F> • 向权威证书颁发机构申请证书</td></tr></table>
用以下命令生成一个csr:

```shell
	$ openssl req -newkey rsa:2048 -new -nodes -keyout my.key -out my.csr
	$ ls
	my.csr  my.key
```
把**csr**交给权威证书颁发机构,权威证书颁发机构对此进行签名,完成.保留好csr,**当权威证书颁发机构颁发的证书过期的时候,你还可以用同样的csr来申请新的证书,key保持不变.**

<table><tr><td bgcolor=#54FF9F> • 或者生成自签名的证书</td></tr></table>

```shell
	$ openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
	$ ls
	cert.pem  key.pem
```

在生成证书的过程中会要你填一堆的东西,其实真正要填的只有Common Name,通常填写你服务器的域名,如"yourcompany.com",或者你服务器的IP地址,其它都可以留空的.
生产环境中还是不要使用自签的证书,否则浏览器会不认,或者如果你是企业应用的话能够强制让用户的浏览器接受你的自签证书也行.向权威机构要证书通常是要钱的,但现在也有免费的,仅仅需要一个简单的域名验证即可.有兴趣的话查查"沃通数字证书".

## **生成证书**

### **一：生成CA证书**
目前不使用第三方权威机构的CA来认证，自己充当CA的角色。
网上下载一个openssl软件
1.创建私钥：
通常是rsa算法  
<table><tr><td bgcolor=#54FF9F>**ca-key.pem**</td></tr></table>

```shell
	$ openssl genrsa -out ca/ca-key.pem 2048
	查看key
	$ openssl rsa -in ca/ca-key.pem -text -noout
	如果是DER格式的话,同理应该这样
	$ openssl rsa -in ca/ca-key.pem -text -noout -inform der
```

2.创建证书请求：  
> 因此在用户向CA申请数字证书时，用户首先需要在自己的电脑中先产生一个公私钥对。用户需要保管好自己的私钥，然后再把公钥和你的个人信息发送给CA机构，CA机构通过你的公钥和个人信息最终签发出数字证书。  
> 而CSR文件，其实就是包含了用户公钥和个人信息的一个数据文件。用户产生出这个CSR文件，再把这个CSR文件发送给CA，CA就会根据CSR中的内容来签发出数字证书。  

在制作csr文件的时，必须使用自己的私钥来签署申，还可以设定一个密钥.
<table><tr><td bgcolor=#54FF9F>**ca-req.csr**</td></tr></table>

```shell
	$ openssl req -new -out ca/ca-req.csr -key ca/ca-key.pem
	  Country Name (2 letter code) [AU]:cn
	  State or Province Name (full name) [Some-State]:zhejiang
	  Locality Name (eg, city) []:hangzhou
	  Organization Name (eg, company) [Internet Widgits Pty Ltd]:skyvision
	  Organizational Unit Name (eg, section) []:test
	  Common Name (eg, YOUR name) []:root
	  Email Address []:sky
```

3.自签署证书 ：
<table><tr><td bgcolor=#54FF9F>**ca/ca-cert.pem**</td></tr></table>

```shell
	$ openssl x509 -req -in ca/ca-req.csr -out ca/ca-cert.pem -signkey ca/ca-key.pem -days 3650
	查看证书格式:
	$ openssl x509 -in ca/ca-cert.pem -text -noout
```

4.将证书导出成浏览器支持的.p12格式 ：

```shell
	$ openssl pkcs12 -export -clcerts -in ca/ca-cert.pem -inkey ca/ca-key.pem -out ca/ca.p12
```
密码：changeit

### **二.生成server证书。**
1.创建私钥 ：
<table><tr><td bgcolor=#54FF9F>**server/server-key.pem**</td></tr></table>

```shell
	$ openssl genrsa -out server/server-key.pem 2048
	查看key
	$ openssl rsa -in server/server-key.pem -text -noout
	如果是DER格式的话,同理应该这样
	$ openssl rsa -in server/server-key.pem -text -noout -inform der
```

2.创建证书请求 ：
<table><tr><td bgcolor=#54FF9F>**server/server-req.csr**</td></tr></table>

```shell
	$ openssl req -new -out server/server-req.csr -key server/server-key.pem
```
```text
	  Country Name (2 letter code) [AU]:cn
	  State or Province Name (full name) [Some-State]:zhejiang
	  Locality Name (eg, city) []:hangzhou
	  Organization Name (eg, company) [Internet Widgits Pty Ltd]:skyvision
	  Organizational Unit Name (eg, section) []:test
	  Common Name (eg, YOUR name) []:192.168.1.246 注释：一定要写服务器所在的ip地址
	  Email Address []:sky
```
查看csr文件内容:
```shell
	$ $ openssl req -in server-req.csr -text -noout // -noout 不用输出csr文件原始内容
```
``` text
	Certificate Request:
	    Data:
	        Version: 0 (0x0)
	        Subject: C=XX, L=Default City, O=Default Company Ltd, CN=10.239.140.186
	        Subject Public Key Info:
	            Public Key Algorithm: rsaEncryption
	                Public-Key: (2048 bit)
	                Modulus:
	                    00:e2:0c:a7:33:33:d9:9b:90:1b:29:30:3c:81:31:
	                    09:97:0a:a9:76:d5:54:be:63:17:21:0c:b9:3a:f0:
	                    a6:02:37:1a:d4:1e:53:4e:e0:c8:d9:5f:27:57:7f:
	                    f3:eb:7f:9d:ad:79:d6:e7:40:64:c8:bc:3d:f3:b4:
	                    16:d6:30:e9:16:04:b6:a0:0e:8f:75:e4:4b:d6:8e:
	                    0a:8e:75:d8:41:89:09:90:96:b0:8d:32:5f:b5:96:
	                    1d:65:d6:a6:b4:c7:eb:3d:3b:f9:62:36:69:7d:07:
	                    6d:05:89:ce:a6:a5:98:a0:b2:5f:ab:bc:25:ba:08:
	                    d8:86:0a:b9:c0:91:ca:f8:d3:bb:36:14:21:f9:c2:
	                    b5:53:43:a9:2c:03:39:9b:93:ef:1d:d9:20:ef:dd:
	                    ff:57:c6:b5:47:e8:bb:46:32:e3:1d:3b:2e:5b:15:
	                    11:80:72:f6:2e:f5:b2:cc:02:7f:b1:d6:e9:3d:8e:
	                    0e:66:f6:6d:45:0e:2f:8c:d5:c3:92:dc:a1:9a:d9:
	                    b0:33:82:30:69:0a:05:ee:08:1b:a6:81:f4:bb:31:
	                    0d:fa:26:37:eb:4f:c8:58:df:e5:be:cc:ac:9a:62:
	                    42:f1:af:8c:35:88:e4:f3:b4:76:8f:6c:13:1f:9a:
	                    61:e0:08:0f:f2:b1:d6:f3:61:b4:0a:5d:9a:61:5f:
	                    e1:0b
	                Exponent: 65537 (0x10001)
	        Attributes:
	            a0:00
	    Signature Algorithm: sha256WithRSAEncryption
	         5b:62:35:07:43:99:dc:af:7c:61:1e:76:4e:f8:ef:59:b2:27:
	         60:71:30:15:5d:f3:0b:b1:b4:53:29:ec:d1:7c:18:48:0a:b3:
	         fe:b7:6d:80:ef:dc:c6:24:04:3d:bd:c1:b8:61:49:f3:1e:fb:
	         22:0f:fb:06:99:ec:db:18:ac:34:ff:4b:15:f8:84:06:01:4d:
	         68:4f:0c:a2:a5:34:dc:1b:61:44:c7:ff:ef:5d:92:a1:09:3f:
	         11:27:1c:a7:30:8e:97:6a:08:03:99:e6:6a:8f:1d:d6:ea:e7:
	         cd:18:a7:eb:36:3d:e7:6b:5e:ef:72:85:ca:eb:89:97:02:cf:
	         fc:38:31:58:e1:66:85:d1:e7:49:e2:72:ef:b1:60:36:55:d7:
	         90:bd:8d:0e:d8:c6:8f:d2:bf:bf:43:85:36:04:2e:f1:ec:5f:
	         d8:1b:17:22:a4:6a:de:a7:b2:2b:00:30:27:e6:4b:32:4d:55:
	         70:b0:61:3d:3f:f2:9d:e7:24:f6:4c:1f:bf:63:6a:d9:16:ef:
	         cb:91:a3:a4:43:b5:1f:11:85:ad:0e:b1:57:39:f2:0a:56:ec:
	         52:90:b0:11:96:c6:28:e0:de:0c:eb:f2:b1:66:ce:04:48:7f:
	         11:90:09:1d:fd:ca:a7:25:66:32:a2:64:33:1a:5e:a9:85:50:
	         8a:2d:90:a5
```

3.自签署证书 ：
<table><tr><td bgcolor=#54FF9F>**server/server-cert.pem**</td></tr></table>

```shell
	$ openssl x509 -req -in server/server-req.csr -out server/server-cert.pem -signkey server/server-key.pem -CA ca/ca-cert.pem -CAkey ca/ca-key.pem -CAcreateserial -days 3650
```
```
	 * -CA选项指明用于被签名的csr证书
	 * -CAkey选项指明用于签名的密钥
	 * -CAcreateserial指明文件不存在时自动生成
```
查看证书格式:
```shell
	$ openssl x509 -in server/server-cert.pem -text -noout
```
可以查看到证书里所包含的public key等相关信息:
```
	Certificate:
	Data:
	    Version: 1 (0x0)
	    Serial Number:
	        cc:db:c0:f2:12:e8:09:27
	Signature Algorithm: sha256WithRSAEncryption
	    Issuer: C=XX, L=Default City, O=Default Company Ltd, CN=AI	// 签发者(CA机构)
	    Validity
	        Not Before: Jul 16 07:01:18 2020 GMT
	        Not After : Jul 14 07:01:18 2030 GMT
	    Subject: C=XX, L=Default City, O=Default Company Ltd, CN=sky
	    Subject Public Key Info:
	        Public Key Algorithm: rsaEncryption
	            Public-Key: (2048 bit)			// public key, 因此本地可以不需要再存储保留public key, 证书里已包含.
	            Modulus:
	                00:cd:d7:ed:e9:c6:5e:fa:bc:ef:1e:4e:92:52:99:
	                f0:34:96:67:7b:32:1b:f6:53:df:ca:7b:e5:72:6a:
	                29:e5:85:27:eb:71:00:c6:90:ac:c1:64:62:0d:2b:
	                b1:bc:b8:ee:e1:d4:54:b7:95:21:1e:de:56:c7:25:
	                4c:d4:2d:29:5f:48:19:8a:05:c4:33:d3:06:16:ec:
	                68:e2:81:07:cf:f9:d1:15:b2:68:3d:da:44:c3:d5:
	                ba:a3:0f:9e:34:34:71:53:4f:02:4b:eb:f8:de:fd:
	                94:3f:f4:ee:12:48:ea:b1:60:62:be:58:47:78:29:
	                59:5b:ae:57:53:23:31:aa:78:cc:6c:f0:f7:e9:76:
	                4a:b9:25:79:3f:9c:05:4e:f0:8e:87:32:df:87:72:
	                67:64:2e:9f:85:15:64:bf:ca:ce:33:71:ee:bb:1a:
	                d3:26:09:34:9b:65:b9:15:71:28:14:37:48:79:1b:
	                b1:99:a4:8c:cc:27:a1:a4:c4:28:8e:01:e5:08:db:
	                e6:45:6e:3d:d9:03:a9:cb:17:25:b7:c9:c9:4b:fb:
	                e5:93:d1:de:31:fe:a9:34:29:c3:29:a4:27:c2:eb:
	                66:99:c6:db:ba:52:07:30:97:d4:0a:1e:1b:5d:72:
	                f6:ff:19:92:22:c0:44:76:74:f7:a7:0d:c5:77:c8:
	                1c:55
	            Exponent: 65537 (0x10001)
	Signature Algorithm: sha256WithRSAEncryption
	     28:d1:d9:29:a5:40:f3:d3:d6:95:87:fd:2c:70:dc:0f:1c:86:
	     08:35:d0:a8:8e:d0:5d:78:28:ae:88:33:61:db:cd:b6:80:1c:
	     88:62:b8:ce:cf:87:14:15:bd:27:9a:3e:77:cb:a1:e0:11:0d:
	     89:ef:f2:e8:b2:2c:cf:96:26:bd:06:3a:7b:8f:4b:fa:b2:c3:
	     f9:14:3e:18:ef:57:b5:37:95:01:a0:0f:bf:6e:5c:c9:47:7b:
	     1a:ed:ca:7a:31:a1:89:e8:0d:4d:95:d2:61:e3:b8:48:e5:86:
	     19:91:3e:00:86:07:50:df:e2:57:29:69:61:c4:cc:55:8f:60:
	     de:20:c1:0d:7d:c7:98:52:f4:34:08:90:c5:90:34:ec:86:0f:
	     ad:9b:e7:1a:d4:7b:d9:dd:59:82:de:54:d3:87:8e:e7:82:ae:
	     22:70:cf:e7:d7:8c:f1:55:57:6d:41:e6:44:3c:83:b7:73:7e:
	     9a:d5:1d:af:72:e9:4e:88:d4:4f:9f:33:2f:dc:1b:50:10:8b:
	     db:cd:e4:0e:e6:96:cd:c6:27:c7:4b:c7:9f:05:74:32:35:7e:
	     99:78:36:30:ae:78:4b:c3:1a:6b:b8:db:62:23:b8:ab:22:11:
	     11:81:95:5d:46:f0:45:15:77:1f:6b:c0:bf:9d:a2:d2:b4:62:
	     c9:b5:2b:dd
```

4.将证书导出成浏览器支持的.p12格式 ：

```shell
	$ openssl pkcs12 -export -clcerts -in server/server-cert.pem -inkey server/server-key.pem -out server/server.p12
```
密码：changeit

Additional：
因为server端的证书是由CA的private key签名(sign)server端的public key及其持有者的真实身份得到的, 因此server端证书里就包含server的public key.  
就不需要用Openssl生成server端的public key. 用server端的**csr（certificate signing requests）**生成server证书即可. client端也一样.  
client端只需要用CA的public key(CA的public key所有人都能获取)解密server端存放在CA的证书文件就可以得到Server端的public key.  
然后client端就可以用解密得到的server端的public key对server端用自身private key加密的信息附带的摘要A(digest)进行解密.  
client端将server端发过来的信息用Hash得到摘要B(digest)与解密得到的摘要A(digest)对比, 如果一直则说明信息没有被黑客篡改.  
如果中间黑客把server端加密的摘要A(digest)修改了，则client用从CA的public key解密得到的server端的public key是解不开摘要A(digest)的, 说明摘要信息被黑客篡改, 内容不可靠.  

如果要生成RSA公钥, command如下:

```shell
	$ openssl rsa -in server/server-key.pem -pubout -out server/server-public-key.pem
```

### **三.生成client证书。**
1.创建私钥 ：
<table><tr><td bgcolor=#54FF9F>**client/client-key.pem**</td></tr></table>

```shell
	$ openssl genrsa -out client/client-key.pem 2048
	查看key
	$ openssl rsa -in client/client-key.pem -text -noout
	如果是DER格式的话,同理应该这样
	$ openssl rsa -in client/client-key.pem -text -noout -inform der
```

2.创建证书请求 ：
<table><tr><td bgcolor=#54FF9F>**client/client-req.csr**</td></tr></table>

```shell
	$ openssl req -new -out client/client-req.csr -key client/client-key.pem
	  Country Name (2 letter code) [AU]:cn
	  State or Province Name (full name) [Some-State]:zhejiang
	  Locality Name (eg, city) []:hangzhou
	  Organization Name (eg, company) [Internet Widgits Pty Ltd]:skyvision
	  Organizational Unit Name (eg, section) []:test
	  Common Name (eg, YOUR name) []:sky
	  Email Address []:sky 注释：就是登入中心的用户（本来用户名应该是Common Name，但是中山公安的不知道为什么使用的Email Address，其他版本没有测试）
	  Please enter the following ‘extra’ attributes
	  to be sent with your certificate request
	  A challenge password []:123456
	  An optional company name []:tsing
```

3.自签署证书 ：
<table><tr><td bgcolor=#54FF9F>**client/client-cert.pem**</td></tr></table>

```shell
	$ openssl x509 -req -in client/client-req.csr -out client/client-cert.pem -signkey client/client-key.pem -CA ca/ca-cert.pem -CAkey ca/ca-key.pem -CAcreateserial -days 3650
	 * -CA选项指明用于被签名的csr证书
	 * -CAkey选项指明用于签名的密钥
	 * -CAcreateserial指明文件不存在时自动生成
	查看证书格式:
	$ openssl x509 -in client/client-cert.pem -text -noout
```

4.将证书导出成浏览器支持的.p12格式 ：

```shell
	$ openssl pkcs12 -export -clcerts -in client/client-cert.pem -inkey client/client-key.pem -out client/client.p12
```
密码：changeit

Additional. 生成RSA公钥:

```shell
	$ openssl rsa -in client/client-key.pem -pubout -out client/client-public-key.pem
```

请一定严格根据里面的步骤来，待实验成功后，修改你自己想要修改的内容。我就是一开始没有安装该填写的来，结果生成的证书就无法配对成功。


