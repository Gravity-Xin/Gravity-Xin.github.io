---
title: Elasticsearch API
date: 2019-02-11 17:47:05
categories: 
- Elasticsearch
tags: 
- index
- type
- document
comments: true
---

ES提供了Restful接口对索引、类型和文档进行CURD操作

## 基本的CURD

- 添加索引

```shell
PUT index1
{
"settings" : {
      "index" : {
        "number_of_shards" : "3",  // 分片
        "number_of_replicas" : "1"  // 副本
    }
  }
}
```

- 查看索引

```shell
GET _all  // 获取所有索引
GET index1
```

- 删除索引

```shell
DELETE index1
```

- 添加文档

```shell
PUT index1/user/1  // 索引名称/类型名称/文档ID
{
  "first_name": "Jane",
  "last_name": "Smith",
  "age": 32,
  "about": "I like to collect rock albums",
  "interests": [
    "music"
  ]
}

POST index1/user  // 索引名称/类型名称
{
  "first_name": "Douglas",
  "last_name": "Fir",
  "age": 23,
  "about": "I like to build cabinets",
  "interests": [
    "forestry"
  ]
}
```

- 查看文档

```shell
GET index1/user/1
GET index1/user/1?_source=first_name,about   // 查看特定字段
```

- 更新文档

```shell
PUT index1/user/1  // 用新的文档覆盖当前文档
{
  "first_name": "Jane",
  "last_name": "Smith",
  "age": 33,
  "about": "I like to collect rock albums",
  "interests": [
    "music",
    "football",
  ]
}

POST index1/user/1/_update  // 可以单独更新某些字段
{
  "doc": {
    "age": 34
  }
}
```

- 删除文档

```shell
DELETE index1/user/1
```

## 基于MultiGet的批量文档查询

```shell
GET /_mget    // 获取不同索引，不同类型下的不同文档
{
  "docs": [
    {
      "_index": "index1",
      "_type": "user",
      "_id": 1,
      "_source": [  // 还可以指定返回的字段列表
        "first_name",
        "age"
      ]
    },
    {
      "_index": "index1",
      "_type": "user",
      "_id": 2
    }
  ]
}

GET index1/user/_mget  // 获取相同索引，相同类型下的多个文档
{
  "docs": [
    {
      "_id": 1
    },
    {
      "_id": 2
    }
  ]
}

GET index1/user/_mget  // 获取相同索引，相同类型下的多个文档
{
  "ids":[1,2]
}
```

## 基于Bulk的批量文档操作

bulk的格式:

```json
{action:{metadata}}
{request_body}
```

action表示操作类型:

- create: 批量添加，如果文档已存在则添加失败，此时必须指定文档ID
- update: 批量更新
- index: 批量添加或替换已存在的文档，可以不指定文档ID，此时ES添加文档并生成ID
- delete: 批量删除

metadata表示文档所属的索引、类型和ID
request_body为文档具体内容

批量添加文档

```shell
POST /index2/books/_bulk
{"create":{"_id":1}}
{"title":"Java","price":55}
{"create":{"_id":2}}
{"title":"C++","price":65}
{"create":{"_id":3}}
{"title":"Golang","price": 45}
```

批量更新文档

```shell
POST /index2/books/_bulk
{"update":{"_id":1}}
{"doc":{"title":"Java1"}}
{"update":{"_id":2}}
{"doc":{"title":"C++1"}}
{"update":{"_id":3}}
{"doc":{"title":"Golang1"}}
```

批量删除文档

```shell
POST /index2/books/_bulk
{"delete":{"_id":1}}  // 删除操作不需要请求体
{"delete":{"_id":2}}
```

批量混合操作

```shell
POST /index2/books/_bulk
{"create":{"_id":1}}
{"title":"Java","price":55}
{"delete":{"_id":3}}
```

bulk操作是将待处理的数据载入到内存当中，因此其对数据大小有限制**一般建议文档数最大为1000-5000个，数据大小最大为5-15MB**