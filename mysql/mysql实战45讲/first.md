# MySQL 实战45讲阅读  一

MySQL模型：server + engine

server：交互

engine：处理

![](https://files.catbox.moe/ou1hli.png)

重要的日志模块
redo log: 账本、黑板速记

![](https://files.catbox.moe/lryo3w.png)

binlog

server层产生的日志，用于归档

区别：

- redo log是InnoDB引擎特有的；binlog是server的，公有的
- redo log是物理日志，记录的是：在某个数据页上做了什么修改；binlog是逻辑日志，记录这个语句的原始逻辑，比如：给Id= 2 的这一行的c字段+1
- redo log循环写入，空间固定，会用完；binlog追加写，不会覆盖以前的日志