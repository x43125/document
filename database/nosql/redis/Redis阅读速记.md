# Redis 阅读笔记

> 目的是速记，不要拘谨

- Redis命令很多，但有互通性，不可死记硬背
- Redis和其数据结构不是万金油，不可胡乱使用
- 通用命令：keys, dbsize, exists, del, expire, ttl, type
- Redis对外数据结构：string, list, hash, set, zset，每个类型都有内部的编码格式，并且不止一个，可以使用 `object encoding` 来查看键的内部编码

**内部编码**

![image-20220203230140624](C:\Users\x43125\AppData\Roaming\Typora\typora-user-images\image-20220203230140624.png)

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

  - 字符串是最基础的数据结构，所有的键都是字符串，其他数据结构也是以字符串为基础

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

- hash：键值对：key:filed:value

  - 命令
    - hset, hget, hdel, hlen, hmget, hmset, hexists, hkeys, hvals, hgetall, hscan
    -  hincrby, hincrbyfloat, hstrlen, 
  - 内部编码
    - ziplist：当元素个数<hash-max-ziplist-entries 且 所有值都<hash-max-ziplist-value值时，更加节省内存
    - hashtable：其他情况。读写速度较快
  - 缺点：稀疏





















































































