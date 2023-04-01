#还没有复习 

- 传输层：Netty 支持 TCP 和 UDP
- 应用层：支持 HTTP，WebSocket，SSL，Protobuf，等多种协议和自定义协议
- API：事件驱动模型，零拷贝技术

![[../../../020 - 附件文件夹/Pasted image 20230401230437.png]]

Netty5 有重大 Bug，所以用 Netty4

[最详细的笔记在尚硅谷](E:\编程资料\尚硅谷\netty-2019-尚硅谷\笔记\尚硅谷_韩顺平_Netty核心技术及源码剖析.pdf)

# Netty 核心功能和线程模型

Netty 和 MQ 的区别是，前者是即时 RPC 框架， 不暂存消息；后者可暂存消息，甚至持久化消息

[参考资料](E:\编程资料\图灵学院\资料\课件代码\四：分布式框架专题\31-Netty核心功能与线程模型精讲-诸葛\02-VIP-Netty核心功能与线程模型精讲.pdf)

## Netty 线程模型

Netty 采用主从 Reactor 模型

![[../../../020 - 附件文件夹/Pasted image 20230401230506.png]]

## Netty 功能组件

- BootStrap 和 ServerBootStrap：引导类，用于绑定主从 NioEventLoopGroup，并启动程序
- NioEventLoopGroup：持有多个线程，每个线程持有 1 个 NioEventLoop，每个 NioEventLoop 持有 1 个 Selector
- Selector：循环执行 select，processSelectedKeys，runAllTasks
- Pipeline：processSelectedKeys 处理 key 时会经过 SocketChannel 上的 Pipeline 链
- ChannelHandler：用户的业务逻辑应该依靠添加在 SocketChannel 上的 Pipeline 链的 ChannelHandler 完成



ChannelHandler 分为入站和出站

- ChannelInboundHandler 用于处理入站 I/O 事件
  - 子类：ChannelInboundHandlerAdapter
- ChannelOutboundHandler 用于处理出站 I/O 操作
  - 子类：ChannelOutboundHandlerAdapter

ChannelHandlerContext：保存 Channel 相关的所有上下文信息，同时关联一个 ChannelHandler 对象

![[../../../020 - 附件文件夹/Pasted image 20230401230523.png]]


# 编解码，粘包拆包和零拷贝，自动重连，心跳机制

[参考资料](E:\编程资料\图灵学院\资料\课件代码\四：分布式框架专题\32-Netty核心功能精讲-诸葛\03-VIP-Netty编解码&粘包拆包&心跳机制&断线自动重连.pdf)



## 编解码机制

编解码功能由 Pipeline 里的 ChannelHandler 实现



handler 链在 pipeline 中，pipeline 中默认有 1 个 head handler 和 1 个 tail handler

Handler 有两种，入栈 handler 和出栈 handler

- 入栈 handler。`ChannelInboundHandlerAdapter`，端接收数据时，数据需要经过入栈 handler 链，所以解码器都是入栈 handler。入栈流程是从 head handler 向 tail handler 方向执行的

- 出栈 handler。`ChannelOutboundHandlerAdapter`，端发送数据时，数据需要经过出栈 handler 链，所以编码器都是出栈 handler。出栈流程是从 tail handler 向 head handler 方向执行的

handler 链是双向链表





常见的编解码器有`StringEncoder`，`ObjectEncoder`（传输各种对象，底层调用的不是 JDK 的序列化方式）

通常采用 protobuf 序列化方案，并且 Netty 也提供了 protobuf 的编解码器，但 protobuf 需要写 proto 文件。因此采用 protobuffer，它基于 protobuf，但简化了使用流程，避免了写 proto 文件

如果不使用任何编解码器时，默认管道传递的是 ByteBuf



## 粘包和拆包

粘包：多个数据被放到了一个 TCP 数据包中传递

拆包：一个完整的数据被拆到了多个 TCP 数据包中传递

![[../../../020 - 附件文件夹/Pasted image 20230401230540.png]]

粘包拆包会在这些场景下出现。客户端收到消息后会显示`客户端收到：msg`

客户端 1 一次性发送大量数据。服务器收到数据后通过发送多个 TCP 数据包转发给客户端 2，客户端 2 收到的数据来自多个数据包。但 OS 感知到 TCP 缓冲区满后直接把数据送到应用程序，而不管数据是否完整（因为 TCP 面向字节流）

期望：客户端 2 显示`客户端收到：aaabbbcccdddeee`

实际：客户端 2 显示`客户端收到：aa`，`客户端收到：abbbc`，`客户端收到：ccdddeee`。即客户端 2 调用了多次 channelRead0 方法，而非调用一次并接收完所有数据



分包解决方案（一个完整的数据被分到多个 TCP 数据包里后，在接收端组装的策略）

- 分隔符 + 转义字符，通过分隔符标记数据未完结或数据的完结（不用）
- 填充位，收发固定长度的数据，数据长度不足就用填充字符填充（不用）
- 发送数据长度 + 数据。比如，前 4 字节表示数据长度（优选）

Netty 提供了分包解码器，但没有提供第三种分包解码器（编码好像得自己做，比如手动在数据后添加分隔符，或在数据头加数据长度字段）



## 心跳机制

心跳的目的是为了实时得知连接状态。如果心跳包能在两端收发，说明连接健在，通信双方都没有挂掉。同时防止连接上长期没有数据传输而自动断掉连接（保活）

防火墙有时也会主动干掉长时间没有进行数据传输的连接

Netty 提供了 IdleStateHandler 作为心跳机制的 handler



Netty 提供的 IdleStateHandler 是这样用的

- 客户端自行编写 handler，并定时发送自定义的心跳包
- 服务端添加 IdleStateHandler，它会定期检查是否超时后仍收到任何数据（包括心跳包和其他数据，心跳包本来就是保活的，如果一直有普通数据收发， 心跳包也就没啥作用了）

IdleStateHandler 定期检查是否发生超时事件，其不是用定时任务实现的，且如果发生超时事件后会调用`ChannelHandlerContext.fireXxx`，它最终会调用`ChannelHandlerContext.xxx`

其逻辑是 IdleStateHandler 主动调用下一个 handler 的 userEventTriggered，所以程序员还需要自行实现一个 handler 并重写其 userEventTriggered 方法。用户在这里实现对超时事件进行处理，比如主动断掉连接

> IdleStateHandler 定期检查是否发生超时事件，其不是用定时任务实现的
>
> 这样做，下次检查超时时间的时间可以在程序运行时动态修改
>
> 比如：设置 3s 为超时时间，2s 时收到 data，下次检查时应该是 1s 后。这样可以保证每 3s 检查一次心跳包
>
> **没理解**



## 自动重连

- 客户端先启动，服务端再启动。客户端一开始没连上服务端，但客户端会不断尝试重连
- 连接突然断掉，客户端尝试自动重连

重连的逻辑应该在客户端

可以在 channelInactive 方法中让 loop 启动一个定时任务，不断尝试重连



# Netty 核心源码剖析

Netty 是对 Java提供的 NIO 的高度封装，做了大量的优化，用了大量的设计模式。但本质上毕竟还是封装了 NIO，所以分析源码时时不时就会看到 NIO 的代码，所以一定要把 NIO 弄熟



**Netty 线程模型图**

![[../../../020 - 附件文件夹/Pasted image 20230401230556.png]]

`NioEventLoop`中的 Selector 是 SocketChannel 和事件的集合。TaskQueue 和事件处理无关

TaskQueue 是用来处理启动稍微耗时的 Netty 内部事务的。比如启动前，先把注册`accept`的注册任务加进这个队列，把启动`NioEventLoop`的`run`方法（启动事件循环，死循环监听事件和处理事件）

`NioEventLoop`有两层循环

- 第一层循环用来循环处理 select（获取 key），processSelectKeys（处理 key 的事件），runAllTasks（处理任务）
- 第二层循环是 processSelectKeys 内部，循环处理 key 的事件（事件循环）








# Netty 高并发性能架构设计精髓

- 主从Reactor线程模型
- NIO多路复用非阻塞
- **无锁串行化**设计思想
- 支持高性能序列化协议
- 零拷贝(直接内存的使用)
- ByteBuf内存池设计
- 灵活的TCP参数配置能力
- 并发优化  

Netty 处理百万连接：

首先机器得是超高配（比如100+处理器核心和超大内存），其次百万链接是理想目标，是实验室数据。但如果百万连接中活跃的只有几十万是活跃的，且不是同一时间建立的连接，那也不是不能可能支持百万连接。单线程同一时刻处理几十，几百个事件还是可以的因为

最后，bossGroup 需要设置多个线程 - 多个 EventLoop，同时每个线程绑定一个端口（bossGroup 的每个线程都能单独绑一个端口）。但通常不这么做，通常只设置 1 个线程，然后在不同服务器上搭建 Netty 集群

Netty 也可搭建集群，Netty 集群需要 zk 支持



# 直接内存和零拷贝



## 直接内存

了解零拷贝前先了解直接内存：JVM 用的堆内存以外的内存。直接内存会和其他进程共享

JVM 的原空间就不是 JVM 规范中定义的内存空间，也被称为非堆

测试操作堆内存和直接内存的分配和访问速度：直接内存读写速度更快，堆内存分配速度更快



堆堆内存的 ByteBuffer 的读写就是对 ByteBuffer 持有的一个 byte 数组的读写

对直接内存的分配，本质上调用了 C 的 malloc 函数，ByteDirectBuffer 只保留元数据和直接内存的地址，对直接内存的读写也是调用 C 语言的函数

直接内存会在回收 ByteDirectBuffer 时也被回收，在 ByteDirectBuffer 申请完直接内存后会创建一个 Cleaner 钩子方法，GC 清理掉 ByteDirectBuffer 的 GCRoot 后就会用 Cleaner 机制回收掉直接内存（但其流程复杂，暂不需要掌握）



## 零拷贝

![[../../../020 - 附件文件夹/Pasted image 20230401230610.png]]

非零拷贝 - 采用堆内内存收发 socket 数据流程中。OS 首先调用 read 系统调用把 socket 缓冲区中的数据读入直接内存

而不是直接读到 JVM 的 ByteBuffer 中，因为 socket 传输数据的行为是 OS 的行为，而不是专为 JVM 特设的，或者说 OS 调用 read 系统调用时并不知道 JVM 进程的存在

> 系统调用只能在 OS 的内核态下才能进行

JVM 再调用 unsafe 读取直接内存（内核空间）中的数据到 JVM（用户空间），这是用户态下的操作。上述操作涉及多次 OS 的状态转换（用户态  - 内核态 - 用户态）



Netty 中执行收发数据的 ByteBuf 默认用的都是直接内存（可以通过设置将其改为堆内内存），但自行创建的比如`Unpooled.copiedBuffer()`是堆内内存

可通过`Unpooled.directCopiedBuffer()`



# 空轮询 Bug

其实问的很少了。空轮询 Bug 在 JDK8 中还没有被完美地解决。此 Bug 不定时出现

问题描述。默认`selector.select()`会阻塞等待。空轮询会导致没有事件发生时也会返回，导致程序一直执行`selector.select()`所在的循环

不是阻塞等待，而是一直执行循环 - 空轮询。这会导致 CPU 100%，因为 JVM 不会因为阻塞而切换线程。所以 CPU 一旦空闲就会切回 JVM 进程执行循环



Netty 解决空轮询的方式是在`selector.select()`后面添加了大量的 if 分支代码，如果`selector.select()`执行的空轮询次数到达阈值，就创建一个新的`Selector`替换掉原有的，并把旧`selector`中的事件注册到新的`selector`中。Netty 只记录空轮询，并用大量的 if 分支针对非空空轮询的执行，会把计数器置 1，让计数器只记录空轮询次数