# redis-study-2-list

## overview

## 常用命令

## 内部编码

### ziplist (压缩列表)

当元素个数少于 `list-max-ziplist-entries` 配置的值时（默认是512个），同时每个元素的值都小于 `list-max-ziplist-value` 配置时（默认是64字节）

### linkedlist(链表)

当列表无法满足ziplist的要求时

> redis3.2 之后提供了 **quicklist** 内部编码

## 使用场景

1. 消息队列
2. 文章列表