# Flink Stream SQL quickstart

> 创建一个从kafka flinkssintest01 topic读取，并写入到flinkssouttest01 topic中的任务

1. 创建flink stream sql文件，命名为`flinkss.sql`

```sql
create table SourceOne (
    id        int,
    name      varchar
) WITH (
    type = 'kafka11',
    bootstrapServers = 'localhost:9092',
    zookeeperQuorum = 'localhost:2181/kafka',
    offsetReset = 'earliest',
    topic = 'flinkssintest01',
    enableKeyPartitions = 'false',
    topicIsPattern = 'false',
    parallelism = '1'
);
    
CREATE TABLE SinkOne (
    id         int,
    name       varchar
) WITH (
    type = 'kafka11',
    bootstrapServers = 'localhost:9092',
    topic = 'flinkssouttest02',
    parallelism = '1',
    updateMode = 'upsert'
);

insert into SinkOne select * from SourceOne;
```

2. 插入Kafka中的数据格式

```json
{"id": 1,"name":"zhangsan"}
{"id": 2,"name":"lisi"}
{"id": 3,"name":"wanger"}
{"id": 4,"name":"piter"}
{"id": 5,"name":"jack"}
{"id": 6,"name":"rose"}
{"id": 7,"name":"mark"}
```

3. 启动任务，执行jar包

```sh
sh submit.sh \
  -mode local \  -- 本地模式启动
  -name local_test \  -- 本次项目的命名
  -sql /opt/code/flink-stream-sql-code/flinkss.sql \  -- sql脚本的位置
  -localSqlPluginPath /opt/software/flink-stream-sql/sqlplugins  -- sqlplugins位置
```

4. 此时向Kafka topic1 flinkssintest01中插入符合格式的数据，在topic2 flinkssouttest01中，即可同步看到该数据