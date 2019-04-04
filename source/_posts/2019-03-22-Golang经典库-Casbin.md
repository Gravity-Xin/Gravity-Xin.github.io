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

- Model:权限控制模型，抽象为PERM(Policy Effect Request Matcher)
  - Request: 权限请求的格式
    - 默认的权限请求格式: Subject Object Action
      Subject: 访问的主体，一般为用户
      Object: 被访问的资源
      Action: 对资源进行的操作
    - 选择合适的权限控制模型: [Online](https://casbin.org/editor/)
      - 支持的权限控制模型:
        - ACL
        - ACL with superuser
        - ACL without users
        - ACL without resources
        - RBAC
        - RBAC with resource roles
        - RBAC with domains/tenants
        - ABAC
  - Policy: 权限控制策略的格式，可以被持久化到外部存储设备中，如mysql、mongodb等
    - p: Policy定义
    - g: 角色定义
  - Matcher: 定义Request和Policy之间是如何匹配的，使用p.eft表示对于当前的Request，该Policy与其是否匹配
  - Effect: 当某个Request对应多个Policy时，判断该请求是否能通过校验
  - Role: 可选，当使用RBAC时，定义不同的角色组，比如用户角色组，资源角色组，用户角色+租户组等

Casbin进行权限校验的流程:![img](/images/Casbin之PERM权限控制模型.png)

## 源码分析

Model定义:

```Go
// Model represents the whole access control model.
type Model map[string]AssertionMap

// AssertionMap is the collection of assertions, can be "r", "p", "g", "e", "m".
type AssertionMap map[string]*Assertion

// Assertion represents an expression in a section of the model.
// For example: r = sub, obj, act
type Assertion struct {
  Key    string
  Value  string
  Tokens []string
  Policy [][]string
  RM     rbac.RoleManager
}

// Effector is the interface for Casbin effectors.
type Effector interface {
  // MergeEffects merges all matching results collected by the enforcer into a single decision.
  MergeEffects(expr string, effects []Effect, results []float64) (bool, error)
}
```

Role定义:

```Go
// RoleManager provides interface to define the operations for managing roles.
type RoleManager interface {
  // Clear clears all stored data and resets the role manager to the initial state.
  Clear() error
  // AddLink adds the inheritance link between two roles. role: name1 and role: name2.
  // domain is a prefix to the roles (can be used for other purposes).
  AddLink(name1 string, name2 string, domain ...string) error
  // DeleteLink deletes the inheritance link between two roles. role: name1 and role: name2.
  // domain is a prefix to the roles (can be used for other purposes).
  DeleteLink(name1 string, name2 string, domain ...string) error
  // HasLink determines whether a link exists between two roles. role: name1 inherits role: name2.
  // domain is a prefix to the roles (can be used for other purposes).
  HasLink(name1 string, name2 string, domain ...string) (bool, error)
  // GetRoles gets the roles that a user inherits.
  // domain is a prefix to the roles (can be used for other purposes).
  GetRoles(name string, domain ...string) ([]string, error)
  // GetUsers gets the users that inherits a role.
  // domain is a prefix to the users (can be used for other purposes).
  GetUsers(name string, domain ...string) ([]string, error)
  // PrintRoles prints all the roles to log.
  PrintRoles() error
}
```

Adapter定义

```Go
// Adapter is the interface for Casbin adapters.
type Adapter interface {
  // LoadPolicy loads all policy rules from the storage.
  LoadPolicy(model model.Model) error
  // SavePolicy saves all policy rules to the storage.
  SavePolicy(model model.Model) error

  // AddPolicy adds a policy rule to the storage.
  // This is part of the Auto-Save feature.
  AddPolicy(sec string, ptype string, rule []string) error
  // RemovePolicy removes a policy rule from the storage.
  // This is part of the Auto-Save feature.
  RemovePolicy(sec string, ptype string, rule []string) error
  // RemoveFilteredPolicy removes policy rules that match the filter from the storage.
  // This is part of the Auto-Save feature.
  RemoveFilteredPolicy(sec string, ptype string, fieldIndex int, fieldValues ...string) error
}
```

Watcher定义:

```Go
// Watcher is the interface for Casbin watchers.
type Watcher interface {
  // SetUpdateCallback sets the callback function that the watcher will call
  // when the policy in DB has been changed by other instances.
  // A classic callback is Enforcer.LoadPolicy().
  SetUpdateCallback(func(string)) error
  // Update calls the update callback of other instances to synchronize their policy.
  // It is usually called after changing the policy in DB, like Enforcer.SavePolicy(),
  // Enforcer.AddPolicy(), Enforcer.RemovePolicy(), etc.
  Update() error
}
```

Enforcer定义:

```Go
// Enforcer is the main interface for authorization enforcement and policy management.
type Enforcer struct {
  modelPath string
  model     model.Model
  fm        model.FunctionMap
  eft       effect.Effector

  adapter persist.Adapter
  watcher persist.Watcher
  rm      rbac.RoleManager

  enabled            bool
  autoSave           bool
  autoBuildRoleLinks bool
}
```