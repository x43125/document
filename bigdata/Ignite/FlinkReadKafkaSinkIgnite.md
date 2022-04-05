# Flink read data from Kafka sink to Ignite

![streamers](streamers.png)

## 1. 示例

### 1.1 结构：

![image-20210510162957787](image-20210510162957787.png)

### 1. 2 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zf</groupId>
    <artifactId>kafka-flink-ignite</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <flink.properties>1.10.1</flink.properties>
        <ignite.version>2.8.1</ignite.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.properties}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_2.11</artifactId>
            <version>${flink.properties}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka-0.10_2.11</artifactId>
            <version>${flink.properties}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.ignite</groupId>
            <artifactId>ignite-core</artifactId>
            <version>${ignite.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.ignite</groupId>
            <artifactId>ignite-spring</artifactId>
            <version>${ignite.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.ignite</groupId>
            <artifactId>ignite-indexing</artifactId>
            <version>${ignite.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.ignite</groupId>
            <artifactId>ignite-flink</artifactId>
            <version>${ignite.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.ignite</groupId>
            <artifactId>ignite-log4j</artifactId>
            <version>${ignite.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.ignite</groupId>
            <artifactId>ignite-rest-http</artifactId>
            <version>${ignite.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>4.3.26.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.1.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.zf.SocketWindowWordCount</mainClass>
                                </transformer>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>META-INF/spring.handlers</resource>
                                </transformer>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>META-INF/spring.schemas</resource>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

### 1. 3 example-ignite.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->

<!--
    Ignite configuration with all defaults and enabled events.
    Used for testing IgniteSink running Ignite in a client mode.
-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/util
        http://www.springframework.org/schema/util/spring-util.xsd">
    <bean id="ignite.cfg" class="org.apache.ignite.configuration.IgniteConfiguration">
        <!-- Enable client mode. -->
        <property name="clientMode" value="false"/>

        <!-- Cache accessed from IgniteSink. -->
        <property name="cacheConfiguration">
            <list>
                <!-- Partitioned cache example configuration with configurations adjusted to server nodes'. -->
                <bean class="org.apache.ignite.configuration.CacheConfiguration">
                    <property name="atomicityMode" value="ATOMIC"/>
                    <property name="name" value="testCache"/>
                </bean>
            </list>
        </property>

        <!-- Enable cache events. -->
        <property name="includeEventTypes">
            <list>
                <!-- Cache events (only EVT_CACHE_OBJECT_PUT for tests). -->
                <util:constant static-field="org.apache.ignite.events.EventType.EVT_CACHE_OBJECT_PUT"/>
                <util:constant static-field="org.apache.ignite.events.EventType.EVT_CACHE_OBJECT_READ"/>
                <util:constant static-field="org.apache.ignite.events.EventType.EVT_CACHE_OBJECT_REMOVED"/>

            </list>
        </property>

        <!-- Explicitly configure TCP discovery SPI to provide list of initial nodes. -->
        <property name="discoverySpi">
            <bean class="org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi">
                <property name="ipFinder">
                    <bean class="org.apache.ignite.spi.discovery.tcp.ipfinder.vm.TcpDiscoveryVmIpFinder">
                        <property name="addresses">
                            <list>
                                <value>127.0.0.1:47500..47509</value>
                            </list>
                        </property>
                    </bean>
                </property>
            </bean>
        </property>
    </bean>
</beans>

```

### 1.4 主函数

```java
package com.zf;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer010;
import org.apache.flink.util.Collector;
import org.apache.ignite.sink.flink.IgniteSink;

import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

public class SocketWindowWordCount {
    public static void main(String[] args) throws Exception {
        /** Ignite test configuration file. */
        final String GRID_CONF_FILE = "example-ignite.xml";

        IgniteSink igniteSink = new IgniteSink("testCache", GRID_CONF_FILE);

        igniteSink.setAllowOverwrite(true);

        igniteSink.setAutoFlushFrequency(5L);

        Configuration p = new Configuration();
        igniteSink.open(p);

        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "test");

        // get the execution environment
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // get input data by connecting to kafka
        DataStream<String> text = env
            .addSource(new FlinkKafkaConsumer010<>("mytopic", new SimpleStringSchema(), properties));

        // parse the data, group it, window it, and aggregate the counts
        SingleOutputStreamOperator<Map<String, Integer>> windowCounts = text
            .flatMap(new Splitter())
            .keyBy(0)
            .timeWindow(Time.seconds(5))
            .sum(1)
            .map(new Mapper());
        // print the results with a single thread, rather than in parallel
        // windowCounts.print();
        windowCounts.addSink(igniteSink);
        env.execute("Window WordCount");
    }

    public static class Splitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String sentence, Collector<Tuple2<String, Integer>> out) throws Exception {
            for (String word: sentence.split(" ")) {
                out.collect(new Tuple2<String, Integer>(word, 1));
            }
        }
    }

    public static class Mapper implements MapFunction<Tuple2<String, Integer>, Map<String, Integer>> {
        @Override
        public Map<String, Integer> map(Tuple2<String, Integer> tuple2) throws Exception {
            Map<String, Integer> myWordMap = new HashMap<>();
            myWordMap.put(tuple2.f0, tuple2.f1);
            return myWordMap;
        }
    }
}
```

## 2. 步骤

### 2.1 安装Flink集群

### 2.2 安装Kafka集群

#### 2.2.1安装

#### 2.2.2 创建Topic

```sh
./bin/kafka-topics.sh --create --topic mytopic --zookeeper localhost:2181 --partitions 1 --replication-factor 1
```

#### 2.2.3 向Topic中写入消息

```sh
./bin/kafka-console-producer.sh --topic mytopic --broker-list localhost:9092
```

>hello java
>hello scala
>hello flink
>hello ignite
>ignite flink kafka

#### 2.2.4 消费消息

```sh
./bin/kafka-console-consumer.sh --topic mytopic --bootstrap-server localhost:9092 --from-beginning
```

### 2.3 安装Ignite集群

### 2.4 打包程序

```sh
mvn clean package
```

### 2.5 执行jar包

```sh
./bin/flink run /opt/code/kafka-flink-ignite.jar
```

## 3. 结果

### 3.1 运行jar包情况如下：

![image-20210511195509106](image-20210511195509106.png)

### 3.2 打开igniteVisor程序，并查看cache具体内容

![image-20210511195535688](image-20210511195535688.png)

### 3.3 向Kafka中插入数据

![image-20210511195552025](image-20210511195552025.png)

## 4. QuickStart

```sh
java -jar /root/demo-ignite/kafka-flink-ignite-1.0.jar
```

另起一个终端

```sh
cd /opt/software/kafka/
./bin/kafka-console-producer.sh --topic mytopic --broker-list localhost:9092
> hello java hello python hello scala hello kotlin hello java
```

另起一个终端

```sh
cd /opt/software/ignite-2.8.1/
./bin/ignitevisorcmd.sh

visor>open
visor>15
visor>cache
visor>cache -scan
visor>0
```



TODO: FlinkKafkaConsumer9