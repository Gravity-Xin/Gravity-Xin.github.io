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
- `准入控制`: 确保该操作对应的后续相关安全检查可以通过

这三个组件都是模块化设计的，在集群中可以有多个插件来实现。当系统中的认证、权限检查或准入控制分别存在多个插件时，用户只需要通过任意一种插件的校验即可

## 账号认证

### User Account（用户账号）

当使用`kubectl`时，会使用kubeconfig文件提供的用户账号信息来访问多个集群的API Server，其文件内容为

- `clusters`: 集群信息
- `users`: 用户账号信息
- `contexts`: 用户账号和集群的对应关系信息

可以使用`kubectl config`命令来查看和修改kubeconfig文件

外部用户可以通过HTTP请求的URL格式来确定操作的资源对象

`apis/apps/v1/namespaces/default/deployments/myapp-deploy/`

- `apps/v1`: 资源的API群组和版本，通过`kubectl api-versions`来查看系统支持的API群组
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

在Pod`spec.serviceAccountName`中指定使用的ServiceAccount资源名称。如果不指定，每一个Pod默认使用名称为`default`的ServiceAccount，该账号只具备通过API Server访问自身信息的权限

ServiceAccount的token信息使用Secret存储卷来存储。当Pod启动时，其使用的ServiceAccount资源对应的Secret存储卷会被挂载到Pod中

创建ServiceAccount资源：

- 通过`secrets`定义关联的Secret存储卷
- 通过`imagePullSecrets`定义拉取镜像的私有Registry认证配置

## 权限检查

使用RBAC来对用户的资源操作进行权限检查，所谓的权限就是指**对某种资源是否具备某种操作的资格**

### Role和ClusterRole(角色定义)

Role: 属于某个Namespace，表示某一Namespace中的角色。其中包含的权限对应的是该Namespace中的资源
ClusterRole: 不属于某个Namespace，而是属于集群。其中包含的资源可以为集群范围内的资源

Role和ClusterRole都是K8S资源对象，可以通过`rules`字段来定义角色的权限

- `apiGroups`: 规定资源所属的API群组和版本
- `nonResourceURLs`: 规定对非资源对象的URL请求
- `resourceNames`: 规定具体的资源名称
- `resources`: 规定资源类型列表
- `verbs`: 规定对资源的操作，如"get", "list", "watch", "create", "update", "patch", "delete"

### RoleBinding和ClusterRoleBinding(角色绑定)

一般是使用RoleBinding将Role和用户进行绑定，使用ClusterRoleBinding将ClusterRole和用户进行绑定

使用RoleBinding将ClusterRole和用户进行绑定时，资源访问权限仍然局限在RoleBinding所在的Namespace中 **在不同的命名空间中复用通用角色**

无法使用ClusterRoleBinding将Role和用户进行绑定

RoleBinding和ClusterRoleBinding都是K8S资源对象，可以通过`roleRef`和`subjects`来定义

- `roleRef`: 规定使用的角色
  - `apiGroup`: 该角色所属于的API群组
  - `kind`: 该角色的类型 Role或ClusterRole
  - `name`: 角色的名称
- `subjects`: 规定被绑定的用户
  - `apiGroup`: 该用户所属于的API群组
  - `kind`: 该用户的类型 User、Group或ServiceAccount
  - `name`: 用户的名称
  - `namespace`: 当用户为ServiceAccount类型时，ServiceAccount所在的Namespace

K8S中默认有名称为`admin`的ClusterRole，可以看做是管理员权限，可以通过RoleBinding将其与用户绑定，从而让用户具备某个Namespace下的管理员权限
K8S中默认有名称为`cluster-admin`的ClusterRole，该权限为对任意资源可进行任意操作，权限最大 **Role、ClusterRole、RoleBinding和ClusterRoleBinding都是集群资源，可以通过kubectl来操作这些资源也是因为kubeconfig中的用户具备这样的权限**
K8S中默认有名称为`cluster-admin`的ClusterRoleb，将`cluster-admin`的ClusterRole绑定到了名称为`system:masters`的Group