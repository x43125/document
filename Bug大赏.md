# BUG鉴赏大会

## 一、docker

> ERROR: network docker_default id 49044fc8722dfa95f075... **has active endpoints** 

原因：先更新了docker-compose.yml文件，后进行的docker-compose down操作



1. centos7直接使用指令会报错，

   ```bash
   # python
   Could not find platform independent libraries <prefix>
   Could not find platform dependent libraries <exec_prefix>
   Consider setting $PYTHONHOME to <prefix>[:<exec_prefix>]
   ImportError: No module named site
   ```

   解决办法:

   * 安装了多个版本的python，但只有其中一个有yum，在设置PYTHONHOME和PYTHONPATH时设置错误
   * 使用find / -type f -executable -name 'python2*'，查看所有的python版本
     * 1. 设置PYTHONPATH：export PYTHONPATH=$:/usr/lib64/python2.7       设置PYTHONHOME为其父目录: export PYTHONHOME=$:/usr/lib64
       2. 报错no module named time
     * 1. 设置PYTHONPATH：export PYTHONPATH=$:/usr/lib/python2.7       设置PYTHONHOME为其父目录: export PYTHONHOME=$:/usr/lib
       2. 报错no module named site

***未解决***





## 二、shell

使用`.`来执行脚本的时候，如果脚本出错会将会话直接关闭，使用`bash`或`sh`来执行则不会



## 三、mybatis-plus

分页的时候，如果是join需要在最外层包一层，否则只会用主表数来返回total

```mysql
select * from (
    join语句
) as temp
```



## 四、maven

### 在powershell中使用的时候报错

在powershell中直接輸入命令：mvnd clean package -Dmaven.test.skip=true

报错：`Unknown lifecycle phase ".test.skip=true"`

**解决方法：** 在参数外面添加 英文单引号

```powershell
mvnd clean package '-Dmaven.test.skip=true'
```





父项目里使用dependencyManagement管理子模块版本，且用properties规定了版本号，那子模块中涉及到的一些依赖的版本需要使用版本号变量

## 五、spring

### 5.1 循环依赖

Has bean injected other bean ；；；； circular reference

找到循环依赖的两个类，并找到相应的注入位置，然后将一个注入加上 `@Lazy` 注解

## 六、idea

### cannot resolve method

**解决方法：**检查import的库是否正确，多数为此错误



## 七、Java

如果 `Javafx`引不进来，有可能是jdk是openjdk的问题。

