---
title: ElasticSearch十六--ES-搜索相关性算分
date: 2021-10-21
tags: ['ES']
category: ES
article: ElasticSearch十六--ES-搜索相关性算分
---

# 搜索相关性算分

ES 会对搜索结果的相关性进行一个算分，算分结果放到`_score` 字段中。

算分是为了排序，ES5之前使用`TF-IDF`算法进行算分，之后使用`BM25`算法


## 词频 (TF)

- term frequency: 检索词在一篇文档中出现的频率
  - 检索词出现的次数除以文档的总字数

- 相关性：简单将搜索中每一个词的TF进行想加
  - TF(区块链) + TF(的) + TF(作用)

- stop word： 没什么作用的词
  - `的` 可能会出现多次，但是他对于相关性并没有什么作用，应该不考虑他的词频， 可以作为一个 stop word

## 逆文档频率 (IDF)

- DF:检索词在所有文档中出现的频率
  - 区块链 在相对较少的文档中出现
  - 作用 在相对较多的文档
  - 的 作为 stop word 在大量的文档中出现

- inverse document frequency: 简单说 = log(存储的全部文档数/检索词出现过的文档数)
- TF-IDF 就是将 TF 求和变成了加权求和

比如，你ES里总共存储了10万条`职位信息`, 你检索 `php`，出现 php 的职位数量是1万，那么IDF = log(10万/1万) = log(10) = 1

- TF-IDF 就是算出分词后所有词的TF和 IDF 并进行处理，比如`区块链的作用` = TF(区块链) * IDF(区块链) + TF(的) * IDF(的) + TF(作用) * IDF(作用)


## BM25 

BM25对之前的TF-IDF 算法进行了一个优化，当TF的词出现的越来越多的时候，如果是TF-IDF 算法，那么分值会增加很多，而如果是 BM25 算法，则会趋于一个极限。



### boosting 

`boosting`可以对算分结果进行影响

正常搜索`php首席`的结果

```
PHP首席架构师
PHP架构师
PHP架构师001
首席软件架构师
首席软件架构师
首席科学家（科研副总经理）
首席科学家（工业传动技术）
商家端资深软件开发工程师（Go/PHP）
```

如果我们想降低`首席`这个词在搜索结果中的算分占比，可以使用`boosting`

```
GET job_ik/_search
{
  "query": {
    "boosting": {  //boosting 关键字
      "positive": {  //positive 关键字
        "match": {
          "job_name": "php"
        }
      },
      "negative": {  //negative 关键字
        "match": {
          "job_name": "首席"
        }
      },
      "negative_boost": 0.2   // 0 - 1，降低首席排序，1-100，提高首席排序
    }
  }
}
```

降低后的结果

```
PHP架构师
PHP架构师001
商家端资深软件开发工程师（Go/PHP）
PHP首席架构师
```

如果把`negative_boost`提升为2，那么结果如下

```
PHP首席架构师
PHP架构师
PHP架构师001
商家端资深软件开发工程师（Go/PHP）
```


> 极客时间 ES 学习笔记
