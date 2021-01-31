---
title: 安装 docker compose
tags: docker
categories:
- technologies
- docker
---

Reference: https://docs.docker.com/compose/install/

<!-- more -->

## 下载docker-compose

Run this command to download the current stable release of Docker Compose:
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
> To install a different version of Compose, substitute 1.28.2 with the version of Compose you want to use.

## 复制到环境变量
Apply executable permissions to the binary:
```
sudo chmod +x /usr/local/bin/docker-compose
```
**`Note:`** If the command docker-compose fails after installation, check your path. You can also create a symbolic link to /usr/bin or any other directory in your path.
For example:
```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

## 命令自动补充

Optionally, install [command completion](https://docs.docker.com/compose/completion/) for the **`bash`** and **`zsh`** shell.

Test the installation.
```
$ docker-compose --version
docker-compose version 1.28.2, build 1110ad01
```


