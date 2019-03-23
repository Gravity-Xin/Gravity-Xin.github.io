---
title: Golang经典库-Casbin
date: 2019-03-22 16:50:58
categories: 
- 编程技术
tags: 
- ACL
- RBAC
---

## 核心概念

权限控制模型 Model 被抽象为PERM(Policy Effect Request Matcher)
  Request: 定义权限请求的格式
    默认的权限请求格式: Subject Object Action
      Subject: 访问的主体，一般为用户
      Object: 被访问的资源
      Action: 对资源进行的操作
  https://casbin.org/editor/ 来选择合适的权限控制模型
  目前之前的权限控制模型包括:
    ACL
    ACL with superuser
    ACL without users
    ACL without resources
    RBAC
    RBAC with resource roles
    RBAC with domains/tenants
    ABAC
  Policy: 定义权限控制策略的格式，可以被持久化到外部存储设备中 如mysql、mongodb等
    p: Policy定义
    g: 角色定义
  Matcher: 定义Request和Policy之间是如何匹配的，使用p.eft表示对于当前的Request，该Policy与其是否匹配
  Effect: 当某个Request对应多个Policy时，判断该请求是否能通过校验
  Role: 可选，当使用RBAC时，定义不同的角色组，比如用户角色组，资源角色组，用户角色+租户组等

Casbin进行权限校验的流程:![img](/images/Casbin之PERM权限控制模型.png)

## 源码分析