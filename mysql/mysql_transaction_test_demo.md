# 事务

> 1. 数据库事务
> 2. 数据库事务测试
> 3. Java语句事务
> 4. springboot事务

## 1. MySQL事务

### 1.1 MySQL事务

### 1.2 MySQL事务隔离级别

#### 1.2.1 隔离级别

#### 1.2.2 可能发生的问题

#### 1.2.3 示例



### 1.3 DEMO

#### 1.3.1 创建一个测试表

```sql
create table testtran(
    id int primary key not null auto_increment,
    username varchar(50),
    money integer(100)
);
```

#### 1.3.2 插入数据

```sql
insert into 
	testtran(username, money) 
values
	('zhangsan', 1000),
	('lisi', 2000),
	('wanger', 0)
;
select * from testtran;
mysql> select * from testtran;
+----+----------+-------+
| id | username | money |
+----+----------+-------+
|  1 | zhangsan |  1000 |
|  2 | lisi     |  2000 |
|  3 | wanger   |     0 |
+----+----------+-------+
3 rows in set (0.00 sec)
```

#### 1.3.3 sql测试语句

```sql
-- 查看提交方式
select @@AUTOCOMMIT;
mysql1> select @@AUTOCOMMIT;
+--------------+
| @@AUTOCOMMIT |
+--------------+
|            1 |
+--------------+
1 row in set (0.00 sec)
-- 两种方式：
-- 1.关闭自动提交，直到重新开启自动提交
SET AUTOCOMMIT=0;
-- 2.手动开启，直到出现rollback或commit，出现之后则不是事务操作
START TRANSACTION;
-- 或
BEGIN;
-- 查询当前表中数据
mysql1> select * from testtran;
+----+----------+-------+
| id | username | money |
+----+----------+-------+
|  1 | zhangsan |  1000 |
|  2 | lisi     |  2000 |
|  3 | wanger   |     0 |
+----+----------+-------+
3 rows in set (0.00 sec)
-- 插入一条数据
mysql1> insert into testtran values(4,'weiwu',0),('5','pangtong',0);
Query OK, 2 row affected (0.00 sec)
-- 查询当前库中数据
mysql1> select * from testtran;
+----+----------+-------+
| id | username | money |
+----+----------+-------+
|  1 | zhangsan |  1000 |
|  2 | lisi     |  2000 |
|  3 | wanger   |     0 |
|  4 | weiwu    |     0 |
|  5 | pangtong |     0 |
+----+----------+-------+
5 rows in set (0.00 sec)
-- 另起一个客户端查看此表中数据
mysql2> select * from testtran;
+----+----------+-------+
| id | username | money |
+----+----------+-------+
|  1 | zhangsan |  1000 |
|  2 | lisi     |  2000 |
|  3 | wanger   |     0 |
+----+----------+-------+
3 rows in set (0.00 sec)
-- 1.回到第一个客户端执行回滚操作
mysql1> rollback;
Query OK, 0 rows affected (0.01 sec)
-- 查询当前数据
mysql1> select * from testtran;
+----+----------+-------+
| id | username | money |
+----+----------+-------+
|  1 | zhangsan |  1000 |
|  2 | lisi     |  2000 |
|  3 | wanger   |     0 |
+----+----------+-------+
3 rows in set (0.00 sec)
-- 2.回到第一个客户端执行提交操作
mysql1> commit;
Query OK, 0 rows affected (0.00 sec)
-- 回到第二个客户端即可查看到新添加的数据
mysql2> select * from testtran;
+----+----------+-------+
| id | username | money |
+----+----------+-------+
|  1 | zhangsan |  1000 |
|  2 | lisi     |  2000 |
|  3 | wanger   |     0 |
|  4 | weiwu    |     0 |
|  5 | pangtong |     0 |
+----+----------+-------+
5 rows in set (0.00 sec)
```

从以上示例中可以看出：

- 当我们开启事务后，在我们手动提交之前，数据虽然可以查看到但并不会写入数据库。
- 在我们手动提交之前，可以执行 `rollback` 操作来回滚操作，回滚到开启事务时。
- 自动提交和事务都是针对客户端的，当关闭当前连接客户端，重开一个连接，则会恢复为默认。

#### 1.3.4 Java代码测试

下面我们使用代码来进一步了解事务：我们使用jdbc连接数据库，采集数据库test表中数据条数，并分别在插入一条数据前，插入后，报错前，报错后，以及回滚前，回滚后分别打印表中条数，从中可以直观地看到结果。

```java
import java.sql.*;

/**
 * @Description: 事务测试
 * @Author: x43125
 * @Date: 2021/05/21 15:25
 */
public class TestTransaction01 {
    static PreparedStatement preparedStatement = null;
    static ResultSet resultSet = null;
    
    public static void main(String[] args) throws Exception {
        String driver = "com.mysql.cj.jdbc.Driver";
        String url = "jdbc:mysql://localhost:3306/test?serverTimeZone=Asia/Shanghai";
        String username = "root";
        String password = "root";
        Class.forName(driver);
        Connection connection =  DriverManager.getConnection(url, username, password);
        // 设置手动提交
        connection.setAutoCommit(false);
        String insertSql = "insert into testtran(username, money) values(?, ?)";
        String querySql = "select count(*) as counts from testtran";
        try {
            preparedStatement = connection.prepareStatement(querySql);
            getResult("插入前: ");
            // 插入一条数据
            preparedStatement = connection.prepareStatement(insertSql);
            preparedStatement.setString(1, "test01");
            preparedStatement.setInt(2, 0);
            preparedStatement.execute();
            preparedStatement = connection.prepareStatement(querySql);
            getResult("插入后，出错前，回滚前: ");
            // 此处引发程序出错，使得sql在提交之前出错，未提交
            System.out.println(1/0);
            connection.commit();
        } catch (Exception throwables) {
            // 查询当前表中数据，此处为rollback之前查询，此时虽然没有commit但已经可以查到插入的数据
            preparedStatement = connection.prepareStatement(querySql);
            getResult("插入后，出错后，回滚前: ");
            // 回滚
            connection.rollback();
            // 回滚后再此查询，已经无法查到刚插入的数据
            getResult("插入后，出错后，回滚后: ");
        } finally {
            resultSet.close();
            preparedStatement.close();
        }
    }

    // 打印条数查询结果
    public static void getResult(String info) throws SQLException {
        resultSet = preparedStatement.executeQuery();
        while (resultSet.next()) {
            System.out.println(info + resultSet.getInt("counts"));
        }
    }
}

------------------------------------------------------------------------------------------------------
插入前: 6
插入后，出错前，回滚前: 7
插入后，出错后，回滚前: 7
插入后，出错后，回滚后: 6

Process finished with exit code 0
```

## 2. spring事务

2.1 开启方式

2.2 隔离级别

2.2.1

对应数据库的隔离级别，spring有5种隔离级别选项：

```java
public enum Isolation {
    DEFAULT(-1),
    READ_UNCOMMITTED(1),
    READ_COMMITTED(2),
    REPEATABLE_READ(4),
    SERIALIZABLE(8);
}
```

- DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是：READ_COMMITTED。
- READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用。
- READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
- REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
- SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能，通常情况下不会用。

2.2.2 注解开启方式：使用 `isolation` 标志开启

```java
@Transactional(isolation = Isolation.DEFAULT)
```

2.3 事务传播行为

2.3.1

spring有6个传播行为选项，分别是：

```java
public enum Propagation {
    REQUIRED(0),
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);
}
```

- REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
- REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
- NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于REQUIRED。

2.3.2 注解开启方式：使用 `propagation` 标志开启

```java
@Transactional(propagation = Propagation.REQUIRED)
```

## 3. mybatis事务

 MyBatis的事务管理分为两种形式：

1. 使用**JDBC**的事务管理机制：即利用java.sql.Connection对象完成对事务的提交（commit()）、回滚（rollback()）、关闭（close()）等。
2. 使用**MANAGED**的事务管理机制：这种机制MyBatis自身不会去实现事务管理，而是让程序的容器如（JBOSS，Weblogic）来实现对事务的管理。



> 直观地讲，JdbcTransaction就是使用 java.sql.Connection 上的commit和rollback功能，JdbcTransaction只是相当于对java.sql.Connection事务处理进行了一次包装（wrapper）