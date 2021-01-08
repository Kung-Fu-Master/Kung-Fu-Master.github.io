---
title: 亚马逊aws-sdk-cpp
tags: security
categories:
- technologies
- security
---

github: https://github.com/aws/aws-sdk-cpp

## Centos 编译执行
1. 更新下curl, 可以参考"curl升级教程"
2. 更新gcc和g++, 可以参考"Devtoolset 升级gcc到8.3.1"

```shell
	wget https://github.com/aws/aws-sdk-cpp/archive/1.8.55.tar.gz
	tar -zxvf 1.8.55.tar.gz
	cd aws-sdk-cpp-1.8.55
	mkdir build
	cd build
	cmake <path-to-root-of-this-source-code(aws-sdk-cpp-1.8.55的绝对路径)> -DCMAKE_BUILD_TYPE=Debug
	make
	sudo make install
```

