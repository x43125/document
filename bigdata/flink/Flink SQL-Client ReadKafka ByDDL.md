# Flink Sql read data from Kafka

## 1. connector

flink sql 从Kafka topic中读取数据首先需要有Kafka 数据源的connector，检查在`$FLINK_HOME/lib`下是否有Kafka的connector jar包 `flink-sql-connector-kafka_2.11-1.12.0.jar`，有则可以直接使用，没有则需要先下载

到官网 https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/connectors/ 下载对应的依赖，这里有各种源的connector，我们需要的是Kafka的，复制该download连接，到`$FLINK_HOME/lib`下执行下方命令

```sh
wget https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-kafka_2.11/1.12.0/flink-sql-connector-kafka_2.11-1.12.0.jar
```

## 2. 启动Flink SQL-Client

到`$FLINK_HOME`下执行下方命令

```sh
./bin/sql-client.sh embedded -l lib/
```

将进入flink sql-client的终端界面

<img src="resources/image-20210426113653618.png" alt="image-20210426113653618" style="zoom:60%;" />

之后的操作即在此终端中执行

## 3. 创建Kafka table

#### 3.1 普通table

先在Kafka中创建待读取的topic，可以另起一个shell：

```sh
/opt/software/kafka$ ./bin/kafka-topics.sh  --create --replication-factor 1 --partitions 1 --topic flinkreadkafka1 --zookeeper localhost:2181
Created topic "flinkreadkafka1".
```

回到flink sql client 输入以下创建Kafka read table sql语句

```sql
CREATE TABLE CustomerStatusChangedEvent(
    customerId int,
    oStatus int,
    nStatus int
) with (
    'connector.type' = 'kafka',  -- 连接类型
    'connector.version' = 'universal',
    'connector.properties.group.id' = 'g2.group1',
    'connector.properties.bootstrap.servers' = '192.168.11.152:9092',  -- kafka bootstrapservers信息，集群则全写
    'connector.properties.zookeeper.connect' = '192.168.11.152:2181',  -- kafka 依赖的zookeeper信息，集群则全写
    'connector.topic' = 'flinkreadkafka1',  -- 待读取的kafka topic
    'connector.startup-mode' = 'earliest-offset',  -- 读取topic方式，from-beginning模式，'last-offset'为读取实时
    'format.type' = 'json' -- topic中数据格式
); 

-- 查看表中数据
select * from CustomerStatusChangedEvent; 
-- 插入数据,
insert into CustomerStatusChangedEvent(customerId,oStatus,nStatus) values(1001,1,2),(1002,10,2),(1003,1,20);
```

**注：**flink sql-client创建出来的表是流表，只保存在当前终端中，关闭终端则表信息及数据不会被保存，但对表的**insert**操作会将数据sink到Kafka对应的topic中

#### 3.2 带有时间戳的table

DDL：

```sh
CREATE TABLE CustomerStatusChangedEvent2(
    rideId int,
    lon float,
    lat float,
    rideTime varchar,
    ts AS TO_TIMESTAMP(rideTime, 'yyyy-MM-dd HH:mm:ss'), -- 定义事件时间
    WATERMARK FOR ts AS ts - INTERVAL '5' SECOND   -- 在ts上定义5 秒延迟的 watermark
) with (
    'connector.type' = 'kafka',  -- 连接类型
    'connector.version' = 'universal',
    'connector.properties.group.id' = 'g2.group1',
    'connector.properties.bootstrap.servers' = '192.168.11.152:9092',  -- kafka bootstrapservers信息，集群则全写
    'connector.properties.zookeeper.connect' = '192.168.11.152:2181',  -- kafka 依赖的zookeeper信息，集群则全写
    'connector.topic' = 'TaxiRides1',  -- 待读取的kafka topic
    'connector.startup-mode' = 'earliest-offset',  -- 读取topic方式，from-beginning模式，'last-offset'为读取实时
    'format.type' = 'json' -- topic中数据格式
);
```

数据：

```sh
{"rideId":1,"lon":0.5,"lat":0.3,"rideTime":"2021-04-21 16:15:15"}
{"rideId":2,"lon":0.4,"lat":0.4,"rideTime":"2021-04-21 16:15:15"}
{"rideId":3,"lon":0.3,"lat":0.5,"rideTime":"2021-04-21 16:15:16"}
{"rideId":4,"lon":0.2,"lat":0.6,"rideTime":"2021-04-21 16:15:17"}
```

结果展示：

![image-20210426165755517](image-20210426165755517.png)