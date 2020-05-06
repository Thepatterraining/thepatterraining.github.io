---
title: mysqlorderby详解-sortbuffer是什么东西
date: 2020-04-26 16:46:24
tags: ['mysql']
category: mysql
article: mysqlorderby详解-sortbuffer是什么东西
---

# mysql是怎么操作order by来进行排序的

mysql的排序用到了sort buffer，sort buffer是一个内存块。

mysql会先取出需要排序的数据，然后把数据放入sort buffer，当所有数据都放入sort buffer或者sort buffer满了就开始排序，然后将排序好的结果返回给客户端。

参数`sort_buffer_size`显示的就是sort buffer的大小。

如果数据量超过sort buffer，那么就会通过磁盘临时文件辅助进行排序，如果数据量比较小，则可以直接在内存中进行。

在内存中排序会使用`快排算法`，而通过磁盘临时文件则会使用`归并排序算法`。

排序步骤可以分为以下几步：

1. 取出select的数据存入sort buffer。
2. 在sort buffer中进行快排或者归并排序算法。
3. 如果有limit按照limit取相应的结果集进行返回。

### rowid排序算法

这里面select出来的数据量可能会很大，跟你要查询的列多少有关，如果你的列很多，那么mysql可能会使用另外一种排序方法，叫做`rowid`排序。

rowid算法不管你查询出来的结果集，它只把必要的字段放入sort buffer中，这样sort buffer就可以存入更多的数据来进行排序。

必要的字段也就是你排序需要的字段和主键字段，比如`order by time`那么他只会放入id 和 time，然后按照time字段排序完成后再通过主键id回表查询一遍数据，然后返回数据。

显然rowid算法还需要再次回表，所以效率上要低一些，所以不是mysql默认使用的排序方法。

只有当你的内存不够用，查询的列太多的时候，mysql才会使用这种算法。

排序步骤可以分为以下几步：

1. 取出排序的字段和id存入sort buffer。
2. 在sort buffer中进行快排或者归并排序算法。
3. 按照排序结果和limit数量回表查询然后返回数据。

### 不需要排序的方法

既然这样，我们为了更快速，可以避免mysql排序。

innoDB的索引是有序的，也就是说，如果我们要排序的字段本身就是有序的，那么就不用排序了。

所以我们可以在排序字段上建立索引，而如果我们查询的字段不多，甚至可以建立覆盖索引，那么速度会快很多。

mysql到底有没有进行排序，可以通过explain的执行计划来看。如果最后有using filesort，就表示使用了排序算法。

