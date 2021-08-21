# SQL Create Table

```sql
CREATE TABLE [IF NOT EXISTS] table_name 
    ( column-definition [ , column-definition  ] * )    
    USING [row | column] // If not specified, a row table is created.
    OPTIONS (
    COLOCATE_WITH 'table-name',  // Default none
    PARTITION_BY 'column-name', // If not specified, replicated table for row tables, and partitioned internally for column tables.
    BUCKETS  'num-partitions', // Default 128. Must be an integer.
    COMPRESSION 'NONE', //By default COMPRESSION is 'ON'. 
    REDUNDANCY       'num-of-copies' , // Must be an integer. By default, REDUNDANCY is set to 0 (zero). '1' is recommended value. Maximum limit is '3'
    EVICTION_BY 'LRUMEMSIZE integer-constant | LRUCOUNT interger-constant | LRUHEAPPERCENT',
    PERSISTENCE  'ASYNCHRONOUS | ASYNC | SYNCHRONOUS | SYNC | NONE’,
    DISKSTORE 'DISKSTORE_NAME', //empty string maps to default diskstore
    OVERFLOW 'true | false', // specifies the action to be executed upon eviction event, 'false' allowed only when EVCITON_BY is not set.
    EXPIRE 'time_to_live_in_seconds',
    COLUMN_BATCH_SIZE 'column-batch-size-in-bytes', // Must be an integer. Only for column table.
    KEY_COLUMNS  'column_name,..', // Only for column table if putInto support is required
    COLUMN_MAX_DELTA_ROWS 'number-of-rows-in-each-bucket', // Must be an integer > 0 and < 2GB. Only for column table.
    )
    [AS select_statement];
```

### 注解：

1. 如果表中已经存在该表名，则报错

| 参数                                   | 描述                                                       |
| -------------------------------------- | ---------------------------------------------------------- |
| `column-definition` (for Column Table) | 指定用于此表的流媒体源。                                   |
| storageLevel                           | 在内存使用量和CPU效率之间提供不同的权衡。                  |
| rowConverter                           | 将非结构化流数据转换为一组行。                             |
| topics                                 | 订阅的Kafka主题。                                          |
| kafkaParams                            | Kafka配置参数，例如meta.broker.list，bootstrap.servers等。 |
| directory                              | HDFS目录以监视新文件。                                     |
| hostname                               | 要连接的主机名，用于接收数据。                             |
| port                                   | 要连接的端口，用于接收数据。                               |
| consumerKey                            | 您的Twitter帐户的使用者密钥（API密钥）。                   |
| consumerSecret                         | 您的Twitter帐户的消费者密钥。                              |
| accessToken                            | 访问您的Twitter帐户的令牌。                                |
| accessTokenSecret                      | 访问您的Twitter帐户的令牌机密。                            |

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

