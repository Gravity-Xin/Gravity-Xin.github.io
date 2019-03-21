---
title: Kubernetes中的Ingress和IngressController
date: 2019-03-20 23:21:15
categories: 
- 系统架构
tags: 
- Ingress
- IngressController
---

Ingress: 为多个Service提供代理，可以根据访问路径或子域名将请求转发到不同的Service中

- Service可以看作是一组Pod的L4级别的代理和负载均衡，因为其基于的iptables和ipvs都是L4级别的机制
- Ingress可以看做是一组Service的L7级别的反向代理和负载均衡，其工作在L7级别
- 使用Service可以保证一组Pod使用相同的L4协议，比如TCP、UPD
- 使用Ingress可以保证一组Service使用相同的L7协议，比如HTTP、HTTPs

IngressController: 本质是一个工作在L7的应用服务，它可以对集群中定义的Ingress资源进行管理并实现Ingress的L7代理功能

- 可以部署为Deployment并使用类型为NodePort的Service对其进行管理；也可以部署为DaemonSet并且共享Node节点网络名称空间

- 当Ingress所管理的Service对应的Pod发生变化时，IngressController会自动更新相应的配置，确保可以正确进行代理，不需要手动修改配置

- 主流的IngressController实现方式
  - Nginx
  - Traefik
  - Envoy
    - Contour
    - Istio

![img](/images/Kubernetes之Ingress与IngressController.png)

## Ingress资源的spec字段

- `backend`: 如果没有找到对应的Service，默认将请求发送到该后端
  - `serviceName`: Service名称
  - `servicePort`: Service的端口
- `rules`: 设置Ingress与Service的关联规则
  - `host`: 定义域名，需要保证该域名可以被解析到IngressController所在Node节点的IP **虚拟主机**
  - `http`: 定义URL和Service的关联规则  **URL映射**
    - `paths`: 定义将请求映射到对应Service的规则
      - `path`: URL路径
      - `backend`: 该URL对应的Service
        - `serviceName`: Service名称
        - `servicePort`: Service的端口
- `tls`: 当需要向用户使用HTTPs协议时，指定TLS配置；该TLS配置工作在IngressController中，而不是后端的Service对应的Pod中
  - `hosts`: 需要使用TLS配置的域名列表
  - `secretName`: Secret资源名称，该资源封装了TLS证书和私钥

## Ingress资源的使用

- 安装部署IngressController [Nginx](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)
- 定义Ingress需要代理的一组Service
- 定义Ingress，将一组Service和Ingress进行关联