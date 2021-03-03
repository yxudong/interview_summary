# Redis

[TOC]

### 1. Redis 和 Memecache 的区别是什么？

1. Redis 不仅仅支持简单的 k/v 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储。Memecache 支持简单的数据类型 String
2. Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用,而 Memecache 把数据全部存在内存之中
3. Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是 Redis 目前是原生支持 cluster 模式的
4. Memcached 是多线程，非阻塞 IO 复用的网络模型；Redis 使用单线程的多路 IO 复用模型



### 2. redis 常见数据结构以及使用场景分析？

1. **String 字符串**
   字符串类型是 Redis 最基础的数据结构，首先键都是字符串类型，而且其他几种数据结构都是在字符串类型基础上构建的。
   常用在缓存、计数、共享Session、限速等。
2. **Hash 哈希**
   在 Redis 中，哈希类型是指键值本身又是一个键值对结构，形如 value={{field1，value1}，...{fieldN，valueN}}。
   哈希可以用来存放用户信息，比如实现购物车。
3. **List 列表**
   列表（list）类型是用来存储多个有序的字符串。
   可以做简单的消息队列的功能。另外，可以利用 lrange 命令，做基于 Redis 的分页功能，性能极佳，用户体验好。
4. **Set 集合**
   集合（set）类型也是用来保存多个的字符串元素，但集合中不允许有重复元素，并且集合中的元素是无序的，不能通过索引下标获取元素。
   利用 Set 的交集、并集、差集等操作，可以计算共同喜好，全部的喜好，自己独有的喜好等功能。
5. **Sorted Set 有序集合**
   Sorted Set 多了一个权重参数 Score，集合中的元素能够按 Score 进行排列。
   可以做排行榜应用，取 TOP N 操作

除此之外还有 3 个高级数据结构

1. Bitmaps bitmaps 应用于信息状态统计
2. HyperLogLog 应用于基数统计
3. GEO 应用于地理位置计算



#### [ZipList](https://blog.csdn.net/zgaoq/article/details/89710600)

ZipList 为节省内存而设计，内存是连续的，没有维护双向指针：prev next，而是存储上一个 entry（可以理解为一个数据）的长度和 当前 entry 的长度，通过长度推算下一个元素在什么地方。

最大的缺点是是连锁更新问题，以时间换空间。 它不利于修改操作，插入和删除操作需要频繁的申请和释放内存。特别是当ziplist长度很长的时候，一次realloc可能会导致大批量的数据拷贝。 

![ziplist](C:/Users/zengni_wei/笔记/技术学习/images/ziplist.png)

- zlbytes: ziplist的长度（单位: 字节)，是一个32位无符号整数
- zltail: ziplist最后一个节点的偏移量，反向遍历ziplist或者pop尾部节点的时候有用。
- zllen: ziplist的节点（entry）个数
- entry: 节点
- zlend: 值为0xFF，用于标记ziplist的结尾

一个 entry 的长度为 len(previous_entry_length) + len(encoding) + len(data)
len(previous_entry_length) 为1字节或5字节
len(encoding) 为1字节、2字节或5字节

```
//空ziplist除了<zlend>的大小
#define ZIPLIST_HEADER_SIZE     (sizeof(uint32_t)*2+sizeof(uint16_t))

//<zlbyte>的指针的值，可读可写
#define ZIPLIST_BYTES(zl)       (*((uint32_t*)(zl)))

//<zltail>的指针的值
#define ZIPLIST_TAIL_OFFSET(zl) (*((uint32_t*)((zl)+sizeof(uint32_t))))

//<zllen>的指针的值
#define ZIPLIST_LENGTH(zl)      (*((uint16_t*)((zl)+sizeof(uint32_t)*2)))

/* Create a new empty ziplist. */
unsigned char *ziplistNew(void) {
    //<zlbytes>4字节<zltail>4字节<zllen>2字节<zlend>1字节，没有entry节点
    unsigned int bytes = ZIPLIST_HEADER_SIZE+1;
    
    unsigned char *zl = zmalloc(bytes);
    
    //<zlbytes>赋值
    ZIPLIST_BYTES(zl) = intrev32ifbe(bytes);
    
    //<zltail>
    ZIPLIST_TAIL_OFFSET(zl) = intrev32ifbe(ZIPLIST_HEADER_SIZE);
    
    //<zllen>
    ZIPLIST_LENGTH(zl) = 0;
    
    //<zlend>
    zl[bytes-1] = ZIP_END;
    
    return zl;
}
```



#### LinkedList

LinkedList 是由一系列不连续的内存块通过指针连接起来的双向链表。缺点是它的内存开销比较大。

首先，它在每个节点上除了要保存数据之外，还要额外保存两个指针，一个指针的大小为4字节（32位）或8字节（64位）。

其次，它的各个节点是单独的内存块，地址不连续，节点多了容易产生内存碎片。

![linkedlist](C:/Users/zengni_wei/笔记/技术学习/images/linkedlist.png)

```
typedef struct listNode {
    //前置节点
    struct listNode * prev;
    //后置节点
    struct listNode * next;
    //节点的值
    void * value;
}listNode;

typedef struct list {
    //表头节点
    listNode * head;
    //表尾节点
    listNode * tail;
    //链表所包含的节点数量
    unsigned long len;
    //节点值复制函数
    void *(*dup)(void *ptr);
    //节点值释放函数
    void (*free)(void *ptr);
    //节点值对比函数
    int (*match)(void *ptr,void *key);
} list;
```





#### QuickList

QuickList 是一个 ZipList 组成的双向链表。

![quicklist](C:/Users/zengni_wei/笔记/技术学习/images/quicklist.png)

```
typedef struct quicklistNode {
    struct quicklistNode *prev;
    struct quicklistNode *next;
    unsigned char *zl;
    unsigned int sz;             /* ziplist size in bytes */
    unsigned int count : 16;     /* count of items in ziplist */
    unsigned int encoding : 2;   /* RAW==1 or LZF==2 */
    unsigned int container : 2;  /* NONE==1 or ZIPLIST==2 */
    unsigned int recompress : 1; /* was this node previous compressed? */
    unsigned int attempted_compress : 1; /* node can't compress; too small */
    unsigned int extra : 10; /* more bits to steal for future usage */
} quicklistNode;
 
typedef struct quicklistLZF {
    unsigned int sz; /* LZF size in bytes*/
    char compressed[];
} quicklistLZF;
 
typedef struct quicklist {
    quicklistNode *head;
    quicklistNode *tail;
    unsigned long count;        /* total count of all entries in all ziplists */
    unsigned int len;           /* number of quicklistNodes */
    int fill : 16;              /* fill factor for individual nodes */
    unsigned int compress : 16; /* depth of end nodes not to compress;0=off */
} quicklist;
```

quicklistNode结构代表quicklist的一个节点，其中各个字段的含义如下：

- prev: 指向链表前一个节点的指针。
- next: 指向链表后一个节点的指针。
- zl: 数据指针。如果当前节点的数据没有压缩，那么它指向一个ziplist结构；否则，它指向一个quicklistLZF结构。
- sz: 表示zl指向的ziplist的总大小（包括`zlbytes`, `zltail`, `zllen`, `zlend`和各个数据项）。需要注意的是：如果ziplist被压缩了，那么这个sz的值仍然是压缩前的ziplist大小。
- count: 表示ziplist里面包含的数据项个数。这个字段只有16bit。稍后我们会一起计算一下这16bit是否够用。
- encoding: 表示ziplist是否压缩了（以及用了哪个压缩算法）。目前只有两种取值：2表示被压缩了（而且用的是[LZF](http://oldhome.schmorp.de/marc/liblzf.html)压缩算法），1表示没有压缩。
- container: 是一个预留字段。本来设计是用来表明一个quicklist节点下面是直接存数据，还是使用ziplist存数据，或者用其它的结构来存数据（用作一个数据容器，所以叫container）。但是，在目前的实现中，这个值是一个固定的值2，表示使用ziplist作为数据容器。
- recompress: 当我们使用类似lindex这样的命令查看了某一项本来压缩的数据时，需要把数据暂时解压，这时就设置recompress=1做一个标记，等有机会再把数据重新压缩。
- attempted_compress: 这个值只对Redis的自动化测试程序有用。我们不用管它。
- extra: 其它扩展字段。目前Redis的实现里也没用上。



#### hashtable

 字典采用hashtable实现，碰撞采用拉链法，也就是通过链表联结key的hash值相同的节点。 

![hashtable](C:/Users/zengni_wei/笔记/技术学习/images/hashtable.jpg)

```
//hash节点
typedef struct dictEntry {
    void *key;//键，一半是sds
    union {//联合体牛逼，导致值可以多用途，可以存储指针或存储整数或浮点数。必定可以存储上面类型之前，通过不同的引用，告诉编译器如何解释对应的内存块。
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;//下一个节点，解决碰撞冲突
} dictEntry;

//定义hash需要使用的函数
typedef struct dictType {//定义函数指针，例如使用的hash函数
    uint64_t (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
} dictType;//定义对应的函数指针，例如定义hash函数

/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
//定义一个hash桶，用来管理hashtable
typedef struct dictht {//管理hashtable
    dictEntry **table;//指针数组，这个hash的桶
    unsigned long size;//元素个数
    unsigned long sizemask;//
    unsigned long used;//
} dictht;

//字典，管理两个dicht，为什么需要定义两个，为了在扩容的时候使用。
typedef struct dict {//管理两个dictht，主要用于动态扩容。
    dictType *type;//函数管理
    void *privdata;
    dictht ht[2];
    long rehashidx; /* 扩容标志rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;

/* If safe is set to 1 this is a safe iterator, that means, you can call
 * dictAdd, dictFind, and other functions against the dictionary even while
 * iterating. Otherwise it is a non safe iterator, and only dictNext()
 * should be called while iterating. */
typedef struct dictIterator {//简单的迭代器封装，STL内部也是这么实现
    dict *d;//字典
    long index;//索引
    int table, safe;
    dictEntry *entry, *nextEntry;//当前元素，下一个元素
    /* unsafe iterator fingerprint for misuse detection. */
    long long fingerprint;
} dictIterator;
```



### 3. Redis String 的实现原理

Redis 内部 String 类型采用 SDS（simple dynamic string）表示

```
struct sdshdr {
    // buf 已占用长度
    int len;
    // buf 剩余可用长度
    int free;
    // 实际保存字符串数据的地方
    char buf[];
};
```

对比 C 字符串， SDS 有以下特性：

             1. 可以高效地执行长度计算（strlen）， 直接取 len 字段
             2. buf 存储内容溢出的问题，首先判断 free 字段是否足够
             3. 可以高效地执行追加操作（append），通过预分配空间，free 字段
             4. 二进制安全，不以 \0 做为结尾标识



### 4. Redis List 的实现原理

Redis3.2 之前，List 底层采用了 ZipList 和 LinkedList 实现的，在 3.2 之后，List 底层采用了 QuickList。

Redis3.2 之前，初始化的 List 使用的 ZipList，当以下两个条件任意一个不满足时，则会被转换成 LinkedList：

      1. List 中存储的每个元素的长度小于 64byte（可以通过配置文件修改）
      2. 元素个数小于 512（可以通过配置文件修改）



### 5. Redis Hash 的实现原理

Redis 的哈希对象的底层存储可以使用 ziplist 和 hashtable。

当 Hash 对象可以同时满足以下两个条件时，哈希对象使用 ziplist 编码：

      1. 哈希对象保存的所有键值对的键和值的字符串长度都小于 64 字节（可以通过配置文件修改）
      2. 哈希对象保存的键值对数量小于 512 个（可以通过配置文件修改）

hashtable 编码的哈希表对象底层使用字典数据结构。

Redis 解决哈希冲突的方式，是链式哈希。

这里存在一个问题，哈希冲突链上的元素只能通过指针逐一查找再操作。

如果哈希表里写入的数据越来越多，哈希冲突可能也会越来越多，会导致某些哈希冲突链过长，进而导致这个链上的元素查找耗时长。

所以，Redis 会对哈希表做 rehash 操作。

#### 渐进式的 rehash

rehash 使用两个 hashtable 1 和 hashtable 2。
随着数据逐步增多，Redis 开始执行 rehash，这个过程分为三步：

    1. 给 hashtable 2 分配更大的空间，例如是当前 hashtable 1 大小的两倍；
    2. 把 hashtable 1 中的数据重新映射并拷贝到 hashtable 2 中；
    3. 释放 hashtable 1 的空间。

在第 2 步 Redis 不是一次性把全部数据 rehash 成功，这样会导致 Redis 对外服务停止，Redis 内部为了处理这种情况采用渐进式的rehash。

Redis 将所有的 rehash 的操作分成多步进行，直到都 rehash 完成。

    1. 为 hashtable 2 分配空间，让字典同时持有 hashtable 1 和 hashtable 2 两个哈希表。
    2. 在字典中维持一个索引计数器变量rehashidx，并将它的值设置为0，表示rehash
       工作正式开始。
    3. 在rehash进行期间，**每次对字典执行添加、删除、查找或者更新操作时，程序除
       了执行指定的操作以外，**还会顺带将 hashtable 1 在 rehashidx 索引上的所有键值对
       rehash到 hashtable 2 ，当rehash工作完成之后，程序将rehashidx+1(表示下次将rehash下一个桶)。
    4. 随着字典操作的不断执行，最终在某个时间点上， hashtable 1 的所有键值对都会被rehash至 hashtable 2 ，这时程序将rehashidx属性的值设为-1，表示rehash操作已完成。



### 6. Redis Set 的实现原理

Set 集合采用 intset（整数集合）和 hashtable 两种方式来实现，当满足以下两个条件的时候，采用 intset 实现，
一旦有一个条件不满足时则采用 hashtable 来实现：

       1. Set 集合中的所有元素都为整数
       2. Set 集合中的元素个数不大于 512（可以通过配置文件修改）



### 7. Redis Sorted Set 的实现原理

Zset 底层同样采用了两种方式来实现，分别是 ZipList 和 SkipList。当同时满足以下两个条件时，采用 ZipList 实现；反之采用 SkipList 实现：
       1. Zset 中保存的元素个数小于 128（可以通过配置文件修改）
              2. Zset 中保存的所有元素长度小于 64byte（可以通过配置文件修改）

**采用 ZipList 的实现原理**
    和 List 的底层实现有些相似，对于 Zset 不同的是，其存储是以键值对的方式依次排列，键存储的是实际 value，值存储的是 value 对应的分值。

**采用 SkipList 的实现原理**
    SkipList 编码的 Zset 底层为一个被称为 zset 的结构体，这个结构体中包含一个字典和一个跳跃表。
    跳跃表按 score 从小到大保存所有集合元素，查找时间复杂度为平均 O(logN)，最坏 O(N) 。
    字典保存从 member 到 score 的映射，这样就可以用 O(1) 的复杂度来查找 member 对应的 score 值。
    跳表是一种并联的链表，它在链表的基础上增加了跳跃功能，正是这个跳跃的功能，使得在查找元素时，跳表能够提供 O(logN) 的时间复杂度。



### 8. Redis 中的 Bitmaps

bitmaps 不是一个真实的数据结构。而是 String 类型上的一组面向 bit 操作的集合。
常用命令：
    setbit key offset value
    getbit key offset
场景举例：

    1. 统计活跃用户（用户登陆情况）
        使用日期作为 key，然后用户 id 为 offset，如果当日活跃过就设置为 1
       2. 统计每天某一部电影是否被点播 统计每天有多少部电影被点播 统计每周/月/年有多少部电影被点播 统计年度哪部电影没有被点播
        日期作为 key，然后电影 id 为 offset，如果点播过就设置为 1



### 9. Redis 中的 HyperLogLog？

Redis HyperLogLog 是用来做**基数统计**的算法，优点是在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。
但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

比如实现 统计 APP 或网页的一个页面，每天有多少用户点击进入的次数，同一个用户的反复点击进入记为 1 次。

#### 什么是基数?

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

#### 命令有

```
    PFADD key element [element ...]
        添加指定元素到 HyperLogLog 中。
    PFCOUNT key [key ...]
        返回给定 HyperLogLog 的基数估算值。
    PFMERGE destkey sourcekey [sourcekey ...]
        将多个 HyperLogLog 合并为一个 HyperLogLog
```



### 10. Redis 过期策略

1. 定期删除，Redis 默认每隔 **100ms** 检查，是否有过期的 key，有过期 key 则删除。
   需要说明的是，Redis 不是每隔 100ms 将所有的 key 检查一次，而是**随机抽取**进行检查(如果每隔 100ms，全部 key 进行检查，Redis 岂不是卡死)。
   因此，如果只采用定期删除策略，会导致很多 key 到时间没有删除。
2. 惰性删除，也就是说在你获取某个 key 的时候，Redis 会检查一下，这个 key 如果设置了过期时间那么是否过期，如果过期了此时就会删除。

过期策略存在的问题，由于 Redis 定期删除是随机抽取检查，不可能扫描清除掉所有过期的 key 并删除，某些 key 由于未被请求，惰性删除也未触发。
这样 Redis 的内存占用会越来越高。此时就需要内存淘汰机制。



### 11. Redis 内存淘汰机制

- no-eviction：默认策略，不淘汰数据；大部分写命令都将返回错误（DEL 等少数除外）。意思是当内存不足以容纳新入数据时，新写入操作就会报错。采用no-enviction策略可以保证数据不被丢失。
- allkeys-lru：从所有数据中根据 LRU 算法挑选数据淘汰
- volatile-lru：从设置了过期时间的数据中根据 LRU 算法挑选数据淘汰
- allkeys-random：从所有数据中随机挑选数据淘汰
- volatile-random：从设置了过期时间的数据中随机挑选数据淘汰
- volatile-ttl：从设置了过期时间的数据中，挑选越早过期的数据进行删除
- allkeys-lfu：从所有数据中根据 LFU 算法挑选数据淘汰（4.0 及以上版本可用）
- volatile-lfu：从设置了过期时间的数据中根据 LFU 算法挑选数据淘汰（4.0 及以上版本可用）



### 12. LRU 与 LFU 的区别

**LRU**：Least Recently Used，最近最少使用
**LFU**：Least Frequently Used，使用频率最少的（最不经常使用的）
如果一条数据仅仅是突然被访问（有可能后续将不再访问），在 LRU 算法下，此数据将被定义为热数据，最晚被淘汰。
但实际生产环境下，我们很多时候需要计算的是一段时间下 key 的访问频率，淘汰此时间段内的冷数据。

LFU 算法相比 LRU，在某些情况下可以提升数据命中率，使用频率更多的数据将更容易被保留。

| 对比项         | 近似LRU算法         | LFU 算法                                             |
| -------------- | ------------------- | ---------------------------------------------------- |
| 最先过期的数据 | 最近未被访问的      | 最近一段时间访问的最少的                             |
| 适用场景       | 数据被连续访问场景  | 数据在一段时间内被连续访问                           |
| 缺点           | 新增 key 将占据缓存 | 历史访问次数超大的 key 淘汰速度取决于 lfu-decay-time |



### 13. Redis 的 LRU 实现

Redis 中的 LRU 与常规的 LRU 实现并不相同，常规 LRU 会准确的淘汰掉**队头**的元素，但是 Redis 的 LRU 并不维护队列，只是根据配置的策略要么从所有的 key 中随机选择 N 个（N 可以配置），要么从所有的设置了过期时间的 key 中选出 N 个键，然后再从这 N 个键中选出最久没有使用的一个 key 进行淘汰。

**为什么要使用近似 LRU？**

1. 性能问题，由于近似 LRU 算法只是最多随机采样 N 个 key 并对其进行排序，如果是精准 LRU 则需要对所有 key 进行排序，这样近似 LRU 性能更高。
2. 内存占用问题，Redis 对内存要求很高，会尽量降低内存使用率，如果是抽样排序可以有效降低内存的占用
3. 实际效果基本相等，如果请求符合长尾法则，那么真实 LRU 与 Redis LRU 之间表现基本无差异.



### 14. 如何保证 Redis 中存放的都是热点数据

限定 Redis 占用的内存，Redis 会根据自身数据淘汰策略，留下热数据到内存。
所以，计算一下热点数据大约占用的内存，然后设置一下 Redis 内存限制，并根据业务场景修改淘汰策略。



### 15. 热点 key 问题

热点 key 问题就是，突然有几十万甚至更大的请求去访问 Redis 上的某个特定 key。
那么，这样会造成流量过于集中，达到 Redis 单实例瓶颈（一般是 10W OPS 级别），或者物理网卡上限，从而导致这台 Redis 的服务器 Hold 不住。

#### 怎么发现热 key？

1. 凭借业务经验，进行预估哪些是热 key

2. 在客户端进行收集

3. 在 Proxy 层做收集

4. 用 Redis 自带命令

   1. monitor 命令，该命令可以实时抓取出 Redis 服务器接收到的命令，然后写代码统计出热 key 是啥。

      ​	当然，也有现成的分析工具可以给你使用，比如 redis-faina。但是该命令在高并发的条件下，有内存增暴增的隐患，还会降低 Redis 的性能。

   2. hotkeys 参数（必须配合 LFU），Redis 4.0.3 提供了 redis-cli 的热点 key 发现功能，执行 redis-cli 时加上 –hotkeys 选项即可。

      ​	但是该参数在执行的时候，如果 key 比较多，执行起来比较慢。

5. 自己抓包评估，Redis 客户端使用TCP协议与服务端进行交互，通信协议采用的是 **RESP**。自己写程序监听端口，按照 RESP 协议解析数据，进行分析。
   缺点就是开发成本高，维护困难，有丢包可能性。

#### 如何解决？

1. 设置二级缓存（推荐）
2. 利用分片算法的特性，对 key 进行打散处理
   hot key 之所以是 hot key，是因为它只有一个 key，落地到一个实例上。
   可以给 hot key 加上前缀或者后缀，把一个 hotkey 的数量经过分片分布到不同的实例上，将访问量均摊到所有实例。



### 16. 大 key 问题

由于 Redis 主线程为单线程模型，大 key 也会带来一些问题，如：

1. 集群模式在 slot 分片均匀情况下，会出现数据和查询倾斜情况，部分有大 key 的 Redis 节点占用内存多，QPS 高。
2. 大 key 相关的删除或者自动过期时，会出现 qps 突降或者突升的情况，极端情况下，会造成主从复制异常，Redis 服务阻塞无法响应请求。

#### 怎么发现大 key？

- Redis 4.0 之前的大 key 的发现与删除方法
  1. redis-rdb-tools 工具。Redis 实例上执行 bgsave，然后对 dump 出来的 rdb 文件进行分析，找到其中的大 key
  2. redis-cli --bigkeys 命令。可以找到某个实例 5 种数据类型（string、hash、list、set、zset）的最大 key
  3. 自定义的扫描脚本，以 Python 脚本居多，方法与 redis-cli --bigkeys 类似
- Redis 4.0 之后的大 key 的发现与删除方法
  Redis 4.0 引入了 `memory usage` 命令和 `lazyfree` 机制，不管是对大 key 的发现，还是解决大 key 删除或者过期造成的阻塞问题都有明显的提升。
  `memory usage` 可以用较小的代价去获取所有key的内存大小。

#### 如何删除大 key？

- Redis 4.0 之前的大 key 的发现与删除方法
  分解删除操作，把 大的 key 分解成小部分逐渐删除：
  list: 逐步 ltrim;
  zset: 逐步 zremrangebyscore
  hset: hscan，然后 hdel 删除
  set: sscan，然后 srem 删除
- Redis 4.0 之后的大 key 的发现与删除方法
  删除大key： lazyfree机制
  unlink 命令，代替 DEL 命令，会把对应的大 key 放到 BIO_LAZY_FREE 后台线程任务队列，然后在后台异步删除。



### 17. 如何保证缓存与数据库双写时的数据一致性？

对于缓存和数据库的操作，主要有以下两种方式。

#### 先删缓存，再更新数据库

先删除缓存，数据库还没有更新成功，此时如果读取缓存，缓存不存在，去数据库中读取到的是旧值，缓存不一致发生。
**解决方案：**
延时双删
延时双删的方案的思路是，为了避免更新数据库的时候，其他线程从缓存中读取不到数据，就在更新完数据库之后，再 sleep 一段时间，然后再次删除缓存。
 sleep 的时间要对业务读写缓存的时间做出评估，sleep 时间大于读写缓存的时间即可。

#### 先更新数据库，再删除缓存

更新数据库成功，如果删除缓存失败或者还没有来得及删除，那么，其他线程从缓存中读取到的就是旧值，还是会发生不一致。

**解决方案：**

1. 消息队列

   先更新数据库，成功后往消息队列发消息，消费到消息后再删除缓存，借助消**息队列的重试机制**来实现，达到最终一致性的效果。
    缺点：引入消息中间件之后，问题更复杂，就算更新数据库和删除缓存都没有发生问题，消息的延迟也会带来短暂的不一致性，不过这个延迟相对来说还是可以接受的

2. 进阶版消息队列
   为了解决缓存一致性的问题单独引入一个消息队列，太复杂。
   其实，一般大公司本身都会有监听 binlog 消息的消息队列存在，主要是为了做一些核对的工作。
   这样，我们可以借助监听 binlog 的消息队列来做删除缓存的操作。
   这样做的好处是，不用你自己引入，侵入到你的业务代码中，中间件帮你做了解耦，同时，中间件的这个东西本身就保证了高可用。
   当然，这样消息延迟的问题依然存在，但是相比单纯引入消息队列的做法更好一点。

3. 设置缓存过期时间
   每次放入缓存的时候，设置一个过期时间，比如 5 分钟，以后的操作只修改数据库，不操作缓存，等待缓存超时后从数据库重新读取。
   如果对于一致性要求不是很高的情况，可以采用这种方案。

**为什么是删除，而不是更新缓存？**

如果是更新的话，那就是先更新数据库，再更新缓存。

举个例子：如果数据库 1h内更新了 1000 次，那么缓存也要更新 1000 次，但是这个缓存可能在 1h 内只被读取了 1 次，那么这 1000 次的更新没有必要。反过来，如果是删除的话，就算数据库更新了 1000 次，那么也只是做了 1 次缓存删除，只有当缓存真正被读取的时候才去数据库加载。



### 18. Redis 如何实现异步队列？

一般使用 list 结构作为队列，rpush 生产消息，lpop 消费消息。当 lpop 没有消息的时候，要适当 sleep 一会再重试。

**如果不用 sleep 呢？**

list 还有个指令叫 blpop，在没有消息的时候，它会阻塞住直到消息到来。

**能不能生产一次消费多次？**

使用 pub/sub 主题订阅者模式，可以实现 1:N 的消息队列。

**pub/sub 有什么缺点？**

在消费者下线的情况下，生产的消息会丢失，改为使用专业的消息队列如 RocketMQ 等。



### 19. Redis 如何实现延时队列？

使用 sortedset，拿时间戳作为 score，消息内容作为 key 调用 zadd 来生产消息，消费者用 zrangebyscore 指令获取 N 秒之前的数据轮询进行处理。



### 20. Pipeline有什么好处，为什么要用 pipeline？

可以将多次 IO 往返的时间缩减为一次，并且减少 Redis 中系统调用



### [21. Redis 如何实现分布式锁？](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/)

Redis 锁主要利用 Redis 的 setnx 命令。

加锁命令：**SETNX key value**，当键不存在时，对键进行设置操作并返回成功，否则返回失败。KEY 是锁的唯一标识，一般按业务来决定命名。

解锁命令：**DEL key**，通过删除键值对释放锁，以便其他线程可以通过 SETNX 命令来获取锁。

锁超时：**EXPIRE key timeout**, 设置 key 的超时时间，以保证即使锁没有被显式释放，锁也可以在一定时间后自动释放，避免资源被永远锁住。

#### 存在的问题

##### 1. SETNX 和 EXPIRE 非原子性

使用 lua 脚本

##### 2. 锁误解除

如果线程 A 成功获取到了锁，并且设置了过期时间 30 秒，但线程 A 执行时间超过了 30 秒，锁过期自动释放，此时线程 B 获取到了锁；

随后 A 执行完成，线程 A 使用 DEL 命令来释放锁，但此时线程 B 加的锁还没有执行完成，线程 A 实际释放的线程 B 加的锁。

**解决办法：**通过在 value 中设置当前线程加锁的标识，在删除之前验证 key 对应的 value 判断锁是否是当前线程持有。可生成一个 UUID 标识当前线程，使用 lua 脚本做验证标识和解锁操作。

##### 3. 超时解锁导致并发

如果线程 A 成功获取锁并设置过期时间 30 秒，但线程 A 执行时间超过了 30 秒，锁过期自动释放，此时线程 B 获取到了锁，A 和 B 并发执行。

A、B 两个线程发生并发显然是不被允许的，一般有两种方式解决该问题：


       1. 将过期时间设置足够长，确保代码逻辑在锁释放之前能够执行完成。
       2. 为获取锁的线程增加守护线程，为将要过期但未释放的锁增加有效时间。

##### 4. 不可重入

当线程在持有锁的情况下再次请求加锁，如果一个锁支持一个线程多次加锁，那么这个锁就是可重入的。

如果一个不可重入锁被再次加锁，由于该锁已经被持有，再次加锁会失败。

Redis 可通过对锁进行重入计数，加锁时加 1，解锁时减 1，当计数归 0 时释放锁。

Redis Hash 数据结构来实现分布式锁，既存锁的标识也对重入次数进行计数。

##### 5. 无法等待锁释放

上述命令执行都是立即返回的，如果客户端可以等待锁释放就无法使用。

1. 可以通过客户端轮询的方式解决该问题，当未获取到锁时，等待一段时间重新获取锁，直到成功获取锁或等待超时。
   这种方式比较消耗服务器资源，当并发量比较大时，会影响服务器的效率。
2. 另一种方式是使用 Redis 的发布订阅功能，当获取锁失败时，订阅锁释放消息，获取锁成功后释放时，发送锁释放消息。

##### 6. 集群

主备切换、集群脑裂时会造成问题，使用 RedLock 算法



### 22.缓存击穿

缓存击穿的概念就是单个 key 并发访问过高，过期时导致所有请求直接打到 db 上.
这个和热 key 的问题比较类似，只是说的点在于过期导致请求全部打到 DB 上而已。

解决方案：
       1. 加锁更新，比如请求查询 A，发现缓存中没有，对 A 这个 key 加锁，同时去数据库查询数据，写入缓存，再返回给用户，这样后面的请求就可以从缓存中拿到数据了。
       2. 将过期时间组合写在 value 中，通过异步的方式不断的刷新过期时间，防止此类现象。



### 23. 缓存穿透

缓存穿透是指查询不存在缓存中的数据，每次请求都会打到DB，就像缓存不存在一样。

还有一种情况是，缓存和数据库中都没有的数据，而用户不断发起请求，如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

#### 解决方案

##### 1. 加一层布隆过滤器。

布隆过滤器的原理是在你存入数据的时候，会通过散列函数将它映射为一个位数组中的 K 个点，同时把他们置为 1。这样当用户再次来查询 A，而 A 在布隆过滤器值为 0，直接返回，就不会产生击穿请求打到 DB 了。

使用布隆过滤器之后会有一个问题就是**误判**，因为它本身是一个数组，可能会有多个值落到同一个位置。

理论上来说只要我们的数组长度够长，误判的概率就会越低，这种问题就根据实际情况来就好了

BloomFilter 用 Bitmap 实现，[参考链接](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/)

##### 2. 接口层增加校验

接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；

##### 3. 设置无效值

从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击



### 24. 缓存雪崩

当某一时刻发生大规模的缓存失效的情况，比如你的缓存服务宕机了，会有大量的请求进来直接打到 DB 上，这样可能导致整个系统的崩溃，称为雪崩。

雪崩和击穿、热 key 的问题不太一样，是指大规模的缓存都过期失效了。

**解决方案：**

1. 针对不同 key 设置不同的过期时间，避免同时过期

2. 限流，如果 Redis 宕机，可以限流，避免同时刻大量请求打崩 DB。
3. 二级缓存。
4. 如果缓存数据库是分布式部署，将热点数据均匀分布在不同的缓存数据库中。
5. 设置热点数据永远不过期。



### 25. Redis 并发竞争key问题如何解决?

1. 分布式锁
2. 消息队列



### 26. 为什么 Redis6.0 之后改用多线程?

Redis 使用多线程并非是完全摒弃单线程。

Redis 还是使用单线程模型来处理客户端的请求，只是使用多线程来处理数据的读写和协议解析，执行命令还是使用单线程。

这样做的目的是因为 Redis 的性能瓶颈在于网络 IO 而非 CPU，使用多线程能提升 IO 读写的效率，从而整体提高 Redis 的性能。



### 27. Redis 的持久化方式

Redis 的持久化主要有两大机制，即 **AOF(Append Only File)** 日志和 **RDB(Redis DataBase)** 快照。

#### RDB 优缺点

##### 优点

           1. RDB是一个紧凑压缩的二进制文件，代表 Redis 在某个时间点上的数据快照。非常适用于备份，全量复制等场景。
           2. 与 AOF 格式的文件相比，RDB 文件可以更快的重启。
           3. RDB 对灾难恢复非常有用，它是一个紧凑的文件，可以更快的传输到远程服务器进行 Redis 服务恢复。

##### 缺点

           1. RDB 方式数据没办法做到实时/秒级持久化，因为 bgsave 每次运行都要执行 fork 操作创建子进程，属于重量级操作，频繁执行成本过高。
           只能保存某个时间间隔的数据，如果在这个期间 Redis 故障了，就会丢失一段时间的数据。

#### AOF 优缺点

#### 优点：

           1. AOF 持久化保存的数据更加完整，即使发生了意外情况，根据配置的保存策略只会丢失短时间内的数据（每次操作保存的话不会丢失）；
           2. OF 持久化文件，非常容易理解和解析，它是把所有 Redis 键值操作命令，以文件的方式存入了磁盘。即使不小心使用 flushall 命令删除了所有信息，只要使用 AOF 文件，删除最后的 flushall 命令，重启 Redis 即可恢复之前误删的数据。

#### 缺点

      1. 对于相同的数据集来说，AOF 文件要大于 RDB 文件
      2. Redis 负载比较高的情况下，RDB 比 AOF 性能更好
      3. 重启恢复数据时不如 RDB 速度快

Redis 4.0 之后新增混合持久化方式，混合持久化是结合了 RDB 和 AOF 的优点，在写入的时候，先把当前的数据以 RDB 的形式写入文件的开头，
再将后续的操作命令以 AOF 的格式存入文件，这样既能保证 Redis 重启时的速度，又能减低数据丢失的风险。



### 28. Redis AOF 日志原理

AOF 日志是写后日志，“写后”的意思是 Redis 是先执行命令，把数据写入内存，然后才记录日志。

为了避免额外的检查开销，Redis 在向 AOF 里面记录日志的时候，并不会先去对这些命令进行语法检查。

如果先记日志再执行命令的话，日志中就有可能记录了错误的命令。

所以，Redis 使用写后日志这一方式的一大好处是，可以避免出现记录错误命令的情况。

除此之外，AOF 还有一个好处：它是在命令执行后才记录日志，所以不会阻塞当前的写操作。

#### 两种潜在的风险

1. 如果刚执行完一个命令，还没有来得及记日志就宕机了，那么这个命令和相应的数据就有丢失的风险。
2. AOF 虽然避免了对当前命令的阻塞，但可能会给下一个操作带来阻塞风险。
   这是因为，AOF 日志也是在主线程中执行的，如果在把日志文件写入磁盘时，磁盘写压力大，就会导致写盘很慢，进而影响后续的操作。

#### 三种写回策略

       1. **Always**，同步写回：每个写命令执行完，立马同步地将日志写回磁盘；
       2. **Everysec**，每秒写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘；
       3. **No**，操作系统控制的写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，由操作系统决定何时将缓冲区内容写回磁盘。

想要获得高性能，就选择 No 策略。如果想要得到高可靠性保证，就选择 Always 策略。如果允许数据有一点丢失，又希望性能别受太大影响的话，那么就选择 Everysec 策略。

#### 重写机制

AOF 文件过大之后再往里面追加命令记录的话，效率会变低，如果日志文件太大，发生宕机恢复过程也会非常缓慢，所以会有 AOF 重写机制。

AOF 重写机制指的是，对过大的 AOF 文件进行重写，以此来压缩AOF文件的大小。

具体的实现是检查当前键值数据库中的键值对，记录键值对的最终状态，将对某个键值对重复操作后产生的多条操作记录压缩成一条，实现压缩 AOF 文件的大小。

#### 重写过程

一个拷贝，两处日志

总结来说，每次 AOF 重写时，Redis 会先执行一个内存拷贝，用于重写；

然后，使用两个日志保证在重写过程中，新写入的数据不会丢失。

而且，因为 Redis 采用额外的线程进行数据重写，所以，这个过程并不会阻塞主线程。



### 29. Redis  RDB 快照

Redis 提供了两个命令来生成 RDB 文件，分别是 save 和 bgsave

**save**：在主线程中执行，会导致阻塞；

**bgsave**：创建一个子进程，专门用于写入 RDB 文件，避免了主线程的阻塞，这也是 Redis RDB 文件生成的默认配置。

Redis 借助操作系统提供的写时复制技术（Copy-On-Write, COW），在执行快照的同时，正常处理写操作。

简单来说，bgsave 子进程是由主线程 fork 生成的，可以共享主线程的所有内存数据。

bgsave 子进程运行后，开始读取主线程的内存数据，并把它们写入 RDB 文件。

此时，如果主线程对这些数据也都是读操作（例如图中的键值对 A），那么，主线程和 bgsave 子进程相互不影响。

但是，如果主线程要修改一块数据，那么，这块数据就会被复制一份，生成该数据的副本。

然后，bgsave 子进程会把这个副本数据写入 RDB 文件，而在这个过程中，主线程仍然可以直接修改原来的数据。

如果频繁地执行全量快照，也会带来两方面的开销：

1. 频繁将全量数据写入磁盘，会给磁盘带来很大压力，多个快照竞争有限的磁盘带宽，前一个快照还没有做完，后一个又开始做了，容易造成恶性循环。

   2. fork 这个创建过程本身会阻塞主线程，而且主线程的内存越大，阻塞时间越长。



### 30. Redis 事务机制

    1. multi——开启事务
    2.命令入队列
        之后所有的命令都会放入事务队列中，并不会立刻执行。
        如果客户端发送的命令为 EXEC，DISCARD 的其中一个，服务器会立刻执行这个命令。
        对于其它命令，服务器并不会立刻执行，而是将这个命令放入一个事务队列中，然后向客户端返回QUEUED回复
    3. exec——执行事务
    4. DISCARD——放弃执行
    
    Redis 的事务机制可以保证一致性和隔离性（watch 机制）。
    持久性取决于是否开启持久化以及持久化机制，极端情况下还是无法保证。
    原子性的情况比较复杂：
        1. 命令入队时就报错，会放弃事务执行，保证原子性
        2. 命令入队时没报错，实际执行时报错，不保证原子性
        3. EXEC 命令执行时实例故障，如果开启了 AOF 日志，可以保证原子性
          （使用 redis-check-aof 工具检查 AOF 日志文件，这个工具可以把未完成的事务操作从 AOF 文件中去除。
           使用 AOF 恢复实例后，事务操作不会再被执行，从而保证原子性。
        只有当事务中使用的命令语法有误时，原子性得不到保证，在其它情况下，事务都可以原子性执行。



#### 31. Redis 主从同步机制

    主从模式是最简单的实现高可用的方案，核心就是主从同步。主从同步的原理如下：
    
    1. slave 发送 sync 命令到 master
    2. master 收到 sync 之后，执行 bgsave，生成 RDB 全量文件
    3. master 把 slave 的写命令记录到缓存
    4. bgsave 执行完毕之后，发送 RDB 文件到 slave，slave 执行
    5. master 发送缓存中的写命令到 slave，slave 执行

<p align='center'>
    <img src='./images/Redis 主从同步机制.jpg'>
</p>

    上面写的命令是 sync，但是在 Redis 2.8 版本之后已经使用 psync 来替代 sync 了，
    原因是 sync 命令非常消耗系统资源，而 psync 的效率更高（增量同步）。
    主从模式下，当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这需要人工干预，费事费力，还会造成一段时间内服务不可用。
    这种方式并不推荐，实际生产中，优先考虑哨兵模式。
    
    优缺点：
       优点：
           1. 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离
           2. Slave 同样可以接受其他 Slaves 的连接和同步请求，这样可以有效地分载 Master 的同步压力
       缺点：
           1. Redis 不具备自动容错和恢复功能，主从不可以自动切换
           2. 主机宕机，宕机前有部分数据未能及时同步到从机，切换 IP 后还会引入数据不一致的问题，降低了系统的可用性
           3. 较难支持在线扩容



#### 32. Redis 哨兵

    哨兵其实就是一个运行在特殊模式下的 Redis 进程，主从库实例运行的同时，它也在运行。
    哨兵主要负责的就是三个任务：监控、选主（选择主库）和通知。
    
    监控：
       哨兵进程会使用 PING 命令检测它自己和主、从库的网络连接情况，用来判断实例的状态。
       如果哨兵发现主库或从库对 PING 命令的响应超时了，那么，哨兵就会先把它标记为“主观下线”。
       如果检测的是从库，那么，哨兵简单地把它标记为“主观下线”就行了。
       但是，如果检测的是主库，那么，哨兵还不能简单地把它标记为“主观下线”，开启主从切换，因为主从切换开销很大，防止误判。
       只有 N/2 + 1 的哨兵实例，都判断主库已经“主观下线”了，主库才会被标记为“客观下线”，
    选主：
       主库挂了以后，在多个从库中，先按照一定的筛选条件，把不符合条件的从库去掉。
       然后，再按照一定的规则，给剩下的从库逐个打分，将得分最高的从库选为新主库。
       筛选过程除了要检查从库的当前在线状态，还要判断它之前的网络连接状态。
       在 down-after-milliseconds 毫秒内，主从节点都没有通过网络联系上，就可以认为主从节点断连了。
       如果发生断连的次数超过了 10 次，就说明这个从库的网络状况不好，不适合作为新主库。
       打分过程分别按照三个规则依次进行三轮打分，这三个规则分别是从库优先级、从库复制进度以及从库 ID 号。
    通知：
       在执行通知任务时，哨兵会把新主库的连接信息发给其他从库，让它们执行 replicaof 命令，和新主库建立连接，并进行数据复制。
       同时，哨兵会把新主库的连接信息通知给客户端，让它们把请求操作发到新主库上。
       客户端和哨兵之间的事件通知通过哨兵自身的 pub/sub 功能实现

<p align='center'>
    <img src='./images/Redis 哨兵.jpg'>
</p>

    优缺点：
       优点：
           1. 主从模式的所有优点
           2. 主从可以自动切换，系统更健壮，可用性更高
       缺点：
           除了支持主从自动切换外的主从模式的所有缺点



#### 33. Redis 集群

    从 Redis 3.0 开始，官方提供了一个名为 Redis Cluster 的方案，用于实现切片集群。
    Redis Cluster 采用哈希槽（Hash Slot），来处理数据和实例之间的映射关系。
    
    在 Redis Cluster 中，一个切片集群共有 16384 个哈希槽，这些哈希槽类似于数据分区，每个键值对都会根据它的 key，被映射到一个哈希槽中。
    
    为什么采用 16384？
    
        1. 在 Redis 节点发送心跳包时需要把所有的槽放到这个心跳包里，以便让节点知道当前集群信息，16384=16k，
           在发送心跳包时使用 char 进行 bitmap 压缩后是 2k（2 * 8 (8 bit) * 1024(1k) = 16K），也就是说使用 2k 的空间创建了 16k 的槽数。
           虽然使用 CRC16 算法最多可以分配 65535（2^16-1）个槽位，65535=65k，压缩后就是 8k（8 * 8 (8 bit) * 1024(1k) =65K），
           也就是说需要需要 8k 的心跳包，作者认为这样做不太值得
        2. 并且一般情况下一个 Redis 集群不会有超过 1000 个 master 节点，所以 16k 的槽位是个比较合适的选择
    
    具体的映射过程分为两大步：
        1. 首先根据键值对的 key，按照 CRC16 算法计算一个 16bit 的值
        2. 然后，再用这个 16bit 值对 16384 取模，得到 0~16383 范围内的模数，每个模数代表一个相应编号的哈希槽
    
    哈希槽和实力的对应：
        部署 Redis Cluster 时，可以使用 cluster create 命令创建集群，此时，Redis 会自动把这些槽平均分布在集群实例上。
        也可以使用 cluster meet 命令手动建立实例间的连接，形成集群，再使用 cluster addslots 命令，指定每个实例上的哈希槽个数。
        当 16384 个 slot 都有节点在处理时，集群处于上线状态，反之只要有一个 slot 没有得到处理都会处理下线状态。
    
    客户端如何定位数据？
        当客户端请求键值对时，会先计算键所对应的哈希槽，然后给相应的实例发送请求。
        在定位键值对数据时，它所处的哈希槽是可以通过计算得到的，这个计算可以在客户端发送请求时来执行。
        知道哈希槽后，客户端如何知道哈希槽分布在哪个实例上？
            Redis 实例会把自己的哈希槽信息发给和它相连接的其它实例，来完成哈希槽分配信息的扩散。
            当实例之间相互连接后，每个实例就有所有哈希槽的映射关系了。
            客户端和集群实例建立连接后，实例就会把哈希槽的分配信息发给客户端。客户端收到哈希槽信息后，会把哈希槽信息缓存在本地。
    
    Redis 集群间通信参用什么协议？
        gossip 协议通信
    
    在集群中，实例和哈希槽的对应关系并不是一成不变的，最常见的变化有两个：
        1. 在集群中，实例有新增或删除，Redis 需要重新分配哈希槽
        2. 为了负载均衡，Redis 需要把哈希槽在所有实例上重新分布一遍。
    此时，实例可以通过相互传递消息，获得最新的哈希槽分配信息，但是，客户端是无法主动感知这些变化的。
    这就会导致，它缓存的分配信息和最新的分配信息不一致。
    
    Redis Cluster 采取重定向机制解决这个问题。
    当客户端把一个键值对的操作请求发给一个实例时，如果这个实例上并没有对应的哈希槽，会给客户端返回 MOVED 响应结果，结果中包含了新实例的访问地址。
    如果此时，旧实例中的数据只有一部分迁移到了新实例，还有部分数据没有迁移，客户端会收到一条 ASK 报错信息。
    代表这个哈希槽正在迁移。此时，客户端需要先给新实例发送一个 ASKING 命令（代表破例执行关于槽的命令一次）。
    然后，客户端再向新实例发送 GET 命令，以读取数据。
    
    故障发现和转移：
        如果节点 A 向节点 B 发送 ping 消息，节点 B 没有在规定的时间内响应 pong，那么节点 A 会标记节点 B 为 pfail 疑似下线状态，
        同时把 B 的状态通过消息的形式发送给其他节点，如果超过半数以上的节点都标记 B 为 pfail 状态，B 就会被标记为 fail 下线状态，此时将会发生故障转移。
        优先从复制数据较多的从节点选择一个成为主节点，并且接管下线节点的 slot，整个过程和哨兵非常类似，都是基于 Raft 协议做选举。
        但是如果下线的主节点没有从节点，整个集群还是处于不可用的状态。
    
    Redis Cluster 并不能保证数据的强一致性，在实际中集群在特定的条件下可能会丢失写操作，原因是集群采用异步复制。



#### 34. Redis 中，sentinel和 cluster 的区别和适用场景是什么？

    哨兵是解决了 Redis 的高可用，而 cluster 则是解决了 Redis 的高并发。
