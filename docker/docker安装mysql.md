# docker 安装部署mysql

### 拉取镜像

```sh
docker pull mysql
# docker pull mysql:5.7  # 指定版本
```

### 启动MySQL

```sh
docker run -p 3306:3306 --name mysql -v /opt/mysql/conf:/etc/mysql/conf.d -v /scy/mysql/logs:/logs -v /scy/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.4

sudo docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7

docker run -p 6033:3306 --name mysql \
-v /opt/mysql/conf:/etc/mysql \
-v /opt/mysql/logs:/var/log/mysql \
-v /opt/mysql/data:/var/lib/mysql \
-v /opt/mysql/mysql-files:/var/lib/mysql-files/ \
-e MYSQL_ROOT_PASSWORD=123ABCdef* \
-d mysql

# 当做了外部目录映射之后也要映射/var/lib/mysql-files/ 
```

### 配置

```sh
# 进入容器
docker exec -it mysql bash
# 登录MySQL
# 第一次进入后直接输入mysql即可
mysql
# 进入MySQL命令行
# 修改root密码
ALTER USER 'test'@'localhost' IDENTIFIED WITH MYSQL_NATIVE_PASSWORD BY '新密码';
update user set host='%' where user='root';

# 赋权
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
# 写入
flush privileges

# 退出重启mysql容器
docker restart mysql
```









































