---
title: Docker Content Trust(DCT) and Notary
tags: docker
categories:
- technologies
- docker
---

reference: https://docs.docker.com/engine/security/trust/

<!-- more -->

**`Note:`** 每次重新部署DCT和Notary时候要重启shell终端, 因为设置了很多临时环境变量会导致重新部署Notary不成功!
```
unset DOCKER_CONTENT_TRUST_SERVER
unset DOCKER_CONTENT_TRUST
```

## Machine configuration

| 机器 | IP | 作用 |
| :----- | :----- | :------: |
| master | 10.239.140.65 | 部署了docker registry仓库和notary服务器 |
| laboratory | 10.239.131.157 | 用来测试上传下载master上的registry中的images |

 * master机器：
```
cat /etc/hosts
......
10.239.140.65 master
10.239.131.157 laboratory
127.0.0.1 notary-server
```
 * laboratory机器:
```
10.239.140.65 master notaryserver
10.239.131.157 laboratory
```

## master机器

### 部署registry本地仓库

```
//改变容器运行时存储位置
$ vim /etc/docker/daemon.json
{
"insecure-registries" :["10.239.140.65:5000", "master:5000"],
"registry-mirrors": ["https://uxk0ognt.mirror.aliyuncs.com"],
"live-restore": true,
"data-root": "/home/<user-name>/data",
}

// 重新加载docker daemon, systemctl daemon-reload会重启doker运行时.
$ systemctl reload docker

// 部署私有仓库
$ docker run -d -p 5000:5000 -v /home/<user-name>/data/registry:/var/lib/registry --restart=always --name registry registry:2
```


### 安装docker-compose
```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

// Apply executable permissions to the binary:
$ sudo chmod +x /usr/local/bin/docker-compose
```

### 部署notary server
**`master机器上部署notary server, signer, mysql服务`**

notary只能在master机器上部署

```
git clone https://github.com/theupdateframework/notary.git
cd notary/fixtures/
vim regenerateTestingCerts.sh
	在"notary-server.cnf"部分的subjectAltName变量修改成如下, 加上master机器IP
	subjectAltName = DNS:notary-server, DNS:notaryserver, DNS:localhost, IP:10.239.140.65, IP:127.0.0.1

// 另外 notary-server.key, notary-signer.key, notary-escrow.key默认用已存在的key
// 需要再在regenerateTestingCerts.sh脚本里添加重新生成这三者
vim regenerateTestingCerts.sh
	......
	56 # Then generate notary-server
	57 openssl genrsa -out "notary-server.key" 3072
	58 # Use the existing notary-server key
	......
	80 # Then generate notary-signer
	81 openssl genrsa -out "notary-signer.key" 3072
	82 # Use the existing notary-signer key
	......
	104 # Then generate notary-escrow
	105 openssl genrsa -out "notary-escrow.key" 3072
	106 # Use the existing notary-escrow key
	......

cd ..    //回到notary根目录
vim  -O server.Dockerfile signer.Dockerfile 添加如下两行配置proxy
	ENV http_proxy=http://child-prc.intel.com:913
	ENV https_proxy=http://child-prc.intel.com:913
docker-compose build  // 构建image, 但不会启动容器
docker-compose up -d  // 启动容器, -d:后台运行
mkdir -p ~/.notary && cp cmd/notary/config.json cmd/notary/root-ca.crt ~/.notary

// 拷贝 notary-server.crt到docker可信赖目录
mkdir -p ~/.docker/tls/127.0.0.1:4443
cp fixtures/notary-server.crt ~/.docker/tls/127.0.0.1:4443/ca.crt
```

**`NOTE: `** 完整的regenerateTestingCerts.sh文件请参考[regenerateTestingCerts.sh](../../../../../02/02/technologies/docker/regenerateTestingCerts/)文档

修改**`~/.notary/config.json`**文件内容如下:

```
{
    "trust_dir" : "~/.docker/trust",
    "remote_server": {
        "url": "https://127.0.0.1:4443",
        "root_ca": "root-ca.crt"
    }
}
```
所有notary server 配置文件设置信息请参考如下连接:
https://docs.docker.com/notary/reference/server-config/

添加**`trust_dir`**后运行如下命令可以查看已存在的key, 但是需要参考下面下载notary client才可执行
```
$ notary key list
```

Add **`127.0.0.1 notary-server`** to your **`/etc/hosts`**, or if using docker-machine, add $(docker-machine ip) notary-server).
You can run through the examples in the getting started docs and advanced usage docs, but without the**` -s`** (server URL) argument to the **`notary`** command since the server URL is specified already in the configuration, file you copied.

You can also leave off the **`-d ~/.docker/trust`** argument if you do not care to use **`notary`** with Docker images.

```
docker ps | grep notary
	bcaa8af05a11        notary_server                    "/usr/bin/env sh -c …"   3 hours ago         Up 3 hours          0.0.0.0:4443->4443/tcp, 0.0.0.0:32769->8080/tcp   notary_server_1
	8a5f51586629        notary_signer                    "/usr/bin/env sh -c …"   3 hours ago         Up 3 hours                                                            notary_signer_1
	17920752ae65        mariadb:10.4                     "docker-entrypoint.s…"   3 hours ago         Up 3 hours          3306/tcp                                          notary_mysql_1
```

验证证书:
```
openssl verify -CAfile <(cat fixtures/intermediate-ca.crt fixtures/root-ca.crt) fixtures/notary-server.crt
```

(可选操作)添加notary根证书到主机 **`/etc/pki/ca-trust/source/anchors/`**目录, 让主机信任证书, 这里可以跳过不设置.
```
sudo cp fixtures/root-ca.crt /etc/pki/ca-trust/source/anchors/
sudo cp fixtures/intermediate-ca.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust
```


测试 connect to notary server

```
openssl s_client -connect 10.239.140.65:4443 -CAfile fixtures/root-ca.crt -no_ssl3 -no_ssl2
openssl s_client -connect 10.239.140.65:4443 -CAfile ~/.notary/root-ca.crt -no_ssl3 -no_ssl2
```

### 设置环境变量
配置notary server端证书到docker目录
```
//export DOCKER_CONTENT_TRUST_SERVER=https://<URL>:<PORT>
export DOCKER_CONTENT_TRUST_SERVER=https://127.0.0.1:4443
export DOCKER_CONTENT_TRUST=1
```


### 生成key和证书.

**可以在master机器上生成key和证书可以用来**`sign image`**, 在laboratory机器上生成key和证书可以用来sign image**, **`此时master机器和laboraty机器作用一样`**

A prerequisite for signing an image is a Docker Registry with a Notary server attached (Such as the Docker Hub ). Instructions for standing up a self-hosted environment can be found here [deploying_notary](https://docs.docker.com/engine/security/trust/deploying_notary/).

To sign a Docker Image you will need a delegation key pair. These keys can be generated locally using **`$ docker trust key generate`** or generated by a certificate authority.

在master机器和laboratry分别执行以下操作生成key和证书, 名字可以一个叫jeff, 另一个叫tester01.
```
openssl genrsa -out key.pem 3072
openssl req -new -sha512 -key key.pem -out key.csr
	填写信息直接回车, 到要填写CN处输入*然后回车
	Common Name (eg, your name or your server's hostname) []:*
openssl x509 -req -sha512 -days 365 -in key.csr -signkey key.pem -out key.crt
chmod 600 key.pem
export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE="12345678" //有这一步后下面的命令就不需要再输入密码

docker trust key load key.pem --name jeff
	输入passphrase: 12345678
```

之后会在 **`~/.docker/trust/private/`** 生成相应的key

Within the Docker CLI we can sign and push a container image with the **`$ docker trust`** command syntax. This is built on top of the Notary feature set, more information on Notary can be found here [notary-get-start](https://docs.docker.com/notary/getting_started/).



### 签名仓库和镜像
**私有仓库(如10.239.140.65:5000/python)先前已经存在, 上传 `jeff` 的 `signer key` 到此私有仓库，于此同时初始化此私有仓库**
```
docker trust signer add --key certs/key.crt jeff 10.239.140.65:5000/python
notary delegation list 10.239.140.65:5000/python
```

**`NOTE:`** 通过registry:2 搭建的private repository里的 **`python目录`** 就相当于一个私有仓库, 而其它目录如 **`dir02`** 就是另外的仓库.
如果用 **`trust signer`** 上传到 **`dir02`** 就会重新生成singer key到 **`dir02`**仓库, 此过程会需要你重新设置 **dir02**仓库singer key的PASSPHRASE
上面操作后应该是会在宿主机 **`ls ~/.docker/trust/private/`** 生成signer key, 下次操作时候可以注意下.

**签名本地image并上传到private repository**
use the delegation private key to sign a particular tag and push it up to the registry：
使用  **`jeff`** 的 **`signer key`**签名 image并上传到部署的registry私有仓库.
```
	docker trust sign 10.239.140.65:5000/python:3.9.1-slim-security-01
```
Remote trust data for a tag or a repository can be viewed by the $ docker trust inspect command:
如果sign过的image显示信息如下:

```
	$ docker trust inspect --pretty 10.239.140.65:5000/python:3.9.1-slim-security-01
	Signatures for 10.239.140.65:5000/python:3.9.1-slim-security-01
	
	SIGNED TAG               DIGEST                                                             SIGNERS
	3.9.1-slim-security-01   b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33   jeff
	
	List of signers and their keys for 10.239.140.65:5000/python:3.9.1-slim-security-01
	
	SIGNER              KEYS
	jeff                3e6bcd979c6c
	
	Administrative keys for 10.239.140.65:5000/python:3.9.1-slim-security-01
	
	  Repository Key:       fd98d671f8da8902c3202891513c743408039cf747185ce74a3790142a0a2535
	  Root Key:     f6ee60c9640fb9ed3a3086c3d884a1fde655a31e5fd0eed70e07ba985c634f95
```

没有sign过的image显示信息如下, 缺少SIGNED TAG信息:
```
	$ docker trust inspect --pretty 10.239.140.65:5000/python:3.9.1-slim
	No signatures for 10.239.140.65:5000/python:3.9.1-slim
	
	List of signers and their keys for 10.239.140.65:5000/python:3.9.1-slim
	
	SIGNER              KEYS
	jeff                3e6bcd979c6c
	
	Administrative keys for 10.239.140.65:5000/python:3.9.1-slim
	
	  Repository Key:       fd98d671f8da8902c3202891513c743408039cf747185ce74a3790142a0a2535
	  Root Key:     f6ee60c9640fb9ed3a3086c3d884a1fde655a31e5fd0eed70e07ba985c634f95
```

### 去掉镜像签名

去除Remote Trust data for a tag:
```
	docker trust revoke 10.239.140.65:5000/python:3.9.1-slim-security-01
```

使docker 客户端强制实行DCT
Content trust is disabled by default in the Docker Client. To enable it, set the DOCKER_CONTENT_TRUST environment variable to 1
```
	export DOCKER_CONTENT_TRUST=1
```

When DCT is enabled in the Docker client, **`docker`** CLI commands that operate on tagged images must either have content signatures or explicit content hashes. The commands that operate with DCT are:
 • push
 • build
 • create
 • pull
 • run

For example, with DCT enabled a docker pull someimage:latest only succeeds if someimage:latest is signed. However, an operation with an explicit content hash always succeeds as long as the hash exists:
```
	$ docker pull 10.239.140.65:5000/python:3.9.1-slim
	No valid trust data for 3.9.1-slim
	
	$ docker pull 10.239.140.65:5000/python:3.9.1-slim-security-01
	Pull (1 of 1): 10.239.140.65:5000/python:3.9.1-slim-security-01@sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33
	sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33: Pulling from python
	Digest: sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33
	Status: Image is up to date for 10.239.140.65:5000/python@sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33
	Tagging 10.239.140.65:5000/python@sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33 as 10.239.140.65:5000/python:3.9.1-slim-security-01
	10.239.140.65:5000/python:3.9.1-slim-security-01
	
	$ docker pull 10.239.140.65:5000/python:3.9.1-slim@sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33
	sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33: Pulling from python
	Digest: sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33
	Status: Image is up to date for 10.239.140.65:5000/python@sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33
	10.239.140.65:5000/python:3.9.1-slim@sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33
	
	$ docker images | grep python 
	看不到上面的10.239.140.65:5000/python:3.9.1-slim@sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33
	
	但是可以通过tag命令给此image再添加标签如:
	$ docker tag 10.239.140.65:5000/python:3.9.1-slim@sha256:b2013807b8af03d66f60a979f20d4e93e4e4a111df1287d9632c8cfd41ecfa33 10.239.140.65:5000/python:test
	$ docker images | grep python
	10.239.140.65:5000/python                               test                           edd87973afe0        2 weeks ago         114MB
```

## laboratory机器

### 配置环境变量
```
cat /etc/hosts
10.239.140.65 master notaryserver
10.239.131.157 laboratory
```

设置DCT 可信任变量
```
export DOCKER_CONTENT_TRUST_SERVER=https://10.239.140.65:4443
export DOCKER_CONTENT_TRUST=1
```

### 拷贝master notary证书
scp master机器的notary-server证书和root-ca证书 到laboratory机器.

```
mkdir -p ~/.docker/tls/10.239.140.65:4443/
scp root@10.239.140.65:/root/.docker/tls/127.0.0.1:4443/ca.crt ~/.docker/tls/10.239.140.65:4443/ca.crt
mkdir -p ~/.notary
scp root@10.239.140.65:/root/.notary/root-ca.crt ~/.notary/root-ca.crt
```

编写notary config 文件
```
cat ~/.notary/config.json
{
        "trust_dir" : "~/.docker/trust",
        "remote_server": {
                "url": "https://10.239.140.65:4443",
                "root_ca": "root-ca.crt"
        }
}
```

### 生成key和证书
在laboratory机器上生成signer **`bob`** 的 key 和 证书, 添加signer **`bob`** 的 key到指定 registry 仓库, 之后sign image
```
mkdir certs
cd certs
openssl genrsa -out key.pem 3072
openssl req -new -sha512 -key key.pem -out key.csr
	填写信息直接回车, 到要填写CN处输入*然后回车
	Common Name (eg, your name or your server's hostname) []:*
openssl x509 -req -sha512 -days 365 -in key.csr -signkey key.pem -out key.crt
chmod 600 key.pem
export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE="12345678" //有这一步后下面的命令就不需要再输入密码

docker trust key load key.pem --name bob
	输入passphrase: 12345678
之后会在 **`~/.docker/trust/private/`** 生成相应的key
```

### 签名仓库和镜像并上传
```
// 添加signer key到10.239.140.65:5000/python-test10仓库
$ docker trust signer add --key certs/key.crt bob 10.239.140.65:5000/python-test10

// 查看delegation key
$ notary delegation list 10.239.140.65:5000/python-test10
  ROLE                PATHS             KEY IDS                                                             THRESHOLD
  ----                -----             -------                                                             ---------
  targets/releases    "" <all paths>    7e1f01ed1d6d0698d5cd5856ec9be27debed33645d2450f20495b70a8a5db83c    1
  targets/bob         "" <all paths>    7e1f01ed1d6d0698d5cd5856ec9be27debed33645d2450f20495b70a8a5db83c    1

// sign image并上传到10.239.140.65:5000/python-test10仓库
$ docker trust sign 10.239.140.65:5000/python-test10:3.9.1-slim-security-01

// 再次sign同一个image会提示image的sign已存在
$ docker trust sign 10.239.140.65:5000/python-test10:3.9.1-slim-security-01
  Signing and pushing trust metadata for 10.239.140.65:5000/python-test10:3.9.1-slim-security-01
  Existing signatures for tag 3.9.1-slim-security-01 digest b9c45d9576c0e70dfe19ec5f481ec4b59daf970a5036463fcb523cf8ebf7fc5c from:
  bob
  Successfully signed 10.239.140.65:5000/python-test10:3.9.1-slim-security-01
```

### 查看registry私有仓库
```
//查看所有仓库
$ curl 10.239.140.65:5000/v2/_catalog

//查看指定仓库所有tag
$ curl 10.239.140.65:5000/v2/python-test10/tags/list
{"name":"python-test10","tags":["3.9.1-slim-security-01", "3.9.1-slim-security-02"]}
```


## notary client
**master和laboratory都需要下载notary client, notary client是一个二进制可执行文件.**

If you would like to use your own Notary server, it is important to use the same or a newer Notary version, as the client for feature compatibility (ex: client version 0.2, server/signer version >= 0.2). (reference: https://docs.docker.com/notary/getting_started/#inspect-a-docker-hub-repository)
查看nodary server signer版本

```
docker images | grep notary

wget https://github.com/theupdateframework/notary/releases/download/v0.6.1/notary-Linux-amd64
chmod +x notary-Linux-amd64
cp notary-Linux-amd64 /usr/local/bin/notary
```

查看notary client版本
```
$ notary version
notary
 Version:    0.6.1
 Git commit: d6e1431f
```

The Notary client used in isolation does not know where the trust repositories are located. So, you must provide the **`-s`** (or long form **`--server`** ) flag to tell the client which repository server it should communicate with.

Additionally, Notary stores your own signing keys, and a cache of previously downloaded trust metadata in a directory, provided with the **`-d`** flag. When interacting with Docker Hub repositories, you must instruct the client to use the associated trust directory, which by default is found at **`.docker/trust`** within the calling user’s home directory (failing to use this directory may result in errors when publishing updates to your trust data):

```
$ notary -s https://notary.docker.io -d ~/.docker/trust list docker.io/library/alpine
   NAME                                 DIGEST                                SIZE (BYTES)    ROLE
------------------------------------------------------------------------------------------------------
  2.6      e9cec9aec697d8b9d450edd32860ecd363f2f3174c8338beb5f809422d182c63   1374           targets
  2.7      9f08005dff552038f0ad2f46b8e65ff3d25641747d3912e3ea8da6785046561a   1374           targets
  3.1      e876b57b2444813cd474523b9c74aacacc238230b288a22bccece9caf2862197   1374           targets
  3.2      4a8c62881c6237b4c1434125661cddf09434d37c6ef26bf26bfaef0b8c5e2f05   1374           targets
  3.3      2d4f890b7eddb390285e3afea9be98a078c2acd2fb311da8c9048e3d1e4864d3   1374           targets
  edge     878c1b1d668830f01c2b1622ebf1656e32ce830850775d26a387b2f11f541239   1374           targets
  latest   24a36bbc059b1345b7e8be0df20f1b23caa3602e85d42fff7ecd9d0bd255de56   1377           targets
```


## Rotate keys
Reference: https://github.com/theupdateframework/notary/blob/master/docs/advanced_usage.md#manage-keys



## 清理notary

```
cd notary
rm -rf ~/.notary
rm -rf ~/.docker
docker-compose down
docker rmi notary_server:latest
docker rmi notary_signer:latest
unset DOCKER_CONTENT_TRUST_SERVER
unset DOCKER_CONTENT_TRUST
```

### 清理notary key 
Remove the key from the local Docker Trust store
```
// 查看有哪些key
notary key list

// 清理key
notary key remove 1091060d7bfd938dfa5be703fa057974f9322a4faef6f580334f3d6df44c02d1
```
以下命令可以直接从本地删除key, 因此避免误删最好备份.
```
rm -rf ~/.notary/trust/private/<key-number>
```




