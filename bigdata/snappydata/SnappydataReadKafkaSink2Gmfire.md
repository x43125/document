# Snappydata with Kafka Source And SnappySink

## 1. 使用snappydata从kafka的topic中读取数据

```sh
./sbin/snappy-start-all.sh
```

```sh
./bin/kafka-topics.sh --create --zookeeper 192.168.11.152:2181 --partitions 4 --replication-factor 1 --topic devices
```

```sh
./bin/kafka-console-producer.sh --broker-list 192.168.11.152:9092 --topic devices
>{"id":"device1", "signal":10}
{"id":"device2", "signal":20}
{"id":"device3", "signal":30}
{"id":"device4", "signal":40}
{"id":"device5", "signal":50}
{"id":"device6", "signal":60}
{"id":"device7", "signal":70}
```

```sh
./bin/spark-shell --master local[*] --conf spark.snappydata.connection=localhost:1527
```

```sh
import org.apache.spark.sql.SnappySession
import org.apache.spark.sql.functions.from_json
import org.apache.spark.sql.streaming.ProcessingTime

val snappy = new SnappySession(sc)

// Create target snappy table. Stream data will be ingested in this table.
snappy.sql("create table if not exists devices(id string, signal int) using column")

val schema = snappy.table("devices").schema

// Create DataFrame representing the stream of input lines from Kafka topic 'devices'
val df = snappy.
  readStream.
  format("kafka").
  option("kafka.bootstrap.servers", "192.168.11.152:9092").  
  option("startingOffsets", "earliest").
  option("subscribe", "devices").
  option("maxOffsetsPerTrigger", 100).  // to restrict the batch size
  load()

// Start the execution of streaming query
val streamingQuery = df.
  select(from_json(df.col("value").cast("string"), schema).alias("jsonObject")).
  selectExpr("jsonObject.*").
  writeStream.
  format("snappysink").
  queryName("deviceStream").  // must be unique across the TIBCO ComputeDB cluster
  trigger(ProcessingTime("1 seconds")).
  option("tableName", "devices").
  option("checkpointLocation", "/opt/code/snappydata-1.2.0-bin/checkpoint").
  start()
```

```sh
./bin/snappy-sql
connect client 'localhost:1527';
select * from devices;
streamingQuery.stop
```





## 2. 将Kafka中读取到的数据进行处理

## 3. 将处理后的数据sink到内存数据库中