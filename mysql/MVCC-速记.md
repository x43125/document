# MVCC

> MVCC主要是为了解决数据库的读写并发问题

数据库可以通过 MVCC + 悲观锁/乐观锁 来解决 读写冲突 + 写写冲突

快照读 & 当前读

主要依靠：**4个隐式字段**，**undo日志** ，**Read View** 来实现

4个隐式字段：DB_ROW_ID, DN_TRX_ID, DB_ROLL_PTR, DELETED_BIT