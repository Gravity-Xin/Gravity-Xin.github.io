---
title: Elasticsearch-Filter查询
date: 2019-02-12 16:06:06
categories: 
- Elasticsearch
tags:
- filter
---

与Query查询不同的是，Filter查询不计算文档与查询条件之间的相关性(score值)，同时可以进行查询结果的缓存，因此Filter查询的速度比Query快

- 简单的过滤查询

```shell
GET /index1/user/_search
{"post_filter": {"term": {"name": "zhaoming"}}}  // 也可以用terms、match、range、exists等其他查询方式
```

- bool过滤查询

通过`must`, `should`和`must_not`可以实现多个过滤条件的组合

must: 类似于SQL中的`AND`
should: 类似于SQL中的`OR`
must_not: 类似于SQL中的`NOT`

```shell
GET /index1/user/_search
{"query":{"bool":{"filter":[{"term":{"name":"zhaoming"}}]}}}  // 普通的过滤查询

GET /index1/user/_search
{"query":{"bool":{"must":[{"term":{"name":"zhaoming"}}],"must_not":[{"term":{"age":20}}]}}}  // 名称必须为zhaoming且年龄不是20的文档

GET /index1/user/_search
{"query":{"bool":{"should":[{"term":{"name":"zhaoming"}},{"term":{"name":"lisi"}}]}}}  // 名称为zhaoming或lisi的文档
```