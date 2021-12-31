# Postgresql安装部署

到 https://www.postgresql.org/download/ 选择相应平台的PG版本，本文以centos7为例

https://www.postgresql.org/download/linux/redhat/ 选择相应的PG版本，平台版本后网站会给出相应的安装命令，复制到平台上安装即可

```sh
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y postgresql13-server
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
sudo systemctl enable postgresql-13
sudo systemctl start postgresql-13
```

执行所有命令后，Postgresql即安装成功，默认只有一个超级用户Postgres，并且关闭远程连接，为了安全使用和方便使用，第一步先创建一个普通用户，再开启远程访问

## 创建用户

```sh
su - postgres
psql
```

```sql
-- 创建了一个叫wx的超级用户，密码为456123，又创建了一个同名数据库，并设置owner为wx，并将该数据库的所有权限赋给用户wx
create role wx with superuser;
alter role wx with password '456123';
create database wx;
alter database wx owner to wx;
grant all on database wx to wx;

```



## 开启远程连接

1. 修改`postgresql.conf`

```sh
vim /var/lib/pgsql/13/data/postgresql.conf
```

将`#listen_addresses = 'localhost'`修改成 `listen_addresses = '*'`

2. 修改`pg_hba.conf`

```sh
vim /var/lib/pgsql/13/data/pg_hba.conf
```

在末尾新增一行：

```sh
host    all             all             0.0.0.0/0               md5
host    all				all				待连接的地址				password
```

保存退出后，重启pg

```sh
systemctl restart postgresql-13
```



