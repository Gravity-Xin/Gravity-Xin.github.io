---
title: Kubernetes中的Pod
date: 2019-03-19 13:43:17
categories: 
- Docker/K8S
tags: 
- Pod
---

Pod: Kubernetes中的最小调度单元，可以包含一个或多个容器

## Pod资源的spec字段

- `containers`: Pod使用的容器列表，一个Pod中可以含有多个容器
  - `args`: 容器启动的参数列表，如果不指定则使用镜像中的CMD参数
  - `command`: 容器启动的命令列表，如果不指定则使用镜像的ENTRYPOINT参数；该命令不会运行在shell中
  - `env`: 容器中的环境变量
  - `name`: 容器名称
  - `image`: 容器使用的镜像
  - `imagePullPolicy`: 拉取镜像的方式，可选值为Always、Never、IfNotPresent
  - `lifecycle`: 设置容器在启动后和结束前的钩子操作，包括ExecAction执行自定义命令、TCPSocketAction向指定的TCP端口发送请求、HTTPGetAction向指定的HTTP服务发送请求
    - `postStart`
    - `preStop`
  - `livenessProbe`: 容器存活的探测方式，包括ExecAction执行自定义命令、TCPSocketAction向指定的TCP端口发送请求、HTTPGetAction向指定的HTTP服务发送请求
  - `readinessProbe`: 容器启动后是否就绪的探测方式，只有当容器就绪之后才会被加入到对应Service的EndPoints中，被外部访问，支持的探测方式和livenessProbe相同
  - `ports`: 需要从容器中暴露的端口列表，仅仅是给系统提供关于容器端口使用的信息。如果不指定，并不会阻止容器中的进程使用的端口被暴露
  - `resources`: 容器需要使用的系统资源列表
  - `volumeMounts`: 存储卷在容器中的挂载点
- `hostIPC`: 设置Pod中的容器是否与共享Node节点的IPC名称空间
- `hostNetwork`: 设置Pod中的容器是否与共享Node节点的Network名称空间，此时可以直接访问Node地址来直接访问Pod，而不再需要通过Service将Pod暴露
- `hostPID`: 设置Pod中的容器是否与共享Node节点的PID名称空间
- `nodeName`: 节点名称，直接指定Pod运行的节点名称
- `nodeSelector`: 节点选择器，被调度器使用，用于选择符合条件的Node节点
- `initContainers`: 串行执行的初始化容器列表，一般用于给主容器初始化系统环境，在主容器启动前退出
- `restartPolicy`: 当容器存活探测失败时Pod的重启策略，Always、OnFailure、Never
- `tolerations`: Pod的容忍度列表，被调度器使用，用于选择符合条件的Node节点
- `volumes`: Pod使用的存储卷列表
- `terminationGracePeriodSeconds`: Pod终止时，会向容器中的进程发送Term信号。在等待设置的时间后，再向容器中的进程发送Kill信号。因此容器响应Term信号的时间应小于该值，从而实现平滑终止

## Pod资源的生命周期

- `Pending`: Pod已被系统接受，但是容器镜像尚未创建。包括Pod调度到节点Node上和通过网络下载镜像的时间
- `Running`: Pod已经和某个Node绑定，并且Pod中的容器都已被创建，至少有一个容器处于启动、运行或重启的状态
  - 使用串行执行的初始化容器执行系统环境初始化操作
  - 启动主容器和其他辅助的容器
  - 在主容器启动后执行postStart钩子操作
  - 执行readinessProbe和livenessProbe探测操作，如果存活探测失败则根据restartPolicy规则来决定是否重启Pod
  - 在主容器结束前执行preStop钩子操作
- `Succeeded`: Pod中的容器已经成功终止且不再会被重启
- `Failed`: Pod所有容器已经终止且至少有一个容器是因为失败而终止，即进程以非0状态退出或者被操作系统终止，此时根据restartPolicy规则来决定是否重启Pod
- `Unknown`: 由于某些原因Master暂时无法获取到Pod的状态，比如网络通信失败等

![img](/images/Kubernetes之Pod生命周期.png)