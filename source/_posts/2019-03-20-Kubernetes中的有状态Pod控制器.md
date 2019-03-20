---
title: Kubernetes中的有状态Pod控制器
date: 2019-03-20 14:18:32
categories: 
- Docker/K8S
tags: 
- StatefulSet
---

有状态Pod的控制器: 每一个Pod会被单独进行管理

- StatefulSet
  - 功能
    - 对有状态服务的Pod进行控制管理，每一个Pod都会被单独管理。当Pod出现故障时，需要自定义操作进行恢复
  - spec字段
    - `revisionHistoryLimit`: Deployment可以保留的历史ReplicaSet数量，从而支持版本回滚
    - `podManagementPolicy`: Pod在初始化的启动策略，OrderedReady、Parallel
    - `replica`:
    - `selector`:
    - `serviceName`:
    - `template`:
    - `updateStrategy`:
    - `volumeClaimTemplates`:
