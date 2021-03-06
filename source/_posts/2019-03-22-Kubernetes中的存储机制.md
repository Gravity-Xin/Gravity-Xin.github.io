---
title: Kubernetes中的存储机制
date: 2019-03-22 10:08:15
categories: 
- 系统架构
tags: 
- Volume
- EmptyDir
- HostPath
- ConfigMap
- Secret
- PersistentVolume
- PersistentVolumeClaim
---

Docker中提供了存储卷的方式来实现容器内数据持久化的功能，本质是将宿主机上的文件和容器中的文件进行绑定

在K8S中，Pod是可被调度的最小单元，有可能会被调度到不同的Node节点上运行，因此Docker中提供的这种方式无法满足Pod在跨节点调度时的数据持久化功能

为了给Pod提供跨Node节点的数据持久化机制，需要将传统网络存储、分布式存储或云存储挂载到Node节点上，然后将存储卷和Node节点上的挂载目录进行绑定

在K8S的Pod的容器中使用存储卷的步骤:

- 在Pod资源定义中使用`volumes`来定义Pod需要的存储卷
  - 使用节点本地存储，只可以在Node节点本地使用
    - `emptyDir`: 可以被Pod访问的临时空间，可以基于磁盘或内存。当Pod删除时，该存储卷也会被删除，即该数据卷的生命周期和Pod相同
      - `medium`: 定义存储卷的存储方式。为空表示使用磁盘存储，Memory表示使用内存存储
      - `sizeLimit`: 定义存储卷的容量
    - `hostPath`: 使用Node节点上的文件路径作为存储卷。当Pod删除时，该存储卷不会被删除，可以看作是Node节点级别的持久化机制  **Docker上的存储卷机制**
      - `path`: Node节点上的路径
      - `type`: 数据卷的类型
        - `DirectoryOrCreate`: 路径为目录，如果目录不存在则自动创建
        - `Directory`: 路径为目录，如果目录不存在则报错
        - `FileOrCreate`: 路径为文件，如果目录不存在则自动创建
        - `File`: 路径为文件，如果目录不存在则报错
        - `Socket`: 路径为Unix Socket文件，如果不存在则报错ß
        - `CharDevice`: 路径为字符设备，如果目录不存在则报错
        - `BlockDevice`: 路径为块设备，如果目录不存在则报错
    - `configMap`: 使用ConfigMap来创建存储卷。主要目的不是给Pod提供存储空间，而是向Pod中提供配置信息
      - `defaultMode`: 配置文件的默认权限
      - `items`: 指定从ConfigMap中要挂载的键列表
      - `name`: 使用的ConfigMap资源名称
      - `optional`: 设置对应的ConfigMap和其中的Key是否必须被定义
    - `secret`: 使用Secret来创建存储卷。主要目的不是给Pod提供存储空间，而是向Pod中提供配置信息
      - `defaultMode`: 配置文件的默认权限
      - `items`: 指定要提供的配置键列表
      - `secretName`: 使用的Secret资源名称
      - `optional`: 设置对应的Secret和其中的配置是否必须被定义
    - `downwardAPI`: 使得一些向下API可以被Pod使用。挂载一个目录，并将请求的数据写入到纯文本文件中
  - 使用传统网络存储、分布式存储或云存储等，可以跨Node节点使用，是真正的持久化。可以直接根据底层存储方式来指定Pod使用的存储卷，如ceph、glusterfs等，也可以通过PV对底层存储进行抽象，通过PVC向PV申请Pod使用的存储卷
    - `awsElasticBlockStore`: AWS云存储
    - `azureDisk`: Azure云存储
    - `azureFile`: Azure云存储
    - `cephfs`: Ceph分布式存储
    - `cinder`: OpenStack云存储
    - `fc`: SAN网络存储
    - `flexVolume`: 存储服务
    - `flocker`: 存储服务
    - `gcePersistentDisk`: Google云存储
    - `gitRepo`: git仓库
      - 建立在emptyDir之上，只是数据卷中的数据初始化为git仓库中的数据
      - git仓库和数据卷中的数据不会自动同步，即不会自动执行git pull和git push的操作
    - `glusterfs`: glusterfs分布式存储
    - `iscsi`: SAN网络存储
    - `nfs`: NFS网络存储
      - `path`: NFS服务器上的导出路径
      - `readOnly`: 对NFS上的文件是否仅可读
      - `server`: NFS服务器地址
    - `persistentVolumeClaim`: 使用PVC向PV申请存储资源
      - `claimName`: PVC资源名称
      - `readOnly`: 申请的存储资源是否仅可读
    - `photonPersistentDisk`
    - `portworxVolume`
    - `projected`: 将几个现有的存储卷映射到同一个目录中
    - `quobyte`
    - `rbd`: Ceph分布式块存储
    - `scaleIO`
    - `storageos`: 存储操作系统，封装了多个云存储
    - `vsphereVolume`: vSphere云存储
- 在Pod资源的容器定义中使用`volumeMounts`定义容器中存储卷的挂载方式
  - `mountPath`: 存储卷在容器中的挂载路径
  - `mountPropagation`: 挂载的存储卷的传播方式 None HostToContainer BiDirection
  - `name`: 挂载的存储卷的名称
  - `readOnly`: 存储卷是否只可读
  - `subPath`: 从存储卷中进行挂载的子目录

## ConfigMap

ConfigMap是Namespace级别的资源，可以被同一个Namespace中的Pod使用

提供了从外部向Pod提供配置信息的功能，且这些信息是明文存储的，可以将Pod和其运行时使用的配置进行分离

使用方式:

- 设置Pod的容器中环境变量的值，基于`spec.containers.env`字段；基于`spec.containers.args`字段来引用环境变量，还可以设置Pod的容器中的命令行参数  **通过环境变量来引用ConfigMap时仅仅在容器创建时注入配置，当ConfigMap中的配置发生变化时，容器中的环境变量值不会随之动态改变**
- 在Pod中使用其创建存储卷，将ConfigMap中的配置变为一个个的文件，然后被Pod中的容器挂载和使用  **通过存储卷来引用ConfigMap时，当ConfigMap中的配置发生变化时，容器中的配置文件会动态改变**

spec字段:

- `binaryData`: 键值对，值为二进制的数据
- `data`: 键值对，值为UTF-8编码的数据

也可以通过`kubectl create configmap`命令从配置文件或键值对中创建

## Secret

Secret是Namespace级别的资源，可以被同一个Namespace中的Pod使用

提供了从外部向Pod提供配置信息的功能，且这些信息使用Base64编码存储的，可以将Pod和其运行时使用的配置进行分离

使用方式: 和ConfigMap类似

spec字段:

- `data`: 键值对，值为任意类型，将会使用Base64进行编码
- `stringData`在: 键值对，值为字符串类型，将会使用Base64进行编码
- `type`: 类型

也可以通过`kubectl create secret`命令从配置文件或键值对来创建

- docker-registry: 私有Docker Registry的访问配置
- generic: 通用类型，比如密码
- tls: TLS配置，私钥和证书

## PersistentVolume/PersistentVolumeClaim

通过PV对集群中的各种存储系统进行封装，Pod通过PVC从PV中申请存储资源，这样便实现了Pod和存储系统的解耦

底层存储系统和PV是一对多的绑定关系

PV和PVC是一对一的绑定关系

PVC和Pod是一对多的绑定关系

PV含有两种创建方式: 静态创建和基于StorageClass动态创建。对于后者来说，PV不再直接向底层存储系统申请存储资源，而是通过StorageClass按需申请

StorageClass也是集群资源，不属于任何Namespace。用于对底层存储系统进行分类，比如按照读写速度、容量大小、收费模式等

StorageClass要求底层存储系统具备Restful风格的请求接口，它将使用该接口来进行底层存储系统的存储资源申请、释放等操作

![img](/images/Kubernetes之PV和PVC.png)

![img](/images/Kubernetes之PV和PVC的使用.png)

![img](/images/Kubernetes之StorageClass.png)

### PV的spec字段

PV是集群级别的资源，不属于任何Namespace

对集群中的各类存储系统进行管理，同时向PVC提供存储资源的访问入口

- 使用的底层存储系统配置: 网络存储、分布式存储、云存储等等，与直接在Pod中定义存储卷类似
- `accessModes`: PV可以被Node节点的挂载方式和使用方式 **还需要保证底层的存储系统支持该模式**
  - `ReadWriteOnce`: 只可以被单个节点挂载，且可读可写
  - `ReadOnlyMany`: 可以被多个节点挂载，且只读
  - `ReadWriteMany`: 可以被多个节点挂载，且可读可写
- `capacity`: 存储资源容量设置
- `claimRef`: PV和PVC之间进行绑定的配置
- `mountOptions`: 存储资源进行挂载操作时的选项
- `nodeAffinity`: 对PV能够被挂载到的Node节点进行限制
- `persistentVolumeReclaimPolicy`: 当PV和PVC之间分离之后的回收策略
  - `Retain`: 保留PV，保留数据
  - `Delete`: 删除PV，清空数据
  - `Recycle`: 保留PV，PV可被重新绑定，清空数据
- `storageClassName`: PV所属的存储类名称
- `volumeMode`: 存储资源的文件系统类型

### PVC的spec的字段

PVC是Namespace级别的资源

对于Pod中需要如何使用存储资源和如何寻找合适的PV进行描述

当PVC被创建之后，会自动从集群中寻找符合条件的PV资源并进行绑定

- `accessModes`: 规定PV应支持的访问模式
- `resources`: 规定PV中应包含的资源
  - `limits`: 设置最大的PV存储空间
  - `requests`: 设置最小的PV存储空间
- `selector`: 使用标签选择器来匹配可以被选择的PV资源
- `storageClassName`: 存储类名称
- `volumeMode`: 存储卷的模式，用于和PV进行绑定
- `volumeName`: 存储卷的名称，用于和PV进行绑定