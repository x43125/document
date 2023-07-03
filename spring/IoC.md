# IoC

> 世界上所有的不利因素，都是因为当事人能力不足造成的。

IoC应该包含哪些部分：

- 加载Bean的配置
- 根据Bean的定义加载生成Bean的实例，并放置在Bean容器中：Bean的依赖注入、嵌套、存放（缓存）等
- 除了基础Bean外，还有常规针对企业级业务的特别Bean：国际化，事件Event
- 对容器中的Bean提供统一的管理和调用：比如工厂模式管理，提供方法根据名字/类的类型等从容器中获取Bean



![img](https://pdai.tech/images/spring/springframework/spring-framework-ioc-source-61.png)





## 脑内风暴

### 什么是IoC？

控制反转，其实简单说就是原本我们在调用某个方法的时候，我们需要首先实例化这个方法的类，如果这个类有依赖其他的类还需要先再实例化依赖的类，然后将依赖类手动注入到这个类里。最后再调用我们需要的方法，实现功能。而如果使用IoC来完成这些功能的话，这个过程就会变成，我们只需要从IoC容器中获取相应的类的实例，然后调用实例的方法即可，此实例会自动创建，自动注入依赖。

### 那么在这个过程中就会涉及到几个问题：

1. 哪些Beans需要加进IoC容器中
2. 怎么加进IoC容器中
3. 依赖关系怎么表示
4. 怎么注入依赖
5. 如何使用IoC容器中的实例

### 依次解答：

#### 1、创建

包括创建、依赖注入、注册进容器中

BeanFactory: 工厂模式定义了IoC容器的基本功能规范

BeanRegistry: 向IoC容器手动注册 **BeanDefinition** 对象的方法

##### 1.1 BeanFactory

bean工厂，即：IoC容器

```java
public interface BeanFactory {    
      
    //用于取消引用实例并将其与FactoryBean创建的bean区分开来。例如，如果命名的bean是FactoryBean，则获取将返回Factory，而不是Factory返回的实例。
    String FACTORY_BEAN_PREFIX = "&"; 
        
    //根据bean的名字和Class类型等来得到bean实例    
    Object getBean(String name) throws BeansException;    
    Object getBean(String name, Class requiredType) throws BeansException;    
    Object getBean(String name, Object... args) throws BeansException;
    <T> T getBean(Class<T> requiredType) throws BeansException;
    <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

    //返回指定bean的Provider
    <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
    <T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);

    //检查工厂中是否包含给定name的bean，或者外部注册的bean
    boolean containsBean(String name);

    //检查所给定name的bean是否为单例/原型
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

    //判断所给name的类型与type是否匹配
    boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
    boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

    //获取给定name的bean的类型
    @Nullable
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;

    //返回给定name的bean的别名
    String[] getAliases(String name);
     
}
```

其中主要就是各种方式的`getBean()`

而BeanFactory的主要接口包括以下几个：

- ListableBeanFactory: 主要定义了访问容器中Bean基本信息的若干方法，如查看Bean个数、获取某一类型Bean的配置名、查看容器中是否包括某一Bean等方法；
- HierarchicalBeanFactory: 父子级联IoC容器的接口，子容器通过接口方法访问父容器，子容器可以访问父容器中的Bean，但父容器不能访问子容器中的Bean；
- ConfigurableBeanFactory: 重要接口，增强了IoC容器的可定制性，她定义了设置类装载器、属性编辑器、容器初始化后置处理器等方法；
- **AutowireCapableBeanFactory**: 定义了将容器中的Bean按某种规则（按名字匹配，类型匹配等）进行自动装配的方法；
- ConfigurableListableBeanFactory: ListableBeanFactory、ConfigurableBeanFactory和AutowireCapableBeanFactory的融合

##### 1.2 BeanRegistry

> 如何将Bean注册进BeanFactory中？

每个Bean在Spring容器中都通过一个BeanDefinition对象来表示，它描述了Bean的配置信息。

BeanDefinition通过**BeanDefinitionRegistry**接口手动注册进容器中



在讲BeanDefinitionRegistry之前要先讲一下BeanDefinition

BeanDefinition: 

Bean对象有依赖嵌套关系，所以设计者设计了一个包装类，BeanDefinition 来描述Bean对象及关系定义

- BeanDefinition: 各种Bean对象及其相互的关系；
- BeanDefinitionReader: BeanDefinition的解析器；
- BeanDefinitionHolder: BeanDefinition的包装类，用来存储BeanDefinition，name及aliases等

##### 1.5 ApplicationContext: IoC接口设计与实现

ApplicationContext是IoC容器的接口类

1. 既然他是IoC容器的接口类，那必然继承了BeanFactory
2. 从名字上看它表示应用上下文，那么除了Bean管理之外，还应该包括
   1. 访问资源：对不同方式的Bean配置进行加载
   2. 国际化：支持信息源，可以实现国际化
   3. 应用事件：支持应用事件

### 再细致一点就是Spring如何加载、解析、生成BeanDefinition并注册到IoC容器中

code：

```java
public ClassPathXmlApplicationContext(String... configLocations) throws BeansException {
		this(configLocations, true, null);
}

public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent) throws BeansException {
	  // 设置Bean资源加载器
    super(parent);
    // 设置配置路径
    setConfigLocations(configLocations);
    // 初始化容器
    if (refresh) {
        // 模版方法：
        // 创建IoC容器前，如果已经有容器存在，则把已有的容器销毁和关闭，以保证在refresh之后使用的是新建立的IoC容器
        // 类似于对IoC容器的重启，在新建立好的容器中对容器进行初始化，载入Bean定义资源 
        refresh();
    }
}
```

**refresh**主体，（非常重要）

```java
@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				// Invoke factory processors registered as beans in the context.
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



> 资源类加载处理型思路

一样：关注顶层设计，而非具体的某一处处理

- 模版方法设计模式：钩子方法
- 将具体的初始化加载方法插入到钩子方法之间
- 将初始化的阶段封装，用来记录当前初始化到什么阶段
- 资源加载初始化有失败等处理（try/catch/finally等）

具体：

![img](https://pdai.tech/images/spring/springframework/spring-framework-ioc-source-8.png)



初始化BeanFactory：obtainFreshBeanFactory

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
  // 委派设计模式：父类定义 refreshBeanFactory() 抽象方法，具体使用的时候调用子类的实现
  refreshBeanFactory();
  return getBeanFactory();
}
```



```java
protected final void refreshBeanFactory() throws BeansException {
  // 如果有则销毁beans，关闭BeanFactory
  if (hasBeanFactory()) {
    destroyBeans();
    closeBeanFactory();
  }
  try {
    // 创建DefaultListableBeanFactory，并调用
    DefaultListableBeanFactory beanFactory = createBeanFactory();
    beanFactory.setSerializationId(getId());
    // 定制化IoC容器，如设置启动参数，开启注解的自动装配等
    customizeBeanFactory(beanFactory);
    // 调用载入Bean定义的方法，有一个委派模式，当前类只定义了一个抽象方法，具体使用子类的实现
    loadBeanDefinitions(beanFactory);
    this.beanFactory = beanFactory;
  }
  catch (IOException ex) {
    throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
  }
}
```



读取xml -> Resource -> 解析成Document对象 -> 再将Document对象解析成BeanDefinition对象 -> 注册到容器中

```java
// 先解析成Doc
Document doc = XmlBeanDefinitionReader.doLoadDocument(inputSource, resource);
// 再解析成beanDefinitionHolder
BeanDefinitionHolder bdHolder = BeanDefinitionParserDelegate.parseBeanDefinitionElement(Element ele);
// 最后将beanDefinitionHolder注册进容器中
BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
```













