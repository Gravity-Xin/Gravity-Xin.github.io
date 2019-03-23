---
title: Kubernetes中的有状态Pod控制器
date: 2019-03-20 14:18:32
categories: 
- 系统架构
tags: 
- StatefulSet
---

有状态Pod的控制器: 每一个Pod会被单独进行管理

- 有状态Pod的部署方式
  - 使用StatefulSet控制器来管理
  - 使用Headless Service来暴露
    - 普通的Service所管理的一组Pod是无序的，Headless Service会给所管理的每一个Pod赋予唯一标识，使得它们有序
    - 对Headless Service的名称DNS解析得到的直接是有状态Pod的IP地址列表
  - 使用VolumeClaimTemplates来创建存储卷
    - 有状态Pod中每一个Pod所使用的存储卷是不同的
    - 无法在有状态Pod中定义自己使用的存储卷，而是在StatefulSet中使用volumeClaimTemplates来定义

- StatefulSet
  - 功能
    - 对有状态服务的Pod进行控制管理，每一个Pod都会被单独管理
    - 支持扩缩容，当replicas字段值发生变化后会自动调整所管理的Pod副本数量
    - 支持滚动更新，通过partition值设置需要被更新的Pod列表
    - 支持版本回滚，使用方式和Deployment类似
  - 对Pod的要求
    - 每一个Pod具备稳定且唯一的网络标识符
    - 每一个Pod具备稳定且持久的存储
    - Pod支持有序、平滑地部署和扩展
    - Pod支持有序、平滑的删除和终止
    - Pod支持有序的滚动更新
  - spec字段
    - `revisionHistoryLimit`: StatefulSet可以保留的历史版本数量，从而支持版本回滚
    - `podManagementPolicy`: Pod在初始化的启动策略，OrderedReady、Parallel
    - `replica`: Pod的副本数
    - `selector`: Pod标签选择器，用来筛选符合条件的Pod列表
    - `serviceName`: 管理Pod的服务名称，同时自动给每一个Pod分配唯一标识符，可以被DNS解析 **POD_NAME.SVC_NAME.NS_NAME.svc.cluster.local**
    - `template`: 创建Pod的模板
    - `updateStrategy`: Pod的更新策略。使用滚动更新机制时，可以更新编号大于等于partition的Pod；通过不断减少partition的值为0，完成所有Pod的更新
    - `volumeClaimTemplates`: 在Pod中创建PVC的模板。该机制可以保证每一个Pod始终与相同的PVC进行绑定，当Pod重新创建后数据不会丢失