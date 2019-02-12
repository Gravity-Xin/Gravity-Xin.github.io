---
title: Elasticsearch-聚合和复合查询
date: 2019-02-12 16:49:09
categories: 
- Elasticsearch
tags:
---

## 聚合查询

- sum: 求和

```shell
GET /index1/user/_search
{"size":0, "aggs":{"sum of age":{"sum":{"field":"age"}}}}
```

- min: 求最小值

```shell
GET /index1/user/_search
{"size":0, "aggs":{"min of age":{"min":{"field":"age"}}}}
```

- max: 求最大值

```shell
GET /index1/user/_search
{"size":0, "aggs":{"max of age":{"max":{"field":"age"}}}}
```

- avg: 求平均值

```shell
GET /index1/user/_search
{"size":0, "aggs":{"avg of age":{"avg":{"field":"age"}}}}
```

- cardinality: 求基数(互不相同的字段值的个数)

```shell
GET /index1/user/_search
{"size":0, "aggs":{"avg of cardinality":{"avg":{"field":"age"}}}}
```

- terms: 分组，类似于SQL中的`GROUP BY`

```shell
GET /index1/user/_search
{"size":0, "aggs":{"group by birthday":{"terms":{"field":"birthday"}}}}

GET /index1/user/_search
{"size":0, "query":{"match":{"interests":"changge"}},"aggs":{"sum of age":{"terms":{"field":"birthday"}}}}  // 对interests包含changge的文档按照birthday进行分组

GET /index1/user/_search
{"size":0,"query":{"match":{"interests":"chang ge"}},"aggs":{"group by birthday":{"terms":{"field":"birthday"},"aggs":{"avg age":{"avg":{"field":"age"}}}}}}  // 对interests包含changge的文档按照birthday进行分组，并且获取每一个组中文档的平均年龄
```

## 复合查询

使用bool将多个基本的查询条件进行组合，得到一个复杂的查询条件

查询条件本身可以嵌套