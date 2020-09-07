---
title: MinIO 03 client
tags: storage
categories:
- storage
---

Official web site: [https://docs.min.io/docs/minio-client-quickstart-guide.html](https://docs.min.io/docs/minio-client-quickstart-guide.html)
下面介绍的是使用MinIO自导的client： mc

## **Download mc tool**
GNU/Linux

	$ wget https://dl.min.io/client/mc/release/linux-amd64/mc
	$ chmod +x mc
	$ ./mc --help
	$ ./mc --version

Microsoft Windows

	下载:https://dl.min.io/client/mc/release/windows-amd64/mc.exe
	mc.exe --help
MinIO Client (mc) provides a modern alternative to UNIX commands like ls, cat, cp, mirror, diff etc. It supports filesystems and Amazon S3 compatible cloud storage service (AWS Signature v2 and v4).

	alias       set, remove and list aliases in configuration file
	ls          list buckets and objects
	mb          make a bucket
	rb          remove a bucket
	cp          copy objects
	mirror      synchronize object(s) to a remote site
	cat         display object contents
	head        display first 'n' lines of an object
	pipe        stream STDIN to an object
	share       generate URL for temporary access to an object
	find        search for objects
	sql         run sql queries on objects
	stat        show object metadata
	mv          move objects
	tree        list buckets and objects in a tree format
	du          summarize disk usage recursively
	retention   set retention for object(s)
	legalhold   set legal hold for object(s)
	diff        list differences in object name, size, and date between two buckets
	rm          remove objects
	versioning  manage bucket versioning
	lock        manage default bucket object lock configuration
	ilm         manage bucket lifecycle
	encrypt     manage bucket encryption config
	event       manage object notifications
	watch       listen for object notification events
	policy      manage anonymous access to buckets and objects
	tag         manage tags for bucket(s) and object(s)
	admin       manage MinIO servers
	update      update mc to latest release

## 添加minio server

	$ set +o history		// 关闭history记录
	$ mc config host add <MinIOName> http://127.0.0.1:30007 <username> <password> --api "s3v4"
	// 部署tls后需要将http换成https
	$ mc config host add hce-minio https://127.0.0.1:30007 <username> <password> --api "s3v4"
	$ set -o history		// 打开history记录
mc 工具config配置文件默认存放位置为: `/root/.mc/`

## **Note**
当MinIO server配置TLS时候，使用`mc`的命令中需要添加 `--insecure`, 如:

	$ ./mc ls myminio --insecure

## Command List
**查看，添加，移除minio host**

	// 查看 host
	$ mc config host list
	// 添加 host, server没有配置TLS
	$ mc config host add myminio http://127.0.0.1:30007 minio minio123 --api S3v4
	// 添加 host, server配置TLS, http变为https
	$ mc config host add myminio https://127.0.0.1:30007 minio minio123 --api S3v4
	// 移除 host
	$ mc config host remove myminio
**查看bucket, object**

	//查看所有bucket
	$ mc ls myminio --insecure //server没有配置TLS可以不加`--insecure`
	//查看bucket下的object
	$ mc ls myminio/my-bucket01/
**创建bucket**

	$ mc mb myminio/my-bucket03 --insecure
**添加,删除文件(object)**

	// 添加文件(objcet)
	$ mc cp minioinstance.yaml myminio/my-bucket03 --insecure
	// 删除文件
	$ mc rm myminio/my-bucket03/minioinstance.yaml --insecure

**查看bucket使用量**

	$ mc du myminio









