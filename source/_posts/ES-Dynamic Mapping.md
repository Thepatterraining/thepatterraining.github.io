---
title: ElasticSearch十一--Dynamic Mapping和常见字段类型
date: 2021-05-05
tags: ['ES']
category: ES
article: ElasticSearch十一--Dynamic Mapping和常见字段类型
---

# Dynamic Mapping和常见字段类型

## Mapping

Mapping 类似数据库中的 `schema`的定义，作用如下：
- 定义索引中的字段的名称
- 定义字段的数据类型，例如字符串，数字，布尔等
- 字段，倒排索引的相关配置

Mapping 会把 JSON 文档映射成 Lucene 所需要的扁平格式

一个Mapping 属于一个索引的 Type
- 每个文档都属于一个Type
- 一个 Type 有一个 Mapping 定义
- 7.0开始，不需要在 Mapping 定义中指定 Type信息

### 字段的数据类型

##### 简单类型
- Text/Keyword
- Date
- Interger/Floating
- Boolean
- IPv4 & IPv6

##### 复杂类型

- 对象类型
- 嵌套类型

##### 特殊类型

geo 是用于存储地理位置信息的类型

- geo_point & geo_shape/ percolator

### Dynamic Mapping

这个就是动态的Mapping，可以不需要手动指定，由 ES 动态识别数据类型，来生成对应的Mapping。

但是这样生成的 Mappping 不一定正确。

如果想要修改Mapping
- 新增加字段
  - Dynamic 设为 true,一旦有新字段，Mapping会自动更新
  - Dynamic 设为 false,一旦有新字段，Mapping不会更新，新增加的字段无法被索引，但是_source里面有数据
  - Dynamic 设为 Strict,文档写入失败
- 已有字段，一旦已经有数据就不支持修改了
  - Lucene实现的倒排索引，一旦生成就不能修改
- 如果想要修改，需要Reindex API ，重建索引

原因：
- 如果修改了字段的数据类型，会导致已被索引的数据无法被搜索
- 但是如果是增加新的字段，就不会有这种情况

注意!!!：
如果`Dynamic`设为false,那么新增字段无法被索引，也就意味着不能拿这个字段去搜索，比如新增加一个`name`字段，那么无法搜索`name`这个字段，比如你想搜索name是张三的，是搜索不出来的。



> 极客时间 ES 学习笔记