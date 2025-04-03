---
title: MySQL零基础教程表设计实战
date: 2025-03-18 10:12:47
tags: ['数据库','数据库原理','sql','mysql']
category: mysql
article: MySQL零基础教程表设计实战
---

# 好的Prompt事半功倍


万能公式：定义角色 + 需求背景 + 实现目标 + 补充要求 + 示例

本教程为零基础教程，零基础小白也可以直接学习，有基础的可以跳到后面的原理篇学习。
基础概念和SQL已经更新完成。

接下来是应用篇，应用篇的内容大致如下图所示。

![概念学习](https://thepatterraining.github.io/images/mysql/mysql1-2.png)
 
## 表设计

表设计可以聊的点其实是比较多的，这个也比较看具体的业务、流量等。

比如，某一个字段是否应该放在这张表里？一张表里应该有哪些字段？如何设计字段类型？

甚至于，如何设计字段的顺序？

这里很多人不知道的一个点在于，字段的顺序也会影响性能，至于为什么，这个就偏低层一些了，下面会讲到。

想要做表设计，那你首先需要知道`表`是什么，所以我们先来看看表到底是什么东西。

### 表是什么？

有的人会说，表就是Navicat上看到的一张表呗，还能是什么啊？

还有的人说，表就是一行一行数据组成的。

其实说的都对，但是这是`逻辑`上的表，也就是mysql给我们展现出来的表。

大家有没有想过，表的物理形式是什么，msyql如何将它转化成逻辑上的表方便我们查看呢？

接下来进行揭秘吧！



## 总结

通常来说，ER图是在设计阶段完成的，先有ER图再有表结构。

可如果你已经有了表结构，有没有办法生成ER图呢？

也是有方法的，比如著名的`Navicat`工具，就支持这么做。

此外，还有一个方法，就是使用在线工具[dbdiagram](https://dbdiagram.io/d)，这个工具可以导入现有的SQL，会生成ER图，如下。

![概念学习](https://thepatterraining.github.io/images/mysql/mysql3-13.png)

这个网站是通过左边的一个叫`dbml`的语言来生成ER图的，也支持直接导入SQL，转化成dbml格式再生成ER图。

## 文末福利

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 发送“AI”即可领取AI学习资料。
> 部分电子书如图所示。

![概念学习](https://thepatterraining.github.io/images/bottom1.png)

![概念学习](https://thepatterraining.github.io/images/bottom2.png)

![概念学习](https://thepatterraining.github.io/images/bottom3.png)

![概念学习](https://thepatterraining.github.io/images/bottom4.png)


