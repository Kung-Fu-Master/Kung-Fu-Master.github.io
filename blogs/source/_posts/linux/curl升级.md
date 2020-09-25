---
title: curl 升级
tags: 
categories:
- linux
---

官网:
https://curl.haxx.se/download.html#LinuxRedhat
https://mirror.city-fan.org/ftp/contrib/sysutils/Mirroring/
http://www.city-fan.org/ftp/contrib/yum-repo/rhel6/x86_64/


## 卸载旧版本curl

	rpm -qa | grep curl
	  libcurl-7.29.0-57.el7_8.1.x86_64
	  python-pycurl-7.19.0-19.el7.x86_64
	  curl-7.29.0-57.el7_8.1.x86_64
	  libcurl-devel-7.29.0-57.el7_8.1.x86_64

卸载: 基本语法

	rpm -e <RPM包的名称>
增加参数 --nodeps ,就可以强制删除，但是一般不推荐这样做，因为依赖于该软件包的程序可能无法运行.  

## 安装curl包
基本语法:

	rpm -ivh  <RPM包全路径名称>

参数说明： i=install 安装 v=verbose 提示 h=hash  进度条

## 升级curl包
需要升级curl包根本原因在于 curl 7.19或7.29 版本不支持 TLS v1.1 以上的协议，如果需要支持 TLS v1.2 版本，至少需要升级到 curl 7.34 版本。

	rpm -Uvh  http://www.city-fan.org/ftp/contrib/yum-repo/rhel6/x86_64/city-fan.org-release-2-1.rhel6.noarch.rpm
	yum --showduplicates list curl --disablerepo="*" --enablerepo="city*"
	(与上面好像不一样)yum --showduplicates list curl --disablerepo="*"  --enablerepo="fan*"
	  Loaded plugins: fastestmirror, langpacks
	  Loading mirror speeds from cached hostfile
	   * city-fan.org: nervion.us.es
	   * city-fan.org-source: nervion.us.es
	  Installed Packages
	  curl.x86_64                                             7.29.0-57.el7_8.1                                               @updates
	  Available Packages
	  curl.x86_64                                             7.72.0-2.0.cf.rhel7                                             city-fan.org
修改该 "city-fan.org" repo的enable为1

	vim /etc/yum.repos.d/city-fan.org.repo
	[city-fan.org]
	name=city-fan.org repository for Red Hat Enterprise Linux (and clones) $releasever ($basearch)
	#baseurl=http://mirror.city-fan.org/ftp/contrib/yum-repo/rhel$releasever/$basearch
	mirrorlist=http://mirror.city-fan.org/ftp/contrib/yum-repo/mirrorlist-rhel$releasever
	enabled=1
	gpgcheck=1
	gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-city-fan.org

安装最新的curl

	yum install curl
	  ......
	  ---> Package libcurl-devel.x86_64 0:7.29.0-57.el7_8.1 will be updated
	  ---> Package libcurl-devel.x86_64 0:7.72.0-2.0.cf.rhel7 will be an update
	  ......
如果提示缺少依赖 libnghttp2.so.14()(64bit)

	rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/Packages/l/libnghttp2-1.6.0-1.el6.1.x86_64.rpm
然后重复第4步即可
查看curl版本

	curl -V
	curl 7.72.0 (x86_64-redhat-linux-gnu) libcurl/7.72.0 NSS/3.44 zlib/1.2.7 libpsl/0.7.0 (+libicu/50.1.2) libssh2/1.9.0 nghttp2/1.33.0
	Release-Date: 2020-08-19
	Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp
	Features: AsynchDNS GSS-API HTTP2 HTTPS-proxy IPv6 Kerberos Largefile libz Metalink NTLM NTLM_WB PSL SPNEGO SSL UnixSockets

(疑惑地方)实践安装后在 curl 7.72 版本中，使用的密码学库并没有从 NSS 替换为 OpenSSL 了，输入以下命令可以看出：

	curl-config --ssl-backends
	  NSS



## reference
https://www.cnblogs.com/kingsonfu/p/10069755.html
https://bbs.huaweicloud.com/blogs/123952



