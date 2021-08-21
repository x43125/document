# Flink Stream SQL read data from Kafka to PostgreSQL

> 创建一个从Kafka读取消息写入到PG中的任务

1. 创建kafka source topic

```sh
./bin/kafka-topics.sh --create --replication-factor 1 --partitions 1 --zookeeper localhost:2181 --topic mqTest03
Created topic "mqTest03".
./bin/kafka-topics.sh --create --replication-factor 1 --partitions 1 --zookeeper localhost:2181 --topic mqTest04
Created topic "mqTest04".
```

2. kafka topic data format

```json
{"name":"maqi","id":1001}  -- Topic mqTest03
{"address":"hz","id":1001}  -- Topic mqTest04
```

3. 创建pg sink table

```sql
create table if not exists userInfo(
    id int,
    name VARCHAR,
    address VARCHAR,
    primary key (id)
)
```

4. 创建flink stream-sql sql文件

```sql
CREATE TABLE source1 (
    id int,
    name VARCHAR
)WITH(
    type ='kafka11',
    bootstrapServers ='localhost:9092',
    zookeeperQuorum ='localhost:2181/kafka',
    offsetReset ='earlist',
    topic ='mqTest03',
    timezone='Asia/Shanghai',
    topicIsPattern ='false'
 );

CREATE TABLE source2(
    id int,
    address VARCHAR
)WITH(
    type ='kafka11',
    bootstrapServers ='localhost:9092',
    zookeeperQuorum ='localhost:2181/kafka',
    offsetReset ='earlist',
    topic ='mqTest04',
    timezone='Asia/Shanghai',
    topicIsPattern ='false'
);

CREATE TABLE MyResult(
    id int,
    name VARCHAR,
    address VARCHAR,
    primary key (id)
)WITH(
    type='postgresql',
    url='jdbc:postgresql://localhost:5432/x43125',
    userName='x43125',
    password='456123',
    tableName='userInfo',
    schema = 'aaa',
    updateMode = 'upsert',
    batchSize = '1'
);

insert into MyResult
select 
	s1.id,
	s1.name,
	s2.address
from 
	source1 s1
left join
	source2 s2
on 	
	s1.id = s2.id
```

5. 启动任务，提交flink ss job

```sh
sh submit.sh \
  -mode local \
  -name local_test \
  -sql /opt/code/flink-stream-sql-code/kafka2pg01.sql \
  -localSqlPluginPath /opt/software/flink-stream-sql/sqlplugins
```

