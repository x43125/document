# Flink Read By Yaml

## 1. Flink Read data Fom CSV By Yaml

### .yaml示例

```yaml
# 定义表，如 source、sink、视图或临时表。

tables:
  - name: MyTableSource
    type: source-table
    update-mode: append
    connector:
      type: filesystem
      path: "/path/to/something.csv"
    format:
      type: csv
      fields:
        - name: MyField1
          data-type: INT
        - name: MyField2
          data-type: VARCHAR
      line-delimiter: "\n"
      comment-prefix: "#"
    schema:
      - name: MyField1
        data-type: INT
      - name: MyField2
        data-type: VARCHAR
  - name: MyCustomView
    type: view
    query: "SELECT MyField2 FROM MyTableSource"

# 定义用户自定义函数

functions:
  - name: myUDF
    from: class
    class: foo.bar.AggregateUDF
    constructor:
      - 7.6
      - false

# 定义 catalogs

catalogs:
   - name: catalog_1
     type: hive
     property-version: 1
     hive-conf-dir: ...
   - name: catalog_2
     type: hive
     property-version: 1
     default-database: mydb2
     hive-conf-dir: ...

# 改变表程序基本的执行行为属性。

execution:
  planner: blink                    # 可选： 'blink' （默认）或 'old'
  type: streaming                   # 必选：执行模式为 'batch' 或 'streaming'
  result-mode: table                # 必选：'table' 或 'changelog'
  max-table-result-rows: 1000000    # 可选：'table' 模式下可维护的最大行数（默认为 1000000，小于 1 则表示无限制）
  time-characteristic: event-time   # 可选： 'processing-time' 或 'event-time' （默认）
  parallelism: 1                    # 可选：Flink 的并行数量（默认为 1）
  periodic-watermarks-interval: 200 # 可选：周期性 watermarks 的间隔时间（默认 200 ms）
  max-parallelism: 16               # 可选：Flink 的最大并行数量（默认 128）
  min-idle-state-retention: 0       # 可选：表程序的最小空闲状态时间
  max-idle-state-retention: 0       # 可选：表程序的最大空闲状态时间
  current-catalog: catalog_1        # 可选：当前会话 catalog 的名称（默认为 'default_catalog'）
  current-database: mydb1           # 可选：当前 catalog 的当前数据库名称
                                    #   （默认为当前 catalog 的默认数据库）
  restart-strategy:                 # 可选：重启策略（restart-strategy）
    type: fallback                  #   默认情况下“回退”到全局重启策略

# 用于调整和调优表程序的配置选项。

# 在专用的”配置”页面上可以找到完整的选项列表及其默认值。
configuration:
  table.optimizer.join-reorder-enabled: true
  table.exec.spill-compression.enabled: true
  table.exec.spill-compression.block-size: 128kb

# 描述表程序提交集群的属性。

deployment:
  response-timeout: 5000
```

上述配置：

- 定义一个从 CSV 文件中读取的 table source `MyTableSource` 所需的环境，
- 定义了一个视图 `MyCustomView` ，该视图是用 SQL 查询声明的虚拟表，
- 定义了一个用户自定义函数 `myUDF`，该函数可以使用类名和两个构造函数参数进行实例化，
- 连接到两个 Hive catalogs 并用 `catalog_1` 来作为当前目录，用 `mydb1` 来作为该目录的当前数据库，
- streaming 模式下用 blink planner 来运行时间特征为 event-time 和并行度为 1 的语句，
- 在 `table` 结果模式下运行试探性的（exploratory）的查询，
- 并通过配置选项对联结（join）重排序和溢出进行一些计划调整。

## 2. Example

从 Kafka 中读取 JSON 文件并作为 table source 的环境配置文件。

将下文保存为 `sql-test.yaml`，并保存到`$FLINK_HOME/conf`目录下

```yaml
tables:
  - name: TaxiRides
    type: source-table
    update-mode: append
    connector:
      property-version: 1
      type: kafka
      version: "universal"
      topic: TaxiRides
      startup-mode: earliest-offset
      properties:
        bootstrap.servers: localhost:9092
        group.id: testGroup
    format:
      property-version: 1
      type: json
     # schema: "ROW<rideId LONG, lon FLOAT, lat FLOAT, rideTime TIMESTAMP>"
      schema: "ROW<rideId LONG, lon FLOAT, lat FLOAT>"
    schema:
      - name: rideId
        data-type: BIGINT
      - name: lon
        data-type: FLOAT
      - name: lat
        data-type: FLOAT
     # - name: rowTime
     #   data-type: TIMESTAMP(3)
     #   rowtime:
     #     timestamps:
     #       type: "from-field"
     #       from: "rideTime"
     #     watermarks:
     #       type: "periodic-bounded"
     #       delay: "60000"
      - name: procTime
        data-type: TIMESTAMP(3)
        proctime: true
```

```sh
$ cd $KAFKA_HOME
./bin/kafka-topics.sh --create --replication-factor 1 --partitions 1 --zookeeper localhost:2181 --topic TaxiRides1
Created topic "TaxiRides1".
./bin/kafka-console-producer.sh --topic TaxiRides1 --broker-list localhost:9092
>
```

插入数据示例如下：

```json
# {"rideId":1,"lon":0.5,"lat":0.3,"rideTime":"2020-11-06 13:46:33.912"}
# {"rideId":2,"lon":0.4,"lat":0.4,"rideTime":"2020-11-06 13:46:33.913"}
# {"rideId":3,"lon":0.3,"lat":0.5,"rideTime":"2020-11-06 13:46:34.122"}
# {"rideId":4,"lon":0.2,"lat":0.6,"rideTime":"2020-11-06 13:46:35.233"}

{"rideId":1,"lon":0.5,"lat":0.3}
{"rideId":2,"lon":0.4,"lat":0.4}
{"rideId":3,"lon":0.3,"lat":0.5}
{"rideId":4,"lon":0.2,"lat":0.6}
```

启动flink sql-client

```sh
./bin/sql-client.sh embedded -d conf/sql-test.yaml
```

```sql
Flink SQL> show tables;
TaxiRides1
Flink SQL> select * from TaxiRides1;
```

![image-20210426161854280](image-20210426161854280.png)

![image-20210426161325267](image-20210426161325267.png)

### 官网的结果展示：

![image-20210426154955130](image-20210426154955130.png)

![image-20210426155045600](image-20210426155045600.png)

## 3.Kafka指令及数据

```sh

./bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic TaxiRides
./bin/kafka-topics.sh --create --replication-factor 1 --partitions 1 --zookeeper localhost:2181 --topic TaxiRides
./bin/kafka-topics.sh --list --zookeeper localhost:2181
./bin/kafka-console-producer.sh --topic TaxiRides --broker-list localhost:9092

{"rideId":1,"lon":0.5,"lat":0.3}
{"rideId":2,"lon":0.4,"lat":0.4}
{"rideId":3,"lon":0.3,"lat":0.5}
{"rideId":4,"lon":0.2,"lat":0.6}

```

