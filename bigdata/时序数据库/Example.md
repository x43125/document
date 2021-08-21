# 时序数据库案例

```sql
# 1.Create a schema for a new hypertable  
CREATE TABLE sensor_data (
    "time" timestamp with time zone NOT NULL,  
    device_id TEXT NOT NULL,  
    location TEXT NULL,  
    temperature NUMERIC NULL,  
    humidity NUMERIC NULL,  
    pm25 NUMERIC  
);
# 2.Create a hypertable from this data 
SELECT create_hypertable ('sensor_data','time','device_id',16);
# 3.Migrate data from existing Postgres table into a TimescaleDB hypertable  
INSERT INTO sensor_data (SELECT * FROM old_data);

# Query hypertable like any SQL table  
SELECT device_id, AVG(temperature) 
from sensor_data  
WHERE temperature IS NOT NULL AND humidity >0.5 AND time > now() - interval'7 day'GROUP BY device_id;

# Metrics about resource-constrained devices  
SELECT 
	time, cpu, freemem, battery 
FROM devops
WHERE device_id='foo'AND cpu >0.7AND freemem <0.2 ORDER BY time DESC LIMIT 100;

# Calculate total errors by latest firmware versions  
# per hour over the last 7 days  
SELECT 
	date_trunc('hour', time) as hour, firmware, COUNT(error_msg)aserrno 
FROM data  
WHERE firmware >50 AND time > now() - interval'7 day' GROUP BY hour, firmware  ORDER BY hour DESC, errno DESC;

# Find average bus speed in last hour  
# for each NYC borough  
SELECT loc.region, AVG(bus.speed) FROM bus  INNER JOIN locON(bus.bus_id = loc.bus_id)  
WHERE loc.city='nyc'AND bus.time > now() - interval'1 hour'GROUP BY loc.region;
```





## Example02

```sql
-- 创建表一张基础表:
CREATE TABLE conditions (
      time TIMESTAMPTZ NOT NULL,
      device INTEGER NOT NULL,
      temperature FLOAT NOT NULL,
      PRIMARY KEY(time, device)
);
-- 将表转成超级表
SELECT create_hypertable('conditions', 'time');

--用timescaledb.continuous view选项创建视图。视图中使用time_bucket函数将温度汇总到按小时为间隔的时间段中。
CREATE MATERIALIZED VIEW conditions_summary_hourly
WITH (timescaledb.continuous) AS
SELECT device,
       time_bucket(INTERVAL '1 hour', time) AS bucket,
       AVG(temperature),
       MAX(temperature),
       MIN(temperature)
FROM conditions
GROUP BY device, bucket;

-- 可以在一个时序基础表中建立多个持续聚合的视图，如下，按天聚合的视图
CREATE MATERIALIZED VIEW conditions_summary_daily
WITH (timescaledb.continuous) AS
SELECT device,
       time_bucket(INTERVAL '1 day', time) AS bucket,
       AVG(temperature),
       MAX(temperature),
       MIN(temperature)
FROM conditions
GROUP BY device, bucket;

--持续聚合视图支持大多数聚合函数的并行计算，如SUM,AVG等，但是order by和distinct 不能使用并行计算，另外filter子句也不支持并行计算。

--如下，查询改视图，可以得到第一季度的device为5的最大，最小，以及平均温度。
SELECT * FROM conditions_summary_daily
WHERE device = 5
  AND bucket >= '2018-01-01' AND bucket < '2018-04-01';
  
--当然也可以做更复杂的一些查询
SELECT * FROM conditions_summary_daily
WHERE max - min > 1800
  AND bucket >= '2018-01-01' AND bucket < '2018-04-01'
ORDER BY bucket DESC, device DESC LIMIT 20;

```



## Example03 楼盘

> 以下SQL查询某楼盘，开盘价，收盘价，最高价，最低价，平均价等信息

```sql
SELECT time_bucket('3 hours', time) AS period
    asset_code,
    first(price, time) AS opening, 
    last(price, time) AS closing,
    max(price) AS high, 
    min(price) AS low,
    avg(price) AS avg
FROM prices
WHERE time > NOW() - interval '7 days'
GROUP BY period, asset_code
ORDER BY period DESC, asset_code;
```

