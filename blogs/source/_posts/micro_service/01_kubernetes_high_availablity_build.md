---
title: 01 Kubernetes build high availability
tags:
- kubernetes
categories:
- microService
- kubernetes
top: 1
---

## **准备机器**

| IP地址 | 主机名 | 角色 |
| :------: | :------: | :------: |
| 10.239.140.133 | master-node | master |
| 10.239.131.156 | laboratory | master |
| 10.239.141.123 | node-1 | master |
| 10.239.141.194 | node-2 | worker |

总共四台机器，三台做master, 一台做work node, 部署好后可以把master上污点去掉, 照样可以部署k8s资源.

![](finish_setup.PNG)

## **环境配置**
每台机器(master和node)都要配置

	// 设置k8s相关系统内核参数
	cat << EOF > /etc/sysctl.d/k8s.conf
	net.bridge.bridege-nf-call-iptables = 1
	net.bridge.bridege-nf-call-ip6tables = 1
	EOF
	sysctl -p
	echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
	echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables
	swapoff -a
	sed -i '/swap/d' /etc/fstab
	systemctl stop firewalld.service
	systemctl disable firewalld
	setenforce 0
	sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
	systemctl stop kubelet
	systemctl stop docker
	systemctl restart kubelet
	systemctl restart docker
	sysctl net.bridge.bridge-nf-call-iptables=1
	sysctl net.bridge.bridge-nf-call-ip6tables=1
	iptables -F
	// 有时候在公司开发机上部署不成功, 需要在~/.bashrc添加NO_PROXY
	// 尤其要加上虚拟出来的VIP-IP
	cat << EOF >> ~/.bashrc
	export NO_PROXY=Node01-IP,Node02-IP,VIP-IP,127.0.0.1,
	EOF
	source ~/.bashrc


## **kube-vip方式部署高可用k8s集群**
official website:  
https://github.com/kubernetes/kubeadm/blob/master/docs/ha-considerations.md#kube-vip  
https://github.com/plunder-app/kube-vip/blob/master/kubernetes-control-plane.md  

## **在三台master机器上添加kube-vip配置文件**
**master-node机器上**

	$ touch /etc/kube-vip/config.yaml
	localPeer:
	  id: master-node			// 机器hostname, 通过$ hostnamectl set-hostname <HostName>修改
	  address: 10.239.140.133	// 指定本地IP地址
	  port: 10000
	remotePeers:
	- id: laboratory
	  address: 10.239.131.156	// 另外一台充当master机器IP地址
	  port: 10000
	- id: node-1
	  address: 10.239.141.123	// 另外一台充当master机器IP地址
	  port: 10000
	# [...]
	vip: 10.239.140.51		// 手动写的IP, 但必须在集群子网内, 部署集群后 $ ip addr 查看p8p1网卡下多出了此IP
	gratuitousARP: true
	singleNode: false
	startAsLeader: true			// 设置作为三台master机器的leader
	interface: p8p1				// 用本机的网卡名字, $ ip addr可查看
	loadBalancers:
	- name: API Server Load Balancer
	  type: tcp
	  port: 6443				// configure the load balancer to sit on the standard API-Server port 6443
	  bindToVip: true
	  backends:
	  - port: 6443		// configure the backends to point to the API-servers that will be configured to run on port 6444
	    address: 10.239.140.133
	  - port: 6443
	    address: 10.239.131.156
	  - port: 6443
	    address: 10.239.141.123
	  # [...]
**laboratory机器上**

	$ touch /etc/kube-vip/config.yaml
	localPeer:
	  id: laboratory			// 改成本机的
	  address: 10.239.131.156
	  port: 10000
	remotePeers:
	- id: master-node
	  address: 10.239.140.133
	  port: 10000
	- id: node-1
	  address: 10.239.141.123
	  port: 10000
	# [...]
	vip: 10.239.140.51
	gratuitousARP: true
	singleNode: false
	startAsLeader: false		// 不要设置成为leader
	interface: eno1				// 改成本机的IP地址网卡名
	loadBalancers:
	- name: API Server Load Balancer
	  type: tcp
	  port: 6443
	  bindToVip: true
	  backends:
	  - port: 6443
	    address: 10.239.140.133
	  - port: 6443
	    address: 10.239.131.156
	  - port: 6443
	    address: 10.239.141.123
	  # [...]
**node-1机器上**

	$ touch /etc/kube-vip/config.yaml
	localPeer:
	  id: node-1				// 改成本机的
	  address: 10.239.141.123
	  port: 10000
	remotePeers:
	- id: master-node
	  address: 10.239.140.133
	  port: 10000
	- id: laboratory
	  address: 10.239.131.156
	  port: 10000
	# [...]
	vip: 10.239.140.51
	gratuitousARP: true
	singleNode: false
	startAsLeader: false		// 不要设置成为leader
	interface: enp0s3			// 改成本机的IP地址网卡名
	loadBalancers:
	- name: API Server Load Balancer
	  type: tcp
	  port: 6443
	  bindToVip: true
	  backends:
	  - port: 6443
	    address: 10.239.140.133
	  - port: 6443
	    address: 10.239.131.156
	  - port: 6443
	    address: 10.239.141.123
	  # [...]
> Use 6443 for both the VIP and the API-Servers, in order to do this we need to specify that the api-server is bound to it's local IP. To do this we use the --apiserver-advertise-address flag as part of the init, this means that we can then bind the same port to the VIP and we wont have a port conflict.

## **部署High Availability K8s集群**
### **1. master-node机器上**
现在master-node机器上配置好K8s集群, 然后再把其它两个master加进来就可以了.  

	docker run -it --rm plndr/kube-vip:0.1.1 /kube-vip sample manifest \
	    | sed "s|plndr/kube-vip:'|plndr/kube-vip:0.1.1'|" \
	    | sudo tee /etc/kubernetes/manifests/kube-vip.yaml
> Ensure that image: plndr/kube-vip:<x> is modified to point to a specific version (0.1.5 at the time of writing), refer to docker hub for details.  
> Also ensure that the hostPath points to the correct kube-vip configuration, if it isn’t the above path.  

	vim /etc/kubernetes/manifests/kube-vip.yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  creationTimestamp: null
	  name: kube-vip
	  namespace: kube-system
	spec:
	  containers:
	  - command:
	    - /kube-vip
	    - start
	    - -c
	    - /vip.yaml
	    image: 'plndr/kube-vip:0.1.1'
	    name: kube-vip
	    resources: {}
	    securityContext:
	      capabilities:
	        add:
	        - NET_ADMIN
	        - SYS_TIME
	    volumeMounts:
	    - mountPath: /vip.yaml
	      name: config
	  hostNetwork: true
	  volumes:
	  - hostPath:
	      path: /etc/kube-vip/config.yaml	// 跟上面的conf.yaml文件路径对应
	    name: config
	status: {}
执行部署K8s集群命令

	$ kubeadm init --control-plane-endpoint vip.mycluster.local:8443 [additional arguments ...] //具体实例如下
	$ kubeadm init --control-plane-endpoint "10.239.140.133:6443" --apiserver-advertise-address 10.239.140.133 --apiserver-bind-port 6443 --upload-certs --kubernetes-version "v1.19.0"
	$ kubectl get pods -A
	  NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
	  <...>
	  kube-system   kube-vip-controlplane01                  1/1     Running   0          10m
这个 --upload-certs 标志用来将在所有控制平面实例之间的共享证书上传到集群.  
当 --upload-certs 与 kubeadm init 一起使用时，主控制平面的证书被加密并上传到 kubeadm-certs 密钥中.  

### **2. laboratory和node-1机器上**
> 先不要在路径/etc/kubernetes/manifests/添加 kube-vip.yaml 文件, this is due to some bizarre kubeadm/kubelet behaviour.  
> 等laboratory和node-1机器都添加进master集群后再添加kube-vip.yaml, kubeadm会自动检测/etc/kubernetes/manifests/文件变化并部署pod.  
直接运行如下命令添加进master集群.

	kubeadm join 192.168.0.75:6443 --token <tkn> \
	    --discovery-token-ca-cert-hash sha256:<hash> \
	    --control-plane --certificate-key <key> 
**配置k8s访问环境变量**  
这样就能在laboratory和node-1机器上执行kubectl命令了.  
第一种:

	$ mkdir -p $HOME/.kube
	$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

第二种:

	$ export KUBECONFIG=/etc/kubernetes/admin.conf
在master-node机器上查看laboratory和node-1机器已经加入master控制层面后, 再在laboratory和node-1机器上添加/etc/kubernetes/manifests/kube-vip.yaml文件.  
**修改api-server访问地址为本机**
这样某一台master机器挂了其它机器照样可以正常访问api-server.

	// laboratory机器上
	vim /etc/kubernetes/admin.conf
	  server: https://10.239.131.156:6443
	// node-1机器上
	vim /etc/kubernetes/admin.conf
	  server: https://10.239.141.123:6443

### **3. master-node机器上**
在master-node机器上运行查看pod运行情况.  

	$ kubectl get pods -A | grep vip
	kube-system   kube-vip-controlplane01                  1/1     Running             1          16m
	kube-system   kube-vip-controlplane02                  1/1     Running             0          18m
	kube-system   kube-vip-controlplane03                  1/1     Running             0          20m
查看 pod/kube-vip-master-node 运行日志

	$ kubectl logs po/kube-vip-master-node -n kube-system
	  time=“2020-08-28T15:33:09Z” level=info msg=“The Node [10.239.140.133:10000] is leading”
	  time=“2020-08-28T15:33:09Z” level=info msg=“The Node [10.239.140.133:10000] is leading”
**部署CNI网络**

	$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

### **4. node-2机器上**
添加work node节点

	$ sudo kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
## **查看kube-vip, api-server服务进程和监听端口**

	$ netstat -nltp | grep 10000	// 列出监听端口10000的进程
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	tcp        0      0 10.239.140.133:10000    0.0.0.0:*               LISTEN      21353/kube-vip
	
	$ netstat -nltp | grep 6443		// 列出监听端口6443的进程
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	tcp6       0      0 :::6443                 :::*                    LISTEN      15700/kube-apiserve

	$ netstat -antp		// 列出所有tcp进程, State不仅包括Listen的, 还包括已建立链接状态为Established的进程.
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	tcp6       0      0 :::6443                 :::*                    LISTEN      15700/kube-apiserve
	tcp        0      0 10.239.140.133:10000    0.0.0.0:*               LISTEN      21353/kube-vip
	tcp        0      0 10.239.140.133:10000    10.239.141.145:48554    ESTABLISHED 21353/kube-vip
`Local Address`可以看作是服务端IP和提供服务的监听端口, `Foreign Address`可以看作是客户端IP和发起链接请求的IP地址和请求端口.  
`ESTABLISHED`表示客户端与服务端已经建立tcp长链接.  
`LISTEN`表示服务端提供服务的端口仍处于监听状态, 等待客户端发起请求.  
TCP才能在Foreign Address看到链接的客户端IP和端口, 而UDP无状态是没有的.  
由以上输出可看到:
 * kube-vip服务进程编号为21353, 监听端口为10000, 所在本机IP为10.239.140.133
 * api-server服务进程编号为15700, 监听端口为6443

查看所有链接本机6443服务端口的客户端IP地址, 地址一致的合并, 然后连接数从高到底排序.

	$ netstat -antp | grep :6443 | awk '{print $5}' | awk -F ":" '{print $1}' | sort | uniq -c | sort -r -n
	      4 10.239.4.100	// 表示从IP地址为10.239.4.100的客户端请求访问本机6443服务端口的进程数为4
	      3 10.239.4.80
	      3 10.239.141.194
	      3 10.239.141.145
	      3
	      2 10.40.0.6
	      2 10.239.140.53
	      2 10.239.140.133
	      2 10.109.19.69
	      1 10.40.0.9
	      1 10.40.0.2
	      1 10.40.0.1
	      1 10.109.19.68

## 查看并去掉node污点(taint)

	// 查看node机器污点
	$ kubectl describe node/<Node-Name> | grep Taint
	  Taints:             node-role.kubernetes.io/master:NoSchedule
	
	// 去掉污点
	$  kubectl taint nodes <Node-Name> node-role.kubernetes.io/master:NoSchedule-


## 网卡上添加删除虚拟网址
网卡上增加一个IP

	ifconfig eth0:1 192.168.0.1 netmask 255.255.255.0
删除网卡的第二个IP地址

	ip addr del 192.168.0.1 dev eth0
