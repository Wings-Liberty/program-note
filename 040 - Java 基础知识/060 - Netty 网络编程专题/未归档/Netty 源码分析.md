#还没有复习 

> 1. Netty怎么用，有什么功能能用。这里不写，这里只写源码分析
> 2. 源码分析程度不深，目的在于对照Netty的执行流程图（看不懂的话，可以看NIO的执行流程图）掌握Netty的运行流程

![[../../../020 - 附件文件夹/Pasted image 20230401231329.png|500]]

![[../../../020 - 附件文件夹/Pasted image 20230401231334.png|500]]


## Example

```java
// NettyServer，这是自己写的类
public static void main(String[] args) {
    // 创建EventLoopGroup时会创建EventLoop数组，数量为 2*CUP核数。并为每个EventLoop添加一个监听器
    EventLoopGroup bossGroup = new NioEventLoopGroup();
    EventLoopGroup workGroup = new NioEventLoopGroup();

    try {
        // 创建服务器的启动类。服务器的生命周期方法和服务器的配置都在这里
        ServerBootstrap serverBootstrap = new ServerBootstrap();

        // 配置
        serverBootstrap.group(bossGroup, workGroup) // 设置两个线程组
            .channel(NioServerSocketChannel.class) // 设置服务器使用的通道的类型，创建一个Factory，工厂使用反射创建真正的ServerChannel
            .option(ChannelOption.SO_BACKLOG, 128) // 设置bossGroup线程队列得到的连接个数（不懂）
            .childOption(ChannelOption.SO_KEEPALIVE, true) // 设置workGroup的连接保持活动连接
            .childHandler(new ChannelInitializer<SocketChannel>() { // 为workGroup创建一个Handler
                // 此类型的handler是用来执行初始化Handler的工作，可以使用此方法向workGroup中添加更多的Handler
                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new NettyServerHandler());
                }
            });

        System.out.println("-----服务器成功启动-----");

        ChannelFuture cf = serverBootstrap.bind(8080).sync();

        // 监听关闭通道的指令，监听到关闭消息是就会关闭通道
        cf.channel().closeFuture().sync();
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        bossGroup.shutdownGracefully();
        workGroup.shutdownGracefully();
    }
}
```



# 启动流程分析



## EventLoopGroup的创建

> 调用构造方法为EventLoopGroup创建NioEventLoop数组

```java
// NettyServer，这是自己写的类
EventLoopGroup bossGroup = new NioEventLoopGroup();
```



```java
// NioEventLoopGroup extends MultithreadEventLoopGroup
protected MultithreadEventExecutorGroup(int nThreads, Executor executor, EventExecutorChooserFactory chooserFactory, Object... args) {

    // 成员变量children，容量默认是2*CPU数量
    children = new EventExecutor[nThreads];

    for (int i = 0; i < nThreads; i ++) {
        // 为每个元素赋值，类型是NioEventLoop
        children[i] = newChild(executor, args);
    }
	// 需要从多个 loop 里选一个时，就调 chooser 选一个
    chooser = chooserFactory.newChooser(children);

    // 将children放到set集合
    Set<EventExecutor> childrenSet = new LinkedHashSet<EventExecutor>(children.length);
    Collections.addAll(childrenSet, children);
    // 创建一个不可修改的set（只读set），并赋值给成员变量
    readonlyChildren = Collections.unmodifiableSet(childrenSet);
}
```

执行完构造方法就完成了 EventLoopGroup 的创建



## ServerBootstrap的启动

> 包含ServerBootStrap启动时，怎么创建的NioServerSocketChannel，怎么创建的Pipeline，怎么绑定的端口号，怎么执行的无限循环

用于启动ServerChannel

````java
// 1. 创建服务器的启动类。服务器的生命周期方法和服务器的配置都在这里
ServerBootstrap serverBootstrap = new ServerBootstrap();

// 2. 配置
serverBootstrap.group(bossGroup, workGroup) // 设置两个线程组
    .channel(NioServerSocketChannel.class) // 设置服务器使用的通道的类型
    .option(ChannelOption.SO_BACKLOG, 128) // 设置bossGroup线程队列得到的连接个数（不懂）
    .childOption(ChannelOption.SO_KEEPALIVE, true) // 设置workGroup的连接保持活动连接
    .childHandler(new ChannelInitializer<SocketChannel>() { // 为workGroup创建一个Handler
        // 此类型的handler是用来执行初始化Handler的工作，可以使用此方法向workGroup中添加更多的Handler
        @Override
        protected void initChannel(SocketChannel ch) throws Exception {
            ch.pipeline().addLast(new NettyServerHandler());
        }
    });

System.out.println("-----服务器成功启动-----");

// 3. 绑定端口号
ChannelFuture cf = serverBootstrap.bind(8080).sync();
````



### 1. 创建ServerBootstrap对象

> 空构造，什么都没干

```java
public ServerBootstrap() { }
```

### 2. 配置

`.group(bossGroup, workGroup)`

```java
// 大致做了这些事
super.group(parentGroup); // 将bossGroup交给父类的变量
this.childGroup = childGroup; // 将workGroup交给成员变量
```

`.channel(NioServerSocketChannel.class)`

```java
// 指定服务端使用的Channel的类型
public B channel(Class<? extends C> channelClass) {
    if (channelClass == null) {
        throw new NullPointerException("channelClass");
    }
    // 创建一个factory，此工厂以后会通过反射创建NioServerSocketChannel（因为我指定了类型）
    return channelFactory(new ReflectiveChannelFactory<C>(channelClass));
}
```

`.option(ChannelOption.SO_BACKLOG, 128)`

```java
// 方法内主要做了这些事
options.put(option, value); // 将参数以kv方式put进成员变量。这些kv是bossGroup的配置信息
```

`.childOption(ChannelOption.SO_KEEPALIVE, true) `

```java
// 方法内主要做了这些事
childOptions.put(childOption, value); // 将参数以kv方式put进成员变量。这些kv是workGroup的配置信息
```

`.childHandler(new ChannelInitializer<SocketChannel>(){...})`

```java
// 方法内主要做了这些事
this.childHandler = childHandler; // 将参数赋给成员变量
```



### 3. 绑定端口号

> 这个方法做了很多事

```java
// NettyServer
serverBootstrap.bind(8080).sync();
```

![[../../../020 - 附件文件夹/Pasted image 20230401231409.png|550]]


**initAndRegister**（这的代码比 doBind0 的代码多，且包含大量重要逻辑）

```java
// initAndRegister方法主要做了这些事
/**
 * channelFactory使用反射创建一个ServerBootStrap.channel指定类型的Channel（NioServerSocketChannel）
 * 1. 创建了一个JDK的channel
 * 2. 创建了一个DefaultChannelPipeline
 * 3. 创建了一个配置类用于对外使用
 * 
 * 初始化channel和channel中的pipleline
 * 
 * 将创建的channel通过bossGroup注册到selector
 **/
channel = channelFactory.newChannel();
init(channel);
config().group().register(channel);


// channelFactory.newChannel();直接使用反射调用无参构造器
public NioServerSocketChannel() {
    this(newSocket(DEFAULT_SELECTOR_PROVIDER)); // newSocket方法会创建一个ServerSocketChannel
}

public NioServerSocketChannel(ServerSocketChannel channel) {
    // 将channel交给父类变量，还配置了SelectionKey.OP_ACCEPT
    super(null, channel, SelectionKey.OP_ACCEPT);
    // 创建一个配置类，并赋值给成员变量
    config = new NioServerSocketChannelConfig(this, javaChannel().socket());
}

// init(channel); 初始化
setChannelOptions(channel, options, logger); // 获取到ServerBootStrap.option设置的kv，并将配置应用到channel（ServerSocketChannel）

// config().group().register(channel); 把channel注册到一个EventLoop中
// next方法会从eventLoopGroup中选择一个eventLoop
public ChannelFuture register(Channel channel) {
    return next().register(channel);
}
// register会调用doRegister
protected void doRegister() throws Exception {
    boolean selected = false;
    for (;;) {
        try {
            // eventLoop().unwrappedSelector() 获取到selector（在创建eventLoop时就创建好了）
            // 并把 accept 事件注册进selector（第二个参数 0 表示 accept 事件）
            selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
            return;
        } catch (CancelledKeyException e) {
            if (!selected) {
                eventLoop().selectNow();
                selected = true;
            } else {
                throw e;
            }
        }
    }
}

// javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
// k是SelectorKey，最后会返回k。这里调的register是nio的方法
k = ((AbstractSelector)sel).register(this, ops, att);
```



ps：pipeline里保存的是ChannelHandlerContext，ChannelHandlerContext中保存有ChannelHandler和各种类



```java
// doBind0
// 最终调用nio的ServerSocketChannel的bind方法绑定端口
```



之后只要一直执行Step Over就能进入NioEventLoop的的run方法，fun方法就是图中的循环

![[../../../020 - 附件文件夹/Pasted image 20230401231432.png|500]]


## 服务端接收并处理请求的流程

> 这里包含  事件循环的死循环中做了什么事
>
> 详细点说。bossGroup 的死循环接收到事件后，是怎么将channel注册到workGroup

- `select`检查有没有需要处理的事件
- `processSelectedKeys`处理事件
- `runAllTasks`执行所有需要执行的任务





> 先分析bossWork接收到请求连接的请求后，服务端是如何处理的
>
> 1. 接收并创建连接SocketChannel并封装到一个NioSocketChannel对象中



在上述的事件循环中，会不断进行select并处理事件。如果有客户端发送请求连接的事件，会发生以下事情



```java
// 获取事件，如果不为空，获取这些SelectKey并遍历处理这些事件
processSelectedKeys()
    
    // 遍历过程中依次执行这些事件
    processSelectedKey(k, (AbstractNioChannel) a);
```



```java
// processSelectedKey中主要做了这些事
// 获取key的事件类型，根据事件类型用if处理。这里讲解的是连接事件，所以readOps的值为16（nio中提到过这些事件的值）
// 事务均有unsafe处理，这个对象包含了NioServerSocketChannel
int readyOps = k.readyOps();

if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
    int ops = k.interestOps();
    ops &= ~SelectionKey.OP_CONNECT;
    k.interestOps(ops);

    unsafe.finishConnect();
}


if ((readyOps & SelectionKey.OP_WRITE) != 0) {
    ch.unsafe().forceFlush();
}

// 客户端请求连接的事件在这里处理
if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
    unsafe.read();
}
```



```java
// unsafe.read(); 主要做了这些事
doReadMessages(readBuf); // 调用nio，调用accept方法创建连接并获取客户端的SoecktChannel对象再将对象封装到NioSocketChannel对象中（会像创建NioSerSocketChannel一样创建SerSocketChannel，配置类，Pipeline），将对象放到readBuf中

pipeline.fireChannelRead(readBuf.get(i));
```



```java
protected int doReadMessages(List<Object> buf) throws Exception {
    SocketChannel ch = SocketUtils.accept(javaChannel());
    try {
        if (ch != null) {
            buf.add(new NioSocketChannel(this, ch));
            return 1;
        }
    } catch (Throwable t) {
        ...
    }
    return 0;
}
```



```java
// pipeline.fireChannelRead(readBuf.get(i)); 这里获取到readBuf中的NioSocketChannel并执行pipeline中的所有入栈Handler
// pipeline中有一个ServerBootStrapAcceptor类型的Handler（ServerBootStrap执行init方法时添加到pipeline中的）
// 以此调用入栈Handler的channelRead方法，这里是ServerBootStrapAcceptor的方法

public ChannelHandlerContext fireChannelRead(final Object msg) {
    // 从当前Handler的后面获取到一个入栈Handler再执行其channelRead方法
    invokeChannelRead(findContextInbound(), msg);
    return this;
}

public void channelRead(ChannelHandlerContext ctx, Object msg) {
    final Channel child = (Channel) msg;

    child.pipeline().addLast(childHandler);

    setChannelOptions(child, childOptions, logger);

    for (Entry<AttributeKey<?>, Object> e: childAttrs) {
        child.attr((AttributeKey<Object>) e.getKey()).set(e.getValue());
    }

    try {
        // child是bossGroup中一个EventLoop中的NioServerSocketChannel接收到的客户端的NioSocketChannel
        // 将其注册到workGroup中
        // 事件循环组的register方法会**调用next选择一个EventLoop*再将channel注册到里面
        // 在前面启动ServerBootStrap时，注册ServerSocketChannel时说过这个方法
        childGroup.register(child).addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                if (!future.isSuccess()) {
                    forceClose(child, future.cause());
                }
            }
        });
    } catch (Throwable t) {
        forceClose(child, t);
    }
}
```

ps：pipeline中head和tail的Handler是固定的



暂时跳过Pipeline的创建和处理器链的调用流程



入栈的调用从head开始。出栈的调用从tail开始



关于`ChannelPipeline`的详细用法见类的注释



暂时跳过心跳处理器（一共有三个处理器）的分析



# 组件分析



## EventLoop

# 问题

![[../../../020 - 附件文件夹/Pasted image 20230401231459.png|500]]


问题

- bossGroup和workGroup中由几个子线程（NioEventLoop）

  默认数量是当前CUP数量的两倍

- NioEventLoop包含什么

  selector，taskQueue（任务队列），scheduleTaskQueue（定时任务队列）

- ChannelHandlerContext，Channel，Pipeline的关系

ChannelHandlerContext持有Channel和Pipeline的引用和其他各种组件的引用。ChannelHandlerContext还是一个双向链表

每一个Channel都包含一个Pipeline对象的引用，但是Channel和Pipeline互相持有对方的引用。Channel更注重读写操作，Pipeline更注处理客户端请求的业务逻辑。两者作用相似，但是各负责不同的部分
  
Channel持有对应的NioEventLoop的引用
  
Pipeline持有Handler（实际处理请求的处理器），taskQueue（普通任务队列），scheduleTaskQueue（定时任务队列）对象
  
- 一个浏览器向服务器请求多次，服务器的Handler处理多次请求。那么处理请求的CahnnelHandlerContext和Pipeline还是同一个对象吗？

Http协议不是长连接协议，所以多个Http请求到后端后，使用的Pipeline不再是同一个Pipeline。但是一个页面同时发送的多个请求到后端使用的Pipeline会是同一个Pipeline

举例：前端一个html中调用2次后端接口，后端处理这两次请求的Pipeline是同一个Pipeline	

但是刷新页面，重新发送两次请求，后端处理这两次请求的Pipeline将和上次的Pipeline不是同一个对象

