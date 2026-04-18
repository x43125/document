# redis-study-n-jedis

> Redis有许多Java客户端，本节讲的是最为广泛的jedis，其他客户端可以自行查阅官网（[Redis-Java-Client](https://redis.io/clients#java)）

## 1.获取Jedis

```java
String host = "47.98.59.193";
int port = 6379;
Jedis jedis = new Jedis(host, port, 1200, false);
// 如果有认证
jedis.auth("123ABCdef*");
```

## 2.Jedis的基本使用

```java
String setResult = jedis.set("hello", "world");
System.out.println("setResult = " + setResult);

Long setnxResult = jedis.setnx("hello", "redis");
System.out.println("setnxResult = " + setnxResult);

String getResult = jedis.get("hello");
System.out.println("getResult = " + getResult);
```

redis客户端需要关闭，否则可能会占用资源，推荐使用 `try-catch-finally` 来捕获遇到的问题并确保客户端在使用完会被关闭

```java
Jedis jedis = null;
try {
    jedis = new Jedis(host, port, 1200, false);
    jedis.auth(password);
    Long setnxRes = jedis.setnx("hello", "world");
    System.out.println("setnxRes = " + setnxRes);
} catch (Exception e) {
    e.printStackTrace();
} finally {
    if (jedis != null) {
        jedis.close();
    }
}

// try-catch 自动关闭资源方式
try (Jedis jedis = new Jedis(host, port, 1200, false)) {
    jedis.auth(password);
    Long setnxRes = jedis.setnx("hello", "world");
    System.out.println("setnxRes = " + setnxRes);
} catch (Exception e) {
    e.printStackTrace();
}
```

数据类型使用

```java
jedis.set("hello", "world");
System.out.println("jedis.get(\"hello\") = " + jedis.get("hello"));
// 2.hash
jedis.hset("myHash", "hashKey1", "hashValue1");
jedis.hset("myHash", "hashKey2", "hashValue2");
System.out.println("jedis.hgetAll(\"myHash\") = " + jedis.hgetAll("myHash"));
// 3.list
jedis.rpush("myList", "1");
jedis.rpush("myList", "2");
jedis.rpush("myList", "3");
jedis.lrange("myList", 0, -1);
// 4.set
jedis.sadd("mySet", "a");
jedis.sadd("mySet", "b");
jedis.sadd("mySet", "c");
System.out.println("jedis.smembers(\"mySet\") = " + jedis.smembers("mySet"));
// 5.zset
jedis.zadd("myZSet", 99, "tom");
jedis.zadd("myZSet", 66, "peter");
jedis.zadd("myZSet", 33, "james");
System.out.println("jedis.zrangeWithScores(\"myZSet\", 0, -1) = " + jedis.zrangeWithScores("myZSet", 0, -1));


// >>>> 结果：
jedis.get("hello") = world
jedis.hgetAll("myHash") = {hashKey2=hashValue2, hashKey1=hashValue1}
jedis.smembers("mySet") = [c, b, a]
jedis.zrangeWithScores("myZSet", 0, -1) = [[james,33.0], [peter,66.0], [tom,99.0]]
```

jedis还支持序列化存取

```java
// 序列化存储
// ProtostuffSerializer 类需自行实现
ProtostuffSerializer protostuffSerializer = new ProtostuffSerializer();

Jedis jedis = new Jedis(host, port, 1200, false);
jedis.auth(password);

String key = "club:1";
Club club = new Club(1, "AC", "米兰", new Date(), 1);
	// 序列化
byte[] clubBytes = protostuffSerializer.serialize(club);
jedis.set(key.getBytes(), clubBytes);
	// 反序列化
byte[] resultBytes = jedis.get(key.getBytes());
Club deserializeClub = protostuffSerializer.deserialize(resultBytes);

System.out.println(deserializeClub);
```





## 3.Jedis连接池使用

## 4.Jedis中Pipeline使用

## 5.Jedis中Lua脚本的使用

