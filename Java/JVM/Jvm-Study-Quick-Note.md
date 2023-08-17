# Jvm Study Quick Note

## 1 内存区域

![image-20220319140415769](resources/image-20220319140415769.png)

## 1.1程序计数器

> Program Counter Register

  程序计数器是**线程私有**的；用于指示当前线程运行到哪个位置，1. 用于在各种循环、if等判断分支中指示接下来的运行语句；2. 用于在多线程切换后指定接着运行的线程的开始运行位置。

  程序计数器是唯一一个不会发生**OOM**的区域。

### 1.2 虚拟机栈

> Java Virtual Machine Stack

  线程私有；Java**方法执行**的线程内存模型；每个方法被执行的时候，JVM都会创建一个**栈帧(Stack Frame)**;用于存储方法执行的**局部变量表、操作数栈、动态连接、方法出口**等信息；每个栈帧在虚拟机栈中从**入栈到出栈**的过程即对应一个方法被**调用到执行**完毕的过程。

#### 1.2.1 局部变量表

1. 存放

    - 编译期可知的各种Java虚拟机基本数据类型（boolean, byte, char, short, int, float, long, double）

    - returnAddress类型（指向了一条字节码指令的地址）

    - 对象引用（不等同于对象本身，可能是一个指向对象起始地址的引用地址；或是指向一个代表对象的句柄；再或是其他与此对象相关的位置）

2. 局部变量槽(Slot)：long和double (64位长度)的数据占用两个slot，其他的占用1个

3. 局部变量表所需的内存空间在编译期间完成分配（当进入一个方法时，这个方法对应栈帧在局部变量空间中分配多少内存是 **完全确定**的）

4. 异常：

    1. 当线程请求的栈深度大于虚拟机允许的深度：**StackOverflowError异常**；（递归调用未正常退出）
    2. 如果JVM虚拟机栈容量可以动态扩展，当扩展到无法申请到足够的内存时：**OutOfMemoryError异常**；(HotSport虚拟机的栈容量不可以动态扩展)

#### 1.2.2 操作数栈

#### 1.2.3 动态连接

#### 1.2.4 方法出口

### 1.3 本地方法栈

> Native Method Stacks

和虚拟机栈相似，只是运行的为本地（Native）方法服务；

HotSpot 虚拟机将 虚拟机栈和本地方法栈 **合二为一** 了

异常：StackOverflowError异常；OutOfMemoryError异常

### 1.4 堆

> Heap

内存中最大的一块







## 案例：

#### CPU占用过高的排查方式：

1. 使用top命令，然后 shift+h 来按照CPU使用率排序，可以得到占用最高的进程信息
2. 从进程信息可以获得进程id, PID
3. 然后使用ps命令获取该进程下的所有线程及线程的CPU占用：`ps H -eo pid,tid,%cpu | grep PID` （不能有多余空格）
4. 从列表中找出CPU占用比较高的线程，然后使用jstack命令查看具体的线程执行信息：jstack pid
5. jstack会打印该进程下的所有线程的执行信息，但其线程会以16进制形式展示，所以需要把第4部得到的线程ID做个转换，在信息中检索，就能找到该线程的具体信息，从而可以知道当前该线程运行的代码位置，一般也就是该位置出现的问题。



#### 迟迟得不到结果：死锁

- 使用jstack pid的方式打印进程运行信息
- 来到进程末尾查看是否有死锁相关信息，如果有会打印出来，deadlock字样，并且会给出哪些线程导致死锁，并且会给出每个线程持有的锁和想要获得的锁，及相关代码行数，根据代码信息，我们就可以去代码中做具体排查了



#### 多次垃圾回收之后，内存占用依然很高

- 首先可以使用jmap在一些特殊时刻打印进程信息，根据打印的内容进行分析，内容一般为各区的内存占用（eden, old, full等）
- 其次可以使用 jconsole 查看堆内存变化的情况，支持GC
- 最后还可以使用 jvisualvm 将当前堆信息做一个堆快照，dump下来，然后对结果进行分析，比如可以使用右侧的查找功能，比如查找前20个堆内存占用最大的对象。支持GC

#### 其他用例可自行分析

- 网站流量暴增后，网站页面反应很慢
- 后台导出数据引发OOM - 偶发



方法区的描述

![image-20230728220748283](/Users/wangxiang/Library/Application Support/typora-user-images/image-20230728220748283.png)





jvm参数

- -Xss 设置每个线程的栈大小: -Xss128k
- -Xmx 设置堆的最大内存
- -Xms 设置堆的起始大小
- -Xmn 设置堆新生代大小
- -XX:MaxMetaspaceSize 设置元空间大小：-XX:MaxMetaspaceSize=8m
- -XX:MaxPermSize 设置永久代大小：-XX:MaxPermSize=8m
- -XX:+PrintStringTableStatistics 打印串池信息
- -XX:+PrintGCDetails -verbose:gc 打印垃圾回收的详细信息
- -XX:StringTableSize=桶个数 字符串池是一个hashmap结构，此参数，确定hashmap的桶个数也就是数组个数
- -XX:+DisableExplicitGC 禁用显示的垃圾回收 （System.gc()）

![image-20230729112202571](/Users/wangxiang/Library/Application Support/typora-user-images/image-20230729112202571.png)

- -XX:+UseSerialGC = Serial+SerialOld
- -XX:+UseParrallelGC ~ -XX:+UseParrallelOldGC
  - -XX:+UseAdaptiveSizepolicy
  - -XX:GCTimeRatio=ratio
  - -XX:MaxGCPauseMillis=ms
  - -XX:ParrallelGCThreads=n
- -XX:+UseConcMarkSweepGC ~ -XX:+UseParNewGC ~ SerialOld



类二进制字节码：类基本信息+常量池+类方法定义+虚拟机指令

javap -v HelloWorld.class // 反编译字节码文件为人可查看的文件内容

常量池：就是一张常量表存储在字节码文件中：虚拟机指令根据这张表找到要执行的类名，方法名，参数类型，字面量等信息

当类被加载进虚拟机后，这个常量池信息就会被放入运行时常量池中，并且把里面的符号地址转成实际地址。



如果代码中有大量重复字符串的使用的话，可以考虑将这些字符串入池，会极大的节约字符串对堆内存的占用。（享元模式）

也就是调用下string.intern()方法



直接内存：Direct Memory 主要用在NIO，用于数据缓冲区，读写效率很高，不受JVM垃圾回收直接回收；但可以通过Unsafe类调用 freeMemory方法来释放，具体实现的时候是在JVM垃圾回收的时候，调用回调方法，回调方法里调用了Unsafe.freeMemory()方法实现。



- 强引用
- 软引用：当没有强引用，且内存不足的时候发生的gc的时候，该引用会被回收；可以配合引用队列使用去剔除已被回收的引用
- 弱引用：当没有强引用，且发生gc的时候，该引用会被回收，但会分区回收，比如只发生了minor gc的时候，只会回收eden区的
- 虚引用



垃圾回收算法：

- 标记-清除
- 标记-整理
- 复制

堆内存结构

新生代：

- eden
- from
- to

老年代：



占用-回收过程：

- 首先小对象放在eden区
- 然后当eden区内存不够的时候，发生一次minor gc，此时将eden区和from survivor区的存活对象放到to survivor区，并将此次存活的对象的年龄增加1，如果某对象的存活年龄超过阈值（此阈值最大为15，默认和JVM有关）则会被放到老年代，而如果此时老年代空间不足，则会触发fullgc，回收老年代内存；然后将eden区和from survivor区的死亡对象回收掉；最后将from survivor区和to survivor区的引用互换。
- 如果是创建了一个大对象，则会直接被放到老年代，如果老年代空间不足，则会触发full gc





#### GC举例

初始堆信息

![image-20230729112700628](/Users/wangxiang/Library/Application Support/typora-user-images/image-20230729112700628.png)





调优

1. GC选择
2. 最快的GC是不发生GC
   1. 数据是不是太多，比如select一张大表，然后放到内存里
   2. 数据表示是不是太臃肿
      1. 对象图
      2. 对象大小
   3. 是否存在内存泄漏
      1. static Map map = （可以使用软、弱引用，或是使用第三方缓存比如redis）
3. 新生代调优
   1. 新生代占总堆内存最好在25%～50%之间
   2. 但因为新生代多数都是朝生夕死的段生命周期的对象，所以更多的对象是标记而非复制，单次复制占用的时间要比单次标记的时间多的多，所以即使把新生代设置大一些，也没那么大的问题，因为多数操作都是标记，复制的很少
   3. 最佳实践：eden内存大小 = 并发数 *（请求数 - 响应数）







i++和++i的区别：





![image-20230729194637132](/Users/wangxiang/Library/Application Support/typora-user-images/image-20230729194637132.png)



类加载过程

- 加载二进制字节码文件到方法区
- 连接
  - 验证：验证二进制字节码文件正确性
  - 准备：分配空间，设置默认值
  - 解析：将常量池中的符号引用解析为直接饮用
- 初始化：赋实际值；调用\<cinit>方法

> 如果是final的，则会直接在准备阶段分配空间➕赋值；除了final一个对象：static final Object o = new Object();





什么时候需要使用自定义类加载器？

- 当我们想要加载一些不在原三类加载器可加载目录下的类的时候
- 框架设计的时候，通过接口来使用不同的实现，实现解耦合
- 当有多个相同类但内容不同又想同时工作的时候（Tomcat）
