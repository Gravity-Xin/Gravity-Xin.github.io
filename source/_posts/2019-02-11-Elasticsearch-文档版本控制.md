---
title: Elasticsearch 文档版本控制
date: 2019-02-11 17:58:10
categories: 
- Elasticsearch
tags:
- document
---

ES使用乐观锁来保证文档数据的一致性，即每个用户对同一个文档的更新或删除操作并不需要进行加锁和解锁的操作，只需要提供要操作的文档版本号即可

内部版本控制: ES的默认版本控制，使用文档的_version字段。当操作提供的版本号与文档当前版本号一致时，ES允许操作顺利进行并将版本号递增，否则提示版本号冲突并抛出异常; 文档内部版本号为[1, 2^63-1]

外部版本控制: 判断操作提供的版本号是否大于文档当前版本号，如果大于，则允许操作顺利进行并将版本号改为提供的版本号，否则提示版本冲突并抛出异常；需要将设置version_type=external

```shell
PUT index3/weather/1?version=4  // 使用内部版本控制
{
  "name": "NJ",
  "status": "Rainy"
}

PUT index3/weather/2?version=5&version_type=external  // 使用外部版本控制
{
  "name": "SH",
  "status": "Sunny"
}
```