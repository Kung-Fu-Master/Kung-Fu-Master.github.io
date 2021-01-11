---
title: Centos安装chrome
tags:
categories:
- linux
---

## Centos安装chrome

<!-- more -->

### 1.配置yum下载源：

在目录 /etc/yum.repos.d/ 下新建文件 google-chrome.repo
```
[root@localhost ~]#  cd /ect/yum.repos.d/
[root@localhost yum.repos.d]#  vim google-chrome.repo
```

编辑google-chrome.repo，内容如下，，编辑后保存退出(：wq)

```
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
```
安装google chrome浏览器：

```
[root@localhost yum.repos.d]# yum -y install google-chrome-stable --nogpgcheck
```
查看浏览器版本

```
google-chrome --version
```

### 2.驱动的安装

下载地址

```
https://chromedriver.storage.googleapis.com/index.html
```
找到相应的驱动发到 /usr/bin/

```
mv chromedriver /usr/bin/
```
查看所属权限

```
ls -l chromedriver
```
赋予执行权限并将其赋给要执行的用户

```
chmod 755 /usr/bin/chromedriver
chown xxx:xxx /usr/bin/chromedriver
```
此上操作就可用程序自动化调用谷歌浏览器



