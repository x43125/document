# spring-source-code-quick-note
## 1 核心内容
### 1.1 IOC
> 控制反转 依赖注入

### 1.1.1 例子
```java
class UserService {
	public void service() {
		UserDao userDao = UserFactory.getUserDao();
		......
	}
}

class UserFactory {
	public static UserDao getUserDao() {
		// 解析xml文件得到类相关信息
		String classValue = getClassValueFromXml();
		class clazz = Class.forName(classValue);
		return (UserDao) clazz.newInstance();
	}

}
```

### 1.1.2 过程：
解析xml文件；
注册进工厂类中；
使用反射创建相应类对象；
### 1.1.3 本质
**IOC本质就是依赖IOC容器实现的，IOC容器底层就是对象工厂**

### 1.1.4 核心类
**BeanFactory**: Spring最顶级接口，Spring内部使用，一般开发人员不使用，懒加载
**ApplicationContext**：继承BeanFactory，更丰富，一般使用，饿汉式加载














## spring最重要的方法：

***org.springframework.context.support.AbstractApplicationContext refresh line545*** 

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

> 以do开头的方法为实际干活的，需要仔细看







spring 奥秘无穷，加油！！！