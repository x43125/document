# MySQL root密码遗忘，重置

1、关闭MySQL服务

使用管理员身份进入cmd，并进入mysql /bin目录下，执行命令

```sh
net stop mysql
```

2、跳过密码验证

执行命令

```sh
mysqld --console --skip-grant-tables --shared-memory
```

3、无密码方式登录

使用管理员另开启一个cmd，到mysql/bin目录下，直接执行命令

```sh
mysql
```

4、讲root用户密码置空

```sql
use mysql; -- (使用mysql数据表)
update user set authentication_string='' where user='root';  -- （将密码置为空）
quit;  -- (然后退出Mysql)
```

5、重新登陆

关闭第一个cmd，在第二个cmd中继续操作

确保MySQL服务已经关闭

开启MySQL服务

```sh
net start mysql
mysql -uroot -p
```

密码为空，直接回车即可进入MySQL终端

更改密码：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';  -- （更改密码）
```

