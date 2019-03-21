---
title: Kubernetes中的无状态Pod控制器
date: 2019-03-19 17:52:56
categories: 
- 系统架构
tags: 
- ReplicaSet
- Deployment
- DaemonSet
- Job
- CronJob
---

无状态Pod的控制器: 将一组Pod看做一个整体，不关注单个Pod

- ReplicaSet
  - 功能
    - 维护一组Pod的持续运行并保证一定数量的Pod能够在集群中正常运行。会持续监视所管理的Pod的状态，当Pod发生故障而数量减少时会启动Pod的副本
    - 支持水平扩缩容，当replicas字段值发生变化后会自动调整所管理的Pod副本数量
    - 不支持自动滚动更新和版本回滚，当template中的镜像发生变化后不会立即使用新镜像来启动新的Pod副本，只有当Pod发生故障或被手动删除而需要重启时，才会使用新的镜像
  - spec字段
    - `replicas`: Pod的副本数量
    - `selector`: 标签选择器，用来过滤出自己所管理的Pod
      - `matchLabels`: 直接给定标签的KV对
      - `matchExpressions`: 基于给定的表达式来定义标签选择器
    - `template`: 当控制器需要创建Pod副本时使用的模板
      - `metadata`: 所管理的Pod的元数据信息
      - `spec`: 所管理的Pod的spec信息

- Deployment
  - 功能
    - 基于ReplicaSet之上，在创建Deployment或者更新Deployment配置时会自动创建对应的ReplicaSet
    - 可以同时管理多个ReplicaSet，并且只有一个ReplicaSet处于激活状态，从而能够方便提供滚动更新、版本回滚、水平扩缩容等功能
    - 在进行滚动更新时，可以设置更新的粒度，即每次更新和删除Pod的数量
    - 当replicas或template等字段发生变化时，自动按照设置的更新策略进行更新，不需要手动操作；同时变更的详细信息会被自动添加到Deployment的Annotations中
    - `kubectl rollout`
      - `history`: 查看Deployment中ReplicaSet的历史版本
      - `undo`: 进行Deployment历史ReplicaSet的版本回滚
      - `pause`: 将Deployment控制器暂停
      - `resume`: 将Deployment从暂停中恢复运行
      - `status`: 查看Deployment进行更新的过程
  - spec字段: 除了ReplicaSet的spec字段之外，还包括
    - `paused`: 设置Deployment控制器是否暂停
    - `revisionHistoryLimit`: Deployment可以保留的历史ReplicaSet数量，从而支持版本回滚
    - `strategy`: 设置使用新Pod替换旧Pod的策略，可以为Recreate、RollingUpdate

![img](/images/Kubernetes之Deployment与ReplicaSet.png)

- DaemonSet
  - 功能
    - 在集群的所有Node节点或者符合Node选择器要求的节点上部署基础服务和守护进程，并且确保某种Pod在节点上只运行一个
    - 支持滚动更新、版本回滚等操作
  - spec字段
    - `revisionHistoryLimit`: DaemonSet可以保留的历史版本数量，从而支持版本回滚
    - `selector`: 标签选择器，用来过滤出自己所管理的Pod
    - `template`: 当控制器需要创建Pod副本时使用的模板
    - `updateStrategy`: 设置使用新Pod替换旧Pod的策略，可以为RollingUpdate或OnDelete

- Job
  - 功能
    - 对一次性任务进行管理，而不是持续运行的守护进程。可以创建并保证一定数量的Pod成功停止
  - spec字段
    - `selector`: 标签选择器，用来过滤出自己所管理的Pod
    - `template`: 当控制器需要创建Pod副本时使用的模板
    - `completions`: 任务对应的需要成功完成的Pod数量
    - `parallelism`: 任务对应的Pod可以并行运行的数量

- CronJob
  - 功能: 用于对周期性的、不需要持续运行的任务进行管理