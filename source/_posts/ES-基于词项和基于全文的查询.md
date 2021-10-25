---
title: ElasticSearch十五--ES-基于词项和基于全文的查询
date: 2021-10-20
tags: ['ES']
category: ES
article: ElasticSearch十五--ES-基于词项和基于全文的查询
---

# term query

`term`是表达语义的最小单位 ,搜索和利用统计语言模型进行自然语言处理都需要处理 term

特点
- term level query: term query/range query/ exists query / prefix query / wildcard query
- 在ES里面，term 查询不做分词，把term查询作为一个整体词汇进行查询，进行一个精确匹配，并对匹配结果进行算分
- 可以通过  `Constant Score` 将查询转换成一个 `filtering` ，避免算分，并利用缓存，提高性能


## term 查询

我们现在有数据如下
```
{
  "_index" : "test",
  "_type" : "_doc",
  "_id" : "ID10032_1194_1634175220_535254",
  "_score" : 6.151313,
  "_source" : {
    "user_id" : 10032,
    "consultant_company_id" : 1194,
    "type" : 9,
    "source" : 1,
    "operation_at" : 1634175220,
    "create_time" : "2021-10-14 09:33:40",
    "update_time" : "2021-10-14 09:33:40",
    "login_type" : 3,
    "true_name" : "张三",
    "id" : "ID10032_1194_1634175220_535254"
  }
}
```

如果我们使用`match`查询

这里`ID10032_1194_1634175220_535254`大写`id10032_1194_1634175220_535254`小写都可以

```
GET test_user_action_detail/_search
{
  "query": {
    "match": {
      "id": "ID10032_1194_1634175220_535254"
    }
  }
}
```

都可以得到搜索结果

如果使用 `term` 查询,`id10032_1194_1634175220_535254`可以查询到数据，但是如果是`ID10032_1194_1634175220_535254`大写，那么`term` 查询就查询不到数据了

```
GET test_user_action_detail/_search
{
  "query": {
    "term": {
      "id": {
        "value": "id10032_1194_1634175220_535254"
      }
    }
  }
}
```

原因在于 该 id 字段的 `mapping` 是 text/keyword

```
"id" : {
  "type" : "text",
  "fields" : {
    "keyword" : {
      "type" : "keyword",
      "ignore_above" : 256
    }
  }
}
```

`text`数据类型会进行分词，也就是大写会被分词成小写。所以text的id数据实际会变成小写`id10032_1194_1634175220_535254`，而`term`查询区分大小写，会进行精确匹配，如果只有搜索小写才能搜索出来结果。如果不想进行分词，那么可以使用`keyword`字段类型搜索，把`term`查询语句改成下面这样

```
GET test_user_action_detail/_search
{
  "query": {
    "term": {
      "id.keyword": {   //这里使用keyword字段查询
        "value": "ID10032_1194_1634175220_535254"
      }
    }
  }
}
```

这样就可以得到结果了，因为`keyword`不进行分词，所以插入的`ID10032_1194_1634175220_535254`就还是`ID10032_1194_1634175220_535254`，不过注意，这样的话小写`id10032_1194_1634175220_535254`就搜索不到结果了



## filter 不进行算分

对于term查询来说，其实没必要算分，因为是精确搜索，那么就可以使用`filter`查询来避免算分并可以有效利用缓存，可以提升性能

```
GET test_user_action_detail/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "id.keyword": {
            "value": "ID10032_1194_1634175220_535254"
          }
        }
      }
    }
  }
}
```

## 基于全文查询

- match query  / match phrase query / query string query

- 特点
  - 索引和搜索都会进行分词，查询字符串先传递到一个合适的分词器，然后生成一个供查询的词项列表
  - 查询时，会先分词，然后对每个词都进行查询，最终将结果进行合并。并为每个文档生成一个算分。
  - 例如查询`matrx reloaded` 会查询到 `matrx` 和 `reloaded`两个词的搜索结果，然后进行合并。


## range query

range 可以进行范围查询

```
range: {
  created_at: {
    gt:'1889237773'
  }
}
```

## exists query

可以判断值是否存在

```
exists:{
  field:"created_at"
}
```

## 处理多值字段

比如有一个数组 `[北京市，上海市]`

如果我们使用`term`查询

```
GET test_user_action_detail/_search
{
  "query": {
    "term": {
      "city.keyword": { 
        "value": "北京市"
      }
    }
  }
}

```

就会搜索出现上面的数据，因为对于数组来说，term 查询是只要包含那么就会搜索出来。


> 极客时间 ES 学习笔记
