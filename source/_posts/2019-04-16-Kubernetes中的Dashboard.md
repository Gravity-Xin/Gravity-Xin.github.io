---
title: Kubernetes中的Dashboard
date: 2019-04-16 16:34:02
categories: 
- 系统架构
tags: 
- Dashboard
---

## 安装

- `kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml`
- 将Dashboard的Service配置修改为NodePort类型，从而可以通过NodeIP进行访问

## 登录

Dashboard以一个Pod的形式运行，它通过与API Server交互从而能够对Kubernetes集群的资源进行管理

因此必须给Dashboard提供一个ServiceAccount，从而才能访问API Server

- 基于Token:
  - 创建一个ServiceAccount，通过ClusterRoleBinding为其赋予权限
  - 通过`kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')`获取登录的Token

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

- 基于Kubeconfig文件:
  - 由于自带的Kubeconfig是UserAccount，而Dashboard必须使用ServiceAccount来登录
  - 同上，创建一个ServiceAccount，通过ClusterRoleBinding为其赋予权限
  - 通过`kubectl config set-credentials`将ServiceAccount生成Kubeconfig文件
  - 通过`kubectl config set-context`将ServiceAccount和集群绑定
  - 通过`kubectl config use-context`启用该ServiceAccount，此时该Kubeconfig文件可以被用来登录Dashboard
