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

#### 2.值得写法

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

    

    

