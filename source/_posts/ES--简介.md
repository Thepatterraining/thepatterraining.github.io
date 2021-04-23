---
title: ElasticSearch一--ES简介
date: 2021-03-19
tags: ['ES']
category: ES
article: ElasticSearch一--ES简介
---

# ES--简介

## 起源

ES起源于Lucene.Lucene是基于JAVA开发的搜索引擎类库。

Lucene具有高性能，易扩展的优点。

Lucene的局限性：
1. 只能基于JAVA开发
2. 类库的接口学习曲线陡峭
3. 原生并不支持水平扩展

## ES的诞生

ES的创始人`Shay Banon`说过：`Search is something that any application should have`。

2004年基于Lucene开发了 Compass

2010年重写了 Compass ，取名 ElasticSearch
- 支持分布式，可水平扩展
- 降低全文检索的学习曲线，可以被任何编程语言调用

## ES的分布式架构

集群规模可以从单个扩展至数百个节点

高可用 & 水平扩展
    - 服务和数据两个维度

支持不同的节点类型
    - 支持 Hot & Warm架构


## 主要功能

- 海量数据的分布式存储以及集群管理
    - 服务与数据的高可用，水平扩展

- 近实时搜索，性能卓越
    - 结构化/全文/地理位置/自动完成

- 海量数据的近实时分析
    - 聚合功能

### 新特性5.x

- Lucene 6.x ，性能提升，默认打分机制从TF-IDF改为BM25
- 支持Ingest节点/ Painless Scripting / Completion suggested 支持/ 原生的JAVA REST客户端
- Type 标记成 deprecated ，支持了keyword类型
- 性能优化
    - 内部引擎移除了避免同一文档并发更新的竞争锁，带来15 - 20%的性能提升
    - Instant aggregation，支持分片上聚合的缓存
    - 新增了Profile API

### 新特性6.x

- Lucene 7.x
- 新功能
    - 跨集群复制（CCR）
    - 索引生命周期管理
    - SQL的支持
- 更友好的升级及数据迁移
    - 在主要版本之间的迁移更为简化，体验升级
    - 全新的基于操作的数据复制框架，可加快回复数据。
- 性能优化
    - 有效存储稀疏字段的新方法，降低了存储成本
    - 在索引时进行排序，可加快排序的查询性能

### 新特性7.x

- Lucene 8.x
- 重大改进 - 正式废除单个索引下多Type的支持
- 7.1开始，Security功能免费使用
- ECK - ElasticSearch Operator on Kubernetes
- 新功能
    - New Cluster coordination
    - Feature-Complete High Level Rest Client
    - Script Score Query
- 性能优化
    - 默认的Primary Shard 数从5变为1，避免Over Sharding
    - 性能优化，更快的Top K