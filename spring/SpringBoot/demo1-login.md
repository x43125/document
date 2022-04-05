# SpringBoot Demo 01

## 用户登录程序

### 1.配置文件

1.1 application.properties

```properties
# 数据库相关配置
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://master02:5432/bdp
spring.datasource.username=x43125
spring.datasource.password=456123

# mybatis路径
mybatis.mapper-locations=classpath:mapper/*.xml
```

1.2 mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace：项目mapper路径 -->
<mapper namespace="com.example.springbootdemo02.mapper.StudentMapper">
    <!--<resultMap id="result" type="com.example.sprintbootdemo02.entity.Student" >
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="password" property="password"/>
    </resultMap>-->
    <!--<resultMap id="getPassWordByName" type="com.study.sprintbootdemo01.entity.dto.NameDTO">
        <result column="name" property="name"/>
        <result column="password" property="password"/>
    </resultMap>-->
    <!--<select id="getStudent" resultMap="result">
        select * from student;
    </select>-->
    <!-- id:mapper对应的方法名 -->
    <select id="getStudentByNameAndPassword" resultType="Integer" >
        select count(id) from student where name=#{name} and password=#{password};
    </select>

</mapper>
```

### 2.程序目录结构

![](D:\WorkSpace\document\SpringBoot\resources\springboot项目结构.png)

### 3.注解

@Resource

@RestController

@Service

@RequestMapping

@Param

@RequestBody

@Mapper

### postman

![](D:\WorkSpace\document\SpringBoot\resources\postman1.png)