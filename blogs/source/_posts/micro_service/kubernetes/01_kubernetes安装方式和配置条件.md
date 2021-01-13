---
title: 01 Kubernetes安装方式和配置条件
tags:
- kubernetes
categories:
- microService
- kubernetes
---

## 安装方法
reference: https://kubernetes.io/docs/setup/production-environment/tools/

<!-- more -->

 * [Bootstrapping clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)
 * [Installing Kubernetes with kops](https://kubernetes.io/docs/setup/production-environment/tools/kops/)
 * [Installing Kubernetes with Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)

## Before you begin
 * One or more machines running one of:
  * Ubuntu 16.04+
  * Debian 9+
  * CentOS 7
  * Red Hat Enterprise Linux (RHEL) 7
  * Fedora 25+
  * HypriotOS v1.0.1+
  * Flatcar Container Linux (tested with 2512.3.0)
 * 2 GB or more of RAM per machine (any less will leave little room for your apps).
 * 2 CPUs or more.
 * Full network connectivity between all machines in the cluster (public or private network is fine).
 * Unique hostname, MAC address, and product_uuid for every node. See here for more details.
 * Certain ports are open on your machines. See here for more details.
 * Swap disabled. You MUST disable swap in order for the kubelet to work properly.

## Verify the MAC address and product_uuid are unique for every node 

 * 您可以使用命令 **`ip link`** 或 **`ifconfig -a`** 来获取网络接口的 MAC 地址
 * 可以使用 **`sudo cat /sys/class/dmi/id/product_uuid`** 命令对 product_uuid 校验
一般来讲，硬件设备会拥有唯一的地址，但是有些虚拟机的地址可能会重复。Kubernetes 使用这些值来唯一确定集群中的节点。 如果这些值在每个节点上不唯一，可能会导致安装[失败](https://github.com/kubernetes/kubeadm/issues/31).

## 检查网络适配器

如果您有一个以上的网络适配器，同时您的 Kubernetes 组件通过默认路由不可达，我们建议您预先添加 IP 路由规则，这样 Kubernetes 集群就可以通过对应的适配器完成连接。

## 确保 iptables 工具不使用 nftables 后端

在 Linux 中，nftables 当前可以作为内核 iptables 子系统的替代品。 **iptables** 工具可以充当兼容性层，其行为类似于 iptables 但实际上是在配置 nftables。 nftables 后端与当前的 kubeadm 软件包不兼容：它会导致重复防火墙规则并破坏 **kube-proxy**。

如果您系统的 **iptables** 工具使用 nftables 后端，则需要把 **iptables** 工具切换到“旧版”模式来避免这些问题。 默认情况下，至少在 Debian 10 (Buster)、Ubuntu 19.04、Fedora 29 和较新的发行版本中会出现这种问题。RHEL 8 不支持切换到旧版本模式，因此与当前的 kubeadm 软件包不兼容


### iptables_legacy

#### "Debian 或 Ubuntu"
```bash
update-alternatives --set iptables /usr/sbin/iptables-legacy
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
update-alternatives --set arptables /usr/sbin/arptables-legacy
update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

#### "Fedora"

```bash
update-alternatives --set iptables /usr/sbin/iptables-legacy
```

Make sure that the **br_netfilter** module is loaded. This can be done by running **`lsmod | grep br_netfilter`**. To load it explicitly call **`sudo modprobe br_netfilter`**.

As a requirement for your Linux Node's iptables to correctly see bridged traffic, you should ensure **`net.bridge.bridge-nf-call-iptables`** is set to 1 in your **`sysctl`** config, e.g.

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
For more details please see the [Network Plugin Requirements page](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#network-plugin-requirements).

## 检查所需端口
### 控制平面节点(s)
| Protocol | Direction | Port Range | Purpose | Used By |
| :------ | :----- | :------ | :------ | :------ |
| TCP | Inbound | 6443* | Kubernetes API server | All |
| TCP | Inbound | 2379-2380 | etcd server client API | kube-apiserver, etcd |
| TCP | Inbound | 10250 | kubelet API | Self, Control plane |
| TCP | Inbound | 10251 | kube-scheduler | Self |
| TCP | Inbound | 10252 | kube-controller-manager | Self |

### 工作节点(s)

| Protocol | Direction | Port Range | Purpose | Used By |
| :------ | :----- | :------ | :------ | :------ |
| TCP | Inbound | 10250 | kubelet API | Self, Control plane |
| TCP | Inbound | 30000-32767 | NodePort Services† | All |

† D Default port range for [NodePort Services](https://kubernetes.io/docs/concepts/services-networking/service/).

Any port numbers marked with * are overridable, so you will need to ensure any custom ports you provide are also open

Although etcd ports are included in control-plane nodes, you can also host your own etcd cluster externally or on custom ports.

The pod network plugin you use (see below) may also require certain ports to be open. Since this differs with each pod network plugin, please see the documentation for the plugins about what port(s) those need.

## 安装 runtime

To run containers in Pods, Kubernetes uses a [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes).
从 v1.6.0 版本起，Kubernetes 开始默认允许使用 CRI（容器运行时接口）。

从 v1.14.0 版本起，kubeadm 将通过观察已知的 UNIX 域套接字来自动检测 Linux 节点上的容器运行时。 下表中是可检测到的正在运行的 runtime 和 socket 路径。

| 运行时 | 域套接字 |
| :------ | :----- |
| Docker | <div style="width: 200pt">/var/run/docker.sock</div> |
| containerd | /run/containerd/containerd.sock |
| CRI-O | /var/run/crio/crio.sock |

如果同时检测到 docker 和 containerd，则优先选择 docker。 这是必然的，因为 docker 18.09 附带了 containerd 并且两者都是可以检测到的。 如果检测到其他两个或多个运行时，kubeadm 将以一个合理的错误信息退出。

在非 Linux 节点上，默认使用 docker 作为容器 runtime。

如果选择的容器 runtime 是 docker，则通过内置 dockershim CRI 在 kubelet 的内部实现其的应用。

基于 CRI 的其他 runtimes 有：

- [containerd](https://github.com/containerd/cri) （containerd 的内置 CRI 插件）
- [cri-o](https://cri-o.io/)
- [frakti](https://github.com/kubernetes/frakti)

请参考 [CRI 安装指南](/zh/docs/setup/production-environment/container-runtimes/)获取更多信息。

## 安装 kubeadm、kubelet 和 kubectl

您需要在每台机器上安装以下的软件包：

* `kubeadm`：用来初始化集群的指令。

* `kubelet`：在集群中的每个节点上用来启动 pod 和容器等。

* `kubectl`：用来与集群通信的命令行工具。

kubeadm **不能** 帮您安装或者管理 `kubelet` 或 `kubectl`，所以您需要确保它们与通过 kubeadm 安装的控制平面的版本相匹配。
如果不这样做，则存在发生版本偏差的风险，可能会导致一些预料之外的错误和问题。
然而，控制平面与 kubelet 间的相差一个次要版本不一致是支持的，但 kubelet 的版本不可以超过 API 服务器的版本。
例如，1.7.0 版本的 kubelet 可以完全兼容 1.8.0 版本的 API 服务器，反之则不可以。

有关安装 `kubectl` 的信息，请参阅[安装和设置 kubectl](/zh/docs/tasks/tools/install-kubectl/)文档。

## **Warning**
> 这些指南不包括系统升级时使用的所有 Kubernetes 程序包。这是因为 kubeadm 和 Kubernetes
> 有[特殊的升级注意事项](/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)。


关于版本偏差的更多信息，请参阅以下文档：

* Kubernetes [版本与版本间的偏差策略](/zh/docs/setup/release/version-skew-policy/)
* Kubeadm-specific [版本偏差策略](/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

## k8s_install
### "Ubuntu、Debian 或 HypriotOS"

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### "CentOS、RHEL 或 Fedora"

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
```

  **请注意：**

  - 通过运行命令 `setenforce 0` 和 `sed ...` 将 SELinux 设置为 permissive 模式可以有效的将其禁用。
    这是允许容器访问主机文件系统所必须的，例如正常使用 pod 网络。
    您必须这么做，直到 kubelet 做出升级支持 SELinux 为止。
  - 一些 RHEL/CentOS 7 的用户曾经遇到过问题：由于 iptables 被绕过而导致流量无法正确路由的问题。您应该确保
    在 `sysctl` 配置中的 `net.bridge.bridge-nf-call-iptables` 被设置为 1。

    ```bash
    cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sysctl --system
    ```
  - 确保在此步骤之前已加载了 `br_netfilter` 模块。这可以通过运行 `lsmod | grep br_netfilter` 来完成。要显示加载它，请调用 `modprobe br_netfilter`。

kubelet 现在每隔几秒就会重启，因为它陷入了一个等待 kubeadm 指令的死循环。


## 在控制平面节点上配置 kubelet 使用的 cgroup 驱动程序

使用 docker 时，kubeadm 会自动为其检测 cgroup 驱动并在运行时对 `/var/lib/kubelet/kubeadm-flags.env` 文件进行配置。

如果您使用不同的 CRI，您需要使用 `cgroup-driver` 值修改 `/etc/default/kubelet` 文件（对于 CentOS、RHEL、Fedora，修改 `/etc/sysconfig/kubelet` 文件），像这样：

```bash
KUBELET_EXTRA_ARGS=--cgroup-driver=<value>
```

这个文件将由 `kubeadm init` 和 `kubeadm join` 使用以获取额外的用户自定义的 kubelet 参数。

请注意，您 **只** 需要在您的 cgroup 驱动程序不是 `cgroupfs` 时这么做，因为它已经是 kubelet 中的默认值。

需要重新启动 kubelet：

```bash
systemctl daemon-reload
systemctl restart kubelet
```

自动检测其他容器运行时的 cgroup 驱动，例如在进程中工作的 CRI-O 和 containerd。

## 故障排查

如果您在使用 kubeadm 时遇到困难，请参阅我们的[故障排查文档](/zh/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/)。



