---
title: Kuberntes 创建 LoadBalancer 类型服务
date: 2019-02-01 14:15:15
categories: 
tags:
- Kubernetes
- 容器学习
---

## 前言

> 我们知道，Service 机制，以及 Kubernetes 里的 DNS 插件，都是在帮助我们解决同样一个问题，即：如何找到某一个容器；而 Service 是由 kube-proxy 组件，加上 iptables 来共同实现的；所谓 Service 的访问入口，其实就是每台宿主机上由 kube-proxy 生成的 iptables 规则，以及 kube-dns 生成的 DNS 记录。而一旦离开了这个集群，这些信息对用户来说，也就自然没有作用了

在使用 Kubernetes 的 Service 时，一个必须要面对和解决的问题就是：**如何从外部（Kubernetes 集群之外），访问到 Kubernetes 里创建的 Service？**

## 从外界连通 Service

### 三种方式

- NodePort
- ExternalIP
- LoadBalance

这里 `NodePort` 和 `externalIPs` 是 K8S 集群本身就支持的特性，这两个方案让很多私有云用户成为了 K8S 世界中的二等公民，而 `NodePort` 也是我们最常用的；

而 kubernetes  没有为裸机群集提供网络负载均衡器（类型为 `LoadBalancer` 的服务）的实现，如果你的 K8S 集群没有在公有云的 IaaS 平台（GCP，AWS，Azure …）上运行，则 LoadBalancers 将在创建时无限期地保持 “挂起” 状态，也就是说只有公有云厂商自家的 kubernetes 支持 LoadBalancer，对 LoadBalancer 类型的服务的支持应该是众多表面差异中最醒目的一个了

### 纯软件解决方案: MetalLB

> 该项目发布于 2017 年底，当前处于 Beta 阶段，旨在解决上述中的不平衡，通过提供与标准网络设备集成的网络 LB 实现来纠正这种不平衡，以便裸机集群上的外部服务也 “尽可能” 地工作；即 MetalLB 能够帮助你在 kubernetes 中创建 LoadBalancer 类型的 kubernetes 服务
>
> 项目地址：https://github.com/google/metallb
>
> 版本说明：https://metallb.universe.tf/release-notes/

`Metallb` 会在 Kubernetes 内运行，监控服务对象的变化，一旦察觉有新的 LoadBalancer 服务运行，并且没有可申请的负载均衡器之后，就会完成两部分的工作：

- **地址分配**

  用户需要在配置中提供一个地址池，Metallb 将会在其中选取地址分配给服务。

- **地址广播**

  根据不同配置，Metallb 会以二层（ARP/NDP）或者 BGP 的方式进行地址的广播

![基本原理图](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2019/metallb.jpg )

## 安装部署演示

### 部署 Metallb 负载均衡器

Metallb 支持 Helm 和 YAML 两种安装方法(这里推荐第二种)

- Helm

  ```sh
  $ helm install --name metallb stable/metallb
  ```

- YAML

  ```bash
  $ kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml
  ```

很简单，`Metallb` 就会开始安装，会生成自己的命名空间以及 RBAC 配置

```sh
[root@lee ~]# kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml
namespace/metallb-system created
serviceaccount/controller created
serviceaccount/speaker created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
role.rbac.authorization.k8s.io/config-watcher created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/config-watcher created
daemonset.apps/speaker created
deployment.apps/controller created

[root@lee ~]# kubectl get pods -n metallb-system -owide
NAME                                             READY   STATUS  RESTARTS   AGE   IP               NODE   NOMINATED NODE
controller-765899887-bdwdk   1/1     Running   0          82s   10.0.0.22                lee    <none>
speaker-qldl6                                  1/1     Running   0          82s   192.168.50.124   lee    <none>
```

目前还没有宣布任何内容，因为我们没有提供 ConfigMap，也没有提供负载均衡地址的服务；
接下来我们要生成一个 ConfigMap文件，为 Metallb 设置网址范围以及协议相关的选择和配置

### 提供 IP pool

通过之前提到的原理图可知，需要创建一个 `ConfigMap` 文件，以提供 `IP 池`；

```sh
$ wget https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/example-layer2-config.yaml
```

**修改 ip 地址池和集群节点网段相同，且必须为一个 IP 区间**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: my-ip-space
      protocol: layer2
      addresses:
      # 与集群节点网段相同
      - 192.168.50.211-192.168.50.220  
```

**IP 地址为 `无占用` **

```bash
[root@lee ~]# ping 192.168.50.211
PING 192.168.50.211 (192.168.50.211) 56(84) bytes of data.
From 192.168.50.124 icmp_seq=1 Destination Host Unreachable
From 192.168.50.124 icmp_seq=2 Destination Host Unreachable
From 192.168.50.124 icmp_seq=3 Destination Host Unreachable
From 192.168.50.124 icmp_seq=4 Destination Host Unreachable
^C
--- 192.168.50.211 ping statistics ---
5 packets transmitted, 0 received, +4 errors, 100% packet loss, time 4001ms
```

部署 `example-layer2-config.yaml` 文件

```sh
$ kubectl apply -f example-layer2-config.yaml
```

### 部署应用服务测试

```sh
$ wget https://raw.githubusercontent.com/google/metallb/master/manifests/tutorial-2.yaml
$ kubectl apply -f tutorial-2.yaml
```

查看 yaml 文件配置，包含了一个 deployment 和一个 LoadBalancer 类型的 service，默认即可；

**查看 service 分配的 EXTERNAL-IP**

```bash
[root@lee ~]# kubectl get svc -owide
NAME               TYPE             CLUSTER-IP     EXTERNAL-IP         PORT(S)                 AGE   SELECTOR
nginx           LoadBalancer   10.0.228.36      192.168.50.211     80:31796/TCP        65s   app=nginx
```

ping 该 ` EXTERNAL-IP` 地址，发现该地址可以访问了

```sh
[root@lee ~]# ping 192.168.50.211
PING 192.168.50.211 (192.168.50.211) 56(84) bytes of data.
64 bytes from 192.168.50.211: icmp_seq=1 ttl=64 time=0.050 ms
64 bytes from 192.168.50.211: icmp_seq=2 ttl=64 time=0.082 ms
64 bytes from 192.168.50.211: icmp_seq=3 ttl=64 time=0.068 ms
```

集群内访问该 ` EXTERNAL-IP` 地址

```bash
[root@lee ~]# curl 192.168.50.211
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...............
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

从集群外访问该 IP 地址

```bash
Administrator at 16:47:09 / $ curl 192.168.50.211
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  597k<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
................
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

另外使用 `Node IP + NodePort` 也可以访问

```bash
Administrator at 16:47:09 / $ curl 192.168.50.124:31796
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  597k<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
................
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```