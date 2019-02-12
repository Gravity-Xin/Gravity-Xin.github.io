---
title: Elasticsearch-Query查询
date: 2019-02-12 13:47:19
categories: 
- Elasticsearch
tags:
- term
- match
- range
- fuzzy
---

- 简单的Query查询

```shell
GET /index1/user/_search?q=name:zhangsan  //单字段匹配

GET /index1/user/_search?q=name:zhangsan&_source=name,age  // 单字段匹配，指定需要返回的字段

GET /index1/user/_search?q=interests:changge&sort=age:desc  // 单字段匹配，指定返回的多个文档的排序规则，默认按照相似度的score值进行降序返回
```

- term和terms查询

向倒排索引中寻找确切的term，不对查询字段进行分词，该查询适合字符串、数值型和日期型等数据类型
适合`精确查找`
term查询: 查询某个字段中包含某个关键词的文档
terms查询: 查询某个字段中包含多个关键词的查询

```shell
GET /index1/user/_search
{"query": {"term": {"interests": "changge"}}}   // 单一关键词查询

GET /index1/user/_search
{"query": {"terms": {"interests": ["changge", "tiaowu"]}}}   // 多个关键词查询，满足一个即可

GET /index1/user/_search
{"query": {"terms": {"interests": ["changge", "tiaowu"]}}, "from": 0, "size": 2}   // 规定返回文档的偏移量和数量

GET /index1/user/_search
{"version": true, "query": {"term": {"interests": "changge"}}}   // 返回文档的版本号

GET /index1/user/_search
{"query": {"term": {"interests": "changge"}}, "sort": [{"age": {"order": "desc"}}]}   // 规定返回文档的排序方式

GET /index1/user/_search
{"_source": ["address", "name"], "query": {"term": {"interests": "changge"}}}   // 规定返回文档中的字段
```

- match查询

match查询会对输入的查询字段先进行分词操作，然后再查询

```shell
GET /index1/user/_search
{"query":{"match":{"interests": "changge tiaowu"}}}  // 先分词，再分别对分词的每一个结果进行查询

GET /index1/user/_search
{"query":{"match":{"age": 23}}}

GET /index1/user/_search
{"query":{"match_all":{}}}   // 使用match_all来查询所有文档

GET /index1/user/_search
{"query":{"multi_match":{"query":"changge","fields":["interests","name"]}}} // 使用multi_match来查询多个字段

GET /index1/user/_search
{"query":{"match_phrase":{"interests":"chang ge tiao wu"}}} // 使用match_phrase来进行短语匹配查询，意味着必须匹配所有分词结果，且各个分词的相对位置不变

GET /index1/user/_search
{"query": {"match_phrase_prefix": {"interests": "changg"}}}  // 使用match_phrase_prefix来进行前缀匹配查询，只需要分词的前缀匹配即可
```

- range查询

查询某字段值在某个范围的文档，适合数值型和日期型字段

```shell
GET /index1/user/_search
{"query":{"range":{"age":{"gt":20,"lte":30}}}}

GET /index1/user/_search
{"query":{"range":{"birthday":{"gte":"1970-10-12","lt":"1987-05-05"}}}}
```

- wildcard查询

使用通配符`*`和`?`进行查询，`*`代表0个或多个字符，`?`代表一个字符

```shell
GET /index1/user/_search
{"query":{"wildcard":{"name": "zh?o*"}}}
```

- fuzzy查询

当对搜索的字符的精确值不清楚时，可以使用fuzzy进行模糊查询

```shell
GET /index1/user/_search
{"query":{"fuzzy":{"name": "lii"}}}
```

- exist查询

判断文档中的某个字段的字段值是否为空
类似于SQL中的`IS NOT NULL`

```shell
GET /index1/user/_search
{"query":{"exists":{"field":"age"}}}  // 获取age字段不为空的所有文档
```