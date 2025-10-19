---
title: 零基础MySQL教程原理篇之Buffer Pool
date: 2025-10-14 10:12:47
tags: ['数据库','数据库原理','sql','mysql']
category: mysql
article: 零基础MySQL教程原理篇之Buffer Pool
---

> 大家好，我是大头，职高毕业，现在大厂资深开发，前上市公司架构师，管理过10人团队！
> 我将持续分享成体系的知识以及我自身的转码经验、面试经验、架构技术分享、AI技术分享等！
> 愿景是带领更多人完成破局、打破信息差！我自身知道走到现在是如何艰难，因此让以后的人少走弯路！
> 无论你是统本CS专业出身、专科出身、还是我和一样职高毕业等。都可以跟着我学习，一起成长！一起涨工资挣钱！
> 关注我一起挣大钱！文末有惊喜哦！

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 发送“AI”即可领取AI学习资料。

# MySQL零基础教程

本教程为零基础教程，零基础小白也可以直接学习。
基础篇和应用篇已经更新完成。
接下来是原理篇，原理篇的内容大致如下图所示。

![原理学习](https://thepatterraining.github.io/images/mysql/mysql1-3.png)
 
## 零基础MySQL教程原理篇之缓冲池实现

MySQL中的`buffer pool(缓冲池)`是`InnoDB存储引擎`的重要组件，它负责在内存中管理数据库中的数据和索引的缓存。

它加速了数据库的运行速度，是数据库和磁盘之间的一个中间层。如果没有缓冲池，那么所有的数据库操作都需要进行磁盘IO，有了缓冲池，就不需要频繁的IO操作了。

本次我们的目标是使用`C++`实现一个缓冲池，这也是`CMU15445`这门课的`Project1`。

基于`BusTub`学术数据库管理系统构建一个面向磁盘的存储管理器。在这样的存储管理器中，数据库的主要存储位置在磁盘。

缓冲池负责在主内存和磁盘之间来回移动物理页面。它允许数据库管理系统支持比系统可用内存量更大的数据库。缓冲池的操作对系统其他部分是透明的。

例如，系统使用其唯一标识符 `（ page_id_t ）`向缓冲池请求一个页面，它不知道这个页面是否已经在内存中，或者系统是否需要从磁盘中检索它。

实现需要是线程安全的。多个线程将并发访问内部数据结构，并且必须确保它们的临界区受到`latches`的保护（在操作系统中这些被称为“锁”）。

实现总共分为三个部分：
1. 实现LRU-K淘汰策略
2. 实现磁盘调度程序
3. 实现缓冲池管理器

### 实现LRU-K淘汰策略

关于LRU-K淘汰策略的一些信息可以看上一篇文章[零基础MySQL教程原理篇之缓冲池原理及实现]()

本次实现主要都在这两个文件中：
- 头文件`src/include/buffer/lru_k_replacer.h`
- 实现文件`src/buffer/lru_k_replacer.cpp`

> LRU-K 算法会淘汰替换器中反向 k 距离最大的页帧。反向 k 距离是指当前时间戳与第 k 次先前访问的时间戳之间的时间差。少于 k 次历史访问的页帧其反向 k 距离被赋予+inf。当多个页帧具有+inf 反向 k 距离时，替换器会淘汰具有最早整体时间戳的页帧（即记录访问最久远的页帧，在所有页帧中整体上最不常被访问的页帧）。

`LRUKReplacer` 的最大大小与缓冲池的大小相同，因为它包含了 `BufferPoolManager` 中所有页帧的占位符。然而，在任意时刻，替换器中的所有页帧并非都被视为可被淘汰。 `LRUKReplacer` 的大小由可被淘汰页帧的数量表示。 `LRUKReplacer` 被初始化为空。只有当页帧被标记为可被淘汰时，替换器的大小才会增加。

总共要实现以下方法：
- Evict(frame_id_t* frame_id):将与所有其他被跟踪的可淘汰帧相比具有最大向后k距离的帧驱逐。将`帧id`存储在输出参数中并返回 True 。如果没有可淘汰帧，则返回 False 。
- RecordAccess(frame_id_t frame_id)：记录给定帧 id 在当前时间戳被访问。此方法应在页面被固定在 `BufferPoolManager` 后调用。
- Remove(frame_id_t frame_id)：清除与帧关联的所有访问历史。此方法仅在页面被删除在 `BufferPoolManager` 时调用。
- SetEvictable(frame_id_t frame_id, bool set_evictable): 此方法控制帧是否可被淘汰。它还控制 `LRUKReplacer` 的大小。当您实现 `BufferPoolManager` 时，您将知道何时调用此函数。具体来说，当页面的固定计数达到 0 时，其对应的帧被标记为可淘汰，并且替换器的大小增加。
- Size()：该方法返回当前在 `LRUKReplacer` 中的可淘汰帧数。

你可以假设不会内存耗尽，但你必须确保你的实现是线程安全的。

以上是`实现要求`.

接下来讨论一下实现的思路。关于LRU-K在数据库中的实现有一篇论文可以参考一下[LRU-K](https://dl.acm.org/doi/pdf/10.1145/170036.170081)

先来看`RecordAccess`方法吧，这个方法算是比较简单的了，也是整个LRU-K算法的入口方法，因为每次访问一个page，都需要调用该方法。

首先，我们可以看到在头文件中有`LRUKNode`的一个类型定义：
```c++
class LRUKNode {
 public:
  explicit LRUKNode(frame_id_t fid) : fid_(fid){};

  ~LRUKNode() = default;

  auto Record(size_t timestamp) -> size_t;

  auto SetEvictable(bool evictable) -> bool;

  auto GetId() const -> frame_id_t { return fid_; }

  auto GetK() const -> size_t { return k_; }

  auto GetKDistance() const -> size_t { return k_distance_; }

  auto IsEvict() const -> bool { return is_evictable_; }

  // 定义比较操作符
  auto operator==(const LRUKNode &other) const -> bool { return fid_ == other.fid_; }

  void Print();

 private:
  /** History of last seen K timestamps of this page. Least recent timestamp stored in front. */
  // Remove maybe_unused if you start using them. Feel free to change the member variables as you want.

  std::list<size_t> history_;  //记录访问历史
  size_t k_distance_;          //当前frame k distance
  size_t k_{0};
  const frame_id_t fid_;      // frame id
  bool is_evictable_{false};  // 是否可淘汰
};
```

在这里我们有一个List类型的`history_`用来记录这个Node的访问历史。`k_distance_`用来计算k距离。`fid_`表示这个Node的帧id，也就是`frame id`，`is_evictable_`代表是否可以被淘汰掉。

因此，在我们的LRUK算法中，需要有一个地方存储这些Node，这里第一个想到的是用一个List来存储，但是，这样每次查找Node都不方便，因此，可以使用一个`HashMap`来存储，定义一个`node_store_`来进行存储,这个map的key是`frame id`，value是一个LRUKNode。我们还需要记录两个大小，一个是LRU淘汰器里面可淘汰的大小，一个是最大的大小。还有最关键的两个数据结构，我们采用两个`queue`来实现真个LRUK算法，一个`lru_node_queue_`来存储没有到达K次的节点，一个`lru_k_queue_`来存储已经到达K次的节点。

因此，在`RecordAccess`方法中，我们的实现思路就是：
1. 给定一个frame id
2. 从node_store_中获取这个frame id对应的节点信息
3. 如果是第一次访问，继续往下。如果不是，跳到8处
4. 首先，我们需要为这个帧创建一个`LRUKNode`
5. 将这个节点插入node_store_中
6. 记录本次对于这个新创建LRUKNode的访问历史
7. 根据访问次数决定放入的队列，如果小于K次，放入lru_node_queue_，否则放入lru_k_queue_
8. 如果node_store_中有这个帧，代表曾经访问过，不需要创建LRUKNode了
9. 从node_store_中取出这个LRUKNode，并记录访问历史
10. 根据访问次数决定放入的队列，如果小于K次，放入lru_node_queue_，否则放入lru_k_queue_

这里面有一个公共的点在于，6、7步骤和9、10步骤，因此可以把这两个步骤抽取出来一个公共方法，我们暂时命名为`RecordHelper`方法。

接下来看`SetEvictable`这个方法，这个方法比较简单，因为只是简单的设置一个LRUKNode的状态为`可以被淘汰`。

同样的，给定一个`frame id`，我们需要先从`node_store_`中进行查询。
1. 给定一个frame id
2. 从node_store_中获取这个frame id对应的节点信息
3. 如果没有查询到，说明不在淘汰器中，因此直接返回错误或者不处理即可。
4. 如果查询到一个LRUKNode，从 node_store_ 中取出这个节点
5. 修改这个LRUKNode的状态
6. 更新可淘汰数量

对于`Size`方法是最简单的，直接返回`可淘汰数量`即可。

对于`Remove`方法，是要将一个节点从淘汰器中删除。
1. 给定一个frame id
2. 从node_store_中获取这个frame id对应的节点信息
3. 如果没有查询到，说明不在淘汰器中，因此直接返回错误或者不处理即可。
4. 如果查询到一个LRUKNode，从 node_store_ 中取出这个节点
5. 从 node_store_ 中删除这个节点
6. 因为我们维护了两个队列，因此这里要判断一下节点的K次，来决定从哪个队列中删除这个节点。

最重要的实现就是`Evict`方法了，这个是整个算法的核心实现。实现思路如下：
1. 优先从 lru_node_queue_ 中淘汰
2. 如果在 lru_node_queue_ 中，则将该节点从 lru_node_queue_ 中删除，并从 node_store_ 中删除
3. 如果 lru_node_queue_ 中没有符合条件的，再从 lru_k_queue_ 中寻找
4. 如果在 lru_k_queue_ 中，则将该节点从 lru_k_queue_ 中删除，并从 node_store_ 中删除。
5. 如果两个队列中都没有可淘汰的页面，则返回 false

### 实现磁盘调度程序

`磁盘调度程序`负责读写磁盘IO，我们需要实现下面两个文件：
- src/include/storage/disk/disk_scheduler.h
- src/storage/disk/disk_scheduler.cpp

磁盘调度器可以被其他组件（在此情况下，任务#3 中的 `缓冲池管理器` ）用来排队磁盘请求，这些请求由 `DiskRequest` 结构体表示（已在 src/include/storage/disk/disk_scheduler.h 中定义）。磁盘调度器将维护一个后台工作线程，负责处理已排队的请求。

> 磁盘调度器将使用一个共享队列来调度和处理 DiskRequests。一个线程会将请求添加到队列中，而磁盘调度器的工作线程会处理队列中的请求。我们在 src/include/common/channel.h 中提供了一个 Channel 类来促进线程间数据的安全共享，但如果你认为有必要，也可以使用自己的实现。

DiskScheduler 的构造函数和析构函数已经实现，它们负责创建和加入后台工作线程。你只需要实现以下方法:
- Schedule(DiskRequest r) : 安排 `DiskManager` 执行请求。 `DiskRequest` 结构体指定请求是读/写操作，数据应写入/读取的位置，以及操作的`page ID`。 `DiskRequest` 还包括一个 `std::promise` ，其值应在请求处理完成后设置为 true。
- StartWorkerThread() : 启动后台工作线程的方法，该方法处理计划中的请求。工作线程在 `DiskScheduler` 构造函数中创建，并调用此方法。此方法负责获取排队中的请求并将它们分派到 `DiskManager` 。记得在 `DiskRequest` 的回调中设置值，以通知请求发起人请求已完成。此方法不应在 `DiskScheduler` 的析构函数被调用之前返回。

最后， `DiskRequest` 中的一个字段是一个 `std::promise` 。如果你不熟悉 C++ 的 `promise` 和 `future`，可以查看它们的文档。在这个项目中，它们基本上为线程提供了一个回调机制，以便线程知道它们的计划请求何时完成。要查看它们可能的使用示例，可以查看 `disk_scheduler_test.cpp` 。

磁盘管理器类（ src/include/storage/disk/disk_manager.h ）负责从磁盘读取和写入页面数据。当磁盘调度器处理读或写请求时，会使用 DiskManager::ReadPage() 和 DiskManager::WritePage() 。

在头文件中，已经给我们定义了一个`request_queue_`属性，这个队列是一个`Channel`类型的，是一个并发安全的类型，可以简单理解为go中的`Channel`或者是一个并发安全的队列。

因此，对于`Schedule(DiskRequest r)`方法，最简单的一个做法就是我们接收到请求以后直接放入这个`Channel`中。
1. 直接放入`request_queue_`中

主要的处理逻辑全部放在`StartWorkerThread`这个方法里面。这里我们首先定一个变量`stop_pool_`来表示磁盘调度程序是否可以结束，如果不结束，我们就一直循环处理即可。
1. 判断stop_pool_是否结束
2. 不结束，我们从request_queue_中获取一个要处理的请求。
3. 判断该请求是否有效
4. 判断请求是写请求还是读请求
5. 如果是写请求，调用`disk manager`进行写入
6. 如果是读请求，调用`disk manager`进行读取
7. 设置返回值true

### 实现缓冲池管理器

这是最核心的一个功能了，那就是`缓冲池`的实现。

缓冲池会调用`磁盘调度程序`来完成磁盘读写请求。当被明确指示执行或需要淘汰Page以腾出空间给新Page时， `缓冲池` 还可以安排脏页写入磁盘。

系统中的所有内存页都由 Page 对象表示。 BufferPoolManager 不需要理解这些页面的内容。

但对于作为系统开发者的你来说，重要的是要理解 Page 对象只是缓冲池中内存的容器，因此它们并不特定于某个唯一页面。也就是说，每个 Page 对象包含一个内存块， DiskManager 将使用这个内存块作为位置来复制它从磁盘读取的物理页面的内容。当 BufferPoolManager 在磁盘之间来回移动时，它会重用同一个 Page 对象来存储数据。这意味着在系统的整个生命周期中，同一个 Page 对象可能包含不同的物理页面。 Page 对象的标识符（ page_id ）会跟踪它包含的物理页面；如果 Page 对象不包含物理页面，那么它的 page_id 必须设置为 INVALID_PAGE_ID 。

每个 Page 对象还维护一个计数器，用于记录有多少线程“固定”了该页。你的 BufferPoolManager 不允许释放被固定的 Page 。每个 Page 对象还跟踪它是否已变脏。在页被取消固定之前，你的工作是要记录该页是否已被修改。你的 BufferPoolManager 必须在对象被重用之前，将脏 Page 的内容写回磁盘。

你的 BufferPoolManager 实现将使用在本次作业前几步中创建的 LRUKReplacer 和 DiskScheduler 类。 LRUKReplacer 将跟踪 Page 对象何时被访问，以便在必须释放一个帧以从磁盘复制新的物理页时决定要驱逐哪一个。在 BufferPoolManager 中将 page_id 映射到 frame_id 时，再次警告：STL 容器不是线程安全的。 DiskScheduler 将在 DiskManager 上调度对磁盘的读写。

该实现涉及到两个文件：
- src/include/buffer/buffer_pool_manager.h
- src/buffer/buffer_pool_manager.cpp

我们需要实现下面的几个方法：
- FetchPage(page_id_t page_id)：如果没有空闲页且所有其他页都被固定，应返回 nullptr。 FlushPage 应无条件刷新页面，无论其固定状态如何。
- UnpinPage(page_id_t page_id, bool is_dirty)： is_dirty 参数用于跟踪页面在固定期间是否被修改。
- FlushPage(page_id_t page_id)
- NewPage(page_id_t* page_id)
- DeletePage(page_id_t page_id)
- FlushAllPages()

















## 结论

本次分享了MySQL中重要的组件`buffer pool`的概念，以及设计理念，具体实现方案，采用的淘汰策略。

并根据这些内容提出了一些性能优化方案。明白了缓冲池的原理及作用以后，根据这些优化方案可以更好的进行数据库的性能优化。

但是，buffer pool也不是越大越好，根据需要来调整，调整以后可以进行一些测试，以测试出最适合自己业务的大小。

## 文末福利

> 关注我发送“MySQL知识图谱”领取完整的MySQL学习路线。
> 发送“电子书”即可领取价值上千的电子书资源。
> 发送“大厂内推”即可获取京东、美团等大厂内推信息，祝你获得高薪职位。
> 发送“AI”即可领取AI学习资料。
> 部分电子书如图所示。

![](https://thepatterraining.github.io/images/bottom1.png)

![](https://thepatterraining.github.io/images/bottom2.png)

![](https://thepatterraining.github.io/images/bottom3.png)

![](https://thepatterraining.github.io/images/bottom4.png)
