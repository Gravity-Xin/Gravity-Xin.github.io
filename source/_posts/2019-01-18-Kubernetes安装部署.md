---
title: Kubernetes安装部署
date: 2019-01-18 17:15:06
categories: 
- Docker/K8S
tags: 
- Kubernetes
---

K8S的安装方式:

- 使用`Minikube`安装K8S单机环境
- 使用二进制安装
  - 将Master节点上的组件(`API Server`、`Controller Manager`、`Scheduler`、`ETCD`等)直接以二进制的形式安装在主机上
  - Node节点上的组件(`Kubelet`、`Docker`、`Kube-Proxy`等)直接以二进制的形式安装在主机上
  - 这是一种传统方式，所有K8S组件都是系统级别的守护进
  - 该方式比较麻烦，因为需要手动去安装和配置各个组件，比如配置节点间安全通信的证书
  - 在该方式下，如果某些K8S组件出现故障，需要手动去进行管理和维护，因此不推荐该方式
[参考](https://github.com/opsnull/follow-me-install-kubernetes-cluster)
- 使用`Kubeadm`自动化工具安装
  - 内部调用`Kubectl`等命令
  - 在所有主机上安装`Docker`、`Kubelet`和`Kubeadm`组件，提供主机上运行容器和Pod的基础环境
  - 将某台主机使用`Kubeadm init`初始化为Master节点
  - 在其他主机上使用`Kubeadm join`来进入到K8S集群中，作为Node节点
  - 在Master节点上以Pod的形式在容器中运行组件(`API Server`、`Controller Manager`、`Scheduler`、`ETCD`等)
  - 在Node节点上以Pod的形式在容器中运行组件(`Kube-Proxy`等)
  - K8S所有组件以Pod的方式来运行，这些Pod为静态Pod，由`Kubelet`进行管理，仅存于特定Node上，不能通过API Server进行管理
  - 在所有主机上以Pod的方式来运行`Flannel`，为Pod提供IP地址分配的功能，该Pod由K8S来进行管理
[参考](https://github.com/kubernetes/kubeadm/blob/master/docs/design/design_v1.10.md)