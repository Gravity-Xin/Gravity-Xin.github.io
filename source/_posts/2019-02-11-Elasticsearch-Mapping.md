---
title: Elasticsearch Mapping
date: 2019-02-11 17:47:19
categories: 
- Elasticsearch
tags:
- mapping
comments: true
---

Mapping是用于描述文档schema的，即对文档中各个字段的数据类型以及该字段如何进行分词进行描述

在添加文档时，ES会根据插入的文档，自动创建Mapping，可以使用`GET /index1/user/_mapping`来查看

也可以在创建索引时，手动创建Mapping

```shell
PUT index4
{
  "mappings": {
    "books": {
      "properties": {
        "ids": {
          "type": "long"
        },
        "price": {
          "type": "long"
        },
        "title": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    }
  }
}
```

Mapping中常用的字段数据类型:

- 核心数据类型
  - 字符型: string, 包括text和keyword
    - text: 对长文本进行索引，在建立索引时对其进行分词，不可用于排序和聚合
    - keyword: 不需要进行分词，只可以使用本身来进行检索过滤，还可以用来排序和聚合
  - 数值型: long, integer, short, byte, double, float，不进行分词
  - 日期型: date，不进行分词
  - 布尔型: boolean，不进行分词
  - 二进制型: binary，不进行分词
- 复杂数据类型
  - 数组类型: json数组
  - 对象类型: _object json对象
  - 嵌套类型: _nested
- 地理位置类型
  - 地理坐标类型: _geo_point
  - 地理形状类型: _geo_shape
- 其他

Mapping描述字段常用的属性:

- store: false 字段不单独进行存储
- index: true 是否对该字段进行分词
- analyzer: ik 指定分词器，默认为standard
- boost: 1.23 字段级别的分数加权
- ignore_above: 100 字段值的长度超过该值，则进行忽略，不被索引
- search_analyzer: ik 默认与analyzer一致
- 其他

在基于字段进行查询时，该字段的mapping决定了查询方式

- text类型基于分词结果进行查询，因此可以进行模糊匹配
- 其他类型不进行分词，必须使用全字匹配来进行查询，因此只可以进行精确匹配

```shell
GET /index1/user/_search?q=first_name:Douglas
GET index1/user/_search?q=age:2
GET index1/user/_search?q=age:23
```