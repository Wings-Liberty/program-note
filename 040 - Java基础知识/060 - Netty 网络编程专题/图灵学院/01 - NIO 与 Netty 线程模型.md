#还没有复习 

> - BIO & NIO & AIO 模型
> - 深入 Hotspot 源码理解 NIO 多路复用器（看了 Linux 下的 Selector 实现类）
> - 深入 Linux 内核理解 Epoll 事件轮询模型（看了 epoll 在 LInux 下的 3 个函数）
> - 阿里面试问的 select、poll、epoll 模型的区别
> - 深入 Redis 源码理解 Redis 高并发线程模型
> - Reactor 响应式编程设计模式
> - Netty 主从 Reactor 高并发线程模型
> - 同步异步阻塞非阻塞



# BIO & NIO & AIO 模型



## BIO

传统 BIO 模型提供的 API 多为阻塞方法。为了同时处理多个客户端，需要为每个连接分配 1 个线程，且只有处理完此客户端的所有事件并断开连接后才会释放此线程

```java
SocketServer server = new SocketServer(9000);
while(true){
    Socket client = server.accept();
    new Thread(()->{
        handlerSocket(client);
    }).start();
}
```

1 个连接 1 个线程的处理效率很差。大量连接会很快耗尽内存，且服务器没那么多线程可用（CPU 核心数有限）

![[../../../020 - 附件文件夹/Pasted image 20230401230053.png|500]]

就算采用连接池效果也差，因为线程池规定了线程数的上限


## NIO

NIO 提供的 API 多为同步非阻塞方法

> 非阻塞不意味着系统自动创建新线程，在后台用新线程处理请求

NIO 处理 Socket 的流程类似计算机处理中断的流程，在处理主业务的同时不断查询 socket 的状态位，如果 socket 状态位表明有事件需要处理，当前线程马上处理其事件，处理完成后继续回去处理主业务

但 NIO 不应该轮询所有 Socket 的状态位，因为可能大量的 socket 只有极少数的处于活跃状态，所以轮询中包含了大量的无效轮询

**多路复用器 - Selector**

所以 NIO 轮询的是一个 key 集合，key 集合中包含了 socket 发送来的事件（连接请求，发送数据，断开连接等事件），并且每处理完 1 个 key 就会将其移出集合

socket 是 NIO 要通信的主体，NIO 的通信单位是 key

![[../../../020 - 附件文件夹/Pasted image 20230401230107.png|650]]

[资料](E:\编程资料\图灵学院\资料\课件代码\四：分布式框架专题\30-深入Hotspot源码与Linux内核理解NIO与Epoll-诸葛\01-VIP-深入Hotspot源码与Linux内核理解NIO与Epoll.pdf)

ServerSocketChannel 向 Selector 注册连接请求事件：收到请求事件后，Selector 会把连接请求事件和发起注册的 ServerSocketChannel 绑到 key 上

收到请求连接事件后，重新获取 ServerSocketChannel，并调 accept 接收请求连接的 SocketChannel

SocketChannel 在服务端向 Selector 注册读事件：收到读事件后，Selector 会把读事件和发起注册的 Socket Channel 帮到 key 上，并等待处理



# Epoll 模型和 NIO 的关系

NIO 底层就是调用了 Linux 的 epoll IO 模型的 3 个核心函数



## Linux 的 3 种 IO 模型

Linux 的 epoll 模型是用来进行网络通信的通信模型，是 OS 的行为，而非 NIO 等应用程序的

对于服务器来说，接收网络数据并令网络进程获取数据的过程如下

1. 网卡收到数据后将其读入内存
2. socket 收到数据后通过中断机制通知在 socket 等待队列上的网络进程

不同 IO 模型的实现不同之处在于监听 socket 的方式不同



- 最原始的方式 recv：1 进程监听 1 个端口，socket 缓冲区收到数据后就唤醒等待队列上的进程

- select 模型：1 个进程遍历所有 socket 并挂到所有 socket 的等待队列上，当有 socket 缓冲区收到数据后唤醒此进程，此进程进入就绪态，通知 OS 遍历所有 socket，因为进程之所以能被唤醒说明必定至少有 1 个 socket 有需要处理的数据（共计遍历了 2 遍 socket 集合）

- poll 模型：和 select 模型很类似，但有少许改进

- epoll 模型：划出两个集合：存放所有 socket 集合 A（等待队列），存放收到数据的 socket 集合 B（就绪列表）。B 中的 socket 来自于 A

  `epoll_ctl`对两个集合进行 curd 管理，`epoll_wait`用于阻塞式检查就绪列表是否有数据并返回相关信息

  一旦`epoll_ctl`将 socket 加到就绪列表中，同时也会通过中断机制唤醒调用`epoll_wait`而进入阻塞的进程

  上述的所有数据结构都是由`epoll_create`创建的`eventpoll`持有的

> 注意：不要说，socket 缓冲区收到数据后，socket 会自行唤醒其 wait queue 下的进程。socket 是静态数据结构，socket 不是动态进程。其收到数据的过程是网卡向其缓冲区写数据的过程。socket 能唤醒其他进程是因为有进程做这件事，而不是说 socket 本身就是一个进程

具体可参考[epoll 和 socket 和中断](https://www.sohu.com/a/317147914_827544)





## NIO 创建 Selector 的流程

NIO 在创建`Selector`时，根据操作系统的不同返回不同的实现类。如果是 Linux 就返回 1 个`EpollSelectorProvider`并调用其`openSelector`方法创建 `EpollSelectorImpl`

在`Selector`调用其`select()`时最终会调用 Linux 的 epoll IO 模型的 3 个核心函数

> 冷知识 JNI 调用 C&C++ 代码时，其 com.a.b.C 在 Java 中的方法名为 d，则其在 C&C++ 中的方法名为 Java_com_a_b_C_d
>
> 此法有助于在自行编译的 JDK 中寻找 native 方法在  C&C++ 中的实现

在创建 `EpollSelectorImpl`时会调用 native 方法，其  C&C++ 方法调用了`epoll_create`，这是 Linux 提供的库函数，是 epoll 模型的核心函数之一

> 在 Windows 下打不开`epoll_create`，因为它是 Linux 提供的库函数，Windows 自然找不到
>
> 但可以在 Linux 下用`man`查找任何命令和函数的说明。调`man epoll_create`即可



## epoll_create

根据`DESCRIPTION`说明，表示此函数创建了 1 个`epoll`实例，并返回 1 个文件描述符

```
NAME
       epoll_create, epoll_create1 - open an epoll file descriptor

SYNOPSIS
       #include <sys/epoll.h>

       int epoll_create(int size);
       int epoll_create1(int flags);

DESCRIPTION
       epoll_create()  creates  a new epoll(7) instance.  Since Linux 2.6.8, the size argument is ignored, but must be greater than zero; see
       NOTES below.

       epoll_create() returns a file descriptor referring to the new epoll instance.  This file descriptor is used  for  all  the  subsequent
       calls  to  the  epoll  interface.   When  no longer required, the file descriptor returned by epoll_create() should be closed by using
```



epoll 实例（eventpoll）里有两个集合

- 第一个集合是所有 socket 的等待队列
- 第二个集合是有待处理数据的 socket 的就绪列表

NIO 通过创建 Selector 创建上述实例，调用 select 方法调用`epoll_select`获取就绪列表的信息并封装为 key 返回给 Java 应用程序



## SocketChannel 向 Selector 中注册事件的流程

首先需要知道，在 Linux 下一切接文件，每次创建 SocketChannel 时都会返回这个文件的文件描述符

SocketChannel 向 Selector 中注册事件的流程实质上就是，声明应用程序关注的事件类型，便于筛选 select 获取到的事件



## Selector 的 select 过程

其核心在于调用了`epoll_wait`函数（这就是 Linux 的 Epoll 模型的 3 大函数），阻塞式检查 eventloop 的就绪队列



### epoll_ctl

`DESCRIPTION`意思大致为：使用文件描述符 epfd 引用的 epoll 实例，对目标文件描述符 fd 执行 op 操作。参数 epfd 表示 epoll 对应的文件描述符，参数 fd 表示 socket 对应的文件描述符

`epoll_ctl`用于维护 eventpoll 的阻塞队列和就绪列表

```
NAME
       epoll_ctl - control interface for an epoll file descriptor

SYNOPSIS
       #include <sys/epoll.h>

       int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

DESCRIPTION
       This  system  call is used to add, modify, or remove entries in the interest list of the epoll(7) instance referred to by the file de‐
       scriptor epfd.  It requests that the operation op be performed for the target file descriptor, fd.

       Valid values for the op argument are:

       EPOLL_CTL_ADD
              Add fd to the interest list and associate the settings specified in event with the internal file linked to fd.

       EPOLL_CTL_MOD
              Change the settings associated with fd in the interest list to the new settings specified in event.

       EPOLL_CTL_DEL
              Remove (deregister) the target file descriptor fd from the interest list.  The event argument is ignored and can be  NULL  (but
              see BUGS below).

```

> epoll 实例（eventpoll）里有两个集合
>
> - 第一个集合是所有 socket 的等待队列
> - 第二个集合是有待处理数据的 socket 的就绪列表



### epoll_wait

`DESCRIPTION`意思为：等待文件描述符 epfd 上的事件。epfd 是 Epoll 对应的文件描述符，events 表示调用者所有可用事件的集合，maxevents 表示最多等到多少个事件就返回，timeout 是超时时间

这个 timeout 也是 NIO 调用`selector.select(timeout)`可以传递超时时间的原因

```
NAME
       epoll_wait, epoll_pwait - wait for an I/O event on an epoll file descriptor

SYNOPSIS
       #include <sys/epoll.h>

       int epoll_wait(int epfd, struct epoll_event *events,
                      int maxevents, int timeout);
       int epoll_pwait(int epfd, struct epoll_event *events,
                      int maxevents, int timeout,
                      const sigset_t *sigmask);

DESCRIPTION
       The  epoll_wait()  system  call  waits  for  events on the epoll(7) instance referred to by the file descriptor epfd.  The memory area
       pointed to by events will contain the events that will be available for the caller.  Up to maxevents  are  returned  by  epoll_wait().
       The maxevents argument must be greater than zero.

       The  timeout argument specifies the number of milliseconds that epoll_wait() will block.  Time is measured against the CLOCK_MONOTONIC
       clock.  The call will block until either:

       *  a file descriptor delivers an event;

       *  the call is interrupted by a signal handler; or

       *  the timeout expires.
```

`epoll_wait`操作的是`epoll`实例中的就绪列表。如果就序列表空，`eopll`就阻塞或限时阻塞（如果设置了超时时间），这也是 NIO 中为数不多的且**必要**的阻塞方法

> epoll 实例（eventpoll）里有两个集合
>
> - 第一个集合是所有 socket 的等待队列
> - 第二个集合是有待处理数据的 socket 的就绪列表



# select，poll 和 epoll 模型的区别

这是一道常见 IO 模型面试题



具体可参考[资料](E:\编程资料\图灵学院\资料\课件代码\四：分布式框架专题\30-深入Hotspot源码与Linux内核理解NIO与Epoll-诸葛\01-VIP-深入Hotspot源码与Linux内核理解NIO与Epoll.pdf)和[epoll 和 socket 和中断](https://www.sohu.com/a/317147914_827544)



Java 最早在 JDK1.4 纳入 NIO 模型，但采用的是 select 模型，后又用 poll 模型，最后又改为 epoll 模型



select 模型中，NIO 面对通信单位是 socket，而不是 key。通过轮询所有 socket 获取并处理所有事件。即同一时间，当大量的 socket 中只有极少数的 socket 在活跃时，轮询中会有大量的无效轮询。既浪费性能又浪费时间



> Netty 基于 epoll 模型，并在 JDK 提供的 NIO 实现上做了更多优化。优化甚至优于 Redis



# Redis 高并发线程模型

Redis 是单线程的，但效率很高

redis 采用 epoll 模型。具体实现见 Redis 源码中的`ae_epoll.c`，其具体实现本质上就是调用了 Linux 的 Epoll 模型的 3 大函数



尽管 redis 采用了 epoll 模型，但受单线程的限制， redis 不能同时高效处理大量耗时的请求

所以 redis 禁止客户端发送耗时大的遍历操作指令，防止单线程处理耗时请求时不能及时响应后面的请求



# Netty 和响应式编程

参考 [Scalable IO in Java](E:\编程资料\图灵学院\资料\课件代码\四：分布式框架专题\31-Netty核心功能与线程模型精讲-诸葛\nio.pdf)

Epoll 是基于事件响应的模型的实现，观察者模式和响应式模型十分类似

在 NIO 中 Reactor 就是 Selector，在 epoll 中 Reactor 就是 eventpoll



IO 模型演化过程如下



## Classic Service Designs

1 个线程仅处理 1 个连接

![[../../../020 - 附件文件夹/Pasted image 20230401230137.png]]

连接驱动的 IO 模型效率低下，由此产生事件驱动模型。Reactor 模型就是事件驱动模型的 1 种实现



## Basic Reactor Design

单线程实现。通过 selector 实现面向事件的 IO 模型

![[../../../020 - 附件文件夹/Pasted image 20230401230154.png]]

单线程的限制导致 long-time task 会延迟后面的任务的执行



## Multithreaded Designs

多线程实现。主线程线程持有 selector，**读取 socket 中的数据**后把任务提交给线程池。待线程处理完后接收返回结果并发送给客户端连接

![[../../../020 - 附件文件夹/Pasted image 20230401230205.png|475]]

线程池不持有任何 Socket，所以主线程承担了读数据，分发事件，向客户端返回响应的压力。仅靠增加线程池中的线程数量无法解决此问题



## Multiple Reactors

主从模型，也是 Netty 采用的模型。实现建立连接事件和读写事件解耦

- 主 Reactor 仅需响应建立连接事件，无需承担读数据，分发事件，向客户端返回响应的压力
- 从 Reactor 持有多个线程，每个线程都持有一个 selector，并注册 socket 的读写事件。客户端读写事件直接由从 Reactor 获取，而不经过主 Reactor

![[../../../020 - 附件文件夹/Pasted image 20230401230219.png]]

Netty 中主从 EventLoopGroup 已经是线程池了，其每一个线程的 EventLoop 又都持有 1 个线程池

因为每个 bossThread 和 workThread 线程都通过 EventLoop 持有一个 selector 用于处理 IO 事件的 select，具体事件处理业务当然应由应用线程处理



这个 IO 模型推演过程需要会（面试可能问）



