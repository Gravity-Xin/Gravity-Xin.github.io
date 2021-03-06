---
title: Kubernetes基本使用
date: 2019-03-09 12:25:57
categories: 
- 系统架构
tags: 
- Kubernetes
---

在Kubernetes中，主要通过Kubectl命令行工具对集群中的资源进行增删改查的操作，集群中的资源包括Pod、Service、各种控制器、Node等等

- `Kubectl`的使用方式
  - 入门命令
    - `create`: 创建集群资源
    - `expose`: 使用Pod、Service或各种控制器来创建Service资源，外部可以通过该Service实现对Pod的访问
    - `run`: 在集群上运行特定镜像，创建Pod资源并使用Deployment或Job来对其进行控制
    - `set`: 给集群中的特定资源进行配置，可以设置Pod中的环境变量、镜像名称、LabelSelector等
  - 中级命令
    - `get`: 查看集群中的各类资源的状态
      - `pod`: 查看Pod资源状态
      - `deployment`: 查看Deployment控制器状态
      - `service`: 查看Service资源状态
    - `explain`: 查看资源中各类资源的说明
    - `edit`: 对集群中的资源对应的yaml文件进行编辑，保存后自动生效
    - `delete`: 对集群中的资源进行删除
  - 部署管理
    - `rollout`: 对资源的更新部署进行管理，比如查看更新的状态，暂停和恢复更新，版本回滚等
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
    - `taint`: 将集群中的某节点增加污点，与Pod的Toleration来配合使用，实现高级的调度
  - 故障排除和调试
    - `describe`: 显示集群某资源的详细信息
      - `pod`: 查看Pod的详细信息
      - `service`: 查看Service的详细信息，以及Service对应Pod的EndPoints列表等
    - `logs`: 查看集群中某个Pod上的日志
    - `attach`: 附加到某个Pod中
    - `exec`: 在某个Pod中运行命令
    - `port-forward`: 设置某个Pod的端口和本地端口的映射
    - `proxy`: 在本地启动对集群API Server的代理
    - `cp`: 在`Pod`和本地之间进行文件拷贝
    - `auth`: 进行操作权限的校验
  - 高级命令
    - `apply`: 对集群中的资源进行配置
    - `patch`: 更新集群中的资源配置
    - `replace`: 对集群中的资源进行替换
    - `convert`: 将配置文件转换为不同的API版本
  - 资源管理
    - `label`: 给资源添加Label
    - `annotate`: 给资源添加Annotation
    - `completion`: 输出不同Shell的自动补全配置
  - 其它
    - `api-versions`: 输出集群支持的API群组
    - `api-resources`: 输出集群支持的资源信息
    - `config`: 修改Kubectl使用的kubeconfig文件
    - `plugin`: 给集群添加插件
    - `version`: 输出客户端和服务端的版本信息
    - `cluster_info`: 查看集群中Master上系统Service信息