# SpringBoot Initiallizer

## 目录结构

* src
    * main
        * java
        * resources
            * static：静态资源（js,css,images）
            * template：模板文件（html,freemarker,thymeleaf）
            * application.properties：默认设置
    * test

## 配置文件

使用全局配置文件，配置文件名固定

* application.properties

* application.yaml

### YAML语法

#### 1.基本语法

k:(空格)v		表示一堆键值对（**空格**必须有）

以**空格**的缩进来控制层级关系，只要是左对齐的一列数据，都是同一个层级

```yaml
server: 
    port: 8081
    path: /hello
```

属性和值大小写敏感

#### 2.值的写法

* 字面量：普通的值（数字、字符串、布尔)

    k: v 	: 字面量直接写

    ​	字符串默认不需要加引号

    ​	双引号其内字符串有转义功能："hello \n world" => hello

    ​																							world

    ​	单引号期内字符串无转移功能：'hello \n world' => hello \n world

* 对象：map（属性、值）

    k: v	：在下一行直接写对象的属性和值的关系

    ​	对象依然是k: v关系

    ```yaml
    friends:
    	name: zhangsan
    	age: 20
    ```

    行内写法：

    ```yaml
    friends: {name: zhangsan,age: 20}
    ```

* 数组（list、set）

    用-值表示数组中的一个元素

    ```yaml
    pets:
     - dog
     - cat
     - horse
    ```

    行内写法

    ```yaml
    pets: [dog,cat,horse]
    ```




## 示例

> 可复制更改

```yaml
server:
  port: 8081

spring:
  application:
    name: shiro
  datasource:
    url: jdbc:mysql://47.98.59.193:3306/selfblog?serverTimeZone=Asia/Shanghai&CharacterEncoding=UTF-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    username: ppdream
    password: 123ABCdef*
  redis:
    host: 47.98.59.193 # Redis服务器地址
    port: 6379 # Redis服务器连接端口
    password: 123ABCdef* # Redis服务器连接密码（默认为空）
    database: 0 # Redis数据库索引（默认为0）
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数（使用负值表示没有限制）
        max-wait: -1ms # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-idle: 8 # 连接池中的最大空闲连接
        min-idle: 0 # 连接池中的最小空闲连接
    timeout: 3000ms # 连接超时时间（毫秒）

mybatis:
  type-aliases-package: com.wx.selfblog.entity
  mapper-locations: classpath:com/**/mapper/*.xml

saltNo: 8
hashNo: 1024

logging:
  level:
    #    root: debug
    com.wx.selfblog.dao: warn

# 自定义jwt key
jwt:
  tokenHeader: Authorization # JWT存储的请求头
  secret: mySecret # JWT加解密使用的密钥
  expiration: 604800 # JWT的超期限时间(60*60*24)
  tokenHead: Bearer # JWT负载中拿到开头

# 上传文件存储位置
upload:
  saveDir: F:/blogSaveDir # 上传的文件存储位置
```



