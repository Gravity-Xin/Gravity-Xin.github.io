---
title: GlusterFS-入门
date: 2019-05-07 15:36:27
categories: 
- 系统架构
tags: 
- GlusterFS
- Node
- Brick
- Volume
---

## 基本特点

- 使用TCP/IP网络协议将物理上分布的存储资源聚集一起，形成统一的、可以弹性扩展的存储资源池
- 使用全局命名空间来管理数据存储池，对上层应用屏蔽了底层的物理存储
- 使用弹性哈希算法在存储池中定位数据，优化了数据分布，不再依赖元数据服务器，避免了单点故障，同时提高了可扩展性
- 高可用性，可以对文件自动进行复制和修复，从而保证数据的可用性
- 适合存储大文件，对于小文件性能较差

## 基本概念

- Brick
  - 基本的存储单元，是存储池中某个服务器上的一个导出目录
  - 通过`Node:/Export`来表示
  - 底层是RAID或磁盘经XFS、EXT文件系统
  - 每一个Node上面的Brick数量不限

![img](/images/GlusterFS之Brick.png)

- Volume
  - 一组Bricks的集合，每个Node上的不同Brick可以属于不同的Volume
  - 是一个可以在Client上挂载的目录
  - 分为以下几类
    - 分布式卷
      - 文件根据弹性哈希算法分布在不同的Brick中
      - 单个Brick失效会导致数据丢失
    - 复制卷
      - 自动复制卷中所有的目录和文件，副本数量一般为2或3
      - 当Node故障或Brick失效时数据仍可用
      - 需要通过事务保证数据的一致性
    - 条带卷
      - 文件切分为多个Chunk，存放在不同的Brick中
      - 一般当文件比较大的时候使用
      - 当Brick失效会导致数据丢失
    - 分布式复制卷
      - 最常见的使用方式
      - 具有分布式卷和复制卷的优点
    - 条带复制卷
    - 分布式条带复制卷

- GFID: GlusterFS的Volume中每一个文件或目录的唯一标识，类似于inode
- Namespace: 每一个Volume都导出为单个命名空间，作为POSIX的挂载点
- Node
  - 物理上的存储设备，包含若干个Brick
  - 各个Node之间是对等关系
  - 每个Node都存储了整个集群的配置信息，需要保证节点间配置信息的一致性

- Trusted Storage Pool: 由多个Node组成的存储池，其中的Node可以动态加入和删除
- Client: 挂载了GlusterFS中的Volume，并对Volume进行读写的客户端

## 基本操作

- 在每一个Node上启动gluster服务

```shell
service glusterd start
```

- 添加和删除Node: 通过一个Node来邀请其他Node加入

```shell
# on node1
gluster peer probe node2
gluster peer detach node3
```

- 查看Node状态

```shell
gluster peer status
```

- 创建分布式卷

```shell
gluster volume create test-volume node1:/export1 node2:/export2
```

![img](/images/GlusterFS之分布式卷.png)

- 创建复制卷

```shell
gluster volume create test-volume replica 2 transport tcp node1:/export1 node2:/export2
```

![img](/images/GlusterFS之复制卷.png)

- 启动/停止/删除数据卷

```shell
gluster volume start test-volume
gluster volume stop test-volume
gluster volume delete test-volume
```

- 查看数据卷信息和状态

```shell
gluster volume info
gluster volume status
```

- 在Client上挂载数据卷

```shell
mount.glusterfs node1:/test-volume /mnt/gluster
```

## 使用流程

- 在Client上启动GlusterFS客户端，此时会向OS注册一个FUSE文件系统，该FUSE文件系统和EXT文件系统位于同一层次，EXT文件系统对本地磁盘进行处理，而FUSE将数据通过/dev/fuse设备文件递交给GlusterFS客户端
- 在Clien上将GlusterFS进行挂载，被挂载的目录对Client是透明的，Client不会察觉到它是本地文件系统还是分布式文件系统
- 在Client上用户对该目录进行读写操作，该操作被交给OS的VFS进行处理，最终递交给GlusterFS客户端进行处理
- GlusterFS客户端将读写操作递交给GlusterFS服务端进行处理，最终将数据从存储池中读取并返回或将数据写入存储池中

![img](/images/GlusterFS之读写流程.png)