---
title: Kubernetes中的RBAC和ServiceAccount
date: 2019-04-02 19:20:07
categories: 
- 系统架构
tags: 
- Role
- ServiceAccount
---

API Server是集群资源操作的入口

- 集群外部的用户可以通过kubectl、http或https接口向其发送操作资源的请求 **kubectl本质上也是发起http请求**
- 集群内部的Pod，如DNS、Dashboard等，也会访问API Server实现资源的操作。此时API Server使用Service进行暴露，从而能够被集群中的Pod访问

API Server对外部请求的校验包含三个部分:

- `认证`: 确保为合法用户，可以是UserAccount或ServiceAccount，具体实现包括基于Token的令牌认证、基于证书的SSL认证等
- `权限检查`: 确保用户对资源的操作可以被执行，具体实现包括ABAC、RBAC、WebHook等
- `准入控制`: 确保后续的相关安全检查可以通过

这三个组件都是模块化设计的，在集群中可以有多个插件来实现。当系统中的认证、权限检查或准入控制分别存在多个插件时，用户只需要通过任意一种插件的校验即可

## 账号认证

### User Account（用户账号）

外部用户可以通过HTTP请求的URL格式来确定操作的资源对象

`apis/apps/v1/namespaces/default/deployments/myapp-deploy/`

- `apps/v1`: 资源的API群组
- `namespaces/default`: 名称空间名称
- `deployment/myapp-deploy`: 资源类型和名称，子资源类型和名称

通过HTTP方法来确定对资源的操作类型

- `GET`: 查看单个或列出多个资源信息，监视单个或多个资源状态变化
- `POST`: 创建或更新资源信息
- `PUT`: 更新资源信息
- `DELETE`: 删除单个或多个资源

可以通过`kubectl proxy`在本地为API Serve提供代理，然后通过curl命令操作集群资源

### Service Account（服务账号）

该资源属于Namespace，可以看做是一个用token表示、用于访问APIServer的服务账号。它用来保证集群中的Pod对API Server的访问

在Pod`spec.serviceAccountName`中指定使用的ServiceAccount资源名称

ServiceAccount的token信息使用Secret存储卷来存储。当Pod启动时，其使用的ServiceAccount资源对应的Secret存储卷会被挂载到Pod中

创建ServiceAccount资源：

- 通过`secrets`定义关联的Secret存储卷
- 通过`imagePullSecrets`定义拉取镜像的私有Registry认证配置

## 授权

基于RBAC