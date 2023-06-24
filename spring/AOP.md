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



### 切入点：execution规则

我们常用execution来声明切入点

```java
execution (modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
```

其中：

- ret-type-pattern(返回类型模式), name-pattern(名字模式), param-pattern(参数模式) **必填，其他选填**。

- modifiers-pattern(修饰符模式):  private, protected, public等
- ret-type-pattern(返回类型模式): 一般都用 ‘*’
- declaring-type-pattern(全限定类型模式): 一个全限定的类型名将只会匹配返回给定类型的方法
- name-pattern(名字模式): 匹配方法名，可以使用 ‘*’ 来表示所有者或部分(可以用：set\* 来表示所有以set开头的方法)
- param -pattern(参数模式)：
  - `()` 匹配一个不接受任何参数的方法 
  - `(..)`匹配一个接受任意数量参数的方法（可以为0）
  - `(*)`匹配一个接受一个任何类型的参数的方法
  - `(*, String)`匹配一个接受两个参数的方法，第一个参数可以是任意类型，第二个参数必须是`String`
  - 以此类推，等等

更多匹配规则如下：

```java
// 任意公共方法的执行：
execution（public * *（..））

// 任何一个名字以“set”开始的方法的执行：
execution（* set*（..））

// AccountService接口定义的任意方法的执行：
execution（* com.xyz.service.AccountService.*（..））

// 在service包中定义的任意方法的执行：
execution（* com.xyz.service.*.*（..））

// 在service包或其子包中定义的任意方法的执行：
execution（* com.xyz.service..*.*（..））

// 在service包中的任意连接点（在Spring AOP中只是方法执行）：
within（com.xyz.service.*）

// 在service包或其子包中的任意连接点（在Spring AOP中只是方法执行）：
within（com.xyz.service..*）

// 实现了AccountService接口的代理对象的任意连接点 （在Spring AOP中只是方法执行）：
this（com.xyz.service.AccountService）// 'this'在绑定表单中更加常用

// 实现AccountService接口的目标对象的任意连接点 （在Spring AOP中只是方法执行）：
target（com.xyz.service.AccountService） // 'target'在绑定表单中更加常用

// 任何一个只接受一个参数，并且运行时所传入的参数是Serializable 接口的连接点（在Spring AOP中只是方法执行）
args（java.io.Serializable） // 'args'在绑定表单中更加常用; 请注意在例子中给出的切入点不同于 execution(* *(java.io.Serializable))： args版本只有在动态运行时候传入参数是Serializable时才匹配，而execution版本在方法签名中声明只有一个 Serializable类型的参数时候匹配。

// 目标对象中有一个 @Transactional 注解的任意连接点 （在Spring AOP中只是方法执行）
@target（org.springframework.transaction.annotation.Transactional）// '@target'在绑定表单中更加常用

// 任何一个目标对象声明的类型有一个 @Transactional 注解的连接点 （在Spring AOP中只是方法执行）：
@within（org.springframework.transaction.annotation.Transactional） // '@within'在绑定表单中更加常用

// 任何一个执行的方法有一个 @Transactional 注解的连接点 （在Spring AOP中只是方法执行）
@annotation（org.springframework.transaction.annotation.Transactional） // '@annotation'在绑定表单中更加常用

// 任何一个只接受一个参数，并且运行时所传入的参数类型具有@Classified 注解的连接点（在Spring AOP中只是方法执行）
@args（com.xyz.security.Classified） // '@args'在绑定表单中更加常用

// 任何一个在名为'tradeService'的Spring bean之上的连接点 （在Spring AOP中只是方法执行）
bean（tradeService）

// 任何一个在名字匹配通配符表达式'*Service'的Spring bean之上的连接点 （在Spring AOP中只是方法执行）
bean（*Service）
```

另外spring还支持以下三个逻辑运算符来组合切入点表达式：

```java
&&：要求连接点同时匹配两个切入点表达式
||：要求连接点匹配任意个切入点表达式
!:：要求连接点不匹配指定的切入点表达式
```

### 当有多种增强通知的时候

最高优先级会先通知：

doBefore：优先级高的先执行

doAfter: 优先级高的后执行

可以通过指定优先级来控制执行顺序：通过实现 ordered接口或者用order注解来指定优先级

