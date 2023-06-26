# Design Pattern of Spring



1. 适配器模式: DispatcherServlet -> HandlerAdapter 
2. 工厂模式: SpringBean创建 
3. 模版方法模式: AbstractApplicationContext#refresh
4. 委派模式
   1. AbstractApplicationContext#obtainFreshBeanFactory
   2. AbstractRefreshableApplicationContext#loadBeanDefinitions