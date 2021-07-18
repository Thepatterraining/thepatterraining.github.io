---
title: ElasticSearch十--ES--query string和simple query string
date: 2021-05-05
tags: ['ES']
category: ES
article: ElasticSearch十--ES--query string和simple query string
---

# query string和simple query string

## query string

类似URI query

指令
```
POST users/_search
{
  "query":{
    "query_string":{
      "default_field":"job_name",  //相当于URI的 df
      "query":"产品"
    }
  }
}
```

还可以搜索多个字段

指令
```
POST users/_search
{
  "query":{
    "query_string":{
      "query":"产品",
      "fields":['job_name'] //搜索job_name是产品的
    }
  }
}
```

还可以直接在query里面使用`AND`,`OR`,`NOT`的操作符

指令
```
POST users/_search
{
  "query":{
    "query_string":{
      "query":"产 OR 品",
      "fields":['job_name'] //搜索job_name是产品的
    }
  }
}

```

## simple query string

类似 `query string` ，但是会忽略错误的语法，同时只支持部分查询语法
- 不支持AND OR NOT ，会当做字符处理
- Term 之间默认的关系是OR，可以指定 `operator`
- 支持 部分逻辑
  - +代替AND
  - |代替OR
  - -代替NOT

这里如果使用了`+`或者`AND`还有`OR`，那么会使用`AND`，而`OR`不生效。

指令
```
POST users/_search
{
  "query":{
    "query_string":{
      "query":"产-品", //这里-代表OR
      "fields":['job_name'] //搜索job_name是产品的
      "default_operator": "OR"  //指定默认操作符
    }
  }
}
```




> 极客时间 ES 学习笔记