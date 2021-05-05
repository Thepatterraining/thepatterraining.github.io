---
title: ElasticSearch八--ES--URI Search 详解
date: 2021-05-05
tags: ['ES']
category: ES
article: ElasticSearch八--ES--URI Search 详解
---

# URI Search

通过URI query 实现搜索

参数：
- q:指定查询语句，使用Query String Syntax
- df:默认字段，不指定会对所有字段进行查询
- sort:排序
- from/size: 用来分页
- Profile 可以查看查询是如何被执行的

## Query String Syntax

指定字段和泛查询

### 泛查询

q参数后面只跟着查询内容会对所有字段进行搜索，可以看到返回值里面`profile`-`query`-`type`是 `DisjunctionMaxQuery`。 `description`是`(customer_name.keyword:产品 | (job_name:产 job_name:品) | job_name.keyword:产品 | (customer_name:产 customer_name:品) | MatchNoDocsQuery("failed [id] query, caused by number_format_exception:[For input string: "产品"]") | (city_names:产 city_names:品) | city_names.keyword:产品)`

指令
```
GET test_home/_search?q=产品
{
  "profile": "true"
}
```

### 指定字段

指定字段有两种方式，一种是df参数

指令
```
GET test_home/_search?q=产品&df=job_name
{
  "profile": "true"
}
```

还有一种是q参数使用冒号`：`来分隔

指令
```
GET test_home/_search?q=job_name:产品
{
  "profile": "true"
}
```

指定字段的返回参数可以看到`type`是`BooleanQuery`,`description`是`job_name:产 job_name:品`

## term query & PhraseQuery

term query 是 or的关系，比如上面搜索产品，只要包含`产`或者`品`就都会搜索出来，可以用()小括号把搜索内容括起来

phrase query 是 and 的关系，需要再搜索内容上加双引号。这样只会搜索出`产品`的结果。

指令
```
GET test_home/_search?q=job_name:"产品"
{
  "profile": "true"
}
```


## boolean query

boolean query可以用`AND`,`OR`,`NOT`来操作
比如既要`产`又要`品`
指令
```
GET test_home/_search?q=job_name:(产 AND 品)
{
  "profile": "true"
}
```

比如只有一个就行
指令
```
GET test_home/_search?q=job_name:(产 OR 品)
{
  "profile": "true"
}
```

比如只有品没有产

指令
```
GET test_home/_search?q=job_name:(品 NOT 产)
{
  "profile": "true"
}
```

## 范围查询

区间表示  []闭区间 {}开区间
- year:[2019 TO 2.18]

## 算术符号

可以使用`>`,`<`,`=`等数学符号

## 通配符查询

通配符查询效率低，占用内存大，不建议使用。

## 正则表达式

可以用正则进行匹配搜索查询

- job_name:[bt]oy

## 模糊匹配与近似查询

模糊查询允许用户输错`查询关键字`

比如上面的查询`产品`，用户打错了，输成了`品阶`。我们通过模糊匹配依旧可以查询出来内容.就是通过在查询内容后面输入`~1`代表允许1个字的错误

指令
```
GET test_home/_search?q=job_name:品阶~1
{
  "profile": "true"
}
```


> 极客时间 ES 学习笔记