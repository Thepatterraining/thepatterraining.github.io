---
title: ElasticSearch十七--ES-bool查询
date: 2021-10-21
tags: ['ES']
category: ES
article: ElasticSearch十七--ES-bool查询
---

# bool查询

一个bool查询里面可以包含多个查询子句

|||
|:----|:----|
|must|必须匹配，贡献算分|
|should| 选择性匹配，算分|
|must_not| filter context 查询子句，必须不能匹配，不算分|
|filter| filter context 必须匹配,但是不算分|

- 子查询可以任意顺序出现
- 可以嵌套多个查询
- 如果bool查询中，没有must条件，should中必须满足一个查询
- should是一个数组

查询语句的结构，会对相关度算分产生影响
- 同一层级下的竞争字段，具有相同的权重
- 通过嵌套bool查询，可以改变对算分的影响


## 单字符串 多字段查询

可以通过 `bool` 的 `should` 来实现

```
POST test_home/_search
{
  "query":{
    "bool":{
      "should":[
        {"match":{"title":"php"}}
        {"match":{"body":"php"}}
      ]
    }
  }
}
```

但是这样实现的算分过程可能不是我们想要的,算分过程如下
- 查询 should 语句中的两个查询
- 加和两个查询的评分
- 乘以匹配语句的总数
- 除以所有语句的总数


我们还可以使用最高算分 `dis_max`- `queries`
```
POST test_home/_search
{
  "query":{
    "dis_max":{
      "queries":[
        {"match":{"title":"php"}}
        {"match":{"body":"php"}}
      ]
    }
  }
}
```

但是有的时候，最高评分是一样的，那怎么办呢，可以通过`tie_breaker`参数来调整
- 获得最佳匹配语句的评分
- 将其他匹配语句的评分与 `tie_breaker`相乘
- 对以上评分求和并规范化
- tie_breaker 是一个 0 - 1的浮点数，0代表最佳匹配，1代表所有语句同等重要。


### 三种场景

- 最佳字段 best_fields 相当于 dis_max 查询
  - 当字段之间相互竞争，又相互关联。例如 title 和 body 这样的，评分来自最佳字段
- 多数字段 most_fields
  - 处理英文内容时：一种常见的手段是，在主字段，抽取词干，加入同义词，以匹配更多的文档。相同的文本，加入子字段，以提供更加精确的匹配。其他字段作为匹配文档提高相关度的信号。匹配字段越多则越好
- 混合字段 cross field
  - 对于某些实体，例如人名，地址，图书，需要在多个字段中确定信息，单个字段只能作为整体的一部分。希望在任何这些列出的字段中找到尽可能多的词


#### multi_match



> 极客时间 ES 学习笔记
