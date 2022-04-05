# SnappyData - example

## Ad Impression Analytics use case

![Architecture Kinda](AdAnalytics_Architecture.png)

- **Find total uniques for a certain AD grouped on geography;**
- **Impression trends for advertisers over time;**
- **Top ads based on uniques count for each Geo.**

## 1. Generating the AdImpression logs

使用程序模拟模拟AdServer，批量向Kafka生成随机的AdImpression Logs （Avro格式）

```scala
val props = new Properties()
props.put("serializer.class", "io.snappydata.adanalytics.AdImpressionLogAvroEncoder")
props.put("partitioner.class", "kafka.producer.DefaultPartitioner")
props.put("key.serializer.class", "kafka.serializer.StringEncoder")
props.put("metadata.broker.list", brokerList)
val config = new ProducerConfig(props)
val producer = new Producer[String, AdImpressionLog](config)
sendToKafka(generateAdImpression())

def generateAdImpression(): AdImpressionLog = {
  val random = new Random()
  val timestamp = System.currentTimeMillis()
  val publisher = Publishers(random.nextInt(NumPublishers))
  val advertiser = Advertisers(random.nextInt(NumAdvertisers))
  val website = s"website_${random.nextInt(Constants.NumWebsites)}.com"
  val cookie = s"cookie_${random.nextInt(Constants.NumCookies)}"
  val geo = Geos(random.nextInt(Geos.size))
  val bid = math.abs(random.nextDouble()) % 1
  val log = new AdImpressionLog()
}

def sendToKafka(log: AdImpressionLog) = {
  producer.send(new KeyedMessage[String, AdImpressionLog](
    Constants.kafkaTopic, log.getTimestamp.toString, log))
}
```

## 2. Spark stream as SQL table and Continuous query

SnappySQLLogAggregator在Kafka源上创建一个流

在流表上注册一个连续查询，该查询每1秒汇总每个发布者和地理位置的指标

```scala
  val sc = new SparkContext(sparkConf)
  val snsc = new SnappyStreamingContext(sc, batchDuration)

  /**
  * AdImpressionStream presents the stream as a Table. It is registered with the Snappy catalog and hence queriable. 
  * Underneath the covers, this is an abstraction over a DStream. DStream batches are emitted as DataFrames here.
  */
  snsc.sql("create stream table adImpressionStream (" +
    " time_stamp timestamp," +
    " publisher string," +
    " advertiser string," +
    " website string," +
    " geo string," +
    " bid double," +
    " cookie string) " +
    " using directkafka_stream options" +
    " (storagelevel 'MEMORY_AND_DISK_SER_2'," +
    " rowConverter 'io.snappydata.adanalytics.AdImpressionToRowsConverter' ," +
    s" kafkaParams 'metadata.broker.list->$brokerList'," +
    s" topics '$kafkaTopic'," +
    " K 'java.lang.String'," +
    " V 'io.snappydata.adanalytics.AdImpressionLog', " +
    " KD 'kafka.serializer.StringDecoder', " +
    " VD 'io.snappydata.adanalytics.AdImpressionLogAvroDecoder')")
    
    // Aggregate metrics for each publisher, geo every few seconds. Just 1 second in this example.
    // With the stream registered as a table, we can execute arbitrary queries.
    // These queries run each time a batch is emitted by the stream. A continuous query.
    val resultStream: SchemaDStream = snsc.registerCQ(
      "select min(time_stamp), publisher, geo, avg(bid) as avg_bid," +
        " count(*) as imps , count(distinct(cookie)) as uniques" +
        " from adImpressionStream window (duration 1 seconds, slide 1 seconds)" +
        " where geo != 'unknown' group by publisher, geo")
```

## 3. Ingesting into Column table

使用spark数据源API写入column表中

```scala
   snsc.sql("create table aggrAdImpressions(time_stamp timestamp, publisher string," +
    " geo string, avg_bid double, imps long, uniques long) " +
     "using column options(buckets '11')")
   //Simple in-memory partitioned, columnar table with 11 partitions. 
   //Other table types, options to replicate, persist, overflow, etc are defined 
   // here -> http://snappydatainc.github.io/snappydata/rowAndColumnTables/
  
   //Persist using the Spark DataSource API 
   resultStream.foreachDataFrame(_.write.insertInto("aggrAdImpressions"))
```

## 4. Ingesting into a Sample table

```scala
  snsc.sql("CREATE SAMPLE TABLE sampledAdImpressions" +
    " OPTIONS(qcs 'geo', fraction '0.03', strataReservoirSize '50', baseTable 'aggrAdImpressions')")
```

## 5. 具体操作

1. 建一个spark-env.sh文件于snappydata/conf下，可以通过复制spark-env.sh.template，并在其中加上下一句

```sh
SPARK_DIST_CLASSPATH=SNAPPY_POC_HOME/assembly/build/libs/snappy-poc-1.0.0-assembly.jar
# SNAPPY_POC_HOME是本程序位于的根目录，如：
SPARK_DIST_CLASSPATH=/opt/code/snappy-examples/assembly/build/libs/snappy-poc-1.0.0-assembly.jar
```

2. 启动流程序

```sh
./bin/snappy-job.sh submit \
--lead localhost:8090 \
--app-name AdAnalytics \
--class io.snappydata.adanalytics.SnappySQLLogAggregatorJob \
--app-jar /opt/code/snappy-examples/assembly/build/libs/snappy-poc-1.0.0-assembly.jar \
--stream
```

3. 生产Kafka数据

```sh
./gradlew generateAdImpressions
```

4. 交互式查看流读取情况

```sh
./bin/snappy-shell
connect client 'localhost:1527';
set spark.sql.shuffle.partitions=7;
show members;
select count(*) from aggrAdImpressions;
select count(*) from adImpressionStream;
select count(*) AS adCount, geo from aggrAdImpressions group by geo order by adCount desc limit 20;
select sum(uniques) AS totalUniques, geo from aggrAdImpressions where publisher='publisher11' group by geo order by totalUniques desc limit 20;
select count(*) AS adCount, geo from aggrAdImpressions group by geo order by adCount desc limit 20 with error 0.20 confidence 0.95 ;
select sum(uniques) AS totalUniques, geo from aggrAdImpressions where publisher='publisher11' group by geo order by totalUniques desc limit 20 with error 0.20 confidence 0.95 ;
select sum(uniques) AS totalUniques, geo from sampledAdImpressions where publisher='publisher11' group by geo order by totalUniques desc;
select count(*) as sample_cnt from sampledAdImpressions;
```

5. 试验结束，关闭集群

```sh
./sbin/snappy-stop-all.sh
```

