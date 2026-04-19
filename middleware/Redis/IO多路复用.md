# IO多路复用

## 事件

- 文件事件：基于Reactor模式：文件事件处理器(file event handler)

  - I/O多路复用：同时监听多个socket，并根据socket目前执行的任务来为socket关联不同的事件处理器

  - 当被监听的socket准备好执行连接accept、read、write、close等操作时，与操作对应的文件事件就会产生，这时文件事件处理器就会调用socket事先关联好的事件处理器来处理这些事件

  - 文件事件处理器的构成：

    > 文件事件是对socket的抽象，I/O多路复用监听多个socket并向dispatcher发送那些产生了事件的socket。socket可能并发的来，但都会被I/O多路复用程序放到一个队列中，通过这个队列有序、同步、每次一个的方式向dispatcher传送socket。上一个事件处理完，才会发送下一个socket到dispatcher

    - socket
    - I/O多路复用程序
      - 实现：对常见I/O多路复用函数库进行包装；单独成文件：ae_select.c, ae_epool.c, ae_kqueue.c...；API相同，故底层实现可互换
    - 文件事件分派器(dispatcher)
    - 事件处理器：一个个函数，定义了某个事件发生时服务器应该执行的操作

  - 事件类型：I/O多路复用程序可以监听 AE_READABLE\AE_WRITABLE事件对应如下：

    - 当socket变得可读时：客户端对socket执行write操作，或者执行close操作；或者有新的可应答（accepted）socket出现时（客户端对服务器的监听socket执行connect操作），socket产生AE_READABLE事件
    - 当socket变得可写时：客户端对socket执行read操作，socket产生AE_WRITABLE事件
    - 如果一个socket即可读又可写时：先读再写

```c
普通方式（你去敲门问）：
─────────────────────────────────
  while (true) {
      if (A有事吗) handleA();
      if (B有事吗) handleB();
      if (C有事吗) handleC();
      // 一个个问，很累，大部分时候都没事
  }
事件驱动（别人举手你才去）：
─────────────────────────────────
  while (true) {
      events = epoll_wait();  // 躺平等通知
      for (e : events) {
          e.handler();        // 谁举手处理谁
      }
  }
```

- 时间事件
  - 定时任务：过期清理、rehash、bgsave等等


