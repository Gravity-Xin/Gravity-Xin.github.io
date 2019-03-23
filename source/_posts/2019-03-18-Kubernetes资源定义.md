---
title: Kubernetes资源定义
date: 2019-03-18 13:44:40
categories: 
- 系统架构
tags: 
- Kubernetes
- Namespace
- Pod
- Service
- Deployment
---

在K8S中，使用`.yaml`配置文件对集群中的资源进行定义

## 核心资源类型

- 工作负载型
  - Pod: 可以被创建、调度和管理的原子单元
  - Pod控制器
    - ReplicaSet
    - Deployment
    - StatefulSet
    - DaemonSet
    - Job
    - CronJob
- 服务发现和负载均衡型
  - Service
  - Ingress
- 配置与存储型
  - ConfigMap
  - Secret
  - PersistentVolumeClaim
- 集群型: 前三者都属于某个名称空间，而集群型资源不属于任何的名称空间
  - Namespace
  - Node
  - Role
  - RoleBinding
  - ClusterRole
  - ClusterRoleBinding
  - PersistentVolume
- 元数据型资源
  - HorizontalPodAutoscaler: HPA
  - PodTemplate
  - LimitRange

## 资源定义方式

使用`kubectl get --output yaml`来查看集群资源的描述

集群中的资源描述文件包含下列字段:

- `apiVersion`: 资源所属的K8S API的群组
- `kind`: 对象资源类型
- `metadata`: 对象的元数据
  - `name`: 对象名称
  - `namespace`: 对象所属的集群名称空间
  - `uid`: 对象唯一标识
  - `labels`: 对象的标签
  - `annotations`: 对象的注解
- `spec`: 对象应该满足的要求。不同类型的资源，该字段中包含的下级字段是有区别的 **K8S的核心就是通过控制器保证资源对象总是在期望的状态**
- `status`: 对象实际的运行状态，该字段由集群进行维护，而不是用户自己定义

使用`kubectl explain`来查看集群资源的定义方式，如`kubectl explain pod`, `kubectl explain pod.spec`等

使用`kubectl apply -f FILENAME`将资源定义文件提交到集群中，可以是创建资源或者更新资源

使用`kubectl delete -f FILENAME`来删除该资源