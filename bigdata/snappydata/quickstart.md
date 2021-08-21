# Quickstart

## 1. 启动Kafka和snappydata命令

```sh
/opt/software/kafka/bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
/opt/software/kafka/bin/kafka-server-start.sh -daemon config/server.properties
/opt/software/snappydata/sbin/snappy-start-all.sh
```

```sh
/opt/software/kafka/bin/kafka-topics.sh --create --zookeeper localhost:2181 --partitions 8 --topic adImpressionsTopic --replication-factor=1

/opt/software/kafka/bin/kafka-console-consumer.sh --topic adImpressionsTopic --bootstrap-server localhost:9092
```

```sh
./gradlew generateAdImpressions
```

```sh
./bin/snappy-job.sh submit \
    --lead localhost:8090 \
    --app-name AdAnalytics \
    --class io.snappydata.adanalytics.SnappySQLLogAggregatorJob \
    --app-jar /opt/code/snappy-examples/assembly/build/libs/snappy-poc-1.0.0-assembly.jar \
    --stream
```

```sh
./bin/snappy-sql
connect client 'localhost:1527';
set spark.sql.shuffle.partitions=7;
show members;

select count(*) from aggrAdImpressions;
```

## 2. sql创建表

```scala
val snsc = new SnappyStreamingContext(sc, Seconds(3))
```

```sh
./bin/snappy-job.sh submit  
--app-name exampleApp 
--class io.snappydata.Example 
--app-jar /path/to/application.jar
```

```sh
./bin/snappy-job.sh submit 
--app-name test01 
--class com.example.snappy.snappydata.example.Main 
--app-jar /opt/code/self-snappydata/snappydata01-1.0-SNAPSHOT.jar
```

