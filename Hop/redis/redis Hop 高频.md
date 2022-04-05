# Redis面试高频

1. 什么是Redis
2. Redis的特点，nosql特点
3. Redis的优点，与其他有什么不同，有哪些好处，相比memcached有什么优势，memcached与Redis的区别都有哪些
4. 持久化机制，优缺点，使用场景，适合场景
5. Redis常见性能问题，解决方案
6. 过期键删除策略，淘汰策略，数据库里有2000w数据，redis中有20w，如何保证Redis中的是热点数据
7. 回收策略
8. Redis同步机制
9. pipeline有什么好处，为什么要用pipeline
10. Redis集群的原理是什么，主从复制模型
11. Redis集群方案什么情况下会导致整个集群不可用
12. Java客户端：Jedis、Redisson、lettuce，分别有什么特点，有什么优缺点
13. Redis hash槽
14. Redis事务
15. 内存优化
16. 一亿个key，其中10万个key是以某个固定的前缀开头的，如何将他们全部找出来；keys 指令会有什么问题，scan的区别，优缺点
17. 如果有大量key在同一时间过期，会发生什么
18. Redis分布式锁