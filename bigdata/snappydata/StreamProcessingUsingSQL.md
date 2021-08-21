```
TIBCO ComputeDB的流功能建立在Spark Streaming的基础上，其主要目的是使构建流应用程序以及与内置商店的集成更加简单。这是Spark Streaming指南中的Spark Streaming的简要概述。

Spark Streaming是核心Spark API的扩展，可实现实时数据流的可伸缩，高吞吐量，容错流处理。可以从许多来源（例如Kafka，Flume，Twitter，ZeroMQ，Kinesis或TCP套接字）中提取数据，并可以使用以高级功能（如map，reduce，join和window）表达的复杂算法来处理数据。

最后，可以将处理后的数据推送到文件系统，数据库和实时仪表板。实际上，您可以在数据流上应用Spark的机器学习和图形处理算法。

在内部，它的工作方式如下。 Spark Streaming接收实时输入数据流，并将数据分成批次，然后由Spark引擎处理，以成批生成最终结果流。

Spark Streaming提供了称为离散流或DStream的高级抽象，它表示连续的数据流。 DStreams可以从来自诸如Kafka，Flume和Kinesis之类的源的输入数据流中创建，也可以通过在其他DStreams上应用高级操作来创建。在内部，DStream表示为RDD序列

此处涵盖了有关Spark Streaming概念和编程的其他详细信息。

通过Spark的TIBCO ComputeDB Streaming扩展提供了以下关于Spark Streaming的增强功能：

以声明方式管理流：与SQL表类似，可以从任何SQL客户端以声明方式定义流，并在SnappyStore的持久性系统目录中将其作为表进行管理。声明性语言遵循SQL语言，并提供对任何Spark Streaming流适配器（例如Kafka或文件输入流）的访问。到达的原始元组可以通过可插拔的变压器转换为适当的结构，从而为自定义过滤或类型转换提供所需的灵活性。
基于SQL的流处理：使用表可见的流，可以将它们与其他流或驻留表（参考数据，历史记录等）结合在一起。本质上，整个SQL语言可用于分析分布式流。
连续查询和时间窗口：类似于流行的流处理产品，应用程序可以在流上注册“连续”查询。默认情况下，Spark流每秒发送一次批处理，并且每次发出批处理时都会执行任何已注册的查询。为了支持任意时间范围，对标准SQL进行了扩展，使其能够指定查询的时间窗口。
OLAP优化：通过将流处理与混合内存存储引擎集成和并置在一起，该产品利用优化器和列存储进行昂贵的扫描和聚合，同时通过RowStore提供基于键的快速操作。
近似流分析：当体积太大时，可以使用各种形式的样本和草图来汇总流，以实现快速的时间序列分析。当应用程序对趋势模式感兴趣时，例如在用户显示器上实时渲染一组趋势线时，这特别有用。

使用流表
TIBCO ComputeDB支持从Twitter，Kafka，文件，套接字源创建流表。

例如，使用kafka source创建流表：

可以从snappy-sql访问以上示例中创建的streamTable，并可以使用临时SQL查询对其进行查询。

通过snappy-sql传输SQL
启动TIBCO ComputeDB集群并通过snappy-sql连接：

SchemaDStream
SchemaDStream是基于SQL的DStream，具有对架构/产品的支持。它提供了在DStreams上操作SQL查询的功能。它与提供类似功能的SchemaRDD相似。在内部，将每个批次工期的RDD视为一张小表，并在这些小表上评估CQ。与DStream中的foreachRDD相似，SchemaDStream提供了foreachDataFrame API。 SchemaDStream可以注册为表。其中一些想法（尤其是命名我们的抽象概念）是从英特尔的Streaming SQL项目中借用的。

注册连续查询

//您可以联接两个流表并生成结果流。
val resultStream = snsc.registerCQ（“从流1窗口中选择s1.id，s1.text（持续时间
    “ 2”秒，滑动“ 2”秒）s1 JOIN stream2 s2 ON s1.id = s2.id“）

//您也可以将DataFrame保存到外部表
dStream.foreachDataFrame（_。write.insertInto（“ yourTableName”））

动态（临时）连续查询
与Spark流不同，您无需在StreamingContext开始之前注册所有流输出转换（在这种情况下为连续查询）。即使在SnappyStreamingContext启动之后，也可以注册连续查询。 
```

