---
title: 零基础MySQL教程原理篇之hash索引原理
date: 2025-10-17 10:12:47
tags: ['数据库','数据库原理','sql','mysql']
category: mysql
article: 零基础MySQL教程原理篇之hash索引原理
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
 
## 零基础MySQL教程原理篇之hash索引原理

MySQL中的`索引`是一个非常重要的组件，面试中最常问的一个问题就是`你是如何优化索引的？`

> 一天，小美遇到了张三，小美说她老公嫌弃她们家衣柜衣服多又乱，早上出门找衣服要找半天！小美问张三有没有好男人介绍一下，哭诉生活太难了
> 张三当即表示我张三就是个好男人啊，能文能武，还会收拾衣柜！
> 张三告诉小美，要想保持衣柜整洁，一定要记得以下三点。。。

我们本次要讲解就是`hash索引`。

### 索引是什么？

`索引`是一种数据结构，它通过特定的方式排列数据，以便于快速查询。可以把索引想象成一本书的目录，通过目录可以快速找到你想要的章节，而不用从头到尾翻书。

在MySQL中，索引通常是通过B树或B+树等数据结构实现的，数据库查询引擎可以利用这些结构快速查找数据。

我们先看一下索引都有哪些类型，最常用的当然是b+树索引啦
- B+树索引：B+树是一种多路自平衡的树结构，所有的叶子节点都在同一层，并且叶子节点通过链表连接。它是MySQL中最常用的索引类型，支持范围查询、精确查找等操作。
- 哈希索引：哈希索引基于哈希表的原理，通过哈希函数对索引列进行计算，得到一个哈希值，哈希值对应的桶中存储数据的指针。
- 全文索引：全文索引是专门用于文本检索的索引结构，通常用于`TEXT`类型的数据。MySQL会将文本字段中的每个单词进行索引，创建倒排索引，以提高文本搜索的效率。
- 空间索引: 空间索引是用于地理空间数据类型（如`POINT`、`LINESTRING`、`POLYGON`）的索引结构，通常使用R树（或改进版的R+树）来存储和查询空间数据。
- 倒排索引：倒排索引主要用于文本搜索。它将文档中每个词的出现位置存储在一个列表中，从而支持高效的文本检索。
- 位图索引：位图索引使用位图（bitmap）来表示数据列中各个值的存在情况。例如，假设某一列有3个不同的值，可以用3个位来表示每个值的出现情况。

### hash索引

`what`什么是hash索引呢？
- hash索引就是通过`hash函数`将数据库中存储的值a映射成一个值b，然后通过该值a进行搜索的时候，可以快速找出这条记录

`why`为什么要使用hash索引呢？
- 很多人都知道b+树索引，那么为什么还要有hash索引呢？
- 其实，hash索引的性能是要高于b+树索引的，但是hash索引的使用范围要小一些，这也是现在很多人使用b+树索引的原因
- hash索引通过hash表来进行查找，时间复杂度为O（1），性能相当可以。

`When`什么时候使用hash索引呢？
- 一些内存存储引擎会使用hash索引
- InnoDB的自适应系统检测热点
- 执行精确相等查询

注意，只有`精确相等查询`才适合使用hash索引，并且性能要比b+索引更好。

比如根据手机号、用户名、邮箱精确的查询用户信息。
下面列出一些使用场景：
1. 幂等查询，有些操作是要求幂等的，因此会存储一些幂等键。
2. session id进行会话管理。session id可以使用hash 索引。
3. 配置信息获取，配置信息通常使用唯一的配置key获取，可以使用hash索引。
4. 验证码存储及获取，验证码的校验一般都是精确查询，也可以使用hash索引。

> 大家还知道哪些场景适合hash索引呢？

性能对比示例
假设有100万条用户会话记录

```sql
  -- 使用哈希索引的MEMORY表
  SELECT * FROM user_sessions WHERE session_id = 'target_session';
  -- 响应时间: ~0.001秒

  -- 如果使用B+树索引的InnoDB表
  SELECT * FROM user_sessions_innodb WHERE session_id = 'target_session';
  -- 响应时间: ~0.01秒
```

为什么这些场景适合哈希索引？

1. 都是精确匹配查询 (= 操作符)
2. 数据量适中 (适合内存存储)
3. 访问频率极高 (性能要求苛刻)
4. 数据生命周期短 (临时性数据)
5. 不需要范围查询 (没有 BETWEEN, >, < 操作)
6. 不需要排序 (没有 ORDER BY 操作)

这些场景的共同点是：快速、精确、临时、高频 - 这正是哈希索引的优势所在！

#### 实现原理

首先，需要通过一个hash函数来进行映射，因此，选择合适的hash函数也是比较重要的。

> 最快的hash函数应该是facebook的xxhash

常见的一些hash函数如下：
1. 简单的hash函数
这应该是最简单的了，直接取模就可以了
```c
// 简单取模
unsigned int simple_hash(int key) {
    return key % TABLE_SIZE;
}

// 加法哈希
unsigned int add_hash(char *str) {
    unsigned int hash = 0;
    while (*str) {
        hash += *str++;
    }
    return hash;
}
```

2. 经典的一些hash函数

- DJB2 Hash
- MurmurHash
- Google开发的CityHash

##### MurmurHash

MurmurHash 是一种`非加密型哈希函数`，适用于一般的哈希检索操作。由`Austin Appleby`在2008年发明，并出现了多个变种，都已经发布到了公有领域(public domain)。与其它流行的哈希函数相比，对于规律性较强的key，MurmurHash的随机分布特征表现更良好.

当前的版本是`MurmurHash3`，能够产生出32-bit或128-bit哈希值。

较早的`MurmurHash2`能产生32-bit或64-bit哈希值。对于大端存储和强制对齐的硬件环境有一个较慢的MurmurHash2可以用。`MurmurHash2A` 变种增加了`Merkle–Damgård` 构造，所以能够以增量方式调用。 有两个变种产生64-bit哈希值：MurmurHash64A，为64位处理器做了优化；MurmurHash64B，为32位处理器做了优化。MurmurHash2-160用于产生160-bit 哈希值，而MurmurHash1已经不再使用。

最初的实现是基于C++的，不过现在主流的语言都支持了，包括Java、Python、Golang、Php、JS、Ruby等。

算法伪代码
```shell
Murmur3_32(key, len, seed)
    c1 
←
{\displaystyle \gets } 0xcc9e2d51
    c2 
←
{\displaystyle \gets } 0x1b873593
    r1 
←
{\displaystyle \gets } 15
    r2 
←
{\displaystyle \gets } 13
    m 
←
{\displaystyle \gets } 5
    n 
←
{\displaystyle \gets } 0xe6546b64
 
    hash 
←
{\displaystyle \gets } seed

    for each fourByteChunk of key
        k 
←
{\displaystyle \gets } fourByteChunk

        k 
←
{\displaystyle \gets } k * c1
        k 
←
{\displaystyle \gets } (k << r1) OR (k >> (32-r1))
        k 
←
{\displaystyle \gets } k * c2

        hash 
←
{\displaystyle \gets } hash XOR k
        hash 
←
{\displaystyle \gets } (hash << r2) OR (hash >> (32-r2))
        hash 
←
{\displaystyle \gets } hash * m + n

    with any remainingBytesInKey
        remainingBytes 
←
{\displaystyle \gets } SwapEndianOrderOf(remainingBytesInKey)
        remainingBytes 
←
{\displaystyle \gets } remainingBytes * c1
        remainingBytes 
←
{\displaystyle \gets } (remainingBytes << r1) OR (remainingBytes >> (32 - r1))
        remainingBytes 
←
{\displaystyle \gets } remainingBytes * c2

        hash 
←
{\displaystyle \gets } hash XOR remainingBytes
 
    hash 
←
{\displaystyle \gets } hash XOR len

    hash 
←
{\displaystyle \gets } hash XOR (hash >> 16)
    hash 
←
{\displaystyle \gets } hash * 0x85ebca6b
    hash 
←
{\displaystyle \gets } hash XOR (hash >> 13)
    hash 
←
{\displaystyle \gets } hash * 0xc2b2ae35
    hash 
←
{\displaystyle \gets } hash XOR (hash >> 16)
```

##### City Hash

简而言之，这是一种非加密哈希算法，比 `MurmurHash` 更快，但更复杂。可以参考[Google的City Hash](https://opensource.googleblog.com/2011/04/introducing-cityhash.html)

Google发布了两个函数，`CityHash64` 和 `CityHash128`。它们分别将字符串哈希为 `64` 位和 `128` 位的哈希码。这些函数不适合用于密码学，但根据google目前的经验，它们非常适合用于哈希表。

Google试图针对谷歌数据中心中常见的 CPU 进行优化，但结果表明大多数台式机和笔记本电脑也具备相关特性。其中重要的是 64 位寄存器、指令级并行和快速非对齐内存访问。

City Hash方法的关键优势在于大多数步骤都包含至少两个独立的数学运算。现代 CPU 通常在这种类型的代码上表现最佳。不利之处在于代码比大多数流行替代方案更复杂。

总的来说，Google认为 `CityHash64` 和 `CityHash128` 是解决经典问题的新颖方法。在真实条件下，预计 `CityHash64` 的速度将至少比以往工作快 `30%`，甚至可能快一倍。此外，据我们所知，这些函数的统计特性是可靠的。请大胆尝试这个快速的新代码！

City Hash的Github仓库地址：[City Hash](https://github.com/google/cityhash)

> 像Click House就使用了City Hash，并在这个基础上做了一些改变


##### xxhash

xxHash 是一种极其快速的非加密哈希算法，运行在内存速度极限。它有四种变体（XXH32、XXH64、XXH3_64bits 和 XXH3_128bits）。最新的变体 XXH3 在各方面都提供了性能提升，特别是在处理小数据时。

详细的可以查看[xxhash](https://xxhash.com/)

#### hash schema

静态Hash table
- liner probe hashing
    - 如果要插入的位置有值了，就往下扫描，扫描到空的位置插入
    - 删除的时候可以增加一个`墓碑`标记，这样就知道这里是有数据的不是空，查找的时候就会继续往下扫描而不会是没找到
    - 删除的时候还可以把后面的数据往前移动，但是这样有的数据就不再原来的位置了，就找不到了。因为只会往下扫描不会往上扫描
- robin hood hashing
    - 记录`距离数`，表示插入的位置和应该插入的位置的距离。从0开始。
    - 插入的时候判断距离数，进行`劫富济贫`，如果你向下扫描到距离数为3的地方插入，而在距离数为2的地方的数据x，x的距离数比你小，比如是0，1.那么你就占据这里，你插入距离数为2的地方，而将x插入你下面，x的距离数会+1.
    - 从整体来看，这个方法牺牲了插入的效率，将数据的距离数变得更加平均
- cuckoo hashing
    - 该方法使用两个或多个`hash table`来记录数据，对A进行两次hash，得出两个hash table中的插入位置，随机选择一个进行插入
    - 如果选择的插入位置已经有数据了，就选择另一个插入
    - 如果两个都有数据了，就占据一个，然后对这个位置上之前的数据B再次hash选择其余位置。

动态hash table
- chained hashing
    - 把所有相同hash的组成一个bucket链表，然后一直往后面增加
    - java的hash table默认就是这样的
- extendible hashing
    - 对 chained hashing 的扩展
    - 有一个slot array，在slot array上有一个 counter, 如果counter = 2，代表看hash以后的数字的前两个bit,slot array就有4个位置，分别是00,01,10,11
    - 每个slot指向一个bucket
    - hash以后找到前两位对应的slot指向的bucket，将数据放进去，如果满了，放不下了就进行拆分
    - 将slot array的counter扩容为3，看前3个bit，slot array变成了8个位置
    - 只将这个满了的bucket拆分成2个，其余的不变，重新进行slot的映射
    - 再次hash这个值，看前3个bit找到对应的slot,在找到对应的bucket，然后插入进去
- linear hashing
    - 对 extendible hashing 的扩展
    - 去掉了 conter，因为他每次加1，都会扩容一倍
    - 增加了`split point`，一开始指向0，然后每次`overflow`需要拆分的时候就拆分split point指向的那个bucket，然后slot array只扩容一个，这个时候出现第二个hash函数并将split point+1
    - 查询的时候如果slot array的位置小于split point，就使用第二个hash函数，因为被拆分了
    - 如果大于等于split point，就使用第一个hash函数

### hash索引的一些优化手段

`hash索引`仅用于使用`=或<=>`运算符的相等比较（但非常快）。它们不用于查找值范围的比较运算符，如`<`。依赖于这种单值查找的系统被称为“键值存储”;要将MySQL用于此类应用程序，请尽可能使用散列索引。

优化器不能使用哈希索引来加速`ORDER BY`操作。

MySQL无法确定两个值之间大约有多少行（这是由范围优化器用来决定使用哪个索引）。如果您将MyISAM或InnoDB表更改为哈希索引的MEMORY表，这可能会影响某些查询。

只能使用整个键来搜索行。

## 结论

本次分享了MySQL中重要的概念Hash索引，接下来会出一个Hash索引具体实现的文章。

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
