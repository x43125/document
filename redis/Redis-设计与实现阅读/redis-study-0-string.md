# Redis-study-0-string

## 1 overview

字符串类型的值可以是字符串、数字、二进制，最大不超过512MB

## 2 命令

### 2.1 常用命令

#### 2.1.1 设置值

> set key value [ex seconds] [px milliseconds] [nx|xx]

| 描述                       | 命令                             | 例子                               | 成功返回   | 失败返回   | 时间复杂度            |
| -------------------------- | :------------------------------- | ---------------------------------- | ---------- | ---------- | --------------------- |
| 设置kv                     | set key value                    | set hello world                    | OK         | 不会失败   | O(1)                  |
| 设置有期限的kv 单位:秒     | set key value ex seconds         | set hello world ex 10              | OK         | (nil)      | O(1)                  |
| 设置有期限的kv 单位:秒     | setex key seconds value          | setex hello 10 world               | OK         | (nil)      | O(1)                  |
| 键不存在的时候才能设置成功 | set key value nx                 | set hello world nx                 | OK         | (nil)      | O(1)                  |
| 键不存在的时候才能设置成功 | setnx key value                  | setnx hello world                  | (integer)1 | (integer)0 | O(1)                  |
| 键存在的时候才能设置成功   | set key value xx                 | set hello world xx                 | OK         | (nil)      | O(1)                  |
| 批量设置值                 | mset key1 value1 key2 value2 ... | set java jedis python redis-py ... | OK         | (nil)      | O(k), k是设置的键个数 |

#### 2.1.2 获取值

> get key

| 描述         | 命令               | 例子             | 成功返回                         | 失败返回 | 时间复杂度            |
| ------------ | ------------------ | ---------------- | -------------------------------- | -------- | --------------------- |
| 根据键获取值 | get key            | get hello        | world                            | (nil)    | O(1)                  |
| 批量获取值   | mget key1 key2 ... | mget java python | 1) ” jedis” <br />2)  “redis-py” | (nil)    | O(k), k是获取的键个数 |
|              |                    |                  |                                  |          |                       |

> !!! 批量操作可以有效的减少网络时间
>
> get * n: n次get耗时 = n次网络时间 + n次命令时间
>
> mget:    n次get耗时 = 1次网络时间 + n次命令时间

#### 2.1.3 其他

| 描述               | 命令                      | 例子                  | 结果                                                         | 时间复杂度            |
| ------------------ | ------------------------- | --------------------- | ------------------------------------------------------------ | --------------------- |
| 键是否存在         | exists key                | exists hello          | 存在：(integer)1<br />不存在：(integer)0                     |                       |
| 删除键值，支持批量 | del key1 key2 key3        | del hello             | 存在：(integer)1<br />不存在：(integer)0                     | O(k), k是删除的键个数 |
| key自增            | incr key                  | incr hello            | value是整数：自增后结果<br />value不是整数：(error) ERR value is not an integer or out of range<br />value不存在：1，并创建该键值 | O(1)                  |
| key自增指定值      | incrby key increment      | incrby hello 10       | value是整数： 自增10后结果<br />value不是整数：报错<br />value不存在：10，并创建该键值 | O(1)                  |
| key自增浮点值      | incrbyfloat key increment | incrbyfloat hello 0.5 | value是数值型：自增0.5后结果<br />value不是数值型：报错<br />value不存在：0.5，并创建该键值 | O(1)                  |
| key自减            | decr key                  | decr hello            | value是整数：自减后结果<br />value不是整数：(error) ERR unknown command `decy`, with args beginning with: `intkey`,<br />value不存在：-1，并创建该键值 | O(1)                  |
| key自减指定值      | decrby key decrement      | decr hello 10         | value是整数：自减10后结果<br />value不是整数：(error) ERR value is not an integer or out of range<br />value不存在：-10，并创建该键值 | O(1)                  |

### 2.2 不常用命令

| 描述                        | 命令                      | 例子               | 结果         | 时间复杂度         |
| --------------------------- | ------------------------- | ------------------ | ------------ | ------------------ |
| 追加值                      | append key value          | append hello world | “worldworld” | O(1)               |
| 字符串长度(中文占用3个字节) | strlen key                | strlen hello       | (integer)10  | O(1)               |
| 设置并返回原值              | getset key value          | getset hello redis | “worldworld” |                    |
| 设置指定位置的字符          | setrange key offset value | setrange hello 0 b | “bedis”      | O(1)               |
| 获取部分字符串              | getrange key start end    | getrange hello 0 2 | “bed”        | O(n),n是字符串长度 |

## 3 内部编码

redis内部每种存储类型都基本上有多种不同的内部编码来适应不同的情况，如更快存取和更节省内存

我们使用命令 `object encoding key` 来检查key的内部编码。

### 3.1 int

> 8个字节的长整型

```sh
127.0.0.1> set key 8653
OK
127.0.0.1> object encoding key
"int"
```

### 3.2 embstr

> 小于等于39个字节的字符串

```sh
127.0.0.1> set key "hello, world"
OK
127.0.0.1>  object encoding key
"embstr"
```

### 3.3 raw

> 大于39个字节的字符串

```sh
127.0.0.1> set key "abcdefghijklmnopqrstuvwxyz1234567890!@#$%^&*()_+-={}[];<>,./?"
OK
127.0.0.1> object encoding key
"raw"
```

**redis会根据值的类型和长度来自行决定使用哪种内部编码**

## 4 使用建议

redis 没有明确对顶键的声明方式，所以如果在开发中不注意规范将会导致在使用的时候非常的混乱，因此建议一定要有统一的规范，如果公司有规范就按照规范来，否则可以使用如下规则或建立自己的规范：

> `业务名:对象名:id:[属性]` 的组合作为键
>
> e.g.: `vs:user:1`



## 5 使用场景

1. 缓存功能：MySQL最典型也应该是使用最多的场景
2. 计数：常见如视频播放数，博客阅览数自增
3. 共享session：分布式web服务将用户的session信息放到redis中进行集中管理
4. 限速： 如限制用户一分钟内收取验证码的次数