---
title: Elasticsearch入门
date: 2019-01-25 17:35:48
categories: 
- Elasticsearch
tags: 
- lucene
- index
- type
- document
- mapping
comments: true
---

一个基于lucene的分布式多用户的全文搜索引擎，提供Restful接口

## 基本概念

- Index: 索引，类似于mysql中的database

- Type: 类型，类似于mysql中的table

- Document: JSON文档，类似于mysql中的row
  - 包含多个Field

- Mapping: JSON文档的schema，类似于mysql中的table schema

- QueryDSL: 查询语句，类似于mysql中的sql语句

- GET/PUT/POST/DELETE: Restful接口，实现文档的增删改查

## 基本功能

- 倒排索引
  - 对所有文档中的内容进行分词，得到单词列表
    - Elasticsearch会使用标准化规则来处理单词中的大小写、单复数、同义词等问题，提高查询的命中率
  - 为每一个单词，赋予唯一的ID
  - 建立文档中的单词ID与文档ID的关系
  - 还可以记录每一个单词在文档中出现的次数称为单词频率信息、每一个单词在文档中出现的位置等
  - 这样便通过分词获取到相关的文档，还可以通过单词频率信息确定单词与文档的相似度以进行排序

- 分词器: 从一串文本中切分出一个一个的词条，同时对每一个词条进行标准化处理
  - 处理过程
    - character filter: 分词前的预处理，如过滤HTML标签、特殊符号转换等
    - tokenizer: 分词操作
    - token filter: 标准化操作
  - 内置分词器
    - standard: 默认分词器，将词汇单元转换为小写形式，去除停用词和标点符号，支持中文的单字切分
    - simple: 使用非字母字符来分割文本信息，将词汇单元统一为小写形式，去除数字类型的字符
    - whitespace: 仅去除空格，不支持中文
    - language: 特定语言的分词器，不支持中文
  - 中文分词器: [ik](https://github.com/medcl/elasticsearch-analysis-ik/)
  
