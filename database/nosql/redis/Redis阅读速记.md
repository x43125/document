# Redis 阅读笔记

> 目的是速记，不要拘谨

## Redis总览

- Redis命令很多，但有互通性，不可死记硬背
- Redis和其数据结构不是万金油，不可胡乱使用
- 通用命令：keys, dbsize, exists, del, expire, ttl, type
- Redis对外数据结构：string, list, hash, set, zset，每个类型都有内部的编码格式，并且不止一个，可以使用 `object encoding` 来查看键的内部编码

- 内部编码的好处：抽象内聚，更换底层编码时不会影响外部使用；适用多种情景，选择最合适的内存时间搭配

- Redis使用了单线程架构和I/O多路复用技术来实现高性能

  - 单线程还能如此之快的原因：
    - 纯内存访问：主要原因
    - 非阻塞I/O：epoll
    - 单线程避免了线程切换和竞态损耗
      - 编码，结构简单
      - 避免线程切换和竞态损耗
  - 单线程存在的问题：
    - 因为是单线程的所以多条命令需要排队，因此每条命令的执行时间不可过长

- 字符串：string

  > 字符串是最基础的数据结构，所有的键都是字符串，其他数据结构也是以字符串为基础

  - 字符串、数字、二进制，不可超过512MB

  - 命令：

    - set, set ex, set px, set nx, set xx, setnx, setex

    - get, mset, mget, incr, decr

      > mset, mget可以减少n-1次网络io时间，redis的效率很高，很多时候瓶颈在网络等其他io耗时
      >
      > 批量操作会减少网络io消耗，但要注释数量，否则可能造成redis或网络阻塞

    - append, strlen, getset, setrange, getrange, 

  > 以上操作除了m批量操作外，时间复杂度都是O(1)，这是因为redis使用了自己的字符串结构，优化了各种查询等操作，而非c语言本身的结构

  - 内部编码
    - int：8字节长整型
    - embstr：<=39字节的字符串
    - raw：>39字节

- redis典型使用场景：缓存，计数，共享session，限速

- 字典：hash

  > 键值对：key:filed:value

- - 命令
  - hset, hget, hdel, hlen, hmget, hmset, hexists, hkeys, hvals, hgetall, hscan
    
    -  hincrby, hincrbyfloat, hstrlen, 
    
  - 内部编码
    - ziplist：当元素个数<hash-max-ziplist-entries 且 所有值都<hash-max-ziplist-value值时，更加节省内存
    - hashtable：其他情况。读写速度较快
    
  - hash类型和关系型数据库的不同：
  
    - hash是稀疏的，关系型数据库是完全结构化的
    - 关系数据库可以做复杂的关联查询
  
  - 三种cache实现方式：
  
    - 字符串直接实现：user1:tom:name tom; user1:tom:birth 1998; user1:tom:sex 1
    - 序列化实现：user1:tom serialize(tom)
    - hash：user1:tom name tom birth 1998 sex 1
  
    > 各有优缺点，自行判断
  
- 列表：list

  > 存储多个有序字符串

  - 一个列表最多可存储2^32 -1个元素
  - 特点
    - 元素有序
    - 元素可重复
  - 命令：lpush, rpush, lrange, linsert, lindex, llen, lpop, rpop, lrem, ltrim, lset
  - blpop, brpop (可以做消息队列)
  - 内部编码
    - ziplist：当元素个数小于 `list-max-ziplist-entries`，且每个元素小于 `list-max-ziplist-value`时是ziplist；节约内存
    - linkedlist：否则是linkedlist；节约时间
    - quicklist：新加类型，结合了ziplist和linkedlist
  - 使用场景
    - 消息队列
    - 文章列表
  - 口诀：
    - lpush + lpop = Stack
    - lpush + rpop = Queue
    - lpush + ltrim = Capped Collection (有限集合)
    - lpush + brpop = Message Queue (消息队列)
  
- 集合：set

  > 无序，无重复

  - 一个集合最多可存储2^32 -1个元素
  - Redis支持集合内**增删改查**，还支持集合间取**交，并，差集**
  - 命令：
    - sadd, srem, scard, sismember, srandmember, spop, smembers, sscan
    - sinter, sunion, sdiff; sinterstore, sunionstore, sdiffstore
  - 内部编码
    -  intset：当元素都是整数 且 元素个数小于 `set-max-intset-entries` (默认512个)
    - hashtable：其他情况为此编码，速度快
  - 使用场景
    - 标签 （用户和标签的关系维护应该在一个事务里进行）
  - 主要场景
    - sadd = 标签
    - spop / srandmember = Random item (随机数，抽奖)
    - sadd + sinter = Social Graph (社交)

- 有序集合：zset

  > 每一个元素都有一个分数属性作为排序依据，元素不可重复，但分数可以重复

  - 命令：zadd, zadd nx, zadd xx, zadd ch, zadd incr
    - zcard, zscore, zrank, zrevrank, zrem, zincrby, zrange, zrevrange
    - zrangebyscore, zrevrangebyscore, zcount, zremrangebyrank, zremrangebyscore
    - zinter, zunion, zinterstore, zunionstore
  - 内部编码
    - ziplist：当元素个数小于`zset-max-ziplist-entries` (默认是128个)，且每个元素的值小于 `zset-max-ziplist-value` （默认是64字节）
    - skiplist：其他情况；读写速度快，占用内存大
  - 使用场景
    - 排行榜
  
- 建管理

  - rename, renamenx, randomkey
  - expire, expireat, pexpire, pexpireat, pttl, persist

- 迁移键

  - move：redis内部数据库间移动
  - dump + restore：从源向目的数据库实例移动
  - migrate：原子执行；将 dump, restore, del 三个操作组合使用；支持迁移多个键；只需要在源redis上执行即可

- 遍历键

  - keys: `glob`风格的匹配

    - keys *
    - keys hel*
    - keys ?ello
    - keys [jr]edis
    - keys \[^j]edis
    - keys [a-z]edis
    - 大量键情况下有可能阻塞

  - scan：渐进式遍历

    > 使用hashtable数据结构

    - scan cursor match type

## Redis数据结构

- hash
    - 入键过程
        - 计算键的hash值：MurmurHash3
        - 散列：
        - rehash：链表法（头插法）
        - 渐进式rehash：为防止表中数量过多，直接从tb0复制到tb1导致阻塞，采用渐进式rehash方式；旧表只减不增

- skiplist：跳表

    > 跳表：平均：O(logN)；最坏：O(N)
    >
    > 大部分情况下，效率媲美平衡树；实现比平衡树简单
    >
    > 有序集合的所有元素都存在一个跳表中
    >
    > 用处：有序集合键；集群节点中用作内部数据结构

    - zskiplist

    ```c
    typedef struct zskiplist {
        // header 跳表头节点
        // tail 跳表尾节点
        struct zskiplistNode *header, *tail;
        // 跳表长度（表头结点的层数不算在内）
        unsigned long length;
        // 跳表层数最大的那层的层数（表头结点不算在内）
        int level;
    } zskiplist;
    ```

    - zskiplistNode

    ```c
    /* ZSETs use a specialized version of Skiplists */
    typedef struct zskiplistNode {
        // 键
        sds ele;
        // 分数
        double score;
        // 后退指针
        struct zskiplistNode *backward;
        struct zskiplistLevel {
            // 前进指针
            struct zskiplistNode *forward;
            // 前进跨度
            unsigned long span;
        } level[];
    } zskiplistNode;
    ```

    ![image-20220207170328230](resources/image-20220207170328230.png)

    - 层
        - level数组包含多个元素，每个元素都包含一个指向其他节点的指针
        - 幂次定律：越大的数出现的概率越小；使用幂次定律在创建新层的时候随即指定一个层的高度







































































