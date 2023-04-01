#还没有复习 

异步和同步：关注的是消息通信机制。调用者调用完 api 后需要等待方法执行结束并返回结果，就叫同步。如果调用方法后不需要等待也不获取结果就继续向下执行，就叫异步。异步调用的结果通常用通知的方式回调，类似观察者模式

阻塞和非阻塞：关注的是调用者的状态。被调用者需要在执行完结果后再返回结果就叫阻塞。如果不需要执行完结果就直接返回或返回给调用者一个 Feature 就叫非阻塞



- 同步阻塞：函数调用者等待函数 f 完成任务后再执行下一步。此时 f 已经执行完逻辑
- 同步非阻塞：函数调用者会等待函数返回一个 future 后再执行下一步。虽然 f 函数已经返回，但 f 函数可能含没有执行完逻辑
- 异步非阻塞：函数调用者不需要等待函数完成，且不关心函数是否执行完逻辑
- 没有异步阻塞：因为既然是异步了就一定不需要等待函数执行完逻辑，自然就不会又阻塞



举例说明

- 同步阻塞：到理发店理发，就一直等理发师，直到轮到自己理发
- 同步非阻塞：到理发店理发，发现前面有其它人理发，给理发师说下，先千其他事情，一会过来看是否轮到自己
- 异步非阻塞：给理发师打电话，让理发师上门服务，自己干其它事情，理发师自己来家给你理发



Java 中的多路复用 IO 的 NIO是同步非阻塞

即需要获取方法执行的返回值，但方法调用可以直接返回一个 Future（同步），而耗时行为由多线程处理，主线程不需要进入阻塞等待（非阻塞）





# 事件循环的核心组件

每个 NioEventLoopGroup 都是一个事件循环组，包含多个 NioEventLoop。每个 NioEventLoop 都是一个事件循环

每个 NioEventLoop 都有一个 Selector，Selector 会把收到的事件发给线程池，所以每个 NioEventLoop 都有一个用于做事件循环的主线程和多个工作线程

所以每个 NioEventLoopGroup 都是一个事件循环组，包含多个 NioEventLoop，每个 NioEventLoop 又都是一个线程池

> 默认 NioEventLoopGroup 中 NioEventLoop 的数量是 CPU 数量的 2 倍
>
> 如果用上述默认值创建 bossGroup 和 workGroup，那么就是多 reactor 多线程，且是多主多从的架构
>
> 通常有一个主就行了。但如果有主 reactor 有多个，要么就让每个主 reactor 绑定一个端口号，要么就绑定一个，但每次处理 accept 事件时都通过 chooser （NioEventLoopGroup 的成员变量）选择选一个主 reactor（eventloop）

所有的 NioEventLoop 的主循环 - 事件循环都在执行以下行为

1. select 获取到活跃的 socket 对应的 SelectedKey
2. processSelectedKey 处理第 1 步获取到的 key
3. runAllTasks 处理任务队列里的任务

其中第 2 步和第 3 步的区别在于



其实还会执行一步：处理空轮询问题



Q：不管是在 Demo 里还是在 BaseChat 里，server 上执行 client 的 pipeline 链时使用的线程都是同一个，因为用标准输出输出的 ThreadName 都是同一个，比如 nioEventLoopGroup-3-1

A：是不是因为每个 client 的 socket 只会注册到一个 loop 上，而 ThreadNam 的命名格式为 nioEventLoopGroup-loopGroupId-loopId。所以其实 netty 的 threadname 不包含线程名，只包含了 loopId



# taskQueue

任务队列的主要运用场景为：当 client 发送的事件需要让 handler 执行很长时间时，可以把任务扔进 netty 提供的任务队列里让线程池慢慢执行

任务队列中的 Task 有 3 种典型使用场景

- 用户程序自定义的普通任务

```java
ctx.channel().eventLoop().ececute(Runnable run); // 尽快执行的任务
```

任务会先被提交到 taskQueue



在 loop 的事件循环里，每次执行 runAllTask 时，任务线程池都会取出一个任务执行，但任务线程池里默认只有一个线程（一个池子就一个线程？搞毛？？）



- 用户自定义定时任务

```java
ctx.channel().eventLoop().schedule(Runnable run); // 定时任务
```

任务被提交到 scheduleTaskQueue

会由 netty 用 基于时间轮实现的定时任务执行





- 非当前 Reactor 线程调用 Channel 的各种方法
  例如在推送系统的**业务线程**里面， 根据用户的标识， 找到对应的 Channel 引用， 然后
  调用 Write 类方法向该用户推送消息， 就会进入到这种场景。 最终的 Write 会提交到
  任务队列中后被异步消费  



# 管道 pipeline

```java
ServerBootstrap serverBootstrap = new ServerBootstrap();

serverBootstrap.group(bossGroup, workGroup)
    .channel(NioServerSocketChannel.class)
    .option(ChannelOption.SO_BACKLOG, 128)
    .childOption(ChannelOption.SO_KEEPALIVE, true)
    .childHandler(new ChannelInitializer<SocketChannel>() {
        @Override
        protected void initChannel(SocketChannel ch) throws Exception {
            ChannelPipeline pipeline = ch.pipeline();
            pipeline.addLast(new HttpServerCodec());
            pipeline.addLast(new ChunkedWriteHandler());
            pipeline.addLast(new HttpObjectAggregator(8192));
            pipeline.addLast(new WebSocketServerProtocolHandler("/chat"));
            pipeline.addLast(new WebSocketHandler());
        }
    });
```



pipeline 是真正的 handler 链，它是双向链表

每个 pipeline 都对应一个 NioChannel （客户端的 channel）。这也是为什么每个 client 的 pipline 里的 handler 对象都不是同一个。因为过来一个 client 连接都会创建一个 pipeline 并调用上述的 ChannelInitializer.initChannel 方法

和 NioServerChannel 是双向绑定的

![[../../../020 - 附件文件夹/Pasted image 20230401230902.png]]

ChannelHandlerContext 保存了 channel（表示客户端的 channel） 的所有信息，同时关联到一个 ChannelHandler 对象

这个 ChannelHandler 对象就是下一个将要被执行的 handler



以处理入站事件为例。在 server  端用 client 找到 client 的 pipeline，pipeline 根据入站标志位遍历 handler 链中的入站 handler

每个 handler 都持有自己的 client handler，pipeline 和集大成的 ChannelHandlerContext

如果在 handler 链执行到某个 handler 时，通过 ctx 向 client 发数据，会执行 client 的 pipline 的出站 handler 链后再继续向下执行



> 注意一下关于编解码器的解码器
>
> 编码器的父类会自行判断将要的数据类型是否可以被当前编码器编码，如果能被编码就调用 coder 实现的 encode 方法，否则就不调用，直接调用 write 方法
>
> 其判断方式是 handler 执行的 ctx.writeAndFlush 传输入的数据类型是不是编码器支持的类型



过滤器链的向后调用

过滤器链的每个节点都是一个 DefaultChannelHandlerCotext，如果还想调用后面的节点，就调用 ctx.fireXxxx（入站 handler） 或 ctx.outXxx（出栈 handler）



# 事件类型

在服务器的 serverChannel 监听端口时或收到 客户端的请求建立连接时，会需要向 selector 注册事件

对于服务器而言

- 服务器收到客户端的 channel 发送来的请求建立连接，就叫 accept 事件
- 服务器收到客户端的 channel 发送来的数据，就叫读事件
- 服务器向客户端的 channel 发送数据，就叫写事件





# 心跳机制

这里的心跳检测不是保活，客户端并不会发送主动发送心跳包

而是服务器单方面检查客户端是否很长时间都没发数据了

> 保活 + 心跳检测 = 真 · 保活



假设要实现这样的心跳检测

- 当服务器超过3秒
- 读时， 就提示读空闲
- 当服务器超过5秒没有写操作时， 就提示写空闲
- 实现当服务器超过7秒没有读或者写操作时， 就提示读写空闲  



心跳机制的实现只需要在服务器上写就行了

如果某个客户端连接在上述规则内还是向服务器没有任何消息，server 就会被提示信息，coder 可写断开连接的代码



```java
/**
 * 使用心跳检测处理器
 * 读空闲 写空闲 读写空闲 的超时时间
 * 最后一个参数是 时间的单位
 * IdleStateHandler发现有空闲的时候 会触发 IdleStateEvent 时间
 * 他会把事件推送给下一个 handler的指定方法 userEventTriggered 去处理
 **/
pipeline.addLast(new IdleStateHandler(3, 5, 7, TimeUnit.SECONDS));

```

