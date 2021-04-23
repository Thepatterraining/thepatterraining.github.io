---
title: ElasticSearch三--ES-基本概念-索引文档REST API
date: 2021-04-23
tags: ['ES']
category: ES
article: ElasticSearch三--ES-基本概念-索引文档REST API
---

# 基本概念

## 文档(Document)

Elasticsearch 是面向`文档`的，文档是所有可搜索数据的最小单位。

- 日志文件中的日志项。
- 电影的具体信息/唱片的具体信息
- MP3播放器里的一首歌/一篇PDF文档中的具体内容

文档会被序列化成 JSON 格式，保存在Elasticsearch中。

- JSON对象由字段组成
- 每个字段都有对应的字段类型（字符串/数值/布尔/日期/二进制/范围类型）

每个文档都有一个Unique ID
- 你可以自己指定ID
- 或者通过Elasticsearch自己生成

### 文档的元数据

- 元数据，用于标注文档的相关信息
    - _index - 文档所属的索引名
    - _type - 文档所属的类型名
    - _id - 文档唯一的ID
    - _source - 文档的原始JSON数据
    - _all - 整合所有字段内容到该字段，已被废除
    - _version - 文档的版本信息
    - _score - 相关性打分

## 索引（Index）

`索引`是`文档`的容器，是一类文档的集合。
- Index 体现了逻辑空间的概念：每个索引都有自己的Mapping定义，用于定义包含的文档的字段名和字段类型
- Shard 体现了物理空间的概念：索引中的数据分散在Shard上

索引的 Mapping 和 Settings
- Mapping 定义文档字段的类型
- Setting 定义不同的数据分布

#### 索引的不同语意

- 名词：一个Elasticsearch中，有多个不同的索引。
- 动词：保存一个文档到Elasticsearch的过程也叫索引（indexing）
    - ES中，创建一个倒排索引的过程
- 名词：一个B树索引，一个倒排索引

## 类型Type

- 7.0 开始 每个索引只能 创建一个 Type - “_doc”

## REST API

Elasticsearch提供了REST API的方式进行调用，这样不管什么语言都可以通过HTTP的方式进行调用。

这些API的文档可以在[Elasticsearch的文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)里面找到.

可以在Kibana里面的`dev tools`进行REST API的测试。
