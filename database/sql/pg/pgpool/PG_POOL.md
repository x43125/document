# pg_pool相关操作
## pg_pool打包：
### pg_pool本体打包
```shell
cd ${PG_POOL_PATH}
./configure --prefix=/pathToInstall
make 
make install
```
### pg 自带插件打包
```shell
cd ${PG_POOL_PATH}/src/sql
make
make install DESTDIR=/install/directory 
```





## 速记

### 1 概念：

pgpool-ii 相当于pg服务器和pg数据库中间件；(我的理解就是相当于zookeeper的概念，给pg提供一个自动切换，HA功能)

提供一个虚拟IP来供外界对pg进行访问使用；

### 2 责任项

流复制数据同步：通过配置pg数据库来实现

虚拟IP自动切换：通过配置pgpool-ii来实现

数据库主备角色切换：通过pgpool-ii 监测机+执行pg的`promote`命令来实现

### 3 部署流程

#### 3.1 pg部署，及流复制配置

- 部pg

- 防火墙 / 开端口

- 配置远程访问

- 配置pg流复制

    - 主库：

        - 创建一个 `repuser` 用户给备库远程访问来获取流
        - 修改 `pg_hba,conf` 、`postgresql.con`
        - 重启pg

    - 从库：

        - 关闭pg
        - 如果没有则创建：pg-data目录并赋权
        - 把主库整个备份到从库

        ```shell
        su – postgres
        pg_basebackup -h 10.242.111.204 -p 5432 -U repuser -Fp -Xs -Pv -R -D /var/lib/pgsql/12/data
        ```

        - 注意事项！！！：
            - `pg_hba.conf` 中配置为 `trust` 则此处无需口令，如果为 `md5` 则需要输入创建用户时的密码
            -  `-R` 参数一定要加，拷贝完成之后会在 `$PG_DATA` 目录下生成 `standby.signal` 标志文件用于标识备库
            - 同步完成后，会自动在 `$PG_DATA` 目录下生成 `postgresql.auto.conf` 文件，优先级大于 `postgresql.conf`

        - 启动备库pg

-  流复制验证

   ```sql
   postgresql=# \x
   postgresql=# select * from pg_stat_replication;
   postgresql=# create database test_204;
   postgresql=# \test_204
   postgresql=# create table test_table(name text);
   postgresql=# insert into test_table(name) values ('china');
   ```
   
- pg主从切换

    - 主库-读写；备库-只读
    - `$PG_HOME/bin/pg_controldata $PG_DATA` 查看数据库运行状态
    - 主库故障，备库升级
        - pg_ctl方式切换备库为主库：在备库主机执行 `pg_ctl promote shell`
        - ***pg12版本以前的方式：***（触发器文件方式：备库配置 `recovery.conf` 文件的 `trigger_file` 参数；pg-12不支持通过recovery.conf的方式进行主备切换，pg13未知；新增recovery.signal, standby.signal文件）
        - `pg_promote(wait boolean DEFAULT true, wait_seconds integer DEFAULT 60)`

    - 切换步骤：
        - 关闭主库
        - 备库上执行切换
            - 方式一：`$PG_HOME/bin/pg_ctl promote -D $PG_DATA`
            - 方式二：`su postgres; psql; postgresql=# select pg_promote(true, 60);`

        - 之后备库则升级为主库，接替集群中的读写操作


> pgpool-ii 中默认的 `failover_command` 切换数据库主备角色脚本就是通过 `pg_ctl promote` 实现的

#### 3.2 pgpool-ii 部署，及配置

> pgpool-ii 功能：完成HA，IP自动切换
>
> - 检测数据库状态，执行相应策略（主库挂掉，切换某个备库为主库）
> - 当某一个pgpool节点不可用，其他节点收到消息进行IP转移（访问入口接管）

- 免密登录 -  ***如果必须要改这一步的话则无法使用在我们的环境中，寻求解决方案***

- 安装pgpool-ii 

    - 源码编译安装

    - rpm包安装

        - 目录：

        ```shell
        #  因为pgpool-ii 的配置中会以 postgres 用户执行一些系统权限命令, 需要使用设置普通用户授权:
        chmod u+x /usr/sbin/ip
        chmod u+s /usr/sbin/arping
        chmod u+s /sbin/ip
        chmod u+s /sbin/ifconfig
        # 配置中相关的日志目录，pid 目录权限：
        chown postgres:postgres /var/run/pgpool
        mkdir -p /var/log/pgpool
        touch /var/log/pgpool/pgpool_status
        chown -R postgres:postgres /var/log/pgpool
        ```

    - 配置pgpool-ii **(主库设置好后，直接发送到备库)**

        - 默认配置文件在：`/etc/pgpool-II `下
        - `pool_hba.conf` （主备相同）
        - `pcp.conf/pool_passwd` （主备相同）这个文件是pgpool用来管理自己的
        - `pgpool.conf` （重点）
            - 通用基础配置

        ```shell
        pid_file_name = '/var/run/pgpool/pgpool.pid'# pid 文件位置， 如果不配置有默认的
        logdir = '/var/run/pgpool'                  # status 文件存储位置
        # 通用
        listen_addresses = '*'
        port = 9999
        pcp_listen_addresses = '*'
        pcp_port = 9898
        # 后台数据库链接信息配置
        backend_hostname0 = 'master'                # 第一台数据库信息
        backend_port0 = 5432
        backend_weight0 = 1                         # 这个权重和后面负载比例相关
        backend_data_directory0 = '/var/lib/pgsql/12/data'
        backend_flag0 = 'ALLOW_TO_FAILOVER'
        
        backend_hostname1 = 'slave'                 # 第二台数据库信息
        backend_port1 = 5432
        backend_weight1 = 1
        backend_data_directory1 = '/var/lib/pgsql/12/data'
        backend_flag1 = 'ALLOW_TO_FAILOVER'
        
        # 流复制相关配置
        replication_mode = off                      # pgpool-ii 中复制模式关闭
        load_balance_mode = on                      # 负载均衡打开
        master_slave_mode = on                      # 主从打开
        master_slave_sub_mode = 'stream'            # 主从之间模式为流传输stream
        
        sr_check_period = 5                         # 流复制检查相关配置
        sr_check_user = 'repuser'
        sr_check_password = 'repuser'
        sr_check_database = 'postgres'
        ```

        - 数据库故障转移配置

        ```shell
        # 数据库运行状况检查，以便Pgpool-II执行故障转移: 数据库的主备切换
        health_check_period = 10                    # Health check period, Disabled (0) by default
        health_check_timeout = 20                   # 健康检查的超时时间,0 永不超时
        health_check_user = 'postgres'              # 健康检查的用户
        health_check_password = 'postgres'          # 健康检查的用户密码
        health_check_database = 'postgres'          # 健康检查的数据库
        
        # 故障后处理, 为了当postgresql数据库挂掉之后执行相应的策略
        # 这个脚本是放在pgpool的目录下, 确切的说是由pgpool执行脚本来维护集群中数据库的状态，就是替代了前文的手动替换备库为主库的步骤
        failover_command = '/etc/pgpool-II/failover.sh %H %R '
        # follow_master_command = ''                # 2台服务器不配置
        # 如果使用3台PostgreSQL服务器，则需要指定follow_master_command在主节点故障转移上的故障转移后运行。
        # 如果只有两台PostgreSQL服务器，则无需设置。
        # 具体脚本文件内容见文末
        ```

        - 看门狗  `watchdog` 配置 （用于检测pgpool-ii 节点本身状态，为pgpool故障提供处理）

        ```shell
        use_watchdog = on                           # 激活看门狗配置
        wd_hostname = 'master'                      # 当前主机(也可使用IP)
        wd_port = 9000                              # 工作端口
        
        # 虚拟IP指定
        delegate_IP = '10.242.111.203'
        if_cmd_path = '/sbin'                       # 如果if_up_cmd, if_down_cmd 以/开头, 忽略此配置
        # 命令中的`ens160` 请根据自己机器上ip addr 实际的网卡名称进行修改
        # 当前节点启动指定虚拟IP的命令
        if_up_cmd = '/usr/bin/sudo /sbin/ip addr add $_IP_$/24 dev ens160 label ens160:0'
        # 当前节点指定关闭虚拟IP的命令
        if_down_cmd = '/usr/bin/sudo /sbin/ip addr del $_IP_$/24 dev ens160'
        
        # watchdog 健康检查
        wd_heartbeat_port = 9694                    # 健康检查端口
        wd_heartbeat_keepalive = 2
        wd_heartbeat_deadtime = 30
        # 其他机器地址配置(多台请增加配置)
        heartbeat_destination0 = 'slave'
        heartbeat_destination_port0 = 9694
        heartbeat_device0 = 'ens160'
        
        # 其他pgpgool节点链接信息(多台请增加配置)
        other_pgpool_hostname0 = 'slave'            # 其他节点地址
        other_pgpool_port0 = 9999
        other_wd_port0 = 9000                       # 其他节点watchdof 端口
        
        # watchdog 发生故障后, 处理的相关配置(宕机, pgpool进程终止)
        # 当某个节点故障后, 
        failover_when_quorum_exists = on
        failover_require_consensus = on
        allow_multiple_failover_requests_from_node = on
        enable_consensus_with_half_votes = on
        ```

        > 注意事项！！！
        >
        > - watchdog本身发生故障后，如果配置打开，其他节点会执行仲裁，如仲裁：从节点中哪一个称为主节点，哪一台接管虚拟IP等；
        > - 这个仲裁本身有投票机制，也有无视仲裁结果等配置
        > - 如果未配置，主pgpool-ii 节点关闭后，可能不会转移 虚拟IP，从而出现暂时不可访问集群的情况

        - 在线恢复（master恢复上线后自动变为备库）

        ```shell
        # 此配置将在多个pgpool-ii 节点时无效
        # 如果有多个pgpool-ii 节点共同维护集群状态, 此配置将不可用, 需要手动恢复同步数据>加入集群
        recovery_user = 'postgres'
        recovery_password = 'postgres'
        recovery_1st_stage_command = 'recovery_1st_stage'   # 这个脚本时放在postgresql数据目录下的
        ```

    - 备库

    ```shell
    scp master:$PG_POOL_CONF_DIR/* $PG_POOL_CONF_DIR/
    ```

- 启动验证

    - 启动 / 停止

    ```shell
    su - postgres
    # 启动命令
    pgpool -n -d -D > $PG_POOL_LOG_DIR/pgpool.log 2>&1 &  # 有debug日志
    pgpool -n -D > $PG_POOL_LOG_DIR/pgpool.log 2>$1 %
    # 停止命令
    pgpool -m fast stop
    ```

    - 查询集群状态

    ```shell
    psql -h vip -p9999 -Upostgres -d postgres
    # 键入密码后进入pg客户端
    postgres# show pool_nodes;
    
     node_id | hostname | port | status | lb_weight |  role   | select_cnt | load_balance_node | replication_delay | replication_state | replication_sync_state | last_status_change
    ---------+----------+------+--------+-----------+---------+------------+-------------------+-------------------+-------------------+------------------------+---------------------
     0       | master   | 5432 | up     | 0.500000  | primary | 0          | true              | 0                 |                   |                        | 2020-06-22 17:48:51
     1       | slave    | 5432 | up     | 0.500000  | standby | 0          | false             | 0                 |                   |                        | 2020-06-22 17:48:51
    (2 rows)
    ```

- 宕机验证
    - pgpool-ii 节点宕机
    - pg数据库宕机



### 资源分配：

| 资源 | 描述 |
| ---- | ---- |
|      |      |



### 注意实现

- `/zfdata/postgresql` 的权限为700或750
- 当数据库或pgpool宕机时，切换需要很长时间，
- 当数据库恢复时，如果只有一台pgpool可以使用脚本自动在线将数据库加入进集群中，如果是多台则需要手动回复，































































































