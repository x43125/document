# redis-study-1-hash

> key: field: value

## overview

## 命令

### 常用命令

### 不常用命令

## 内部编码

## 使用建议

## 使用场景

1. 用户信息保存
    - 字符串模式：存储混乱，并使用大量的key占用内存
        - set user:1:name tom
        - set user:1:age 1
        - set user:1:city tianjin
    - 序列化模式：需要良好的序列化，每次序列化耗费时间
        - set user:1 serialize(name tom, age 1, city tianjin)
    - hash模式：要控制内部编码在ziplist 和 hashtable之间的转换，hashtable比较耗内存
        - hmset user:1 name tom age 1 city tianjin

