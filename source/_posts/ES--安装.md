---
title: ElasticSearch二--ES安装
date: 2021-03-25
tags: ['ES']
category: ES
article: ElasticSearch二--ES安装
---

# ES--安装

## 安装

- 运行ElasticSearch, 需安装并配置JDK
    - 设置 $JAVA_HOME

- 各个版本对 Java的依赖
    - ElasticSearch 5 需要 Java 8 以上的版本
    - ElasticSearch 从6.5开始支持 Java 11
    - https://www.elastic.co/support/matrix#matrix_jvm
    - 7.0开始，内置了Java环境

从官网（https://www.elastic.co/cn/downloads/elasticsearch)可以进行下载安装

## 目录说明

- bin : 脚本文件，包括启动 ElasticSearch，安装插件。运行统计数据等。
- config : 配置文件：elasticsearch.yml。集群配置文件，user,role based相关配置
- JDK: Java运行环境
- data: path.data 数据文件
- lib : Java类库
- logs: path.log 日志文件
- modules: 包含所有ES模块
- plugins: 包含所有已安装插件

## JVM配置

- 修改JVM - config/jvm.options
    - 7.1 下载的默认设置是1GB

- 配置的建议
    - Xmx 和 Xms设置成一样
    - Xmx 不要超过机器内存的50%
    - 不要超过 30GB - https://www.elastic.co/blog/a-heap-of-trouble

## 启动

#### 单节点启动

```
/bin/elasticsearch
```

启动后可以访问 `localhost:9200` 进行查看信息

#### 多节点启动

cluster.name 指定同一个集群，node.name指定每个节点的名称，path.data指定数据存放位置

```
/bin/elasticsearch -E node.name=node1 -E cluster.name=test -E path.data=node1_data -d
/bin/elasticsearch -E node.name=node2 -E cluster.name=test -E path.data=node2_data -d
/bin/elasticsearch -E node.name=node3 -E cluster.name=test -E path.data=node3_data -d
```

启动完成以后可以直接访问`localhost:9200/_cat/nodes`查看node信息


## 安装插件

使用elasticsearch-plugin进行插件的安装和查看

```
/bin/elasticsearch-plugin list   //查询已安装的插件列表
/bin/elasticsearch-plugin install analysis-icu  //安装analysis-icu插件
```

还可以通过api进行查看已安装的插件，访问`localhost:9200/_cat/plugins`可以查看已安装的插件

