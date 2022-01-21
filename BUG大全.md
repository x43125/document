# BUG鉴赏大会

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



2. idea： cannot resolve method

检查import的库是否正确，多数为此错误



## shell

使用`.`来执行脚本的时候，如果脚本出错会将会话直接关闭，使用`bash`或`sh`来执行则不会

## mybatis-plus
分页的时候，如果是join需要在最外层包一层，否则只会用主表数来返回total
```mysql
select * from (
    join语句
) as temp
```
