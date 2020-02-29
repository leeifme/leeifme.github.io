---
title: 暴力学习 k8s - 集群搭建
date: 2018-09-01 14:15:15
categories: 
tags:
- Kubernetes
- 容器学习
---

## 前言

> 其实，搭建一个 Kubernetes（K8S）集群不是一件容易的事情，主要困难有两个：
>
> - 那一道厚厚的墙
> - 对 K8S 的知识不熟悉
>
> 只要能解决上面两个问题，搭建的过程实际上就没有那么复杂了。
>
> 本系列是我在搭建过程中踩的无数坑 、以及查阅众多相关问题解决的文章的一些记录和总结。

## 集群规划

### 网络配置

- 节点网络： 192.168.31.0/24
- service 网络： 10.96.0.0/12
- pod 网络： 10.244.0.0/16

### kubeadm 部署

#### 基本情况

- kubeadm 项目[链接地址](https://github.com/kubernetes/kubeadm)
- master、node： 安装 `kubelet`、 `kubeadm`、 `docker`
- master： `kubeadm init`
- node： `kubeadm join`
- `apiserver`、`scheduler`、`Controller-manager`、`etcd` 在 master 上以 Pod 运行
- `kubeproxy` 以 Pod 方式运行在每一个 node 节点上。
- 以上 pod 均为静态 Pod
- 每一个节点都需要运行 `flannel`（也是以 Pod 方式运行），以提供 Pod 网络
- kubeadm 的[介绍](https://github.com/kubernetes/kubeadm/blob/master/docs/design/design_v1.10.md)

#### 安装步骤

1. master，node 需要安装 kubelet， kubeadm， docker
2. master 节点上运行 `kubeadm init`
3. node 节点上运行 `kubeadm join` 加入集群

## 准备环境

### 我的环境

```sh
[root@master yum.repos.d]# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 
[root@master yum.repos.d]# uname -a
Linux master 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

### 节点解析

通过 `/etc/hosts` 文件解析

```sh
192.168.31.81 master.test.com master
192.168.31.82 node01.test.com node01
192.168.31.83 node02.test.com node02
```

**集群通过时间服务器做时钟同步**，我没做

### 节点互信

可以按照此[文档](https://www.cnblogs.com/jyzhao/p/3781072.html)配置节点互信，也没有做

### 选择版本

使用 kubernetes v1.11.2

## 开始部署

**确保 iptables firewalld 等未启动, 且不开机自启动**，可参考我之前的 CentOS 安装

### 配置 yum 仓库

使用 aliyun 源，[链接](https://opsx.alibaba.com/mirror)

#### docker 源使用如下命令获取

```sh
cd /etc/yum.repos.d
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### kubernetes 源

```sh
[root@master yum.repos.d]# cat kubernetes.repo 
[kubernetes]
name=Kubernetes Repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
enabled=1
```

#### 查看源是否生效

```
# yum clean all
# yum repolist
*****
*****
Determining fastest mirrors
kubernetes                                                         243/243
repo id                         repo name                            status
base/7/x86_64                   CentOS-7 - Base - 163.com            9,911
docker-ce-stable/x86_64         Docker CE Stable - x86_64               16
extras/7/x86_64                 CentOS-7 - Extras - 163.com            370
kubernetes                      Kubernetes Repo                        243
updates/7/x86_64                CentOS-7 - Updates - 163.com         1,054
repolist: 11,594
```

### 安装软件

**三台机器都需要安装**

使用 `yum install docker-ce kubelet kubeadm kubectl` 安装

安装的软件包如下：

```
  Installing : kubectl-1.11.2-0.x86_64                                 1/7 
  Installing : cri-tools-1.11.0-0.x86_64                               2/7 
  Installing : socat-1.7.3.2-2.el7.x86_64                              3/7 
  Installing : kubernetes-cni-0.6.0-0.x86_64                           4/7 
  Installing : kubelet-1.11.2-0.x86_64                                 5/7 
  Installing : kubeadm-1.11.2-0.x86_64                                 6/7 
  Installing : docker-ce-18.06.0.ce-3.el7.x86_64                       7/7 
```

### 启动 docker 服务等

由于国内网络原因，kubernetes 的镜像托管在 google 云上，无法直接下载，需要设置 proxy
在 `/usr/lib/systemd/system/docker.service` 文件中添加如下两行

```shell
[root@master ~]# cat /usr/lib/systemd/system/docker.service |grep PROXY
Environment="HTTPS_PROXY=http://www.ik8s.io:10080"
Environment="NO_PROXY=127.0.0.0/8,192.168.31.0/24"
```

之后，启动 docker

```shell
systemctl daemon-reload
systemctl start docker
systemctl enable docker
```

确认 proc 的这两个参数如下，均为 1

```shell
[root@master ~]# cat /proc/sys/net/bridge/bridge-nf-call-iptables 
1
[root@master ~]# cat /proc/sys/net/bridge/bridge-nf-call-ip6tables 
1

解决方法：修改系统文件是的机器 bridge 模式开启

设置机器开机启动的时候执行下面两条命令

编辑 vim  /etc/rc.d/rc.local 添加下面两条命令

        echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables

        echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables

centos7 需要增加执行权限：

        chmod +x　/etc/rc,d/rc.local

重启系统
```

### 设置 kubelet

查看 kubelet 安装生成了哪些文件

```shell
[root@master ~]# rpm -ql kubelet
/etc/kubernetes/manifests              # 清单目录
/etc/sysconfig/kubelet                 # 配置文件
/etc/systemd/system/kubelet.service    # unit file
/usr/bin/kubelet                       # 主程序
```

**默认**的配置文件

```shell
[root@master ~]# cat /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=
```

修改 kubelet 的配置文件

```shell
[root@master ~]# cat /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--fail-swap-on=false"
```

此时还无法正常启动 kubelet，先设置 kubelet 开机自启动，使用如下命令： `systemctl enable kubelet` 。

### kubeadm init

在 master 节点上执行

```shell
kubeadm init --kubernetes-version=v1.11.2 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap
```

kubeadm init 的输出可见于此[链接](https://segmentfault.com/n/1330000016057102)

此命令，下载了如下 image

```shell
[root@master ~]# docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy-amd64                v1.11.2             46a3cd725628        7 days ago          97.8MB
k8s.gcr.io/kube-apiserver-amd64            v1.11.2             821507941e9c        7 days ago          187MB
k8s.gcr.io/kube-controller-manager-amd64   v1.11.2             38521457c799        7 days ago          155MB
k8s.gcr.io/kube-scheduler-amd64            v1.11.2             37a1403e6c1a        7 days ago          56.8MB
k8s.gcr.io/coredns                         1.1.3               b3b94275d97c        2 months ago        45.6MB
k8s.gcr.io/etcd-amd64                      3.2.18              b8df3b177be2        4 months ago        219MB
k8s.gcr.io/pause                           3.1                 da86e6ba6ca1        7 months ago        742kB
```

现在，正在运行的 docker 如下

```sh
[root@master ~]# docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
1c03e043e6b7        46a3cd725628           "/usr/local/bin/kube??   3 minutes ago       Up 3 minutes                            k8s_kube-proxy_kube-proxy-6fgjm_kube-system_f85e8660-a090-11e8-8ee7-000c29f71e04_0
5f166bd11566        k8s.gcr.io/pause:3.1   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_kube-proxy-6fgjm_kube-system_f85e8660-a090-11e8-8ee7-000c29f71e04_0
0f306f98cc52        b8df3b177be2           "etcd --advertise-cl??   3 minutes ago       Up 3 minutes                            k8s_etcd_etcd-master_kube-system_2cc1c8a24b68ab9b46bca47e153e74c6_0
8f01317b9e20        37a1403e6c1a           "kube-scheduler --ad??   3 minutes ago       Up 3 minutes                            k8s_kube-scheduler_kube-scheduler-master_kube-system_a00c35e56ebd0bdfcd77d53674a5d2a1_0
4e6a71ab20d3        821507941e9c           "kube-apiserver --au??   3 minutes ago       Up 3 minutes                            k8s_kube-apiserver_kube-apiserver-master_kube-system_d25d40ebb427821464356bb27a38f487_0
69e4c5dae335        38521457c799           "kube-controller-man??   3 minutes ago       Up 3 minutes                            k8s_kube-controller-manager_kube-controller-manager-master_kube-system_6363f7ebf727b0b95d9a9ef72516a0e5_0
da5981dc546a        k8s.gcr.io/pause:3.1   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_kube-controller-manager-master_kube-system_6363f7ebf727b0b95d9a9ef72516a0e5_0
b7a8fdc35029        k8s.gcr.io/pause:3.1   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_kube-apiserver-master_kube-system_d25d40ebb427821464356bb27a38f487_0
b09efc7ff7bd        k8s.gcr.io/pause:3.1   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_kube-scheduler-master_kube-system_a00c35e56ebd0bdfcd77d53674a5d2a1_0
ab11d6ffadab        k8s.gcr.io/pause:3.1   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_etcd-master_kube-system_2cc1c8a24b68ab9b46bca47e153e74c6_0
```

node 节点可以通过如下命令加入集群 `kubeadm join 192.168.31.81:6443 --token n84v6t.c7d83cn4mo2z8wyr --discovery-token-ca-cert-hash sha256:b946c145416fe1995e1d4d002c149e71a897acc7b106d94cee2920cb2c85ce29` 

在 kubeadm init 的[输出](https://segmentfault.com/a/1190000016026998)中，提示我们需要以普通用户做如下操作，我此时用 root 执行

```sh
[root@master ~]# mkdir -p $HOME/.kube
[root@master ~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

此时可以通过 `kubelet get` 获取各种资源信息。比如

```sh
[root@master ~]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"} 
```

此时的监听状态

```sh
[root@master ~]# ss -tnl
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN      0      128    127.0.0.1:2379                  *:*                  
LISTEN      0      128    127.0.0.1:10251                 *:*                  
LISTEN      0      128    127.0.0.1:2380                  *:*                  
LISTEN      0      128    127.0.0.1:10252                 *:*                  
LISTEN      0      128       *:22                    *:*                  
LISTEN      0      128    127.0.0.1:33881                 *:*                  
LISTEN      0      100    127.0.0.1:25                    *:*                  
LISTEN      0      128    192.168.18.128:10010                 *:*                  
LISTEN      0      128    127.0.0.1:10248                 *:*                  
LISTEN      0      128    127.0.0.1:10249                 *:*                  
LISTEN      0      128      :::6443                 :::*                  
LISTEN      0      128      :::10256                :::*                  
LISTEN      0      128      :::22                   :::*                  
LISTEN      0      100     ::1:25                   :::*                  
LISTEN      0      128      :::10250                :::*         
```

此时的节点状态

```sh
[root@master ~]# kubectl get nodes
NAME      STATUS     ROLES     AGE       VERSION
master    NotReady   master    9m        v1.11.2
```

状态为 `NotReady` ， 需要部署 flannel，[链接](https://github.com/coreos/flannel)

### 部署 flannel

在文档中，找到如下命令，部署

```sh
[root@master ~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created
```

按如下方法查看：

```sh
[root@master ~]# kubectl get pods -n  kube-system
NAME                             READY     STATUS    RESTARTS   AGE
coredns-78fcdf6894-cv4gp         1/1       Running   0          13m
coredns-78fcdf6894-wmd25         1/1       Running   0          13m
etcd-master                      1/1       Running   0          49s
kube-apiserver-master            1/1       Running   0          49s
kube-controller-manager-master   1/1       Running   0          48s
kube-flannel-ds-amd64-r42wr      1/1       Running   0          2m
kube-proxy-6fgjm                 1/1       Running   0          13m
kube-scheduler-master            1/1       Running   0          48s
[root@master ~]# docker images |grep flannel
quay.io/coreos/flannel                     v0.10.0-amd64       f0fad859c909        6 months ago        44.6MB
[root@master ~]# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    14m       v1.11.2
```

此时 master 节点状态变为 `Ready` 。

### 在 node 节点上执行 `kubeadm join` 命令

使用 master 节点执行 `kubeadm init` 命令的输出，在 node 上执行，使其加入集群。

```sh
[root@node01 ~]#   kubeadm join 192.168.18.128:6443 --token n84v6t.c7d83cn4mo2z8wyr --discovery-token-ca-cert-hash sha256:b946c145416fe1995e1d4d002c149e71a897acc7b106d94cee2920cb2c85ce29 --ignore-preflight-errors=Swap

[preflight] running pre-flight checks
    [WARNING RequiredIPVSKernelModulesAvailable]: the IPVS proxier will not be used, because the following required kernel modules are not loaded: [ip_vs ip_vs_rr ip_vs_wrr ip_vs_sh] or no builtin kernel ipvs support: map[ip_vs_sh:{} nf_conntrack_ipv4:{} ip_vs:{} ip_vs_rr:{} ip_vs_wrr:{}]
you can solve this problem with following methods:
 1. Run 'modprobe -- ' to load missing kernel modules;
2. Provide the missing builtin kernel ipvs support

    [WARNING Swap]: running with swap on is not supported. Please disable swap
I0815 22:02:30.751069   15460 kernel_validator.go:81] Validating kernel version
I0815 22:02:30.751145   15460 kernel_validator.go:96] Validating kernel config
    [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 18.06.0-ce. Max validated version: 17.03
[discovery] Trying to connect to API Server "192.168.18.128:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://192.168.18.128:6443"
[discovery] Requesting info from "https://192.168.18.128:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "192.168.18.128:6443"
[discovery] Successfully established connection with API Server "192.168.18.128:6443"
[kubelet] Downloading configuration for the kubelet from the "kubelet-config-1.11" ConfigMap in the kube-system namespace
[kubelet] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[preflight] Activating the kubelet service
[tlsbootstrap] Waiting for the kubelet to perform the TLS Bootstrap...
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "node01" as an annotation

This node has joined the cluster:
* Certificate signing request was sent to master and a response
  was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```

在两个节点上，执行完毕上述命令后，在 master 上查看

```sh
[root@master ~]# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    23m       v1.11.2
node01    Ready     <none>    2m        v1.11.2
node02    Ready     <none>    1m        v1.11.2
```

部署成功。

## 部署完成后的观察

**检查现在正在运行的 pod**

```sh
[root@master ~]# kubectl get pods -n kube-system -o wide
NAME                             READY     STATUS    RESTARTS   AGE       IP               NODE      NOMINATED NODE
coredns-78fcdf6894-cv4gp         1/1       Running   0          28m       10.244.0.3       master    <none>
coredns-78fcdf6894-wmd25         1/1       Running   0          28m       10.244.0.2       master    <none>
etcd-master                      1/1       Running   0          15m       192.168.18.128   master    <none>
kube-apiserver-master            1/1       Running   0          15m       192.168.18.128   master    <none>
kube-controller-manager-master   1/1       Running   0          15m       192.168.18.128   master    <none>
kube-flannel-ds-amd64-48rvq      1/1       Running   3          6m        192.168.18.130   node02    <none>
kube-flannel-ds-amd64-7dw42      1/1       Running   3          7m        192.168.18.129   node01    <none>
kube-flannel-ds-amd64-r42wr      1/1       Running   0          16m       192.168.18.128   master    <none>
kube-proxy-6fgjm                 1/1       Running   0          28m       192.168.18.128   master    <none>
kube-proxy-6mngv                 1/1       Running   0          7m        192.168.18.129   node01    <none>
kube-proxy-9sh2n                 1/1       Running   0          6m        192.168.18.130   node02    <none>
kube-scheduler-master            1/1       Running   0          15m       192.168.18.128   master    <none>
```

可以发现：

1. kube-apiserver， kube-scheduler， kube-controller，etcd-master 运行在 master 上，
2. kube-flannel 在三个节点上均有运行
3. kube-proxy 在三个节点均有运行

