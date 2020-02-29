---
title: Opeartor-SDK 简单上手
date: 2019-02-28 14:15:15
categories: 
tags:
- Kubernetes
- 容器学习
---

## 前言

>   本篇介绍了CoreOS（已被红帽收购）的开源项目 [Operator-SDK](https://github.com/operator-framework/operator-sdk) 的基本使用。该项目是 `Operator Framework` 的一个组件，它是一个开源工具包，以有效，自动化和可扩展的方式管理称为 `Operators` 的Kubernetes原生应用程序

### 概述

[Operators](https://coreos.com/operators/) 可以在Kubernetes之上轻松地管理复杂有状态的应用程序。然而，由于诸如使用低级API，编写样板以及缺乏模块导致重复性工作等挑战，导致目前编写Operator可能很困难

Operator SDK是一个框架，旨在简化Operator的编写，它提供如下功能：

-   高级API和抽象，更直观地编写操作逻辑
-   用于脚手架和代码生成的工具，可以快速引导新项目
-   扩展以涵盖常见的操作员用例

## 工作流程

SDK提供以下工作流程来开发新的Operator：

1.  使用SDK命令行界面（CLI）创建新的Operator项目
2.  通过添加自定义资源定义（CRD）定义新资源API
3.  使用SDK API监控指定的资源
4.  在指定的处理程序中定义Operator协调逻辑(对比期望状态与实际状态)，并使用SDK API与资源进行交互
5.  使用SDK CLI构建并生成Operator部署manifests

Operator使用SDK在用户自定义的处理程序中以高级API处理监视资源的事件，并采取措施来reconcile（对比期望状态与实际状态）应用程序的状态

## 快速开始

### 安装 Operator-SDK CLI

```bash
$ mkdir -p $GOPATH/src/github.com/operator-framework
$ cd $GOPATH/src/github.com/operator-framework
$ git clone https://github.com/operator-framework/operator-sdk
$ cd operator-sdk
$ git checkout master
$ make dep
$ make install
```

**注：**这里可知，`Operator-SDK` 默认采用 `dep` 作为包管理方式，但 `dep` 当前的执行效率的确不高，如果你的 project 依赖的外部包很多，那么等待的时间可能会很长。并且由于 dep 会下载依赖包，一旦下载 qiang 外的包，那么 dep 可能会 “阻塞” 在那里！因此这里可以选择自己拉包后 `go install`

### 创建 app-operator

```bash
# 创建一个定义 App 用户资源的 app-operator 项目
$ mkdir -p $GOPATH/src/github.com/<GitHub_ID>

# 创建 app-operator 项目
$ cd $GOPATH/src/github.com/<GitHub_ID>
$ operator-sdk new app-operator  
$ cd app-operator

# 为AppService用户资源添加一个新的 API
$ operator-sdk add api --api-version=app.example.com/v1alpha1 --kind=AppService
# Required: --kind - 是 CRD 要定義的 kind
# Required: --api-version - 是CRD 想定义的 group/version。

# 添加一个新的控制器来监控 AppService
$ operator-sdk add controller --api-version=app.example.com/v1alpha1 --kind=AppService

# 构建并推送 app-operator 镜像到一个公开的 registry
# 这里 build 镜像，使用的基础镜像，由于不可描述的原因，非常耗时
# 可替换为 ： FROM docker.io/ericnie2017/helm-operator:latest
$ operator-sdk build app-operator:v0.1.0
$ docker push 

# 更新 operator manifest 来使用新构建的镜像
$ sed -i 's|REPLACE_IMAGE|app-operator:v0.1.0|g' deploy/operator.yaml
```

**注：**这里采用 `operator-sdk new ` 创建 app-operator 时，会自动生成 `Gopkg.toml` 并使用 `dep` 下载依赖包，最好挂代理，不然非常耗时，也可以直接跳过，自己拉包编译

### 创建 app-operator for Helm

```bash
# 其他步骤都一样
# 只是再创建 app-operator 时，增加 --type=helm，并且再创建 helm 类型的 operator 时，可以直接 add api 和 kind
$ operator-sdk new <project-name> --type=helm --kind=<kind> --api-version=<group/version> 

# 例如
$ operator-sdk new app-operator api --api-version=app.example.com/v1alpha1 --kind=AppService --type=helm
```

### 部署 app-operator

上述两种创建方式，都会自动生成 `/deploy` 文件夹，其中包含一组可以 deploy 到 K8s cluser 上的 resource files；

-   **deploy/service_account.yaml**: 建立一个该自定义 operator 名称的 ServiceAccount
-   **deploy/role.yaml**: 建立一个 role 定义它能存取 K8s cluster 的规则
-   **deploy/role_binding.yaml**: 把上面的 ServiceAccount 和 Role 绑定起来，使该 ServiceAccount 拥有这个 Role 定义的存取 rule
-   **deploy/operator.yaml**: 是 deploy operator 到 K8s cluster 上的 deployment，使用的 ServiceAccount 是我们 `deploy/service_account.yaml` 定义的名称
-   **deploy/crds/GROUP_VERSION_CRDNAME_crd.yaml**: 就是我們自己定义 resource type，More detail: [Extend the Kubernetes API with CustomResourceDefinitions](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/)
-   **deploy/crds/GROUP_VERSION_CRDNAME_cr.yaml**: 是我们要 deploy 自定义的 resource，CR 代表 Custom Resource ，More detail: [Create custom objects](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/#create-custom-objects) ，该文件中的 spec 就是完全复制 `helm-charts` 下的 values.yaml，所以原本用 helm 怎么定义 values.yaml 中的值，该文件中就怎么写

```bash
# 建立Service Account
$ kubectl create -f deploy/service_account.yaml

# 建立RBAC
$ kubectl create -f deploy/role.yaml
$ kubectl create -f deploy/role_binding.yaml

# 建立CRD
$ kubectl create -f deploy/crds/app_v1alpha1_appservice_crd.yaml

# 部署app-operator
# 这里部署时，由于自动生成的 operator.yaml 文件中的 拉取镜像，默认为 imagePullPolicy: Always ；到镜像仓库拉取
# 我一般镜像打包到本地，所以替换为  imagePullPolicy: IfNotPresent ；本地有，就不到镜像仓库拉取
$ kubectl create -f deploy/operator.yaml

# 创建一个AppService用户资源
# 默认控制器将监视AppService对象并为每一个CR创建一个pod
$ kubectl create -f deploy/crds/app_v1alpha1_appservice_cr.yaml

# 验证pod是否创建
$ kubectl get pod -l app=example-appservice
NAME                     READY     STATUS    RESTARTS   AGE
example-appservice-pod   1/1       Running   0          1m
```

## 最后

这样通过 `operator-framework/operator-sdk ` 要完成自己的 CRD 和 operator 真的很快速和方便，可以预见，如果是开发或运维人员，在了解需求后，可以不用写很多的 operator 复杂和繁琐的重复程序，就可以快速开发和部署 operator ，且后续一样可以很方便的使用 K8s 的 API 资源；不过，是否要使用 `operator-framework/operator-sdk` ，一切还是需要自行的评估和结合当前应用服务的体量来看是否适合使用，小心服用。