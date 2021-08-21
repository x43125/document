```scala
 val spark: SparkSession = SparkSession.builder.appName("SparkApp").master("local[4]").getOrCreate
 val snsc = new SnappyStreamingContext(sc, Duration(1))
 snsc.sql("create stream table streamTable (userId string, clickStreamLog string) " +
     "using kafka_stream options (" +
     "storagelevel 'MEMORY_AND_DISK_SER_2', " +
     "rowConverter 'io.snappydata.app.streaming.KafkaStreamToRowsConverter', " +
     "kafkaParams 'zookeeper.connect->localhost:2181;auto.offset.reset->smallest;group.id->myGroupId', " +
     "subscribe 'streamTopic:01')")

 // You can get a handle of underlying DStream of the table
 val dStream = snsc.getSchemaDStream("streamTable")
 // You can also save the DataFrames to an external table
 dStream.foreachDataFrame(_.write.insertInto(tableName))
```

```scala
class AdImpressionLogToRowRDD extends Serializable {
  def convert(logRdd: RDD[AdImpressionLog]): RDD[Row] = {
    logRdd.map(log => {
      Row(log.getTimestamp,
        log.getPublisher.toString,
        log.getAdvertiser.toString,
        log.getWebsite.toString,
        log.getGeo.toString,
        log.getBid,
        log.getCookie.toString)
    })
  }
}
```

```scala
snsc.sql("create stream table if not exists directKafkaStream (" +
    " publisher string, advertiser string)" +
    " using directkafka_stream options(" +
    " rowConverter 'org.apache.spark.sql.streaming.RowsConverter' ," +
    s" kafkaParams 'metadata.broker.list->$brokeraddrs;auto.offset.reset->smallest'," +
    s" topics '$topic1')")
```

```scala
snsc.sql("create stream table if not exists kafkaStream (" +
    " publisher string, advertiser string) " +
    " using kafka_stream options(" +
    " rowConverter 'org.apache.spark.sql.streaming.RowsConverter'," +
    s" kafkaParams 'zookeeper.connect->$zkadd;auto.offset.reset->smallest;group.id->myId'," +
    s" topics '$topic:1,$topic2:1')")
```

```scala
/**
  * Converts an input stream message into an org.apache.spark.sql.Row
  * instance
  */
class RowsConverter extends StreamToRowsConverter with Serializable {
  override def toRows(message: Any): Seq[Row] = {
    val log = message.asInstanceOf[String]
    val fields = log.split(",")
    Seq(Row.fromSeq(Seq(new java.sql.Timestamp(fields(0).toLong),
      fields(1),
      fields(2),
      fields(3),
      fields(4),
      fields(5).toDouble,
      fields(6)
    )))
  }
}
```

