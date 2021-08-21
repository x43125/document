# SQLCreateStreamTable

```sql
// DDL for creating a stream table
CREATE STREAM TABLE [IF NOT EXISTS] table_name
    ( column-definition [ , column-definition  ] * )
    USING kafka_stream | file_stream | twitter_stream | socket_stream
    OPTIONS (
    // multiple stream source specific options
      storagelevel 'cache-data-option',
      rowConverter 'rowconverter-class-name',
      subscribe 'comma-seperated-topic-name',
      kafkaParams 'kafka-related-params',
      consumerKey 'consumer-key',
      consumerSecret 'consumer-secret',
      accessToken 'access-token',
      accessTokenSecret 'access-token-secret',
      hostname 'socket-streaming-hostname',
      port 'socket-streaming-port-number',
      directory 'file-streaming-directory'
    )
```

### 注解：

1. 如果表中已经存在该表名，则报错

| 参数                 | 描述                                                       |
| -------------------- | ---------------------------------------------------------- |
| USING \<data source> | 指定用于此表的流媒体源。                                   |
| storageLevel         | 在内存使用量和CPU效率之间提供不同的权衡。                  |
| rowConverter         | 将非结构化流数据转换为一组行。                             |
| topics               | 订阅的Kafka主题。                                          |
| kafkaParams          | Kafka配置参数，例如meta.broker.list，bootstrap.servers等。 |
| directory            | HDFS目录以监视新文件。                                     |
| hostname             | 要连接的主机名，用于接收数据。                             |
| port                 | 要连接的端口，用于接收数据。                               |
| consumerKey          | 您的Twitter帐户的使用者密钥（API密钥）。                   |
| consumerSecret       | 您的Twitter帐户的消费者密钥。                              |
| accessToken          | 访问您的Twitter帐户的令牌。                                |
| accessTokenSecret    | 访问您的Twitter帐户的令牌机密。                            |

**注：**

> You need to register to https://apps.twitter.com/ to get the `consumerKey`, `consumerSecret`, `accessToken` and `accessTokenSecret` credentials.
>
> 您需要注册到https://apps.twitter.com/以获取consumerKey，consumerSecret，accessToken和accessTokenSecret凭证。 

### 案例

```sql
//create a connection
snappy> connect client 'localhost:1527';

// Initialize streaming with batchInterval of 2 seconds
snappy> streaming init 2secs;

// Create a stream table
snappy> create stream table streamTable (id long, text string, fullName string, country string,
        retweets int, hashtag  string) using twitter_stream options (consumerKey '', consumerSecret '',
        accessToken '', accessTokenSecret '', rowConverter 'org.apache.spark.sql.streaming.TweetToRowsConverter');

// Start the streaming
snappy> streaming start;

//Run ad-hoc queries on the streamTable on current batch of data
snappy> select id, text, fullName from streamTable where text like '%snappy%';

// Drop the streamTable
snappy> drop table streamTable;

// Stop the streaming
snappy> streaming stop;
```

