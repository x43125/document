# Tomcat Overview

## 常用参数

- acceptCount: accept队列的长度；默认值为100；当accept队列中的连接数达到acceptCount之后的请求将都被拒绝
- maxConnections: Tomcat在任意时刻接收的最大连接数；当连接数达到maxConnections时，Acceptor将不会再从mq中读取连接，accept队列中的连接将一直被阻塞，直到连接数小于maxConnections。如果maxConnections设置为-1，则指不受限制，可以无限读。默认值与使用的实现有关：NIO=10000; APR/native=8192; BIO=maxThreads.
- maxThreads: 请求的最大处理线程数；默认值是200；