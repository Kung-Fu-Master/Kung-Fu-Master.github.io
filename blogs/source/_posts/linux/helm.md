---
title: helm
tags:
categories:
- linux
---

## Helm

[Helm offical installation tutorial](https://helm.sh/docs/intro/install/)
Manually download and install the Helm release version: [release](https://github.com/helm/helm/releases)

	$ curl -O https://get.helm.sh/helm-v3.3.1-linux-amd64.tar.gz
	$ tar -zxvf helm-v3.3.1-linux-amd64.tar.gz
	$ mv linux-amd64/helm /usr/local/bin/helm

From there, you should be able to run the client and add the stable repo: `helm help`