# Spring Running Process

## 1. spring项目启动过程

> 大的维度：

### 1.1 xml方式

#### 1.1.1 容器创建时做了什么

- 解析xml文件 （**XmlBeanDefinitionReader**），得到想要创建的对象，这个对象依赖哪些对象，是不是懒加载，是不是单例等。将这些信息放到一个**BeanDefinition**中，有多少个对象，就有多少个BeanDefinition。然后将这些BeanDefinition放到一个**Map**中。
- 创建好所有的BeanDefinition后，实例化：
    - BeanFactory：懒加载，用到的时候才实例化
    - ApplicationContext: 饿汉式，将所有非标注成懒加载的直接实例化（节约使用时候的时间）
    - 过程：根据BeanDefinition中的信息，再结合反射，即可实例化出对应对象
        - 如果Bean依赖其他的Bean，则先将被依赖Bean实例化出来，然后通过构造函数或setter方式注入到bean中
- 实例化好的单例对象将被存放到另一个**Map**中复用，非单例的不会存储，用完即扔，回收

#### 1.1.2 getBean时做了什么

- 获取bean实例，如果是单例的且未实例化则按照上面的流程实例化一个返回
- 如果已实例化则直接返回
- 如果是多例则新实例化一个返回
