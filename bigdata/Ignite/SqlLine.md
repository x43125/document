# SQLLine连接模式

在开启了集群之后，使用如下命令进入sqlLine模式

```sh
sqlline.sh -u jdbc:ignite:thin:[host]
```

如果集群开启了认证，则需要做相应认证

```sh
jdbc:ignite:thin://[address]:[port];user=[username];password=[password]
```

## sqlLine支持的命令列表

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

