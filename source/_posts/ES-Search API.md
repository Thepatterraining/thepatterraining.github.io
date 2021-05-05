---
title: ElasticSearch七--ES--Search API
date: 2021-05-05
tags: ['ES']
category: ES
article: ElasticSearch七--ES--Search API
---

# Search API

## 两种方法

URI Search
- 通过Url参数来进行查询

url指定参数q=field:搜索内容

例子：
http://localhost:9200/{index}/_search?q={field}:搜索内容

url:
http://localhost:9200/test_home/_search?q=job_name:php

Request Body Search
- 通过ES提供的JSON格式的DSL语句进行查询

| 语法 | 范围 |
|:-----|:-----|
|/_search|集群上所有索引|
|/{index}/_search| 对应index上的数据|
|/{index},{index}/_search| 对应多个index上的数据|
|/test*/_search| 所有test开头的index上的数据|

支持GET POST两种方式

例子：
```
GET/POST test_home/_search
{
  "query":{
    "match_all": {}
  }
}
```


## 搜索返回的结果

- took: 查询花的时间
- hits-total: 符合条件的总文档数
- hints: 结果集
- _score: 相关度评分

### 相关度

Infomation Retrieval
- Precision(查准率) - 尽可能返回较少的无关文档
- Recall (查全率) - 尽量返回较多的相关文档
- Ranking - 是否能按相关度进行排序




> 极客时间 ES 学习笔记