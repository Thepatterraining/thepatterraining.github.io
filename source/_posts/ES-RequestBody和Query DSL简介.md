---
title: ElasticSearch九--RequestBody和Query DSL简介
date: 2021-05-05
tags: ['ES']
category: ES
article: ElasticSearch九--RequestBody和Query DSL简介
---

# RequestBody和Query DSL简介

通过RequestBody 实现搜索

参数：
- q:指定查询语句，使用Query String Syntax
- df:默认字段，不指定会对所有字段进行查询
- sort:排序
- from/size: 用来分页
- Profile 可以查看查询是如何被执行的

这个方法的参数和URI是一样的，比如我们在kibana里面执行下面的指令：
```
GET test_home/_search
{
  "profile": "true",
  "from":0,
  "size":10,
  "sort":[{"id":"desc"}]
}
```

## 查询指定字段

这种方式还可以查询指定字段，通过`_source`参数来指定。支持通配符

指令
```
GET test_home/_search
{
  "profile": "true",
  "query":{
    "match_all":{}
  },
  "_source":['job_name','id'] //指定字段
}
```

## 脚本字段

可以进行一些简单运算，拼接字符串等操作

指令
```
GET test_home/_search
{
  "profile": "true",
  "query":{
    "match_all":{}
  },
  "script_fields":{
    "new_field":{
      "script":{
        "lang":"painless" //使用这个脚本
        "source": "doc['job_name'].value+'_hello'" //在职位名称后面拼接_hello
      }
    }
  }
}
```

结果
```
"fields":{
  "new_fields":[
    "产品_hello"
  ]
}
```

## query DSL

通过DSL查询产品怎么办呢，可以使用query参数

```
GET test_home/_search
{
  "profile": "true",
  "query":{
    "match":{   //使用match 而不是 match_all
      "job_name":"产品"  //这里就相当于出现产 或者 品 都会搜出来 左边的 job_name 就是搜索的字段
    }
  },
  "_source":['job_name','id'] //指定字段
}
```

如果想使用`AND`可以增加参数，如下：

```
GET test_home/_search
{
  "profile": "true",
  "query":{
    "match":{   //使用match 而不是 match_all
      "job_name": {  //这里变成一个对象 左边的 job_name 就是搜索的字段
        "query":"产品", //这里是搜索内容
        "operator":"AND"  //这里指定AND OR NOT
      }
    }
  },
  "_source":['job_name','id'] //指定字段
}
```

### 短语搜索 match phrase

这里就是短语，单词搜索，等价于URI的phrase search

指令
```
GET test_home/_search
{
  "profile": "true",
  "query":{
    "match_phrase":{   //使用match_phrase
      "job_name": {  //这里变成一个对象 左边的 job_name 就是搜索的字段
        "query":"产品", //这里是搜索内容
        "slop":1, //代表可以有一个错误
      }
    }
  },
  "_source":['job_name','id'] //指定字段
}
```



> 极客时间 ES 学习笔记