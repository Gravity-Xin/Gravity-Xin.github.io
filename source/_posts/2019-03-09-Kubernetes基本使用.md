---
title: Kubernetes基本使用
date: 2019-03-09 12:25:57
categories: 
- Docker/K8S
tags: 
- Kubernetes
---

在Kubernetes中，主要通过`Kubectl`命令行工具对集群中的资源进行增删改查的操作，集群中的资源包括`Pod`、`Service`、各种控制器、`Node`等等

- 集群资源类型
  - certificatesigningrequests (aka 'csr')  
  - clusterrolebindings  
  - clusterroles  
  - componentstatuses (aka 'cs')  
  - configmaps (aka 'cm')  
  - controllerrevisions  
  - cronjobs  
  - customresourcedefinition (aka 'crd') 
  - daemonsets (aka 'ds')  
  - deployments (aka 'deploy')  
  - endpoints (aka 'ep')  
  - events (aka 'ev')  
  - horizontalpodautoscalers (aka 'hpa')  
  - ingresses (aka 'ing')  
  - jobs  
  - limitranges (aka 'limits')  
  - namespaces (aka 'ns')  
  - networkpolicies (aka 'netpol')  
  - nodes (aka 'no')  
  - persistentvolumeclaims (aka 'pvc')  
  - persistentvolumes (aka 'pv')  
  - poddisruptionbudgets (aka 'pdb')  
  - podpreset  
  - pods (aka 'po')  
  - podsecuritypolicies (aka 'psp')  
  - podtemplates  
  - replicasets (aka 'rs')  
  - replicationcontrollers (aka 'rc')  
  - resourcequotas (aka 'quota')  
  - rolebindings  
  - roles  
  - secrets  
  - serviceaccounts (aka 'sa')  
  - services (aka 'svc')  
  - statefulsets (aka 'sts')  
  - storageclasses (aka 'sc')error: Required resource not specified.

- `Kubectl`的使用方式
  - 入门命令
    - `create`: 创建集群资源
    - `expose`: 使用`Pod`、`Service`或各种控制器来创建`Service`资源
    - `run`: 在集群上运行特定镜像，创建Pod资源
    - `set`: 给集群中的特定资源进行配置
  - 中级命令
    - `get`: 查看集群中的各类资源的状态
    - `explain`: 查看资源中各类资源的说明
    - `edit`: 对集群中的资源进行编辑
    - `delete`: 对集群中的资源进行删除
  - 部署管理
    - `rollout`: 对资源的部署进行管理，比如暂停、恢复、回滚等
    - `rolling-update`: 对资源的部署进行滚动更新
    - `scale`: 手动对资源进行扩容或缩容
    - `autoscale`: 设置资源的自动水平伸缩
  - 集群管理
    - `certificate`: 修改集群认证信息
    - `cluster-info`: 查询集群信息
    - `top`: 查看集群中资源使用情况，如CPU、内存、磁盘等
    - `cordon`: 标记集群中某节点不可被调度
    - `uncordon`: 标记集群中某节点可以被调度
    - `drain`: 将集群中某节点上运行的资源赶到其它节点上
    - `taint`: 将集群中的某节点增加污点，与`Pod`的`Toleration`来配合使用
  - 故障排除和调试
    - `describe`: 显示集群某资源的详细信息
    - `logs`: 查看集群中某个`Pod`上的日志
    - `attach`: 附加到某个`Pod`中
    - `exec`: 在某个`Pod`中运行命令
    - `port-forward`: 设置某个`Pod`的端口和本地端口的映射
    - `proxy`: 在本地启动对集群API Server的代理
    - `cp`: 在`Pod`和本地之间进行文件拷贝
    - `auth`: 进行操作权限的校验
  - 高级命令
    - `apply`: 对集群中的资源进行配置
    - `patch`: 更新集群中的资源配置
    - `replace`: 对集群中的资源进行替换
    - `convert`: 将配置文件转换为不同的API版本
  - 资源管理
    - `label`: 给资源添加`Label`
    - `annotate`: 给资源添加`Annotation`
    - `completion`: 输出不同Shell的自动补全配置
  - 其它
    - `api-versions`: 输出API版本
    - `api-resources`: 输出集群支持的资源信息
    - `config`: 修改`Kubectl`使用的kubeconfig文件
    - `plugin`: 给集群添加插件
    - `version`: 输出客户端和服务端的版本信息

`kubectl get componentstatus`: 获取Master节点上各个组件的运行状态

`kubectl get namespaces`: 获取集群中的名称空间信息

`kubectl get nodes`: 获取集群Node节点信息

`kubectl get pods`: 获取集群中的Pod信息
