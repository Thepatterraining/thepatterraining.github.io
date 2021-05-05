---
title: ElasticSearch六--ES--Analyzer分词
date: 2021-05-05
tags: ['ES']
category: ES
article: ElasticSearch六--ES--Analyzer分词
---

# Analyzer分词

## Analysis 和 Analyzer

Analysis - 文本分析是把全文本转换成一系列单词（term/token）的过程，也叫分词

Analysis 是通过 Analyzer 来实现的
- 可使用 Elasticsearch 内置的分析器，或者按需制定分析器

除了在数据写入时转换词条，匹配 Query 语句时候也需要用相同的分析器对查询语句进行分析

## Analyzer 的组成

分词器是专门处理分词的组件，由三部分组成
- Character Filters (针对原始文本处理，例如去除html)
- Tokenizer (按照规则切分为单词)
- Token Filter(将切分的单词进行加工，小写，删除 stopwords,增加同义词)


例子：

将 `Mastering Elasticsearch & Elasticsearch in Action` 经过上面的步骤就会产生
- master
- elasticsearch
- action


## ES的内置分词器

- Standard Analyzer - 默认分词器，按词切分，小写处理
- Simple Analyzer - 按照非字母切分（符号被过滤），小写处理
- Stop Analyzer - 小写处理，停用词过滤（the a is）
- WhiteSpace Analyzer - 按照空格切分，不转小写
- Keyword Analyzer - 不分词，直接将输入当做输出
- Patter Analyzer - 正则表达式，默认\W+(非字符分隔)
- Language - 提供了30多种常见语言的分词器
- Customer Analyzer 自定义分词器


## 使用 _analyzer API

直接指定 Analyzer 进行测试

指令
```
GET /_analyze
{
    "analyzer":"standard",
    "text":"Mastering Elasticsearch & Elasticsearch in Action"
}
```

结果
```
{
  "tokens" : [
    {
      "token" : "mastering",
      "start_offset" : 0,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "elasticsearch",
      "start_offset" : 10,
      "end_offset" : 23,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "elasticsearch",
      "start_offset" : 25,
      "end_offset" : 38,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "in",
      "start_offset" : 39,
      "end_offset" : 41,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "action",
      "start_offset" : 42,
      "end_offset" : 48,
      "type" : "<ALPHANUM>",
      "position" : 4
    }
  ]
}
```

指定索引的字段进行测试

指令
```
POST test_home/_analyze
{
  "field": "job_name",
  "text":"Mastering Elasticsearch, Elasticsearch in Action"
}
```

结果

```
{
  "tokens" : [
    {
      "token" : "mastering",
      "start_offset" : 0,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "elasticsearch",
      "start_offset" : 10,
      "end_offset" : 23,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "elasticsearch",
      "start_offset" : 25,
      "end_offset" : 38,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "in",
      "start_offset" : 39,
      "end_offset" : 41,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "action",
      "start_offset" : 42,
      "end_offset" : 48,
      "type" : "<ALPHANUM>",
      "position" : 4
    }
  ]
}
```

自定义分词进行测试

指令

```
POST _analyze
{
  "tokenizer": "standard",
  "filter": ["lowercase"],
  "text":"Mastering Elasticsearch, Elasticsearch in Action"
}
```

结果

```
{
  "tokens" : [
    {
      "token" : "mastering",
      "start_offset" : 0,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "elasticsearch",
      "start_offset" : 10,
      "end_offset" : 23,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "elasticsearch",
      "start_offset" : 25,
      "end_offset" : 38,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "in",
      "start_offset" : 39,
      "end_offset" : 41,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "action",
      "start_offset" : 42,
      "end_offset" : 48,
      "type" : "<ALPHANUM>",
      "position" : 4
    }
  ]
}
```

## 中文分词的难点

中文句子，切分成一个个词（不是一个个字）

英文字，单词有自然地空格进行切分

一句中文，在不同的上下文，有不同的理解

推荐的一些中文分词器：
- icu
- ik : https://github.com/medcl/elasticsearch-analysis-ik
- thulac : https://github.com/microbun/elasticsearch-thulac-plugin


> 极客时间 ES 学习笔记