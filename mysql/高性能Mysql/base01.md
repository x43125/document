# Base 01



## 间歇性问题分析：

### 方法：

1. show global status: `mysqladmin ext -i1 -uroot -proot | awk '/Queries/{q=$4-qp;qp=$4}/Threads_connected/{tc=$4}/Threads_running/{printf "%5d %5d %5d\n", q, tc, $4}'`，此命令每秒捕获一次 SHOW GLOBAL STATUS 的数据，输出出来
2. show processlist: `mysql -e 'show processlist\G' -uroot -proot |grep 'State'|sort|uniq -c| sort -rn`， 此命令不停的捕获 SHOW PROCESSLIST 的输出，来观察是否有大量线程处于不正常状态或者有其他不正常的特征。（例如：查询很少会长时间处于 `statistics`状态，此状态一般很快；也很少见到大量线程报告当前连接用户是 “未经验证的用户”）
   1. 大量线程处于 `freeing items`状态是出现了大量有问题查询的很明显的特征。
   2. 也可以在 information_schema下的PROCESSLIST表中查看线程情况
3. 使用查询日志：`awk '/^# Time:/{print $3, $4, c;c=0}/^# User/{c++}' /var/lib/mysql/2ba3731e4a26-slow.log`，此命令可以根据MySQL每秒将当前时间写入日志中的模式统计每秒的查询数量。使用查询日志的前置操作：
   1. 开启慢查询日志
   2. 设置全局级别`long_query_time=0`
   3. 重置所有连接

### 触发器

1. 合适的触发器
2. 合适的阈值，不宜过高或过低（不要将阈值设置在刚好发生问题的地方，发生问题的上升趋势更重要）

### 收集什么样的数据

- 系统状态
- CPU利用率
- 磁盘利用率、可用空间
- ps的输出采样
- 内存利用率
- mysql信息
  - SHOW STATUS
  - SHOW PROCESSLIST
  - SHOW INNODB STATUS

可能还有更多

