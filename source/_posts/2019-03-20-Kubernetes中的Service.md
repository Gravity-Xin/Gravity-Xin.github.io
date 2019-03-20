---
title: Kubernetes中的Service
date: 2019-03-20 14:20:12
categories: 
- Docker/K8S
tags: 
- Service
---

Service: 是一组Pod的逻辑集合和访问方式的抽象

当Service或者Pod发生变化时:

- ServiceController会基于云提供商的负载均衡API来创建Serivice的负载均衡器；EndpointController会维护Service对象对应的Pod列表、创建Pod列表对应的Endpoint列表、将Endpoint列表和Service关联  **Service->Endpoints->Pods**
- 每一个Node节点上的kube-proxy会自动改变节点上iptables或ipvs中的规则，从而维护Service对象和一组Pod之间的关系

kube-proxy对Service的代理有三种模式:

- `userspace`: 运行在用户空间

![img](/images/Kubernetes之Service的userspace代理.png)

- `iptables`: 运行在内核空间

![img](/images/Kubernetes之Service的iptables代理.png)

- `ipvs`: 运行在内核空间 **默认模式**

![img](/images/Kubernetes之Service的ipvs代理.png)

## Service资源的spec字段

- `clusterIP`: Service在集群中的地址，一般由系统随机指定，当Service类型不为ExternalName才有用。当用户将其手动指定为None时，Service变为HeadLess Service，对Service名称的DNS解析不再返回clusterIP，而是返回Service管理的一组Pod的IP地址
- `externalName`: 当Service的类型为ExternalName时，Service对应的外部服务地址 **通过DNS将Service地址解析为externalName**
- `ports`: Service暴露的端口信息列表
  - `name`: 端口名称
  - `nodePort`: 当Service类型为NodePort或LoadBalancer时，Service映射到Node节点上的端口
  - `port`: Service对外提供服务的端口
  - `targetPort`: Service管理的Pod中容器的端口
- `selector`: 标签选择器，用来过滤出自己所管理的Pod
- `type`: Service的类型
  - `ClusterIP`: 使用集群中的私有地址，此时Service只能被Node上的Pod访问
  - `NodePort`: 除了使用ClusterIP之外，也将Service的端口映射到每一个Node的端口上。这样便可以在集群外部通过访问Node的端口来访问Pod **比ClusterIP多一层数据转发**
  - `LoadBalancer`: 使用云提供商的负载均衡API来创建Service的负责均衡器，然后访问ClusterIP或NodePort上的Service资源 **多了一层负载均衡**
  - `ExternalName`: 为Pod提供了访问集群外部服务的能力。通过CNAME将某个Service和ExternalName进行绑定，Pod便可以通过访问该Service来访问集群外部的服务 **本质是将Service地址通过DNS解析为集群外部服务的地址**
- `sessionAffinity`: 对于Service的请求被转发到Service对应的Pod列表时使用的策略
  - ClientIP: 同一个客户端的请求总是被转发到同一个Pod中
  - None: 随机选择一个Pod

## Service资源的使用

Service资源被创建后，在集群的DNS中便存在对应的Service资源和对应ClusterIP的记录

Service资源格式: `SVC_NAME.NS_NAME.svc.cluster.local`