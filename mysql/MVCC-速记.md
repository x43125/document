# MVCC

> MVCC主要是为了解决数据库的**读写并发**问题

数据库可以通过 MVCC + 悲观锁/乐观锁 来解决 读写冲突 + 写写冲突

快照读 & 当前读

主要依靠：**4个隐式字段**，**undo日志** ，**Read View** 来实现

4个隐式字段：DB_ROW_ID, DN_TRX_ID, DB_ROLL_PTR, DELETED_BIT

DB_ROW_ID: 隐含的自增ID（隐藏主键）

> MySQL有一个 _rowid 键，如果该表有整型主键或有非空唯一整型索引的话则 _rowid 直接与之关联
>
> 如果没有以上两种的话，则InnoDB会自动以DB_ROW_ID产生一个聚簇索引 从0到 2^48 - 1，如果超过 2^48-1会从0重新开始，所以会有覆盖旧数据的情况
>
> 在InnoDB中有一个全局变量 `dictsys.row_id` 所有DB_ROW_ID共享这个变量，每插入一条需要DB_ROW_ID的记录的时候，DB_ROW_ID会拿这个全局变量当作自己的主键，然后再自增这个全局变量

DN_TRX_ID: 最近修改/插入事务的ID

DB_ROLL_PTR: 回滚指针，指向这条记录的上一个版本

DELETED_BIT: 记录被更新或删除，不代表真的删除，只是删除flag变了
