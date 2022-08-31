# docker安装redis

### 拉取镜像

```sh
docker pull redis
```

### 获取配置文件

官网下载一个，修改部分配置

```sh
# bind 127.0.0.1   # 注释掉这部分，使redis可以外部访问
daemonize yes  # 用守护线程的方式启动
# requirepass 你的密码  # 给redis设置密码
appendonly yes  # redis持久化 默认是no
```

准备环境

```sh
mkdir -p /opt/redis/data
cp redis.conf /opt/redis/data
```

### 启动

```sh
docker run -p 6379:6379 \
 --name redis \
 -v /opt/redis/redis.conf:/etc/redis/redis.conf  \
 -v /opt/redis/data:/data \
 -d redis redis-server /etc/redis/redis.conf \
 --appendonly yes 
```



