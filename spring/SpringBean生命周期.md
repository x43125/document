# SpringBean生命周期

## 零、简介

核心逻辑、过程、步骤在: `AbstractApplicationContext.refresh()`中

其他的都是具体的多种实现，比如：各种读取bean方法

各种后置处理器

各种注册方法

>  最后：世界上所有的不利因素都是因为当事人能力不足导致的！

## 一、bean的总体创建过程

![img](https://img-blog.csdnimg.cn/20200224111702108.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NjM0MTgx,size_16,color_FFFFFF,t_70)

那么从上图可以看出，主要分为两个过程：1.Java类生成beanDefinition; 2.beanDefinition生成bean

**那么主要就看这两个过程中做了什么**？

## 二、Bean的具体创建过程

![img](https://pdai.tech/images/spring/springframework/spring-framework-ioc-source-102.png)



![image-20230628100211806](/Users/wangxiang/Library/Application Support/typora-user-images/image-20230628100211806.png)

![image-20230628110720210](/Users/wangxiang/Library/Application Support/typora-user-images/image-20230628110720210.png)

## 三、核心：AbstractApplicationContext#refresh

```java
public void refresh() throws BeansException, IllegalStateException {
  synchronized (this.startupShutdownMonitor) {
    StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

    // Prepare this context for refreshing.
    prepareRefresh();

    // Tell the subclass to refresh the internal bean factory.
    // 获取beanFactory，根据配置方式，读取xml或注解等，解析成document，再将document解析成BeanDefinitions
    ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

    // Prepare the bean factory for use in this context.
    // 准备factory的一些配置，如添加BeanPostProcessor;添加 忽略某些依赖接口等
    prepareBeanFactory(beanFactory);

    try {
      // Allows post-processing of the bean factory in context subclasses.
      postProcessBeanFactory(beanFactory);

      StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
      // Invoke factory processors registered as beans in the context.
      // 1. 执行BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry
      // 2. 执行BeanFactoryPostProcessors#postProcessBeanFactory
      invokeBeanFactoryPostProcessors(beanFactory);

      // Register bean processors that intercept bean creation.
      registerBeanPostProcessors(beanFactory);
      beanPostProcess.end();

      // Initialize message source for this context.
      initMessageSource();

      // Initialize event multicaster for this context.
      initApplicationEventMulticaster();

      // Initialize other special beans in specific context subclasses.
      onRefresh();

      // Check for listener beans and register them.
      registerListeners();

      // Instantiate all remaining (non-lazy-init) singletons.
      finishBeanFactoryInitialization(beanFactory);

      // Last step: publish corresponding event.
      finishRefresh();
    }

    catch (BeansException ex) {
      if (logger.isWarnEnabled()) {
        logger.warn("Exception encountered during context initialization - " +
            "cancelling refresh attempt: " + ex);
      }

      // Destroy already created singletons to avoid dangling resources.
      destroyBeans();

      // Reset 'active' flag.
      cancelRefresh(ex);

      // Propagate exception to caller.
      throw ex;
    }

    finally {
      // Reset common introspection caches in Spring's core, since we
      // might not ever need metadata for singleton beans anymore...
      resetCommonCaches();
      contextRefresh.end();
    }
  }
}
```

### 3.1 obtainFreshBeanFactory()

根据配置方式读取xml,annotation等解析成document,再解析document成BeanDefinition

最后将BeanDefinition放到缓存中: beanDefinitionMap,beanDefinitionNames,aliasMap等

### 3.2 invokeBeanFactoryPostProcessors(beanFactory)

这一步会实例化和调用BeanFactoryPostProcessor及它的子类BeanDefinitionRegistryPostProcessor

BeanDefinitionRegistryPostProcessor继承自BeanFactoryPostProcessor，有着更高的优先级

> BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry
>
> BeanFactoryPostProcessors#postProcessBeanFactory

BeanFactoryPostProcessor 接口是 Spring 初始化 BeanFactory 时对外暴露的扩展点，Spring IoC 容器允许 BeanFactoryPostProcessor 在容器实例化任何 bean 之前读取 bean 的定义，并可以修改它。





spring-bean生命周期主要分为：

- InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation
- SmartInstantiationAwareBeanPostProcessor#deteminCandidateConstructors
- **实例化: instantiation**
- MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition
- SmartInstantiationAwareBeanPostProcessor#getEarlyBeanReference
- InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation
- InstantiationAwareBeanPostProcessor#postProcessProperties
- **属性赋值: populate**
- invokeAwareMethods
- BeanPostProcessor#postProcessBeforeInitialization
- **初始化: initialization**
  - InitializingBean#afterPropertiesSet
  - bean#init
- BeanPostProcessor#postProcessAfterInitialization
- SmartInitializingSingleTon#afterSingletonsInstantiated
- **销毁: destruction**









## BeanFactory & ApplicationContext

BeanFactory:不会做的事

- 不会主动调用**BeanFactoryPostProcessor**
- 不会主动添加**BeanPostProcessor**
- 不会主动初始化单例
- 不会解析BeanFactory，也不会解析 `${}`, `#[]`

ApplicationContext





## Bean生命周期

bean本身

1. 构造函数
2. 属性注入
3. 初始化(@PostConstructor)
4. 销毁

<img src="/Users/wangxiang/Library/Application Support/typora-user-images/image-20230628173633786.png" alt="image-20230628173633786" style="zoom:50%;" />



添加BeanPostProcessor之后

<img src="/Users/wangxiang/Library/Application Support/typora-user-images/image-20230628173654022.png" alt="image-20230628173654022" style="zoom:50%;" />



BeanFactoryPostprocessor 是补充Bean的定义

BeanPostProcessor 是Bean生命周期各阶段提供扩展功能



JDK代理的过程

- 生成字节码
- 类加载器加载字节码
- 获取代理类构造器
- 构建代理类对象
- 使用代理类对象
