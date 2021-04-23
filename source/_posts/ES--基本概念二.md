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