---
title: ElasticSearch十四--ES-index template
date: 2021-05-09
tags: ['ES']
category: ES
article: ElasticSearch十四--ES-index template
---

# index template

## index template

index template 可以帮助你设定 Mappings 和 Settins ，并按照一定的规则，自动匹配到新创建的索引上

- 模板仅在一个索引被新创建时，才会产生作用。修改模板不会影响已经创建的索引
- 可以设定多个索引模板，这些设置会根据一定的规则 合并到一起
- 可以指定 Order的数值 控制 合并 的过程

### 实例

```
PUT _template/template_default
{
  "index_patterns":["*"],
  "order": 0,
  "version": 1,
  "settings":{
    "number_of_shards":1,
    "number_of_replicas":1
  }
}
```

### index templalte 的工作方式

当一个新索引被创建
- 应用 ES 的默认 settins 和 mappings
- 应用 order 数值低的 index template 的 settings mappings
- 应用 order 数值高的 index template 的 settings mappings
- 应用 用户 自定义的 settings mappings


## Dynamic template

根据 ES 识别的数据类型，结合字段名称，来动态设定字段类型
- 所有的字符串类型都设定成 keyword 或者关闭 keyword 字段
- is 开头的字段都设置成 boolean
- long_ 开头的都设置成 long 类型


## Aggregation 聚合

ES 除了搜索以外提供了统计分析的功能
- 实时性高
- Hadoop

通过聚合可以得到数据的概览，是分析和总结全套的数据，而不是寻找单个文档

高性能，只需要一条语句，就可以从 ES 得到结果
- 无需在客户端自己去实现分析逻辑

比如 Kibana 中的可视化报表就是使用 聚合 实现的

### 分类

Bucket Aggregation : 一些满足特定条件的集合，相当于 sql 里面的 group by 分组功能

Metric Aggregatino : 提供数学运算，比如最大值，最小值，平均值 相当于 sql 里面的 count(),max()等函数功能

Pipeline Aggregation: 可以二次聚合

Matrix Aggregation: 支持对多个字段的操作并提供一个结果矩阵





> 极客时间 ES 学习笔记