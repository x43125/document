# overview

- ApplicationContext()
- @ComponentScan
- @Component
- getBean()





步骤：

现在让我们来模仿下日常开发一个项目，首先我们会新建一个spring项目，

然后我们会

- 新建一个controller或者service，给他放上@Controller\@Service等注解，相信我们稍微了解过的都知道这些注解其实除了名称之外和@Component没什么区别，只做区分，所以接下来我们统一用@Component来表示，标这个注解的目的是把这些类加入到spring的bean工厂里，由spring来帮我们管理，比如：依赖注入等。但只标注解不行，spring不知道哪些地方归他管理，所以还需要添加另一个注解@ComponentScan来开启扫描这些类，让spring知道哪些位置由他来处理，因此第一步，我们来模拟 **@ComponentScan**
- 







```java
package com.shawn.spring;

import java.beans.Introspector;
import java.io.File;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.net.URL;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author Shawn
 * @date 2024/8/25 22:29
 * @description
 */
public class SimulatorApplicationContext {

    private Class configClass;
    private ConcurrentHashMap<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>();
    private ConcurrentHashMap<String, Object> singletonObjects = new ConcurrentHashMap<>();

    public SimulatorApplicationContext(Class configClass) {
        this.configClass = configClass;

        // 扫描 --> BeanDefinition --> beanDefinitionMap
        if (configClass.isAnnotationPresent(ComponentScan.class)) {
            ComponentScan componentScanAnnotation = (ComponentScan) configClass.getAnnotation(ComponentScan.class);
            // 配置的扫描路径：com.shawn.service
            String path = componentScanAnnotation.value();

            path = path.replace(".", "/");

            ClassLoader classLoader = SimulatorApplicationContext.class.getClassLoader();
            URL resource = classLoader.getResource(path);
            File file = new File(resource.getFile());
//            System.out.println(file);
            if (file.isDirectory()) {
                File[] files = file.listFiles();
                for (File f : files) {
                    String fileName = f.getAbsolutePath();
//                    System.out.println(fileName);
                    if (fileName.endsWith(".class")) {
                        String className = fileName.substring(fileName.indexOf("com"), fileName.indexOf(".class"));
                        className = className.replace("/", ".");
                        try {
                            Class<?> clazz = classLoader.loadClass(className);
                            if (clazz.isAnnotationPresent(Component.class)) {
                                Component component = clazz.getAnnotation(Component.class);
                                String beanName = component.value();
                                if ("".equals(beanName)) {
                                    // 将大驼峰类名 转成 小驼峰对象名
                                    beanName = Introspector.decapitalize(clazz.getSimpleName());
                                }

                                BeanDefinition beanDefinition = new BeanDefinition();
                                beanDefinition.setType(clazz);
                                if (clazz.isAnnotationPresent(Scope.class)) {
                                    Scope scopeAnnotation = clazz.getAnnotation(Scope.class);
                                    beanDefinition.setScope(scopeAnnotation.value());
                                } else {
                                    beanDefinition.setScope("singleton");
                                }

                                beanDefinitionMap.put(beanName, beanDefinition);
                            }
                        } catch (ClassNotFoundException e) {
                            throw new RuntimeException(e);
                        }
                    }
                }
            }
        }

        // 懒加载 实例化单例bean
        for (Map.Entry<String, BeanDefinition> entry : beanDefinitionMap.entrySet()) {
            String beanName = entry.getKey();
            BeanDefinition beanDefinition = entry.getValue();
            if (beanDefinition.getScope().equals("singleton")) {
                Object bean = createBean(beanName, beanDefinition);
                singletonObjects.put(beanName, bean);
            }
        }
    }

    // 创建bean
    private Object createBean(String beanName, BeanDefinition beanDefinition) {
        Class clazz = beanDefinition.getType();
        try {
            // 实例化bean
            Object instance = clazz.getConstructor().newInstance();
            // 依赖注入：
            // 遍历所有属性
            for (Field field : clazz.getDeclaredFields()) {
                // 如果有autowired注解
                if (field.isAnnotationPresent(Autowired.class)) {
                    // 设置属性可访问
                    field.setAccessible(true);
                    // 为该依赖注入该属性的bean
                    field.set(instance, getBean(field.getName()));
                }
            }

            // Aware
            if (instance instanceof BeanNameAware) {
                ((BeanNameAware) instance).setBeanName(beanName);
            }

            // 初始化
            if (instance instanceof InitializingBean) {
                // 让程序员可以干预初始化
                ((InitializingBean) instance).afterPropertiesSet();
            }

            // BeanPostProcessor 初始化处理器

            return instance;
        } catch (InstantiationException | IllegalAccessException | InvocationTargetException |
                 NoSuchMethodException e) {
            throw new RuntimeException(e);
        }
    }

    // 获取bean
    public Object getBean(String beanName) {
        BeanDefinition beanDefinition = beanDefinitionMap.get(beanName);
        if (beanDefinition == null) {
            throw new RuntimeException("Bean not found: " + beanName);
        }

        String scope = beanDefinition.getScope();
        if (scope.equals("singleton")) {
            Object bean = singletonObjects.get(beanName);
            if (bean == null) {
                // 如果获取不到，则创建，初始化先后顺序导致
                Object o = createBean(beanName, beanDefinition);
                singletonObjects.put(beanName, o);
            }
            return bean;
        } else {
            return createBean(beanName, beanDefinition);
        }
    }
}

```

