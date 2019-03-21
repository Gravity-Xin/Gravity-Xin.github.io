---
title: Docker的Linux原理
date: 2019-01-18 14:07:53
categories: 
- 系统架构
tags: 
- namespace
- cgroup
- union-filesystem
- copy-on-write
---

## 使用namespace实现资源隔离

- UTS: 主机与域名隔离，使用独立的主机名称，从而能够网络中标识自己

- IPC: 进程间通信隔离，如信号量、消息队列、共享内存等

- PID: 进程隔离，与宿主机中的进程ID隔离

- Network: 网络设备、网络栈、端口等隔离

- Mount: 文件系统隔离，使用`chroot`来切换根目录的挂载点

- User: 用户，用户组和权限的隔离

## 使用cgroups实现资源限制

- CPU

- Memory

- Disk I/O

- Network I/O

## 使用union-filesystem实现镜像

- 基于copy-on-write，写时复制

- 多个layer，最上层为一个可写的层

- 所有对容器的修改其实都是对最上层的修改
