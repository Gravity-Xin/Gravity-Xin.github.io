---
title: Kubernetes入门
date: 2019-01-17 16:31:32
categories: 
- 系统架构
tags: 
- Kubernetes
- Namespace
- Pod
- Service
comments: true
---

当前主流的容器编排技术包括

- `docker compose`: Docker官方的单机版本的容器编排工具
- `docker swarm`: Docker官方的多机版本的容器编排工具
- `mesos + marathon`: Apache提供的集群资源管理和容器编排工具
- `kubernetes`: Google提供的容器编排工具，事实上已经成为容器编排的标准工具

## 主要功能

- 基于容器的应用的自动部署、滚动升级、自动修复和版本回滚
- 应用配置的管理
- 服务的自动伸缩和水平扩展
- 负载均衡和服务发现
- 跨机器和跨地区的集群调度
- 无状态和有状态服务
- 广泛的数据卷支持和管理
- 插件机制保证可扩展性

## 系统架构

Master-Slave(Node)结构

![kubernetes architecture](/images/Kubernetes之系统架构.png)

- `Kubectl`: 用户的命令行交互工具

- K8S Control Plane(Master): 管理整个集群
  - 各个组件可以以分布式的形式在多台机器上运行
  - 为保证高可用，推荐部署3个Master节点
  - 主要组件包括:
    - `API Server`: 用户API入口
      - 将K8S API以`JSON over HTTP`的形式暴露给用户
      - 用户可以通过kubectl发起请求，对集群中的Pod在Node上进行调度
      - 请求处理的结果最终保存到ETCD中
    - `Controller Manager`: 控制器管理器
      - 管理多个Pod控制器，一般一个Pod需要一个Pod控制器
      - 常用的Pod控制器包括:
        - `ReplicationController`: 早期版本的副本控制器
        - `ReplicaSet`: 副本集控制器，一般不直接使用
        - `Deployment`: 声明式的无状态应用部署控制器，支持滚动升级和版本回滚
          - 支持HPA(HorizontalPodAutoscaling)，可以对应用进行自动伸缩
        - `DaemonSet`: 无状态应用，保证一个Node上运行一个Pod，对应系统级别的后台进程
        - `StatefulSet`: 有状态应用
        - `Job`: 一次性任务，即完成后便退出的应用
        - `Cronjob`: 周期性任务，不需要持续运行的应用
      - 每个控制器分别负责监控集群中的某些状态，使之符合Object.status的要求
      - 与API Server进行通信，实现资源(Pod, Service)的创建、更新和删除
    - `Scheduler`: 调度器
      - 将集群中所有的Node的资源看做是一个整体
      - 监控各个Node上的资源使用情况
      - 对资源进行调度，按照预定的策略将Pod调度到对应的Node上
    - `ETCD`: 集群状态存储
      - 轻量级的分布式一致性KV存储服务
      - API Server使用其提供的watch机制对资源状态进行监控
      - 存储整个集群的状态信息
    - `AddONs`: 插件，也是以Pod+Service的形式运行
      - DNS: 解析规则会根据集群状态的变化而自动改变
      - Dashboard: 集群管理页面
      - Heapster: 集群监控
      - IngressController: Ingress资源管理

- K8S Node(Slave): Pod真正运行的机器
  - 带有容器的运行时，比如Docker
  - 还包括以下组件
    - `Kubelet`
      - 对容器、数据卷、网络等进行管理
      - 监控容器运行状态和资源使用状态并上报到Master
    - `Container`
      - 处于Pod的内部，包含了应用所需的依赖
    - `Kube-Proxy`
      - 实现了网络代理、服务发现、负载均衡等功能
      - 当Service对象发生变化后，自动在iptables中修改对应的规则，相当于为Service对象提供了一层抽象
    - `cAdvisor`
      - Kubelet使用cAdvisor进行容器资源使用情况的收集
      - 对各个容器使用的系统资源(CPU,Memory,Network,File)进行收集

## 基本概念

- `Namespace`: 用于创建多个虚拟集群，对资源进行隔离
  - 命名空间的名称在系统中唯一
  - 删除一个命名空间会自动删除属于该空间的资源
  - 名称为default和kube-system为系统使用，不可以删除
  - 大多数的资源都属于命名空间，如Pod, Service, Replica, Deployment等
  - 低级别的资源如Node, PersistentVolume则不属于任何命名空间

- `Pod`: 系统中可以进行调度的最小对象，可以包含一个或多个容器
  - Pod在系统中会被分配一个唯一的IP地址
  - Pod中的多个容器共享网络、数据卷等 **使用pause容器作为根容器，Pod中的其它容器共享pause容器的各类名称空间**
  - Pod内部的容器之间可以通过localhost进行访问 **本质是Pod中的容器使用的是联盟式网络**
  - Pod是有生命周期的，可能会被集群启动、修改和删除
  - 跨Pod的容器之间的访问可以通过Service对象来实现

- `Service`: 一个或多个Pod组成，是Pod之上的概念
  - 通过Label Selector来筛选Pod，符合条件的Pod的IP和端口组成了一个EndPoints列表
  - 每一个Service会被分配一个Cluster IP和DNS名称
  - 外部用户和其他Service可以通过该Cluster IP或DNS名来访问服务，而不需要了解服务对应的Pod列表，可以将Service看做是对Pod的代理
  - Kube-Proxy将用户对Service的访问均衡负载到这些EndPoints上，从而实现了负载均衡和服务发现的功能  **本质上是DNS解析和在iptables添加了DNAT数据转发规则**

- `Volume`: 为Pod提供存储空间和文件共享

管理`Object`的机制包括:

- `Label`: 识别Object的标签
  - key/value对
    - key: 长度小于63字符，只能为数字、字母、下划线、_和.，且以字母开头
    - value: 长度小于63字符，可以为空，只能为数字、字母、下划线、_和.，且以字母或数字开头结尾
  - 使用`kubectl get --selector`命令可以通过指定的标签选择器过滤出符合条件的Object
    - 等式: app=nginx env!=production
    - 集合: env in (production, test)
    - 组合: app=nginx, env=test
  - 使用`kubectl label`命令更新对象的标签
  - 许多资源如`ReplicaSet`、`Deployment`支持使用内嵌字段来定义它们的标签选择器，从而从集群中获取符合条件的资源
    - `matchLabels`: 直接给定标签的KV对
    - `matchExpressions`: 基于给定的表达式来定义标签选择器
- `Annotation`: 对Object的描述
  - key/value对，键值大小不受限制
  - 记录Object的一些附属信息