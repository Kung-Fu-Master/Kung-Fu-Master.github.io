---
title: k8s部署遇到的问题
tags:
- kubernetes
categories:
- microService
- kubernetes
---

## (仍需确认)问题01 failed to get cgroup stats for "/system.slice/kubelet.service"

通过`kubectl get po -n kube-system` 查看api-server启动异常一直重启
通过`systemctl status kubelet` 查看kubelet服务出现如下类似异常

**failed to get cgroup stats for "/system.slice/kubelet.service"**
**解决方法:** https://github.com/kubernetes/kubernetes/issues/56850
参考最新的：

	# ubuntu
	cat << EOF | sudo tee /etc/systemd/system/kubelet.service.d/12-after-docker.conf
	[Unit]
	After=docker.service
	EOF
	
	# centos
	cat << EOF | sudo tee /usr/lib/systemd/system/kubelet.service.d/12-after-docker.conf
	[Unit]
	After=docker.service
	EOF

	$ systemctl daemon-reload && systemctl restart kubelet // restart kubelet, after adding this file
实际重启kubelet后仍然会报同样的异常, 没有管, 当时也没观察api-server是否正常启动, 过一段时间后发现api-server正常启动.




