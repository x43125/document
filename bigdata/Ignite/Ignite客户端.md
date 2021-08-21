# Ignite客户端

> Ignite服务端启动后，我们则可以使用客户端连接Ignite了
>
> Ignite客户端包括REST，SQL，JAVA/.NET几种

## REST

> 为了使用REST Client需要保证ignite-rest-http模块在JVM的classpath中，将 $IGNITE_HOME/libs/optional 中 ignite-rest-http*模块拷贝到 $IGNITE_HOME/libs 目录中

- 查看server节点版本

```sh
curl 'http://192.168.11.152:8080/ignite?cmd=version'
{"successStatus":0,"sessionToken":null,"error":null,"response":"2.8.1"}
```

- 创建一个cache

```sh
curl 'http://192.168.11.152:8080/ignite?cmd=getorcreate&cacheName=myfirstcache'

StatusCode        : 200
StatusDescription : OK
Content           : {"successStatus":0,"sessionToken":null,"error":null,"response":null}
RawContent        : HTTP/1.1 200 OK
                    Content-Length: 68
                    Content-Type: application/json;charset=utf-8
                    Date: Tue, 11 May 2021 02:53:38 GMT
                    Server: Jetty(9.4.25.v20191220)

                    {"successStatus":0,"sessionToken":null,"error...
Forms             : {}
Headers           : {[Content-Length, 68], [Content-Type, application/json;charset=utf-8], [Date, Tue, 11 May 2021 02:5
                    3:38 GMT], [Server, Jetty(9.4.25.v20191220)]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 68
```

- 向cache中添加数据

```sh
curl 'http://192.168.11.152:8080/ignite?cmd=put&key=Toronto&val=Ontario&cacheName=myfirstcache'

StatusCode        : 200
StatusDescription : OK
Content           : {"successStatus":0,"affinityNodeId":"ce8f77d2-341f-4449-bb32-cd8409a925aa","sessionToken":null,"err
                    or":null,"response":true}
RawContent        : HTTP/1.1 200 OK
                    Content-Length: 124
                    Content-Type: application/json;charset=utf-8
                    Date: Tue, 11 May 2021 02:57:56 GMT
                    Server: Jetty(9.4.25.v20191220)

                    {"successStatus":0,"affinityNodeId":"ce8f77d...
Forms             : {}
Headers           : {[Content-Length, 124], [Content-Type, application/json;charset=utf-8], [Date, Tue, 11 May 2021 02:
                    57:56 GMT], [Server, Jetty(9.4.25.v20191220)]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 124

curl 'http://192.168.11.152:8080/ignite?cmd=put&key=Edmonton&val=Alberta&cacheName=myfirstcache'

StatusCode        : 200
StatusDescription : OK
Content           : {"successStatus":0,"affinityNodeId":"ce8f77d2-341f-4449-bb32-cd8409a925aa","sessionToken":null,"err
                    or":null,"response":true}
RawContent        : HTTP/1.1 200 OK
                    Content-Length: 124
                    Content-Type: application/json;charset=utf-8
                    Date: Tue, 11 May 2021 03:00:19 GMT
                    Server: Jetty(9.4.25.v20191220)

                    {"successStatus":0,"affinityNodeId":"ce8f77d...
Forms             : {}
Headers           : {[Content-Length, 124], [Content-Type, application/json;charset=utf-8], [Date, Tue, 11 May 2021 03:
                    00:19 GMT], [Server, Jetty(9.4.25.v20191220)]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 124
```

- 读取cache中数据

```sh
curl 'http://192.168.11.152:8080/ignite?cmd=get&key=Toronto&cacheName=myfirstcache'

StatusCode        : 200
StatusDescription : OK
Content           : {"successStatus":0,"affinityNodeId":"ce8f77d2-341f-4449-bb32-cd8409a925aa","sessionToken":null,"err
                    or":null,"response":"Ontario"}
RawContent        : HTTP/1.1 200 OK
                    Content-Length: 129
                    Content-Type: application/json;charset=utf-8
                    Date: Tue, 11 May 2021 03:02:35 GMT
                    Server: Jetty(9.4.25.v20191220)

                    {"successStatus":0,"affinityNodeId":"ce8f77d...
Forms             : {}
Headers           : {[Content-Length, 129], [Content-Type, application/json;charset=utf-8], [Date, Tue, 11 May 2021 03:
                    02:35 GMT], [Server, Jetty(9.4.25.v20191220)]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 129

curl 'http://192.168.11.152:8080/ignite?cmd=get&key=Edmonton&cacheName=myfirstcache'
```

## SQL客户端

- ignite官方支持DBeaver数据库，下载DBeaver并安装，连接Ignite数据库，端口为10800

除了部分语法，几乎和其他sql一致

下面sql创建了两张表，添加了索引，并添加了部分数据，最后进行了一个简单查询

```sql
CREATE TABLE IF NOT EXISTS province (
	id long PRIMARY KEY,
	name varchar
) WITH "template=replicated";

CREATE TABLE IF NOT EXISTS city (
	id long,
	name varchar,
	province_id long,
	PRIMARY KEY (id, province_id)
)WITH "backups=1, affinityKey=province_id";

CREATE INDEX IF NOT EXISTS idx_province_name ON province(name);
CREATE INDEX IF NOT EXISTS idx_city_name ON city(name);

INSERT INTO province(id, name) values(1, 'Ontario');
INSERT INTO province(id, name) values(2, 'Alberta');
INSERT INTO province(id, name) values(3, 'Quebec');

INSERT INTO city(id, name, province_id) values(1, 'Toronto', 1);
INSERT INTO city(id, name, province_id) values(2, 'Edmonton', 2);
INSERT INTO city(id, name, province_id) values(3, 'Calgary', 2);
INSERT INTO city(id, name, province_id) values(4, 'Montreal', 3);

SELECT p.name AS PROVINCE, c.name AS CITY FROM province p, city c WHERE p.id = c.province_id
```

- ignite还自带了一个SQL的命令行工具，SQLLine

使用下面命令启动自带的SQL工具

```sh
cd $IGNITE_HOME
./bin/sqlline.sh --verbose=true -u jdbc:ignite:thin://192.168.11.152/
```

sqlline支持的命令

> 输入的方式为，先输入命令 加一个 空格 再输入具体的命令，如sql命令
>
> !sql select * from province;

| 命令            | 描述                            |
| :-------------- | :------------------------------ |
| `!all`          | 在当前的所有连接中执行指定的SQL |
| `!batch`        | 开始执行一批SQL语句             |
| `!brief`        | 启动简易输出模式                |
| `!closeall`     | 关闭所有目前已打开的连接        |
| `!columns`      | 显示表中的列                    |
| `!connect`      | 接入数据库                      |
| `!dbinfo`       | 列出当前连接的元数据信息        |
| `!dropall`      | 删除数据库中的所有表            |
| `!go`           | 转换到另一个活动连接            |
| `!help`         | 显示帮助信息                    |
| `!history`      | 显示命令历史                    |
| `!indexes`      | 显示表的索引                    |
| `!list`         | 显示所有的活动连接              |
| `!manual`       | 显示SQLLine手册                 |
| `!metadata`     | 调用任意的元数据命令            |
| `!nickname`     | 为连接命名（更新命令提示）      |
| `!outputformat` | 改变显示SQL结果的方法           |
| `!primarykeys`  | 显示表的主键列                  |
| `!properties`   | 使用指定的属性文件接入数据库    |
| `!quit`         | 退出SQLLine                     |
| `!reconnect`    | 重新连接当前的数据库            |
| `!record`       | 开始记录SQL命令的所有输出       |
| `!run`          | 执行一个命令脚本                |
| `!script`       | 将已执行的命令保存到一个文件    |
| `!sql`          | 在数据库上执行一个SQL           |
| `!tables`       | 列出数据库中的所有表            |
| `!verbose`      | 启动详细输出模式                |

## Ignite Visor 命令行工具

官方还提供了一个简单的命令行工具

```sh
cd $IGNITE_HOME
./bin/ignitevisorcmd.sh
```

启动后，通过open命令连接到当前使用的ignite集群

```sh
open
-- 选择一个配置文件连接，输入一个数字回车即可
top

visor> top
Hosts: 1
+=======================================================================================================================================================================================+
|   Int./Ext. IPs    |   Node ID8(@)    |                Node consistent ID                 | Node Type |                   OS                    | CPUs |       MACs        | CPU Load |
+=======================================================================================================================================================================================+
| 0:0:0:0:0:0:0:1%lo | 1: CE8F77D2(@n0) | 0:0:0:0:0:0:0:1%lo,127.0.0.1,192.168.11.152:47500 | Server    | Linux amd64 3.10.0-1160.24.1.el7.x86_64 | 2    | 00:0C:29:B6:0F:66 | 1.33 %   |
| 127.0.0.1          |                  |                                                   |           |                                         |      |                   |          |
| 192.168.11.152     |                  |                                                   |           |                                         |      |                   |          |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Summary:
+--------------------------------------+
| Active         | true                |
| Total hosts    | 1                   |
| Total nodes    | 1                   |
| Total CPUs     | 2                   |
| Avg. CPU load  | 1.33 %              |
| Avg. free heap | 87.00 %             |
| Avg. Up time   | 00:50:08            |
| Snapshot time  | 2021-05-10 23:42:20 |
+--------------------------------------+

-- 从中可以看到我们的集群目前只有一个节点，cpu*2

visor> cache
Time of the snapshot: 2021-05-10 23:43:44
+======================================================================================================================================================================+
|         Name(@)          |    Mode     | Nodes | Total entries (Heap / Off-heap) | Primary entries (Heap / Off-heap) |   Hits    |  Misses   |   Reads   |  Writes   |
+======================================================================================================================================================================+
| myfirstcache(@c0)        | PARTITIONED | 1     | 2 (0 / 2)                       | min: 2 (0 / 2)                    | min: 0    | min: 0    | min: 0    | min: 0    |
|                          |             |       |                                 | avg: 2.00 (0.00 / 2.00)           | avg: 0.00 | avg: 0.00 | avg: 0.00 | avg: 0.00 |
|                          |             |       |                                 | max: 2 (0 / 2)                    | max: 0    | max: 0    | max: 0    | max: 0    |
+--------------------------+-------------+-------+---------------------------------+-----------------------------------+-----------+-----------+-----------+-----------+
| SQL_PUBLIC_CITY(@c1)     | PARTITIONED | 1     | 4 (0 / 4)                       | min: 4 (0 / 4)                    | min: 0    | min: 0    | min: 0    | min: 0    |
|                          |             |       |                                 | avg: 4.00 (0.00 / 4.00)           | avg: 0.00 | avg: 0.00 | avg: 0.00 | avg: 0.00 |
|                          |             |       |                                 | max: 4 (0 / 4)                    | max: 0    | max: 0    | max: 0    | max: 0    |
+--------------------------+-------------+-------+---------------------------------+-----------------------------------+-----------+-----------+-----------+-----------+
| SQL_PUBLIC_PROVINCE(@c2) | REPLICATED  | 1     | 3 (0 / 3)                       | min: 3 (0 / 3)                    | min: 0    | min: 0    | min: 0    | min: 0    |
|                          |             |       |                                 | avg: 3.00 (0.00 / 3.00)           | avg: 0.00 | avg: 0.00 | avg: 0.00 | avg: 0.00 |
|                          |             |       |                                 | max: 3 (0 / 3)                    | max: 0    | max: 0    | max: 0    | max: 0    |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Use "-a" flag to see detailed statistics.

-- 从中可以看到我们当前有三个cache
-- 一个是我们通过REST API建立的myfirstcache缓存
-- 剩下两个是我们在DBeaver里面创建的两张数据库表
-- 同时我们还可以看到每个缓存的统计信息
-- 比如缓存是REPLICATED还是PARTITIONED模式，缓存中有多少条目，读写次数，命中率等

visor> help
+=======================================================================+
|   Command   |                       Description                       |
+=======================================================================+
| ack         | Acks arguments on all remote nodes.                     |
| alert       | Alerts for user-defined events.                         |
| cache       | Prints cache statistics, clears cache, prints list of a |
|             | ll entries from cache.                                  |
| close       | Disconnects Visor console from the grid.                |
| config      | Prints node configuration.                              |
| deploy      | Copies file or folder to remote host.                   |
| disco       | Prints topology change log.                             |
| events      | Print events from a node.                               |
| gc          | Runs GC on remote nodes.                                |
| help (?)    | Prints Visor console help.                              |
| kill        | Kills or restarts node.                                 |
| log         | Starts or stops grid-wide events logging.               |
| mcompact    | Fills gap in Visor console memory variables.            |
| mget        | Gets Visor console memory variable.                     |
| mlist       | Prints Visor console memory variables.                  |
| modify      | Modify cache by put/get/remove value.                   |
| node        | Prints node statistics.                                 |
| open        | Connects Visor console to the grid.                     |
| ping        | Pings node.                                             |
| quit (exit) | Quit from Visor console.                                |
| start       | Starts or restarts nodes on remote hosts.               |
| status (!)  | Prints Visor console status.                            |
| tasks       | Prints tasks execution statistics.                      |
| top         | Prints current topology.                                |
| vvm         | Opens VisualVM for nodes in topology.                   |
+-----------------------------------------------------------------------+

Type 'help "command name"' to see how to use this command.

-- 以上为visor的其他命令
```

## Thin Client

### Java Thin Client

```sh
java -cp /opt/software/ignite-2.8.1/libs/*:./IgniteDemo01-1.0.jar \
com.zf.ignite.IgniteThinClientExample \
Toronto Markham Edmonton Calgary

Ignite thin client example started.

Begin create cache and insert data.
Successfully insert all provinces data.
Successfully insert all city data.

Begin query cache.
Find Toronto in province Ontario
Cannot find Markham in any province.  -- 因为并没有Markham所以查不到
Find Edmonton in province Alberta
Find Calgary in province Alberta
```

## Java启动Ignite节点代码

### Java启动Server节点代码

1. 不指定配置文件

```sh
java -cp /opt/software/ignite-2.8.1/libs/*:./IgniteDemo01-1.0.jar \
com.zf.ignite.IgniteServerNodeExample
```

2. 指定配置文件

```sh
java -cp /opt/software/ignite-2.8.1/libs/*:$IGNITE_HOME/libs/ignite-spring/*:./IgniteDemo01-1.0.jar \
com.zf.ignite.IgniteServerNodeExample \
example-server.xml
```

### Java启动client节点代码

1. 不指定配置文件

```sh
java -cp $IGNITE_HOME/libs/*:./IgniteDemo01-1.0.jar \
com.zf.ignite.IgniteClientNodeExample
```

2. 指定配置文件

```sh
java -cp $IGNITE_HOME/libs/*:$IGNITE_HOME/libs/ignite-spring/*:./IgniteDemo01-1.0.jar \
com.zf.ignite.IgniteClientNodeExample \
example-client.xml
```

### 注

启动server和client节点时要注意以下两点：

- Client节点启动时必须有可用的Server节点在集群中，但也可以通过配置强制启动
- 如果通过传入xml配置文件的方式启动节点，则需要在CLASS_PATH中包含ignite-spring模块的jar文件

