---
title: ElasticSearch三--ES-基本概念-节点集群分片副本
date: 2021-04-23
tags: ['ES']
category: ES
article: ElasticSearch三--ES-基本概念-节点集群分片副本
---

# 基本概念

## 分布式系统的可用性与扩展性

高可用性

- 服务可用性：允许有节点停止服务。
- 数据可用性：部分节点丢失，不会丢失数据。

可扩展性

- 请求量提升/数据的不断增长（将数据分布到所有节点上）

## 分布式特性

Elasticsearch 的分布式架构好处
- 存储的水平扩容
- 提高系统的可用性，部分节点停止服务，整个集群的服务不受影响

Elasticsearch的分布式架构
- 不同的集群通过不同的名字来区分，默认名字“Elasticsearch”
- 通过配置文件修改，或者在命令行中 -E cluster.name=xxx来设置
- 一个集群可以有一个或多个节点

## 节点

节点是一个Elasticsearch 的实例
- 本质上就是一个JAVA进程
- 一台机器上可以运行多个Elasticsearch进程，但是线上建议一个机器一个节点

每一个节点都有名字，通过配置文件配置，或者启动时 -E node.name=xxx 来设置

每个节点在启动之后会有一个UID，保存在data目录下

### Master-eligible nodes 和 Master Node

每个节点启动后，默认是一个 Master eligible 节点。
- 可以设置 node.master:false 禁止
Master-eligible 节点可以参加选主流程，成为 master 节点

当第一个节点启动时候，它会将自己选举成为 master 节点

每个节点上都保存了集群的状态，只有 master 节点才能修改。
- 集群状态(Cluster State)，维护了一个集群中必要的信息
    - 所有的节点信息
    - 所有的索引和其相关的 Mapping 与 Settings 信息
    - 分片的路由信息
- 任意节点都可以修改会导致数据的不一致


### Data Node 和 Coordinating Node

Data Node

- 可以保存数据的节点，叫做Data Node。 负责保存分片数据。在数据扩展上起到了至关重要的作用。

Coordinating Node
- 负责接受Client 的请求，将请求分发到合适的节点，最终把结果汇集到一起。
- 每个节点默认都起到了Coordinating Node的职责


### 其他的节点类型

Hot & Warm Node
- 不同硬件配置的Data Node ，用来实现 Hot & Warm 架构，降低集群部署的成本。

Machine Learning Node
- 负责跑机器学习的Job,用来做异常检测

Tribe Node
- (5.3开始使用 Cross Cluster Search) Tribe Node 连接到不同的Elasticsearch 集群，并且支持将这些集群当成一个单独的集群处理

### 配置节点类型

生产环境建议一个节点担任一个角色

|节点类型|配置参数|默认值|
|:----    |:---|:----- |
|maste eligible| node.master| true|
|data          | node.data  | true|
|ingest        | node.ingest| true|
|coordinating only| 无| 每个节点默认都是coordinating节点，设置其他类型全部为false|
|machine learning| node.ml | true(需要 enable x-pack)|

## 分片（Primary Shard & Replica Shard）

主分片，用以解决数据水平扩展的问题。通过主分片，可以将数据分布到集群内的所有节点之上。
- 一个分片是一个运行的Lucene的实例
- 主分片数在索引创建时指定，后续不允许修改，除非Reindex

副本，用以解决数据高可用的问题。分片是主分片的拷贝
- 副本分片数，可以动态调整
- 增加副本数，还可以在一定程度上提高服务的可用性（读取的吞吐）

### 分片的设定

对于生产环境，需要提前做好容量规划

分片数设置过小
- 导致后续无法增加节点实现水平扩展
- 单个分片的数据量太大，导致数据重新分配耗时

分片数设置过大，7.0开始，默认分片设置为1，解决了over-sharding的问题
- 影响搜索结果的相关性打分，影响统计结果的准确性
- 单个节点上过多的分片，会导致资源浪费，同时也会影响性能

### 查询集群的健康状态

使用GET方法访问 _cluster/health

```
GET /_cluster/health
```

或者访问 `http://localhost:9200/_cluster/health`

Green : 集群健康
Yello : 主分片分配，副本未分配
Red   : 主分片未分配



