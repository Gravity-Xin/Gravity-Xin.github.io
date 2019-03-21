---
title: Kubernetes网络基础
date: 2019-03-08 15:57:50
categories: 
- 系统架构
tags: 
- Node
- Service
- Pod
---

- 三种网络
  - 节点网络: 所有Node组成的网络，是真实有效的IP地址，与物理网卡或虚拟网卡绑定
  - 集群网络: 所有Service组成的网络，是虚拟的IP地址，不与物理网卡或虚拟网卡绑定。通过DNS对Service名称进行解析，通过在iptables、ipvs中添加规则实现对Pod的访问
  - Pod网络: 所有Pod组成的网络，是真是有效的IP地址，与虚拟网卡绑定

三者处于不同的网段中，外部的访问分别经过节点网络、集群网络和Pod网络  **所有的网络相关，其本质上都是iptables或者ipvs中的规则在起作用，以及dns解析在起作用**

![img](/images/Kubernetes之三种网络.png)

- 三种通信
  - Pod内部的多个容器间使用联盟式网络，共享网络设备，可以直接通过localhost进行访问
  - Pod与Pod的容器之间
    - 两个Pod如果位于同一个Node上，容器间可以利用`docker0`网桥相连接进行通信
    - 两个Pod如果在不同Node上，此时容器分别被`docker0`网桥隐藏，无法直接连接，此时需要Flannel来接管Pod的IP分配，以Overlay Network叠加式网络的方式进行连接
  - Pod与Service之间，由于Service在K8S集群中有独立的IP地址和DNS名称，可以通过其与Service直接进行通信

- 网络配置和网络策略: 提供了`CNI`容器网络接口规范，可以将第三方插件纳入到集群中
  - 网络配置: Pod和Service的IP分配
  - 网络策略: Pod之间是否可以互相访问