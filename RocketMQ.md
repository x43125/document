# rocketMQ

## 速记

### 命令：

```sh
################################################ 启动namesrv
$ nohup sh bin/mqnamesrv &
 
### 验证namesrv是否启动成功
$ tail -f ~/logs/rocketmqlogs/namesrv.log
The Name Server boot success...

################################################ 先启动broker
$ nohup sh bin/mqbroker -n localhost:9876 --enable-proxy &

### 验证broker是否启动成功, 比如, broker的ip是192.168.1.2 然后名字是broker-a
$ tail -f ~/logs/rocketmqlogs/broker_default.log 
The broker[bsroker-a,192.169.1.2:10911] boot success...

### 使用自带用例工具测试消息收发
$ export NAMESRV_ADDR=localhost:9876
$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 SendResult [sendStatus=SEND_OK, msgId= ...

$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
 
### 命令创建topic
$ sh bin/mqadmin updatetopic -n localhost:9876 -b localhost:10911 -t TestTopic

################################################ 关闭broker
$ sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker with proxy enable OK(36695)

################################################ 关闭namesrv
$ sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```

## 安装rocketmq-console

```sh
wget https://github.com/apache/rocketmq-externals/archive/rocketmq-console-1.0.0.tar.gz
tar -xf rocketmq-console-1.0.0.tar.gz
### 重命名，为了方便后续操作
mv rocketmq-externals-rocketmq-console-1.0.0/rocketmq-console  rocketmq-console
### 修改配置文件 修改namesrvAddr=127.0.0.1:9876
cd rocketmq-console
vi src/main/resources/applications.properties
### maven编译
mvn clean  package -DskipTests
### 编译成功后，会在target目录下生成一个jar文件
cp target/rocketmq-console-ng-1.0.0.jar /opt/
################################################ 启动RocketMQ-Console
nohup java -jar rocketmq-console-ng-1.0.0.jar &
```

## bug-fix

```java
// group命名不符合规范
Exception in thread "main" java.lang.IllegalArgumentException: consumerGroup does not match the regex [regex=^[%a-zA-Z0-9_-]+$]

```

## overview

### 为何要选择RocketMQ

[官方给出的各种MQ差异比较](https://rocketmq.apache.org/zh/docs/)

于我而言他是用java开发的，这就够了。^^

### Topic

用于标识同一类业务逻辑的消息，通过TopicName来做唯一标识和区分。属于顶层资源和容器。

### NameServer

Topic 的路由注册中心，为客户根据Topic提供路由服务，Name Server之间节点不通信，**路由信息在NameServer及群众数据一致性采取的最终一致性。**（不通顺，不懂？以后再理吧）。

### Broker

消息存储服务器：Master & Slave，主服务器负责读写，从服务器备份，当主服务器存在压力时，从服务器可承担**读**服务。所有broker每30秒会想NameServer发送心跳，**心跳中包含Broker的所有Topic的路由信息**。

### Client

消息客户端：Producer & Consumer。客户端在同一时间只会连接一台NameServer，只有在连接出现异常的时候才会尝试连接另一台。客户端每隔30s向NameServer发起Topic的路由信息查询。

> NameServer是在内存中存储Topic的路由信息，实际持久化的是在Broker中，即`${ROCKETMQ_HOME/store/config/topics.json}` 

### 消息订阅模型

#### 发布与订阅模式

- Topic: 一类消息的集合；
- ConsumerGroup: 消息消费组，一个消费单位的“群体”，消费组在启动时需要订阅消费Topic，消费组和Topic是多对多的关系。一个消费组拥有多个消费者。

#### 消费模式

广播或集群模式

- 广播模式：一个消费组内的所有消费者完全独立的去消费Topic下的每一条消息，通常用于刷新内存缓存；
- 集群模式：一个消费组内的所有消费者共同消费一个Topic的消息，分工协作，每个消费者消费部分数据，启动负载均衡。

> 集群模式是非常普遍的模式，符合分布式架构的基本理念，即横向扩容，当前消费者如果无法快速及时处理消息时，可以通过增加消费者的个数横向扩容，快速提高消费能力，及时处理挤压的消息。

#### 负载算法 & 重平衡机制

例如：一个Topic有16个队列，由一个3个消费者的消费组来消费

> 在MQ领域有一个不成文的约定：**同一个消费者同一时间可以分配多个队列，但一个队列同一时间只会分配个一个消费者**。

##### 常用负载算法

- AllocateMessageQueueAveragely: 平均分配

对上面的16个队列进行编号：q0~q15，消费组：c0~c2

C0: q0 q1 q2 q3 q4 q5

C1: q6 q7 q8 q9 q10

C2: q11 q12 q13 q14 q15

- AllocateMessageQueueAveragelyByCircle: 轮流平均分配

C0: q0 q3 q6 q9 q12 q15

C1: q1 q4 q7 q10 q13

C2: q2 q5 q8 q11 q14

#### 消息发送方式

- 同步：客户端发送一次消息后会等待服务器返回结果
- 异步：不等结果，服务器响应好后会调回掉函数
- OneWay：不管结果，发完即止

##### OneWay

通常用于发送一些不重要的消息，如操作日志等，丢失一些也没关系。

##### 同步 or 异步

一般都使用同步，很少用异步

### Key

Rocket MQ提供了丰富的消息查询机制，比如 `消息Key`，我们可以通过索引Key进行查询消息。

如果要指定多个key通过空格分开即可，查询方式如下：

![image-20230115154207175](/Users/wangxiang/Library/Application Support/typora-user-images/image-20230115154207175.png)

### Tag

可以为Topic设置Tag，消费端可以用tag过滤Topic中的消息。

### msgId

全剧唯一

- 客户端发送 IP，支持 IPV4 和 IPV6
- 进程 PID（2 字节）
- 类加载器的 hashcode（4 字节）
- 当前系统时间戳与启动时间戳的差值（4 字节）
- 自增序列（2 字节）

### offsetMsgId

消息所在Broker的物理偏移量，即在commitlog文件中的偏移量

- Broker 的 IP 与端口号
- commitlog 中的物理偏移量

> 温馨提示：可以根据 offsetMsgId 即可以定位到具体的消息，无需知道该消息的 Topic 等其他一切信息。

这两个消息 ID 有时候在排查问题的时候，特别是项目能提供 msgID，但是在消息集群中无法查询到时**可以通过解码这消息 ID，从而得知消息发送者的 IP 或消息存储 Broker 的 IP。**

其中 msgId 可以通过 MessageClientIDSetter 的 getIPStrFromID 方法获取 IP，而 OffsetMsgId 可以通过 MessageDecoder 的 decodeMessageId 方法解码。

## example

### 订阅，消费

A每10s发送一次消息

B消费消息























































