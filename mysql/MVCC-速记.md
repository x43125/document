# MVCC

> MVCC主要是为了解决数据库的**读写并发**问题

数据库可以通过 MVCC + 悲观锁/乐观锁 来解决 读写冲突 + 写写冲突

快照读 & 当前读

主要依靠：**4个隐式字段**，**undo日志** ，**Read View** 来实现

## 4个隐式字段

> DB_ROW_ID, DN_TRX_ID, DB_ROLL_PTR, DELETED_BIT

DB_ROW_ID: 隐含的自增ID（隐藏主键）

> MySQL有一个 _rowid 键，如果该表有整型主键或有非空唯一整型索引的话则 _rowid 直接与之关联
>
> 如果没有以上两种的话，则InnoDB会自动以DB_ROW_ID产生一个聚簇索引 从0到 2^48 - 1，如果超过 2^48-1会从0重新开始，所以会有覆盖旧数据的情况
>
> 在InnoDB中有一个全局变量 `dictsys.row_id` 所有DB_ROW_ID共享这个变量，每插入一条需要DB_ROW_ID的记录的时候，DB_ROW_ID会拿这个全局变量当作自己的主键，然后再自增这个全局变量

DN_TRX_ID: 最近修改/插入事务的ID

DB_ROLL_PTR: 回滚指针，指向这条记录的上一个版本

DELETED_BIT: 记录被更新或删除，不代表真的删除，只是删除flag变了

## undo log

为了能够回滚而记录的这些数据称之为 `undo log`

> 在查询的时候不会产生修改操作，所以无需记录 undo log

undo log 主要分为三种：

> insert undo log，update undo log，delete undo log

insert undo log：至少要记录这条记录的主键，以便回滚

update undo log：至少要记录这条记录修改前的全部旧值，以便回滚后修改为旧值

delete undo log：至少要记录这条记录删除前的全部旧值，以便回滚后将原值再重新插入表中

> 删除操作一般只是记录下老记录的删除标记`DELETED_BIT`并非真正删除
>
> 需要再研究：具体机制
>
> 为了节省磁盘空间，InnoDB有专门的purge线程来清理DELETED_BIT为true的记录。为了不影响MVCC的正常工作，purge线程自己也维护了一个read view（这个read view相当于系统中最老活跃事务的read view）;如果某个记录的DELETED_BIT为true，并且DB_TRX_ID相对于purge线程的read view可见，那么这条记录一定是可以被安全清除的。
>
> ------
>
> 著作权归@pdai所有 原文链接：https://pdai.tech/md/db/sql-mysql/sql-mysql-mvcc.html

所以，主要就是update undo log

我们先来看一个update的例子：

### 示例：



## Read View

