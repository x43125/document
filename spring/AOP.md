# AOP

> Aspect Oriented Programming（面向切面编程） 本质：解耦，一种设计思想

OOP面向对象编程，针对业务处理过程的实体及属性和行为进行抽象封装，从而使得各逻辑单元之间划分清晰明了；

AOP则是针对业务处理过程中的切面进行提取，他是处理过程的某个步骤或阶段，以获得逻辑过程中各部分之间的降低耦合的效果。

> 横向解决代码重复的问题：性能监视，事务管理，安全检查，缓存，关闭连接，写日志等

## 术语

- 连接点（Joinpoint）：需要在程序中插入横切关注点的扩展点，可能是：类初始化，方法执行，方法调用，字段调用，异常处理等等。（spring只支持方法执行连接点），在AOP中表现为**在哪里干**；
- 切入点（Pointcut）：选择一组相关连接点的模式，可以认为是连接点的集合。spring支持perl5和AspectJ切入点模式在AOP中表现为**在哪里干的集合**；
- 通知（Advice）：在连接点上执行的行为；包括前置通知，后置通知，环绕通知等。在Spring中通过代理模式来实现AOP，并通过拦截器模式以环绕连接点的拦截器链织入通知；在AOP中表现为**干什么**；
- 方面/切面（Aspect）：横切关注点的模块化，比如日志组件。可以认为是通知、引入和切入点的组合；在Spring中可以使用Schema和@AspectJ方式组织实现；在AOP中表现为**在哪干和干什么集合**；
- 引入（inter-type declaration）：在AOP中表现为**引入什么**
- 目标对象（Target Object）：需要被织入横切关注点的对象，在AOP中表现为**对谁干**
- 织入（Weaving）：在AOP中表现为**怎么实现的**
- AOP代理（AOP proxy）：在AOP中表现为**怎么实现的一种典型方式**



### 一个例子☝️🌰：

1. 首先我们要有一个目标类，也就是我们的日常的一些业务代码；
2. 我们要定义切面类即我们需要在业务代码中插入的逻辑，比如日志模块，监控，权限管理等；
   1. 在切面类里可以编写各种切入时机的具体增强逻辑，如：doBefore, doAround, doAfter, doThrowing, doAfterReturning；
3. 配置AOP，有两种方式，一种是XML方式，一种是基于@AspectJ注解的方式

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd
 http://www.springframework.org/schema/context
 http://www.springframework.org/schema/context/spring-context.xsd
">

    <context:component-scan base-package="com.shawn.springstudy" />

    <aop:aspectj-autoproxy/>

    <!-- 目标类 -->
    <bean id="demoService" class="com.shawn.springstudy.service.AopDemoServiceImpl">
        <!-- configure properties of bean here as normal -->
    </bean>

    <!-- 切面 -->
    <bean id="logAspect" class="com.shawn.springstudy.aspect.LogAspect">
        <!-- configure properties of aspect here as normal -->
    </bean>

    <aop:config>
        <!-- 配置切面 -->
        <aop:aspect ref="logAspect">
            <!-- 配置切入点 -->
            <aop:pointcut id="pointCutMethod" expression="execution(* com.shawn.springstudy.service.*.*(..))"/>
            <!-- 环绕通知 -->
            <aop:around method="doAround" pointcut-ref="pointCutMethod"/>
            <!-- 前置通知 -->
            <aop:before method="doBefore" pointcut-ref="pointCutMethod"/>
            <!-- 后置通知；returning属性：用于设置后置通知的第二个参数的名称，类型是Object -->
            <aop:after-returning method="doAfterReturning" pointcut-ref="pointCutMethod" returning="result"/>
            <!-- 异常通知：如果没有异常，将不会执行增强；throwing属性：用于设置通知第二个参数的的名称、类型-->
            <aop:after-throwing method="doAfterThrowing" pointcut-ref="pointCutMethod" throwing="e"/>
            <!-- 最终通知 -->
            <aop:after method="doAfter" pointcut-ref="pointCutMethod"/>
        </aop:aspect>
    </aop:config>

    <!-- more bean definitions for data access objects go here -->
</beans>
```

4. 最后添加一个测试类

```java
/**
  * main interfaces.
  *
  * @param args args
  */
public static void main(String[] args) {
    // create and configure beans
    ApplicationContext context = new ClassPathXmlApplicationContext("aspects.xml");

    // retrieve configured instance
    AopDemoServiceImpl service = context.getBean("demoService", AopDemoServiceImpl.class);

    // use configured instance
    service.doMethod1();
    service.doMethod2();
    try {
        service.doMethod3();
    } catch (Exception e) {
        // e.printStackTrace();
    }
}
```

5. 结果

```java
-----------------------
环绕通知: 进入方法
前置通知
AopDemoServiceImpl.doMethod1
环绕通知: 退出方法
最终通知
-----------------------
环绕通知: 进入方法
前置通知
AopDemoServiceImpl.doMethod2
环绕通知: 退出方法
后置通知, 返回值: hello world
最终通知
-----------------------
环绕通知: 进入方法
前置通知
AopDemoServiceImpl.doMethod3
异常通知, 异常: some exception
最终通知
```

