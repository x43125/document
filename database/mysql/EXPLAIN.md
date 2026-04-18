# EXPLAIN功能

## 字段

select_type（查询类型）：简单类型，联合查询，子查询等

type: 全表扫描，范围扫描，唯一索引扫描，常数引用

possible_keys & key: 可能用到的index，实际用到的index

rows: 预计需要扫描的行数

Extra: 

- 空：在索引中使用where过滤，存储引擎层完成；
- Using index：使用索引覆盖扫描，服务器层完成，无需回表查询；
- Using Where：从数据表中返回数据，然后过滤不符合的记录，在服务器层完成，需要先从数据库表中读取记录，再进行过滤。