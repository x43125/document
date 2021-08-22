# 阿里云服务器，centos7安装部署mysql8

> centos7安装mysql有多种方法，本文采用较为简单的一种，yum源安装

1. 下载yum源

```sh
wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
```

2. 安装yum 源

```sh
yum localinstall mysql80-community-release-el7-1.noarch.rpm
```

3. 检查，一般是不会出现什么问题的，但也可以检查一下是否安装成功

```sh
yum repolist enabled | grep "mysql.*-community.*"
```

4. 安装mysql

```sh
yum install -y mysql
```

5. 启动mysql并设置开机自启

```sh
systemctl start mysqld
systemctl enable mysqld
```

6. 测试，mysql默认会在`/var/log/mysqld.log`下生成一个随机密码，可以使用这个密码来登录，我们先得到这个密码，并登录进去

```sh
grep 'temporary password' /var/log/mysqld.log

mysql -uroot -p 
# 输入密码，回车即可，可以直接复制
```

7. 修改密码，创建用户，修改对外端口号

    > 因为root用户权限过多，一般我们都不使用root用户来操作，也不给root用户远程访问的权力
    >
    > 所以我们需要创建一个新的用户用于操作

    - 创建新用户前我们先把root用户的密码改掉，以防止需要使用时遗忘

    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED BY '你的新密码';  
    -- 密码默认检查策略很高，需要包含大小写，数字和特殊符号且不少于8位
    -- Tips：很多网站都会建议这个模式的密码，也是因为这种程度的密码比较安全
    --  笔者建议每一个用户都准备一条这样的密码用于日常使用，不要再简单的使用生日姓名组合啦 ^_^
    ```

    - 创建新用户，授权，开启远程访问

    ```sql
    CREATE USER 'username'@'host' IDENTIFIED BY '密码';
    GRANT ALL ON *.* TO 'username'@'host'
    -- 'host'即可以使用此用户名登录的IP号，填'%'代表所有IP都可以
    -- 如果想要添加多个IP可访问的话可以使用如下命令
    grant all privileges on *.* to 'root'@'192.168.*.*' identified by '密码';
    flush privileges;
    -- 以上命令我执行时出错，尚未解决，有知道的可以教教我，报错如下
    ERROR 1410 (42000): You are not allowed to create a user with GRANT
    ```

    - 最后是修改对外端口号，一些不法分子经常会针对一些特定端口号如mysql的3306等进行攻击，所以我们更换默认端口号可以起到很好的保护作用

    ```sh
    vim /etc/my.cnf
    # 添加一行
    port=13306
    ```

    - 重启mysql

    ```sh
    systemctl restart mysqld
    ```

    - 最最后针对**阿里云**服务器的用户，阿里云服务器安全组里默认是不对外开放所有端口的，需要自己去安全组里添加对外开放的端口

    ![image-20210605011853745](C:\Users\x3125\AppData\Roaming\Typora\typora-user-images\image-20210605011853745.png)