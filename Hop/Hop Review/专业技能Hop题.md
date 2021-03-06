# 专业技能面试题

## 1 简单汇总

- openstack 基本概念
- docker 制作镜像

- shiro + 权限 + 数据库设计
- 采集sql元数据信息的方法
- 采集es元数据的方法    
- 读取excel使用的什么方法
- 读取yaml使用的什么方法
- 基础组件架构设计：回调函数；基础插件化 与 业务代码相分离； 抽象
- 业务组件架构设计：底层逻辑 和 业务代码相分离；各种防错设施；内存 与 数据库分离，一致性；必须可以卸载；设计模式
- 了解rocketMQ，和kafka的优缺点
- pg, mysql区别 优缺点
- 集合、反射、注解、lambda函数、并发、JVM内存结构、GC算法、类加载机制
- SSM，SpringBoot
- 常用设计模式，代码设计原则，代码编写规范
- Linux命令，Shell脚本，
- 熟练掌握内存数据库Redis，熟悉Redis基础操作、底层结构、持久化机制、缓存设计、分布式锁等；熟悉C语言；
- 熟悉PostgreSQL，MySQL数据库；熟悉数据结构，计算机网络，操作系统，计算机组成原理等基础知识；
- 解Elasticsearch；熟悉大数据相关组件Zookeeper, Kafka,Hadoop, Spark, Flink等；
    熟悉Python语言，了解深度学习，图像目标检测相关知识；了解Nginx, Apollo等中间件; 了解OpenStack云管理平台、Docker容器化技术；

## 2 分解

### 2.1 简单了解

#### 2.1.1 OpenStack

当今最具影响力的云计算管理工具 -- 通过Web可视化面板来管理 IaaS 云端资源池（服务器、存储和网络）

支持：KVM、Docker等

开发语言：python

使用版本：Queens

部署：kolla-ansible 

> kolla-ansible是从kolla项目中分离出来专门部署openstack基础服务的

#### 2.1.2 Docker

##### 1 简介

Docker是一个开源的应用容器引擎 -- 集装箱 （简单理解就是：用来运行特定应用环境的）

##### 2 开发语言

go

##### 3 构成

DockerClient客户端；Docker Daemon守护进程；Docker Image 镜像；DokcerContainer 容器

##### 4 镜像

类似安装包的一种模板资源，通常使用方式为将一个特定环境做成一个镜像供专门使用，比如：MySQL数据库镜像，Redis镜像；

特点：镜像只包含能够让指定环境运行的最简化环境，体积小；部署快；不包含6个不同的shell甚至不包含shell；不包含内核（容器共享主机的内核），所以会说容器只包含操作系统（操作系统文件和文件系统对象）

拉取镜像：docker image pull redis:latest

查看镜像：docker image ls; 后+ `--filter` 过滤操作 过滤大小，标签等； 后+ `--format` 格式化

搜索镜像：docker search

镜像仓库：类似maven仓库；yum仓库等存储镜像源；官方镜像源：Docker Hub；也可以自己做镜像并存到自己的私有仓库里

**镜像制作**：Docker镜像由一些松耦合的只读镜像层组成；查看分层：docker image inspect

​	所有Docker镜像都起始于一个基础镜像层，修改或新增内容则在其上累加新的镜像层

**核心：** Dockerfile

1. 准备好镜像源，即要制作成镜像的基础环境（centos,ubuntu,microsoft...），以`FROM`开头
2. 编写执行命令：每一层的制作 以 `RUN` 开头

镜像制作的核心文件：Dockerfile，

1. 镜像源以`FROM`开头
2. 每一层的制作 以 `RUN` 开头

**例: MySQL Docker Image**

```sh
# This dockerfile uses the centos image
# VERSION 
# Author: JianJie
# Command format: Instruction [arguments / command] ..

# Base image to use, this must be set as the first line
FROM centos:latest

# Maintainer: docker_user <docker_user at email.com> (@docker_user)
MAINTAINER JianJie jie.jian@transwarp.cn

# Command to update the image
WORKDIR /root
ENV MYSQL_DATA_DIR=/mydata/data
ENV MYSQL_USER=mysql
ENV MYSQL_EXTRACT_DIR=/usr/local
ENV MYSQL_PORT=3306

COPY libaio-0.3.109-13.el7.x86_64.rpm $MYSQL_EXTRACT_DIR
COPY mariadb-10.1.35-linux-x86_64.tar.gz $MYSQL_EXTRACT_DIR
COPY setup.sh /root
RUN chmod +x /root/setup.sh
RUN mkdir -p $MYSQL_DATA_DIR && cd $MYSQL_EXTRACT_DIR && yum install -y libaio-0.3.109-13.el7.x86_64.rpm && \
    echo "Unpacking mysql ..." && tar xf mariadb-10.1.35-linux-x86_64.tar.gz && \
    ln -sf mariadb-10.1.35-linux-x86_64 mysql && rm -rf mariadb-10.1.35-linux-x86_64.tar.gz
RUN groupadd -r -g 306 mysql && useradd -r -g 306 -u 306 mysql && cd $MYSQL_EXTRACT_DIR/mysql && \
    chown -R mysql:mysql ./* && chown -R mysql:mysql $MYSQL_DATA_DIR
RUN cd $MYSQL_EXTRACT_DIR/mysql && cp support-files/mysql.server /etc/rc.d/init.d/mysqld && \
    chkconfig --add mysqld && chkconfig mysqld on && echo "export PATH=$MYSQL_EXTRACT_DIR/mysql/bin:$PATH" >> /etc/profile.d/mysql.sh && \
    source /etc/profile.d/mysql.sh
RUN mkdir -p /etc/mysql && cd $MYSQL_EXTRACT_DIR/mysql && cp support-files/my-large.cnf /etc/mysql/my.cnf && \
    sed -i '/\[mysqld\]/a \datadir = '"$MYSQL_DATA_DIR"'\ninnodb_file_per_table = on\nskip_name_resolve = on' /etc/mysql/my.cnf

RUN yum clean all
EXPOSE 3306

CMD ["/root/setup.sh"]
```

**制作命令**

docker build -t docker-user/mysql:vl /root/dockerfile

##### 5 容器

启动：docker container run 

##### 6 与虚拟机的区别

每个虚拟机都有自己的虚拟cpu、ram、磁盘等资源；都有自己的操作系统；启动慢

容器具有宿主机的单个内核；容器共享一个操作系统 / 内核；启动快；不需要去做：定位、解压、初始化过程；更不用对硬件遍历、初始化；

#### 2.1.3 shiro + 权限设计 + 数据库设计

##### 1 shiro

是Apache基金会下的一个Java安全框架，执行身份验证、授权、密码和会话管理

**核心组件：**

Subject: 当前操作用户 (不单指人，第三方进程、后台账户等与软件交互的东西)

SecurityManager:  Facade模式

Realms：充当Shiro与应用安全数据间的桥梁或连接器（当对用户执行认证和授权验证时，Shiro会从应用配置的Realm中查找用户及其权限消息）;Shiro内置了大量安全数据源，如果不满足可以自定义

##### 2 RBAC

基于角色的访问控制：用户通过角色与权限进行关联：一个用户拥有若干角色；一个角色拥有若干权限

用户 - 角色 - 权限: 多对多

攻防演练系统：root用户、教师、学生

[参考](https://www.cnblogs.com/myseries/p/10871633.html)

**角色：**权限的载体，是一系列权限的集合

当用户量大的时候，给每个用户授权就很繁琐，可以建立用户分组，然后给用户组赋权，因此用户的权限就是用户自己的权限 + 用户所在组的权限和

**权限：**有些会将功能操作与 文件、菜单、页面等分离开构成：用户-角色-权限-资源 模型；但在设计数据库的时候可以不分开统一处理，这样更具便捷性和扩展性

最终关系可见下图：

![img](c07d99bc-e19d-302d-8dea-dc98309bf919.jpg)

最后为了方便管理，可以引入角色组对角色进行分类管理，角色组不参与授权

#### 2.1.4 RocketMQ、RabbitMQ、Kafka

##### 1 什么是消息队列

消息队列是一个有发布者和订阅者两个部分组成的结构，发布者像消息队列中发布消息，订阅者接收到发布的消息后执行操作的一个过程

##### 2 为什么要用消息队列

***解耦，异步，削峰***

**解耦：**传统模式下，每个其他系统接入到主系统中获取消息，主系统都需要编写相应的接入代码，很麻烦不易扩展

消息队列模式下，主系统将消息发送到队列中，其它系统通过订阅获取相关信息，从而主系统只需要对接消息队列即可，剩下的操作由各个系统自行完成

**异步：**传统模式下，主系统与其他业务逻辑同步运行，耗费时间；而通过将消息写入消息队列使非必要的业务逻辑以异步的方式运行，加快响应速度

**削峰：**传统模式下：并发量大的时候，所有请求直接怼到数据库，造成数据库连接异常；而通过消息队列的方式将所有请求放到消息队列中，让数据库连接充当订阅者以数据库能够承担的连接数慢慢的从队列中取请求执行操作

##### 3 使用消息队列的缺点

- 系统可用性降低：如果消息队列挂了，系统就阻塞了
- 系统复杂性提高：需要考虑的东西就变多了，如一致性问题、消息不被重复消费、消息不被忽略、消息可靠传输等

##### 4 选型

***ActiveMQ, RabbitMQ, RocketMQ, Kafka***

| 特性           | ActiveMQ                                                     | RabbitMQ (开源)                                              | RocketMQ（阿里）                                             | kafka（大数据）                                              |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 开发语言       | java                                                         | erlang                                                       | java                                                         | scala                                                        |
| 单机吞吐量     | 万级                                                         | 万级                                                         | 10万级                                                       | 10万级                                                       |
| 时效性         | ms级                                                         | us级                                                         | ms级                                                         | ms级以内                                                     |
| 可用性         | 高(主从架构)                                                 | 高(主从架构)                                                 | 非常高(分布式架构)                                           | 非常高(分布式架构)                                           |
| 功能特性       | 成熟的产品，在很多公司得到应用；有较多的文档；各种协议支持较好 | 基于erlang开发，所以并发能力很强，性能极其好，延时很低;管理界面较丰富 | MQ功能比较完备，扩展性佳                                     | 只支持主要的MQ功能，像一些消息查询，消息回溯等功能没有提供，毕竟是为大数据准备的，在大数据领域应用广。 |
| 中小型公司选择 | 更新速度较慢，官方工作重心转移                               | 中小型公司选择，erlang具备高并发特性，管理界面使用方便，社区活跃 | 阿里出品，如果阿里放弃维护，中小型公司一般抽不出人力来定制化开发，不推荐 | 中小型公司没有那么大数据量，首选功能完善的，所以不用kafka    |
| 大型公司选择   |                                                              |                                                              | 人力财力充足，能维护                                         | 如果有日志收集需要                                           |

**5 高可用**

> 生产环境中没人使用单机模式的消息队列

Kafka保持高可用

zookeeper高可用，**选举规则**

**应能手画出高可用架构图**



##### 6 保证消息不被重复消费

**为什么：**消息队列在被消费后会发回确认消息，导致重复消费的原因主要是网络等原因阻塞了确认消息的及时传到队列，队列未接收到确认消息则认定消费失败，则允许其他消费者消费。

**解决：**

- 比如如果拿到数据是做insert操作，则给这个消息做一个唯一主键，那么就算重复消费也会因为冲突不会插入数据库出现脏数据；
- redis set操作则不需要做操作，因为redis set操作本身就是幂等操作；
- 准备一个第三方介质，来做消费记录。没消费一次某消息则将该消息存入redis中，每次消费某消息前先去redis中确认一遍是否已经有该纪律。

##### 7 可靠性传输

**为什么：**生产者丢数据、消息队列丢数据、消费者丢数据

- 生产者丢数据：RabbitMQ提供transaction和confirm模式来确保生产者不丢消息
    - transaction：发送消息前开启事务，然后发送消息，发送过程中出现问题则会回滚；但吞吐量下降
    - confirm：给信道上面发布的消息都只拍一个唯一id，等到全部投递完全的时候发送一个Ack给生产者，生产者知道消息已经到达目的地队列了，如果rabbitMQ没有处理该消息，则会发送一个Nack给用户，进行重试操作
- 消息队列丢数据：开启持久化磁盘配置
- 消费者丢数据：自动确认消息模式

**8 顺序性**

保证入队有序即可，出对顺序由消费者自己去保证



































































