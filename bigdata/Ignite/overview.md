# Apache Ignit

## POM.XML

常用依赖

```xml
<properties>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
    <!-- 要和搭建的版本一致 -->
    <ignite.version>2.9.1</ignite.version>
</properties>

<dependencies>
    <!-- core -->
    <dependency>
        <groupId>org.apache.ignite</groupId>
        <artifactId>ignite-core</artifactId>
        <version>${ignite.version}</version>
    </dependency>
    <!-- support for XML-based configuration -->
    <dependency>
        <groupId>org.apache.ignite</groupId>
        <artifactId>ignite-spring</artifactId>
        <version>${ignite.version}</version>
    </dependency>
    <!-- support for SQL indexing -->
    <dependency>
        <groupId>org.apache.ignite</groupId>
        <artifactId>ignite-indexing</artifactId>
        <version>${ignite.version}</version>
    </dependency>
</dependencies>
```

## 设置启动配置

```sh
ignite.sh ignite-config.xml
```

## Work Directory

Ignit用一个Work Directory来存储，程序运行产生的数据（如果使用的是 `Native Persistence` 方式），index files, metadata information, logs和其他文件

默认的Work Directory：

- 如果使用`./bin/ignite.sh`来启动程序的话，则工作路径为： `$IGNITE_HOME/work`，默认工作路径

- 如果执行jar包的方式运行，则在运行jar包的位置创建work

## 启动模块

Ignite支持多种模块以支持众多功能，但除了`ignite-core,ignite-spring,ignite-indexing`默认都是关闭的，需手动开启，这些模块都被放置在`lib/optional`中

其中`ignite-hibernate,ignite-geospatial,ignite-schedule`需要编译再使用，因为他们不在maven中央仓库中，比如需要使用`ignite-hibernate`，到source目录执行以下语句再讲包添加到本地maven仓库

```sh
mvn clean install -DskipTests -Plgpl -pl modules/hibernate -am
```

## Setting JVM Options

<https://ignite.apache.org/docs/latest/setup>

## 4种生命周期事件

| Event Type        | Description                                                |
| :---------------- | :--------------------------------------------------------- |
| BEFORE_NODE_START | Invoked before the node’s startup routine is initiated.    |
| AFTER_NODE_START  | Invoked right after then node has started.                 |
| BEFORE_NODE_STOP  | Invoked right before the node’s stop routine is initiated. |
| AFTER_NODE_STOP   | Invoked right after then node has stopped.                 |

## 2种集群互联方式

### TCP/IP互联方式

> Tcp/Ip互联方式适合100个左右节点互联的集群

### Zookeeper互联方式

> Zookeeper互联方式适合100到1000个节点间互联的集群