---
title: Kubernetes入门
date: 2019-01-17 16:31:32
categories: 
- Docker/K8S
tags: 
- Kubernetes
- Namespace
- Pod
- Service
comments: true
---

## 主要功能

- 基于容器的应用部署、维护和滚动升级
- 负载均衡和服务发现
- 跨机器和跨地区的集群调度
- 自动伸缩
- 无状态和有状态服务
- 广泛的数据卷支持
- 插件机制保证可扩展性

## 基本概念

- Object
  - 对持久实体进行描述
  - 一般包含两个嵌套对象
    - spec: 对象所需要的状态**K8S的核心就是保证应用总是在期望的状态**
    - status: 对象实际的状态
  - 使用K8S API对对象进行管理
  - 创建对象时，一般使用`.yaml`文件来进行描述
    - 必填字段
      - apiVersion: K8S API的版本
      - kind: 对象类型
      - metadata: 对象唯一标识，包含`name, uid, namespace`等信息
      - spec: 对象所需要的状态

常用的`Object`对象包括:

- Namespace: 用于创建多个虚拟集群，对资源进行隔离
  - 命名空间的名称在系统中唯一
  - 删除一个命名空间会自动删除属于该空间的资源
  - 名称为`default`和`kube-system`为系统使用，不可以删除
  - 大多数的资源都属于命名空间，如`Pod, Service, Replica, Deployment`等
  - 低级别的资源如`Node, PersistentVolume`则不属于任何命名空间

- Pod: 系统中可以进行调度的最小对象，可以包含一个或多个容器
  - Pod在系统中会被分配一个唯一的IP地址
  - Pod中的多个容器共享网络和文件系统
  - Pod内部的容器之间可以通过`localhost`进行访问
  - 跨Pod的容器之间的访问可以通过`Service`对象来实现

- Service: 多个Pod组成，应用服务的抽象
  - 通过`Label`为应用提供负载均衡和服务发现，匹配`Label`的Pod的IP和端口组成了一个`EndPoints`列表
  - 每一个`Service`会被分配一个`Cluster IP`和DNS名称
  - 其他Pod可以通过该`Cluster IP`或DNS名来访问服务，而不需要了解服务对应的Pod列表
  - `kube-proxy`将对`Service`的访问均衡负载到这些`EndPoints`上

- Volume: 为Pod提供存储空间和文件共享

管理`Object`的机制包括:

- Label: 识别Object的标签
  - key/value对
  - 使用`Label Selector`用于过滤出符合标签条件的Object
    - 等式: app=nginx env != production
    - 集合: env in (production, test)
    - 组合: app=nginx, env=test

- Annotation: 对Object的描述
  - key/value对
  - 记录Object的一些附属信息

## 系统架构

master-slave结构

![kubernetes architecture](/images/kubernetes_architecture.png)

- kubectl: 用户的命令行交互工具

- k8s control plane(master): 管理整个集群
  - 各个组件可以以分布式的形式在多台机器上运行
  - 主要组件包括:
    - api server
      - 将K8S API以`JSON over HTTP`的形式暴露给用户
      - 用户可以通过`kubectl`发起请求，对集群中的Pod在Node上进行调度
      - 请求处理的结果最终保存到ectd中
    - controller manager
      - 维护集群中各个资源的运行状态，使之符合`Object.status`的要求
      - 与api server进行通信，实现资源(Pod, Service)的创建、更新和删除
    - scheduler
      - 监控各个Node上的负载情况
      - 对资源进行调度，按照预定的策略将Pod调度到对应的Node上
    - etcd
      - 轻量级的分布式一致性KV存储服务
      - api server使用其提供的`watch`机制对资源状态进行监控
      - 存储整个集群的状态信息
    - addons: 插件
      - DNS
      - dashboard

- k8s node(slave): Pod真正运行的机器
  - 带有容器的运行时，比如Docker
  - 还包括以下组件
    - kubelet
      - 对容器、数据卷、网络等进行管理
      - 监控容器运行状态和容器状态上报
    - container
      - 处于Pod的内部，包含了应用所需的依赖
    - kube-proxy
      - 实现了网络代理、服务发现、负载均衡等功能
      - 为`Service`对象提供了一层抽象
    - cAdvisor
      - 对各个容器使用的系统资源(CPU,Memory,Network,File)进行收集和上报