# 多线程基础

## 一、线程的创建

- Thread
- Runnable
- Callable
- 线程池

## 二、常用方法

### 1、sleep

```java
Thread.sleep(ms);
```

线程休眠指定毫秒；会让出CPU、但不会释放锁

#### 状态变更

开始休眠：RUNNABLE -> TIME_WAITING

休眠结束：TIME_WAITING -> RUNNABLE

#### 可以被打断

可以使用 `threadName.intterupt` 方法打断

会抛一个异常 `InterruptedException`

### 2、yield

```java
Thread.yield();
```

调用yield让出CPU，重新进入CPU竞争

#### 状态变更

Running -> Runnable

### 3、join

```java
threadName.join();
threadName.join(ms);
```

调用指定线程的join方法，来阻塞当前线程，直到指定线程执行完成。

可以传一个时间参数，来取消等待：等待了一段时间后指定线程还是没有完成，那就不等了。

### 4、interrupt

```java
threadName.interrupt();
```

打断指定线程，可以打断两种状态的线程：阻塞 & 正在运行

阻塞：sleep, wait, join 的线程

##### 注：

- 此打断只是添加了一个打断标记
- 可以使用 `isInterrupted(), interrupted()`两个方法来查看当前线程的打断状态
- 其中 `interrupted()` 在调用之后会清除打断标记，`isInterrupted()`不会
- 打断`park()` 线程之后，如果线程再调用`park()`，就会失效：即打断标记为真的时候，park会失效



不推荐使用：

stop()

suspend()

resume()

### 5、wait & notify & notifyAll

wait(): 释放CPU、释放锁，进入等待队列，等待被唤醒

notify(): 唤醒某一个wait()的线程

notifyAll(): 唤醒所有wait()的线程，进入竞争锁资源，选出一个

> 这几个想要使用的前提是：已经获得锁了，才能使用

e.g.:

```java
new Thread(() -> {
  synchronized(this) {
    this.wait();
  }
}).start();
  
new Thread(() -> {
  synchronized(this) {
    this.notifyAll();
  }
}).start();
```

### 6、park & unpark

park也会阻塞线程，当调用了某线程的unpark又会解锁该线程，但他的实现逻辑不同：先调unpark再调park,也可以释放阻塞；如果先notify 再wait则不会释放线程



### 守护线程

只要有线程在运行的时候，Java进程就不会结束。但不包括守护线程，当进程中其他的线程都运行完，只有守护线程还在运行的时候，进程会结束。

## 线程状态

NEW：新建

RUNNABLE：可运行，运行，阻塞（操作系统层面的阻塞：比如IO操作等）

BLOCKED：获得锁失败

WAITING：永远阻塞，比如 join等待其他线程执行完

TIMED_WAITING：有时间的等待，比如 Thread.sleep(1000)

TERMINATED：线程执行完之后的状态

## 锁

### synchronized

#### 定义

> 同步锁：加在对象上的锁，锁具体的**一个对象**，当多个线程需要获取这**同一个对象的锁**的时候，会发生占用问题。

#### 作用范围

加在普通对象上：锁该对象

加在普通方法上：相当于锁 this

加在静态对象上：锁 该类对象

加在静态方法上：锁 类对象

e.g.：当一个线程对一个对象加锁，一个线程不加锁直接访问，那么不会发生同步，即：有可能发生并发问题

#### 锁升级

##### 对象头：

Java对象的字节码包括：对象头 + 成员变量

对象头又包括：Mark Word + Kclass Word

Mark Word：使用64位的信息来存储：对象信息和加锁状态 以及占位填充

以下为不同加锁状态下的Mark Word的字节码位数的具体对应信息

| Makr Word(64bits)             |             |          |       |               |      | State              |
| ----------------------------- | ----------- | -------- | ----- | ------------- | ---- | ------------------ |
| unused:25                     | hashcode:31 | unused:1 | age:4 | biased_lock:0 | 01   | Normal             |
| thread:54                     | epoch:2     | unused:1 | age:4 | biased_lock:1 | 01   | Biased             |
| ptr_to_lock_record:62         |             |          |       |               | 00   | LightWeight Locked |
| ptr_to_heavyweight_monitor:62 |             |          |       |               | 10   | HeavyWeight Locked |
|                               |             |          |       |               | 11   | Marked For GC      |

锁升级：synchronized 包含 三种锁阶段：偏向锁；轻量级锁；重量级锁

默认为偏向锁；

> 但偏向锁需要在主任务启动几秒后才会生效，所以一开始就打印一个对象的对象头的话，是不会显示有偏向锁的，对象头会是 ：000……001
>
> 但如果等了几秒钟之后再创建对象，则会展示为：000……101 即是偏向锁状态

当同一个多线程多次加锁，没有产生其他线程占用锁的时候，则一直停留在 偏向锁状态

当有其他线程试图加锁，如果多个线程之间对锁的占用时间刚好可以错开，则偏向锁会升级成轻量级锁，锁的状态为：……000

> 其中前半部分的字节码则会改为轻量级锁指向的线程的栈帧记录；而原先栈帧中的lock record中则会记录该对象的对象头信息，等待释放锁的时候，再拷贝回去

当有多个线程试图加锁，并且时间上未错开，产生了冲突，则会原地自旋多次，

如果自旋之后获得锁了，则依然是轻量级锁，如果自旋之后仍未获得锁，则会继续升级成重量级锁，锁的状态为：……010

> 其中前半部分的字节码会升级成指向Monitor的地址，Monitor中的owner则会指向加锁的线程栈帧

一个小知识：

> 初始创建的对象的对象头会是包含偏向锁的信息，但如果此时调用了一次hashcode，则会消除偏向锁信息，因为对象头64位，其中的54位要用来存偏向锁的线程信息，就放不下hashcode了，所以放了hashcode就无法再放偏向锁信息了，之所轻量级锁、重量级锁可以hashcode，是因为他们的对象信息，有其他地方可以存。

#### Monitor:

有几个东西组成：WaitSet, EntryList, Owner

## 三、并发出现问题的因素

### 1、可见性：CPU缓存引起

> 一个线程对共享变量的修改，另外一个线程能够立刻看到。

```java
// 线程1执行的代码
int i = 0;
i = 10;
// 线程2执行的代码
int j = i;
```

### 2、原子性：分时复用技术

> 一串操作，要么全部执行，要么全部失败

```java
int i = 1;
i++;
```

i++ 这部操作：隐含3个操作

- 将变量i从内存读取到CPU寄存器
- 在寄存器中执行i+1操作
- 将i的新值写回到内存

### 3、有序性：重排序引起

> 程序的执行顺序按照代码的先后顺序执行

```java
int i = 0;
boolean flag = false;
i = 1;
flag = true;
```

在执行程序时为了提高性能，编译器和处理器会对指令做重排序，重排序分三种：

- 编译器优化的重排序：编译器在不改变单线程程序语义的前提下，可以重新排序安排语句的执行顺序
- 指令级并行重排序：处理器采用了指令级并行技术将多条指令重叠执行，如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
- 内存系统的重排序：由于处理器使用缓存和 读/写 缓冲区，使得加载和存储操作看上去是乱序执行的。

## 四、Java是怎么解决并发问题的

> JMM（Java内存模型）

### 1、理解一：

JMM本质上可以理解为：Java内存模型规范了JVM如何提供按需**禁用**缓存和编译优化的方法。主要包括：

- volatile、synchronized 和 final 三个关键字
- Happens-Before 规则

### 2、理解二：

可见性、原子性、有序性

原子性：JMM只保证了基本读取和赋值操作是原子性操作，如果其他的大范围操作，可以使用synchronized和lock来实现

可见性：Java提供了volatile来保证可见性。当一个变量被volatile修饰时，它会保证修改的值会立即被更新到内存中，当其他线程需要读取时，会去内存中读取新值。（synchronized和lock也可以保证可见性）

有序性：可以通过volatile来保证一定的“有序性”，（synchronized和lock也可以保证有序性）JMM是通过Happens-Before规则来保证有序性的。

## 五、Happens-Before

> 规定了对共享变量的写操作对其他线程的读操作可见

### 1、单一线程原则 （Single Thread Rule）

在一个线程内，在程序前面的操作先行发生于后面的操作

### 2、管程锁定规则（Monitor Lock Rule）

一个unlock操作先发生于后面对同一个锁的lock操作

### 3、volatile变量规则（Volatile Variable Rule）

对一个volatile变量的写操作先行发生于后面对这个变量的读操作

### 4、线程启动规则（Thread Start Rule）

Thread对象的start方法调用先行发生于此线程的每一个动作

### 5、线程加入规则（Thread Join Rule）

Thread对象的结束先行发生于join方法返回

### 6、线程中断规则（Thread Interrupt Rule）

对线程interrupt方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过interrupt方法检测到是否有中断发生

### 7、对象终结规则（Finalizer Rule）

一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize方法的开始

### 8、传递性（Transitivity）

如果操作A先行发生于B，操作B先行发生于C，那么操作A先行发生于操作C







