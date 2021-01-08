---
title: greenplum(psql) 远程连接k8s部署的greenplum
tags: storage
categories:
- storage
- greenplum
---


## 第一种方式, 本地主机连接Kubernetes上部署的greenplum

查看greenplum在kubernetes上部署的service

```shell
	$ kubectl get svc -n greenplum
	NAME        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
	agent       ClusterIP      None            <none>        22/TCP           6m1s
	greenplum   LoadBalancer   10.99.154.225   <pending>     5432:32518/TCP   6m1s
```

宿主主机安装psql客户端和服务端
Reference Link: https://www.cnblogs.com/zhi-leaf/p/11432054.html

```shell
	$ yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
	$ yum install postgresql10
	$ yum install postgresql10-server
	$ /usr/pgsql-10/bin/postgresql-10-setup initdb
	(option) $ systemctl start postgresql-10
```

宿主主机安装完psql后会在如下路径生成配置文件

```
	/var/lib/pgsql/10/data/
```

**遇到的问题**

```shell
	$ psql -Ugpadmin -p32518 -h10.67.108.211
	psql: FATAL:  no pg_hba.conf entry for host "10.32.0.1", user "gpadmin", database "gpadmin", SSL off
```

**解决方法**

```shell
	$ kubectl exec po/master-0 -n greenplum -it -- /bin/bash
	$ vim /greenplum/data-1/pg_hba.conf
```
添加如下内容:
```
	host     all         gpadmin         10.46.0.0/32    trust
```
在上面登陆的greenplum的容器里运行如下命令Reload pg_hba.conf and postgresql.conf

```shell
	$ gpstop -u
```

## 第二种方式, 编写dockerfile, 程序在docker里连接greenplum

1. psql_lib.py内容如下

``` py
	import psycopg2
	import sqlparse
	
	class Greenplum_psql(object):
	    def __init__(self, database, host, user, port):
	        self.__database = database
	        self.__host = host
	        self.__user = user
	        self.__port = port
	
	    def connect(self):
	        conn = psycopg2.connect(
	            dbname=self.__database, host=self.__host, user=self.__user, port=self.__port)
	        cursor = conn.cursor()
	        return conn, cursor
	
	    def __parse_psql(self, sql_cmd):
	        parsed_cmd = sqlparse.split(sql_cmd)
	        return parsed_cmd
	
	    def process_psql(self, conn, cursor, cmd):
	        parsed_cmd = self.__parse_psql(cmd)
	        for cmd in parsed_cmd:
	            cursor.execute(cmd)
	        conn.commit()
	
	    def close(self, conn, cursor):
	        cursor.close()
	        conn.close()
```

2. main.py内容如下

```py
	import psql_lib
	
	if __name__ == "__main__":
	    dbname = "template1"
	    host = "10.99.154.225"
	    user = "gpadmin"
	    port = "5432"
	    cmd = "DROP TABLE test_conn1;\
	CREATE TABLE test_conn1(id int, mediaURI text, date text);\
	INSERT INTO test_conn1 values(111, 'URI-text', '2020-11-06, 10:05');\
	select * from test_conn1;"
	
	    psql = psql_lib.Greenplum_psql(dbname, host, user, port)
	    conn, cursor = psql.connect()
	    psql.process_psql(conn, cursor, cmd)
	
	    rows = cursor.fetchall()
	    for row in rows:
	        print('id = ', row[0], 'name = ', row[1], 'date = ', row[2])
	
	    psql.close(conn, cursor)
```
3. Dockerfile 内容如下

```
	$ vim Dockerfile
	FROM python:3.6.8
	LABEL storage="greenplum-sdk"
	WORKDIR /home/
	COPY psql_lib.py main.py ./
	ARG proxy
	RUN pip install psycopg2==2.8.3 sqlparse==0.4.1 --proxy ${proxy}
	CMD ["python", "main.py"]
```
**Build the image**

```shell
	$ docker build --build-arg proxy=http://child-prc.intel.com:913 -t greenplum-client:0.1 .
```
**Run the container**

```shell
	$ docker run --name python-greenplum greenplum-client:0.1
```






