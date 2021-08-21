# SnappyData Join Test01 - error

```scala
import org.apache.spark.sql.SnappySession
import org.apache.spark.sql.functions.from_json
import org.apache.spark.sql.streaming.ProcessingTime

val snappy = new SnappySession(sc)

// Create target snappy table. Stream data will be ingested in this table.
snappy.sql("create table if not exists devices01(id string, signal int) using column")
snappy.sql("create table if not exists devices02(id string, age int) using column")
val schema1 = snappy.table("devices01").schema
val schema2 = snappy.table("devices02").schema
val df1 = snappy.
  readStream.
  format("kafka").
  option("kafka.bootstrap.servers", "192.168.11.152:9092").  
  option("startingOffsets", "earliest").
  option("subscribe", "devices01").
  option("maxOffsetsPerTrigger", 100).  // to restrict the batch size
  load()

// Start the execution of streaming query
val streamingQuery = df1.
  select(from_json(df1.col("value").cast("string"), schema1).alias("jsonObject")).
  selectExpr("jsonObject.*").
  writeStream.
  format("snappysink").
  queryName("deviceStream").  // must be unique across the TIBCO ComputeDB cluster
  trigger(ProcessingTime("1 seconds")).
  option("tableName", "devices01").
  option("checkpointLocation", "/opt/code/snappydata-1.2.0-bin/checkpoint").
  start()

val df2 = snappy.
  readStream.
  format("kafka").
  option("kafka.bootstrap.servers", "192.168.11.152:9092").  
  option("startingOffsets", "earliest").
  option("subscribe", "devices02").
  option("maxOffsetsPerTrigger", 100).  // to restrict the batch size
  load()

// Start the execution of streaming query
val streamingQuery = df2.
  select(from_json(df2.col("value").cast("string"), schema2).alias("jsonObject")).
  selectExpr("jsonObject.*").
  writeStream.
  format("snappysink").
  queryName("deviceStream").  // must be unique across the TIBCO ComputeDB cluster
  trigger(ProcessingTime("1 seconds")).
  option("tableName", "devices02").
  option("checkpointLocation", "/opt/code/snappydata-1.2.0-bin/checkpoint").
  start()
```

