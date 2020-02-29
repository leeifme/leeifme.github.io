---
title: 简化 Kubernetes 应用部署工具 -- Helm
date: 2018-11-10 14:15:15
categories: 
tags:
- Kubernetes
- 容器学习
---

### 先区分下概念

- **`Docker`**: 镜像是把一个单纯的 App 和它的安装环境整合在一起。
- **`Kubertnetes`**: 管理 Docker 容器的生成和毁灭，保证 Docker 容器对应 App 的高可用（监控、自动创建）和易维护（部署和对外暴露、动态扩容、启动停止删除等）。
- **`Helm`**: 是为了方便配置和部署、升级和回滚应用，尤其是多个 Service 组合创建的一个大型应用，比如网站

### 为什么要用？

> 首先在原来项目中都是基于 yaml 文件来进行部署发布的，而目前项目大部分微服务化或者模块化，会分成很多个组件来部署，每个组件可能对应一个 deployment.yaml, 一个 service.yaml, 一个 Ingress.yaml 还可能存在各种依赖关系，这样一个项目如果有 5 个组件，很可能就有 15 个不同的 yaml 文件，这些 yaml 分散存放，如果某天进行项目恢复的话，很难知道部署顺序，依赖关系等，而所有这些包括
>
> - 基于 yaml 配置的集中存放
> - 基于项目的打包
> - 组件间的依赖
>
> 都可以通过 helm 来进行解决

 

### Helm 基本概念

> Helm 可以理解为 Kubernetes 的包管理工具，可以方便地发现、共享和使用为 Kubernetes 构建的应用，它包含几个基本概念
>
> - Chart：一个 Helm 包，其中包含了运行一个应用所需要的镜像、依赖和资源定义等，还可能包含 Kubernetes 集群中的服务定义
> - Release: 在 Kubernetes 集群上运行的 Chart 的一个实例。在同一个集群上，一个 Chart 可以安装很多次每次安装都会创建一个新的 release。例如一个 MySQL Chart，如果想在服务器上运行两个数据库，就可以把这个 Chart 安装两次。每次安装都会生成自己的 Release，会有自己的 Release 名称
> - Repository：用于发布和存储 Chart 的仓库

#### 要点

- Helm 是一个 Chart 管理器: [GitHub - kubernetes/helm: The Kubernetes Package Manager](https://github.com/kubernetes/helm)
- Charts 是一组配置好的 Kubernetes 资源（定义）组合
- Release 是一组已经部署到 Kubernetes 上的资源集合

#### Chart 的基本结构

```
.
├── Chart.yaml
├── README.md
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── pvc.yaml
│   ├── secrets.yaml
│   └── svc.yaml
└── values.yaml
```

#### 用途

1. 创建可配置的 Release
2. 升级、删除、查看由 Helm 创建的 Release

#### 组成

1. **`Helm Client`** 客户端
   - 制作、拉取、查找和验证 Chart
   - 安装服务端 Tiller
   - 指示服务端 Tiller 做事，比如根据 chart 创建一个 Release
2. helm 服务端 **`tiller`**
   安装在 Kubernetes 集群内的一个应用， 用来执行客户端发来的命令，管理 Release



### Helm的安装

#### `Helm Client` 安装

下载 helm 的相关版本: https://github.com/kubernetes/helm/releases

安装过程如下：

1. 下载 Helm 
2. 解包：tar -zxvf helm-v2.11.0-linux-amd64.tgz
3. helm 二进制文件移到 / usr/local/bin 目录

#### `Helm Tiller ` 安装

> 因为 Kubernetes APIServer 开启了 RBAC 访问控制，所以需要创建 tiller 使用的 service account: tiller 并分配合适的角色给它。这里简单起见直接分配 cluster-admin 这个集群内置的 ClusterRole 给它。创建 rbac-config.yaml 文件：

```yaml
# rbac-config.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

接下来使用 helm 部署 tiller:

```bash
helm init --service-account tiller --skip-refresh
```

如果直接执行部署语句，要链接 https://kubernetes-charts.storage.googleapis.com 去下载镜像；在国内这个地址不能直接访问，所以需要下载其他镜像；docker pull fengzos/tiller:v2.11.0 这个镜像是直接从 gcr.io/kubernetes-helm/tiller:v2.11.0 继承过来的，可以直接使用。

```bash
docker pull fengzos/tiller
helm init --service-account tiller --upgrade -i fengzos/tiller:latest  --skip-refresh
```

#### 帮助文档

1. `helm help` 查看 helm 支持的命令
2. `helm somecommand -h` 查看某个命令的使用方法
3. `helm version` 查看客户端和服务端的版本，如果只显示了客户端版本，说明没有连上服务端。 它会自动去 K8s 上 kube-system 命名空间下查找是否有 Tiller 的 Pod 在运行，除非你通过 `--tiller-namespace`标签 or `TILLER_NAMESPACE`环境变量指定

### Helm的使用

#### 使用 Chart

- `helm search` 查找可用的 Charts
- `helm inspect` 查看指定 Chart 的基本信息
- `helm install` 根据指定的 Chart 部署一个 Release 到 K8s
- `helm create` 创建自己的 Chart
- `helm package` 打包 Chart，一般是一个压缩包文件

#### 管理 Release

- `helm list` 列出已经部署的 Release
- `helm delete [RELEASE]` 删除一个 Release. 并没有物理删除， 出于审计需要，历史可查
- `helm delete --purge [RELEASE]` 移除所有与指定 Release 相关的 K8s资源和所有这个 Release 的记录
- `helm status [RELEASE]` 查看指定的 Release 信息，即使使用`helm delete`命令删除的 Release.
- `helm upgrade` 升级某个 Release
- `helm rollback [RELEASE] [REVISION]` 回滚 Release 到指定发布序列
- `helm get values [RELEASE]` 查看 Release 的配置文件值

#### 管理 Chart Repository

- `helm repo list`
- `helm repo add [RepoName] [RepoUrl]`
- `helm repo update`