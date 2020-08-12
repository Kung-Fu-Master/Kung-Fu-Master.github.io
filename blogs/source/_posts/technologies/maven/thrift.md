---
title: thrift
tags: 
categories:
- technologies
- maven
---

## Thrift 简介
Thrift可以定义多种语言的函数接然后并生成源码文件.

## Ubuntu18.04 安装 Thrift
 * 1. sudo apt-get install automake bison flex g++ git libboost-all-dev libevent-dev libssl-dev libtool make pkg-config
 * 2. tar -zxvf thrift-0.9.3.tar.gz
 * 3. cd thrift-0.9.3
 * 4.  ./configure CXXFLAGS='-ggdb3' --with-java --with-python --without-cpp --without-boost --without-csharp --without-erlang --without-perl --without-php --without-php_extension --without-ruby --without-haskell --without-go
 * 5. make
 * 6. make install
 * 7. thrift -version
 * 8. cd thrift-0.9.3/lib/py
 * 9. python setup.py install

### 遇到的问题:
#### 问题1: from thrift.Thrift import TType, TMessageType, TFrozenDict, TException, TApplicationException ImportError: cannot import name TFrozenDict
#### 问题2: 安装sasl的时候报错：sasl/saslwrapper.h:22:23: fatal error: sasl/sasl.h: No such file or directory
 * 久经折腾，最终发现缺少包：https://pypi.python.org/pypi/sasl/0.1.3
 * Debian/Ubuntu:$ apt-get install python-dev libsasl2-dev gcc
 * CentOS/RHEL:$ yum install gcc-c++ python-devel.x86_64 cyrus-sasl-devel.x86_64
 * 最后安装: 某些包没有关联上，装包时加上[hive]后缀 $ pip install pyhive[hive]


