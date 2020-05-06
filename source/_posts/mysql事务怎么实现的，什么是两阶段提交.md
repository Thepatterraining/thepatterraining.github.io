---
title: mysql事务怎么实现的，什么是两阶段提交
date: 2020-04-17 18:34:10
tags: ['mysql']
category: mysql
article: mysql事务怎么实现的，什么是两阶段提交
---

# mysql事务怎么实现的，什么是两阶段提交

我们都知道使用mysql的事务，准确来说是innoDB引擎的事务，可以保证数据的一致性，原子性等。那么为什么呢？

### redo log

innoDB里面用到了一个叫做`redo log`（重做日志）的东西。

当你进行数据库操作的时候，innoDB并不会直接操作磁盘数据，因为这样很慢很慢。它使用了`wal`的机制，所有的操作先记录在redo log。等没事干了或者redo log满了再把数据刷到磁盘里面。这样的话他刷盘的时候就是顺序io，避免了使用随机io。redo log的大小我们可以通过参数设置。

### 缓冲池

在innoDB里面还有一个缓冲池，buffer_pool，这个缓冲池是在内存中的。为啥有缓冲池，因为磁盘的速度太慢了，如果每次都直接操作磁盘，那就完犊子了。就需要用到内存来缓冲一下，提升性能。

从innoDB里面读取数据的时候，实际上读取的是缓冲池，只有当缓冲池中没有记录的时候，才会读取磁盘，并且将记录放入缓冲池方便下次读取。

缓冲池的大小可以通过`innodb_buffer_pool_size`参数来指定。

缓存池中基本有以下组件：
- 索引页
- 数据页
- undo页
- 插入缓冲（insert buffer)
- 自适应哈希索引
- 锁信息
- 数据字典
- redo log buffer

缓冲池既然是在内存中，有固定大小，就会有淘汰机制，缓冲池使用的是LRU机制，也就是最近最少使用的会被淘汰掉。

这里面还有一个redo log buffer，这里面存储的就是上面说的redo log的信息了，会先写入这个buffer，然后再刷入redo log里面。有一个参数`innodb_log_buffer_size`来控制redo log buffer的大小。这个不用太大，因为一般每秒都会把这里面的刷新到redo log。

redo log buffer刷新到redo log的三种情况：
- 每秒刷新
- 事务提交时候刷新
- 当redo log buffer剩余空间少于一半的时候刷新

### binlog

redo log是innoDB引擎的日志，那mysql自己同样有日志，就是这个binlog日志。binlog 里面记载了mysql的所有变化，可以用来恢复数据库，创建备库，从库等。可以查看我的[mysql读写分离](https://blog.csdn.net/Thepatterraining/article/details/105248917)。

那为什么有两个日志呢，这是因为mysql最开始默认使用的是myisam引擎，myisam引擎没有redo log，mysql就有自己的binlog。innoDB引擎是后来加入mysql的。所以他们各自有各自的日志。

这两个日志有什么区别呢？

1. rodo log是innoDB引擎的，binlog 是mysql的，binlog所有引擎都可以使用。
2. redo log记录的是“再某个数据页上修改了啥”，binlog记录的是你的sql。
3. redo log是固定大小，循环写入的，binlog没有大小限制，不断追加。

### 事务

那一个事务怎么执行的呢，比如更新一个登录时间。

```sql
update user set login_time = ? where id = 1;
```

- 走完了前面的连接器，分析器，优化器，执行器。执行器会去innoDB更新数据了。
- 他会先取出这条数据，取出id = 1的数据，id是主键索引，和数据在一起。innoDB直接查找主键索引树找到对应数据就返回了。当然，如果数据再缓冲池，那么直接返回了。
- 执行器拿到以后再修改登陆时间，然后再次调用innoDB引擎的接口。
- innoDB把数据更新到内存中，同时更新redo log，把redo log标记成待提交状态。
- 执行器写入binlog。
- 执行器调用innoDB引擎的接口，innoDB把redo log标记成提交状态。

这个时候整个更新事务完成。

为什么要这么做呢？

因为有两个日志的原因，所以两个日志都要写入，要保证这两个日志的一致性。那么如果不使用两阶段提交的方式，直接写入redo log然后写入binlog有什么问题呢？

假设，写完redo log，系统挂了。那么重启后innoDB引擎会根据redo log日志来恢复数据库。这时候数据库里面的数据是正确的。但是binlog丢失了啊。如果你有从库，那么从库的数据就错误了。因为从库的数据是通过binlog同步的。

如果把这两个步骤反过来呢，先写入binlog 再写入redo log呢？

那么就会redo log丢失，数据库实际上没有更新。但是从库通过binlog更新了。还是数据不一致。

所以需要`两阶段提交`来保证数据一致性。如果这时候写完redo log后挂掉了，因为redo log和binlog都没有数据，所以会回滚事务。
如果binlog和redo log都写入了，但是没有提交，那么重启后会提交事务。这样binlog和数据库就都有数据了。




