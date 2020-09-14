---
title: ssh端口映射
tags: SSH-port
categories:
- linux
---

# 简介
可以将远端服务器一个端口remote_port绑定到本地端口port，其中-C是进行数据压缩，-f是后台操作，只有当提示用户名密码的时候才转向前台。-N是不执行远端命令，在只是端口转发时这条命令很有用处。-g 是允许远端主机连接本地转发端口。-R表明是将远端主机端口映射到本地端口。如果是-L，则是将本地端口映射到远端主机端口。
ssh的三个强大的端口转发命令：
转发到远端：ssh -C -f -N -g -L 本地端口:目标IP:目标端口 用户名@目标IP
转发到本地：ssh -C -f -N -g –R 本地端口:目标IP:目标端口 用户名@目标IP

	ssh -C -f -N -g -D listen_port user@Tunnel_Host
	-C: 压缩数据传输。
	-f: 后台认证用户/密码，通常和-N连用，不用登录到远程主机。
	-N: 不执行脚本或命令，通常与-f连用。
	-g: 在-L/-R/-D参数中，允许远程主机连接到建立的转发的端口，如果不加这个参数，只允许本地主机建立连接。
	-L: localport:remotehost:remotehostport sshserver(本地机器端口:目标机器IP:目标机器端口 中转机器IP)

## **本地端口转发**
将访问本机的80端口访问转发到192.168.1.1的8080端口
ssh -C -f -N -g -L 80:192.168.1.1:8080  user@192.168.1.1
如下图，假如host3和host1、host2都同互相通信，但是host1和host2之间不能通信，如何从host1连接上host2？

对于实现ssh连接来说，实现方式很简单，从host1 ssh到host3，再ssh到host2，也就是将host3作为跳板的方式。但是如果不是ssh，而是http的80端口呢？如何让host1能访问host2的80端口？
![](local_port_transmit.png)
ssh支持本地端口转发，语法格式为：

	 ssh -L [local_bind_addr:]local_port:remote:remote_port middle_host
以上图为例，实现方式是在host1上执行:

	[root@xuexi ~]# ssh -g -L 2222:host2:80 host3
其中**"-L"**选项表示本地端口转发，其工作方式为：在本地指定一个由ssh监听的转发端口(2222)，将远程主机的端口(host2:80)映射为本地端口(2222)，当有主机连接本地映射端口(2222)时，本地ssh就将此端口的数据包转发给中间主机(host3)，然后host3再与远程主机的端口(host2:80)通信。
现在就可以通过访问host1的2222端口来达到访问host2:80的目的了。例如：
![](local_port_transmit1.png)
再来解释下"-g"选项，指定该选项表示允许外界主机连接本地转发端口(2222)，如果不指定"-g"，则host4将无法通过访问host1:2222达到访问host2:80的目的。甚至，host1自身也不能使用172.16.10.5:2222，**而只能使用localhost:2222或127.0.0.1:2222这样的方式达到访问host2:80的目的，**之所以如此，是因为本地转发端口默认绑定在回环地址上。可以使用bind_addr来改变转发端口的绑定地址，例如：  

	[root@xuexi ~]# ssh -L 172.16.10.5:2222:host2:80 host3
这样，**host1自身就能通过访问172.16.10.5:2222的方式达到访问host2:80的目的。**  
一般来说，使用转发端口，都建议同时使用"-g"选项，否则将只有自身能访问转发端口.  

### **再来分析下转发端口通信的过程**
![](port_transmit_protocol.png)
当host4发起172.16.10.5:2222的连接时(即步骤①)，数据包的目标地址和端口为"172.16.10.5:2222"。由于host1上ssh已经监听了2222端口，并且知道该端口映射自哪台主机哪个端口，所以将会把该数据包目标地址和端口替换为"172.16.10.3:80"，并将此数据包通过转发给host3。当host3收到该数据包时，发现是host1转发过来请求访问host2:80的数据包，所以host3将代为访问host2的80端口。  

所以，**host1和host3之间的通信方式是SSH协议**，这段连接是安全加密的，因此称为"安全隧道"，**而host3和host2之间通信协议则是HTTP而不是ssh。**

### **现在再来考虑下，通过本地端口转发的方式如何实现ssh跳板的功能呢？仍以上图为例**

	[root@xuexi ~]# ssh -g -L 22333:host2:22 host3]
这样只需使用ssh连上host1的22333端口就等于连接了host2的22端口。

### **最后，关于端口转发有一个需要注意的问题：ssh命令中带有要执行的命令。考虑了下面的三条在host1上执行的命令的区别。**

	[root@xuexi ~]# ssh -g -L 22333:host2:22 host3
	[root@xuexi ~]# ssh -g -L 22333:host2:22 host3 "ifconfig"
	[root@xuexi ~]# ssh -g -L 22333:host2:22 host3 "sleep 10"
第一条命令开启了本地端口转发，且是以登录到host3的方式开启的，所以执行完该命令后，将跳到host3主机上，当退出host3时，端口转发功能将被关闭。另外，host1上之所以要开启端口转发，目的是为了与host2进行通信，而不是跳到host3上，所以应该在ssh命令行上加上"-f"选项让ssh在本机host1上以后台方式提供端口转发功能，而不是跳到host3上来提供端口转发功能。  

第二条命令在开启本地转发的时候还指定了要在host3上执行"ifconfig"命令，但是ssh的工作机制是远程命令执行完毕的那一刻，ssh关闭连接，所以此命令开启的本地端口转发功能有效期只有执行ifconfig命令的一瞬间。  

第三条命令和第二条命令类似，只不过指定的是睡眠10秒命令，所以此命令开启的本地转发功能有效期只有10秒。  

结合上面的分析，开启端口转发功能时，建议让ssh以后台方式提供端口转发功能，且明确指示不要执行任何ssh命令行上的远程命令。即最佳开启方式为：  

	[root@xuexi ~]# ssh -f -N -g -L 22333:host2:22 host3


## **远程端口转发**
将访问192.168.1.1的8080访问转发到本机的80端口
ssh -C -f -N -g -R 80:192.168.1.1:8080 user@192.168.1.1
请求开启的转发端口是在远程主机上的，所以称为"远程端口转发"


## K8s:
查看Server机上k8s部署的Pod

	$ kubectl get po -n rook-ceph
	NAME                               READY   STATUS      RESTARTS   AGE     IP              NODE         NOMINATED NODE   READINESS GATES
	rook-ceph-mgr-a-778576cbbc-mxvcp   1/1     Running     0          4h24m   10.36.0.11      hci-node04   <none>           <none>            app=rook-ceph-mgr,ceph_daemon_id=a,instance=a,mgr=a,pod-template-hash=778576cbbc,rook_cluster=rook-ceph

	$ kubectl get svc -n rook-ceph
	NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE     LABELS
	rook-ceph-mgr-dashboard    ClusterIP   10.108.115.137   <none>        8443/TCP            4h25m   app=rook-ceph-mgr,rook_cluster=rook-ceph
将server机器的上部署的pod端口或svc的ClusterIP改为NodePort后的端口映射到自己的开发机上，并用浏览器访问两种方法:
第一种: 也可以修改svc的ClusterIP类型为NodePort，这样就不用再运行下面的Pod端口映射到Node上了

	$ kubectl edit svc -n rook-ceph rook-ceph-mgr-dashboard
	spec:
      clusterIP: 10.108.115.137
	  ......
	  type: NodePort	// 把ClusterIP改为NodePort
	$ kubectl get svc -n rook-ceph
	NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
	rook-ceph-mgr-dashboard    NodePort    10.101.80.177    <none>        8443:31180/TCP      46m
在server机上用curl访问看看

	$ curl -v https://127.0.0.1:31180 --noproxy "*" -k	//不通过机器配置的proxy访问
可以看到映射到server的31180端口, 因此端口映射改为31180而不是8443，因为8443是svc的端口

	$ ssh -L MyLabIP:31180:10.67.108.211:31180 root@10.67.108.211

第二种: Server机上将Pod端口映射到Node上

	$ kubectl port-forward rook-ceph-mgr-a-778576cbbc-mxvcp -n rook-ceph 8443:8443
Server机上访问

	$ curl -v https://127.0.0.1:8443 --noproxy "*" -k	// 访问不经过设置的公司Proxy, 直接用局域网访问

在开发机上设置端口转发

	在自己开发机上运行下面command，将访问自己开发机的8443端口流量转发到server机器10.67.108.211的8443端口
	浏览器或curl访问路径为 https://10.67.108.211:8443
	$ ssh -L 8443:10.67.108.211:8443 root@10.67.108.211
	
	(管用!!!自己开发机上运行)运行下面command后浏览器或curl访问路径为 https://127.0.0.1:8443
	$ ssh -L 8443:127.0.0.1:8443 root@10.67.108.211		// 将访问本机8443(第一个)端口的路由转发到10.67.108.211机器8443端口
	$ ssh -L MyLabIP:8443:127.0.0.1:8443 root@10.67.108.211	//如果不在本机运行上面命令需要加上要转发8443端口的主机IP
	浏览器输入https://127.0.0.1:31440

## istio dashboard端口转发
### 如在master机器A上运行istio dashboard

	istioctl dashboard kiali
	http://localhost:38610/kiali
查看kiali服务监听端口

	netstat -nltp | grep 38610
	tcp        0      0 127.0.0.1:38610         0.0.0.0:*               LISTEN      303611/istioctl
	tcp6       0      0 ::1:38610               :::*                    LISTEN      303611/istioctl

### 在可打开浏览器的机器B上运行如下端口转发命令

	ssh -fL 38610:127.0.0.1:38610 root@<master-IP>
	root@<master-IP>'s password:	//输入密码登陆即可
然后再打开机器B终端, 运行终端打开浏览器命令如下, 或直接在浏览器输入http://127.0.0.1:38610

	firefox





