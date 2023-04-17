#还没有复习 

Zookeeper——分布式协同服务系统，也是系统级应用



# 课程内容概述

> - 第 1 部分：基础知识
>   - Zookeeper 基础
>   - Zookeeper 开发
>   - Zookeeper 运维
> - 第 2 部分：进阶知识
>   - Zookeeper 开发进阶
>   - 比较 Zookeeper、etcd 和 Chubby
>   - Zookeeper 实现原理和源码解读



- 第 1 部分：基础知识
  - Zookeeper 基础
    -  ZooKeeper的安装配置，ZooKeeper的基本概念和zkCli.sh的使用
  - Zookeeper 开发
    - Apache Curator 的使用和如何使用 Apache Curator 进行协同服务的开发
  - Zookeeper 运维
    - ZooKeeper生产环境的安裝配置和 ZooKeeper的监控。 ZooKeeper自带的监控工具和其他监控系统的集成
- 第 2 部分：进阶知识
  - Zookeeper 开发进阶
    - Zookeeper 实现一个服务发现的实战项目
    - Kafka 如何使用 Zookeeper
  - 比较 Zookeeper、etcd 和 Chubby
    - 对etcd和 Chubby做一个简单的介绍，并把他们和 ZooKeeper做一个比较
  - Zookeeper 实现原理和源码解读
    - 结合 ZooKeeper的源码讲解 ZooKeeper 设计实现原理
    - 阅读 ZooKeeper的核心模块源码
    - 相关的计算机理论知识点



此外还要学习

- 如何实际一个本地数据节点
- 分布式环境中节点之间如何通讯
- 如何从 0 到 1 设计一个 RPC 子系统
- 如何使用数据一致性协议保证数据的高可用
- 如何在数据一致性和系统性能之间做取舍



# zk 基础

> 以下使用`zk`代替`Zookeeper`



## 什么是 Zookeeper

**Zookeeper 的介绍**

ZooKeeper 是一个开源的分布式协同服务系统。 ZooKeeper 的设计目标是将那些复杂且容易出错的分布式协同服务封装起来，抽象出一个高效可靠的原语集，并以一系列简单的接口提供给集群中的节点使用。
被用于:分布式锁,节点选举等

**Zookeeper 的应用场景**

- 配置管理（configuration management）   比如微服务中水平节点使用的配置文件
- DNS 服务
- 组成员管理( group membership)     比如：HBase
- 各种分布式锁，分布式队列



- Hadoop：使用 ZooKeeper 做 Namenode 的高可用
- HBase：保证集群中只有一个 master，保存集群中的 RegionServer 列表，保存 hbase:meta 表的位置
- Kafka：集群成员管理，controller 节点选举



ZooKeeper 适用于存储和协同相关的关键数据,不适合用于大数据量存



## Zookeeper 的数据模型

Zookeeper 客户端负责和 Zookeeper 集群的交互

![Snipaste_2021-02-03_18-23-48](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-03_18-23-48.png)

### zk 的数据模型

zk 的数据模型是层次模型，层次模型常见于文件系统（主流的数据模型是层次模型和 vk 模型）

zk 使用层次模型的原因

- 文件系统的属性结构百年与表达数据之间的层次关系
- 树形结构便于为不同的应用分配独立的命名空间（namespace）
  - 比如 Dubbo 向 zk 中注册消费者时，节点名为 /dubbo/com.xx.xx.xxService/consummers | providers | routers | configurators                 其中包括了消费者，生产者，路由，配置等数据信息

zk 的层次模型被称作 data tree。树节点叫 znode，每个节点都能保存数据，每个节点都有一个版本号（从 0 开始计数）

一般的文件系统中，节点类型分为目录和文件。只有文件能保存数据。zk 中所有节点均能保存数据

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-03_18-37-04.png" alt="Snipaste_2021-02-03_18-37-04" style="zoom:67%;" />

上图是一个简单的成员协议集群管理示例。/app 下每个节点都代表一个进程，只要进程没死节点就存在，进程死，节点就被删除



### data tree 接口

data tree 接口
即操作一个 znode 时,不影响同时操作
另一

ZooKeeper对外提供一个用来访问 data tree的简化文件系统 API

- 使用 UNIX 风格的路径名定位 znode
- znde 的数据只支持全量写入和读取，没有像通用文件系统那样支持部分写入和读取
- data tree 的所有 API 都是 wait-free 的。正在执行中的 API 调用不会影响其他 API 的完成（即操作节点 A 时，不影响同时操作节点 B）
- zk 不直接提供锁这样的分布式协同机制。但 data tree 的 API 非常强大，可以用来实现多种分布式协同机制



### zk 的节点类型

- 持久节点（节点数据被持久化）
- 临时节点（客户端结束会话时，zk 集群会删除临时节点。临时节点不允许拥有子节点）
- 持久顺序节点（znode 名后追加一个 zk 维护的自增整数）
- 临时顺序节点


Zookeeper主要有以上4种。还有其他类型的 znode ，但是开发不常用。好像是 Econtent znode



## zk 实现分布式锁和 Master-Worker 协同



### 分布式锁

分布式锁要求：如果锁的持有者宕机，锁就可以被释放。zk 的临时节点恰好具备这样的特性

1. A 执行`create -e /lock`，成功创建节点 /lock（A 持有锁）
2. B 执行`create -e /lock`，结果节点已存在，创建失败（B 尝试获取锁，获取失败）
3. B 执行`stat -w /lock`，添加监听器（ -w 是添加监听的选项，stat 是获取节点状态的命令）
4. A 执行`quit`，临时节点 /lock 被删除
5. B 收到 /lock 节点被删除的事件
6. B 执行`create -w /lock`，成功创建节点 /lock（B持有锁）



### Master-Worker

需求：多个节点。一个 master，多个 worker。master 能监控其他 worker 的状态，master 宕机后 worker 能选出一个新的 master

- 首先，使用上述分布式锁实现多个节点中只有一个节点能当 master
- 每个节点的值为：主机名+与其他节点进行通信的端口号
- 创建的节点类型必须都是临时节点。这样能保证动态扩容
- 成功创建 /master 节点为 master
- /workers 下的子节点是 worker 。成功创建 /workers/workerXX 的节点为 worker



节点获取 master/worker 身份的流程

- 初始化集群/节点刚启动时，尝试创建 /master 获取 master 身份
  - 如果创建成功，说明获取到 master 身份。然后 master 节点 监听 /workers 节点（以便实时掌握 worker 节点的状态）
  - 如果创建失败，说明 master 已存在。那么就创建 /workers/workerXX 并监听 /master 节点（以便 master 宕机时再次尝试创建 /master）



## Zookeeper 总体架构

- zk 有两种模式：standalone 单机模式和 quorum 集群模式
- zk 客户端连接 zk 集群时会和集群中的某个节点建立一个 Session 
  - 客户端可主动关闭 Session
  - 服务端如果超过 Session 指定的时间还没收到过客户端的消息，Session 也会被关闭
  - 如果 zk 客户端和节点正常连接时节点突然宕机，zk 客户端会自动尝试和其他节点建立连接
- quorum 集群模式下。leader 节点可以处理读写请求，follower 只能处理读请求。follower 接收到写请求时会把写请求转发给 leader 来处理
  - leader 会以广播形式通知其他 follower 进行写操作，如果半数节点完成此写操作
    - 如果接收写请求的节点是 leader ，leader 会通知客户端写完成
    - 如果接收写请求的节点是 follower，leader 会通知此 follower 告诉它的客户端写完成
- zk 集群能保证数据一致性的原因
  - 全局可线性化写入：先到达 leader 的写请求会被先处理，leader 决定写请求的执行顺序
  - 客户端 FIFO 顺序：来自给定客户端的请求按照发送顺序执行



PS：搭建集群时，配置文件中`server.n=host:port1:port2`。n 是节点的 myid ，port1 是节点与其他节点通信的端口号，port2 是选举 master 时用于通票的通信端口号



# zk 开发



## zk Java API

zk Java 有 3 种：zk官方提供的（也就是下面使用的 API），Apache Curator（推荐使用。封装了 zk 官方提供的 API） ，开源的 zkClient（不推荐）



> zk 主要使用 org.apache.zookeeper.Zookeeper 这个类使用 zk服务

**org.apache.zookeeper.Zookeeper 的主要方法**

- create   创建节点
- delete   删除节点
- exists    判断节点是否存在
- getData   获取指定节点数据
- setData    修改指定节点数据
- getChildren    获取指定节点下的子节点列表
- sync    同步当前 session 连接的节点和 leader 节点的数据
- ACL （Access Control List）相关方法

详细 API 见[这里](https://zookeeper.apache.org/doc/r3.6.2/apidocs/zookeeper-server/index.html)



**方法说明**

- 所有 API 均可传递一个 watch 对象作为监听器
- 所有更新节点的 API 都提供有条件版本和无条件版本（version 为 -1 表示无条件更新，否则表示有条件更新）
- 所有方法都有同步和异步两个版本。同步版本的方法发送请求后同步等待响应，异步版本把请求放入客户端的请求队列中并马上返回。异步版本通过回调方法接收服务端的响应



**zk Java API 的同步方法中可能会抛出的异常**

- KeeperException 表示 zk 服务端出错
  - KeeperException 的子类 ConnectionLossException 表示客户端和当前连接的 zk 节点断开了连接
    - 网络分区和 zk 节点失败都会导致这个异常出现
    - 发生此异常的时机可能是在 zk 节点处理客户端请求之前，也可能是在 zk 节点处理客户端请求之后
    - 出现 ConnectionLossException 异常之后，客户端会进行自动重新连接，但是我们必须要检查以前客户端请求是否被成功执行过
- InterruptedException 表示方法被中断了。可以使用 Thread. interrupt()来中断 API 的执行。



具体用法见[这里](D:\workspace\Typora-workspace\Zookeeper.md)的 md 简单介绍



## 搭建 zk 源码环境

> zk 源码地址[在这里](https://github.com/apache/zookeeper)

关于 zk 版本问题

旧版本的 zk 使用的 build 工具是 Ant+Ivy，比如：3.5.5 和 3.5.6

新版本的 zk 使用的 build 工具是 maven，比如：3.6.1

因为 zk 3.6.1 里 zooekkper-recipes 下的子项目/模块都是空的，zk 3.5.5 里 zooekkper-recipes 提供了分布式锁，队列等类和测试类，所以使用 3.5.5



搭建过程见[这里](https://www.cnblogs.com/heyonggang/p/12123991.html)



## zk 实现分布式队列&分布式锁&选举

zk 3.5.5 中 zooekkper-recipes 项目下提供了分布式队列，分布式锁，分布式选举等的实现

上述 zk 提供的分布式队列，分布式锁，分布式选举等，都是 zk 给用户（或想要封装 zk Java API 以便提供更强大的 zk 客户端的用户）提供的工具类，可有可无。它们不是 zk 的核心代码。这也是 zk 新版本不再提供这些实现类的原因



### 分布式队列

> zk 中提供的分布式队列实现是  zooekkper-recipes  项目下的 org.apache.zookeeper.recipes.queue.DistributedQueue

**设计**

使用路径为 /queue 的 znode 下的节点表示队列中的元素。/queue 下的节点都是顺序持化 znode。这些 znode 名字的后缀数字表示了对应队列元素在队列中的位置。Znode 名字后缀数字越小，对应队列元素在队列中的位置越靠前。Recipe 说明：[Queues](https://zookeeper.apache.org/doc/r3.5.5/recipes.html)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-04_12-49-30.png" alt="Snipaste_2021-02-04_12-49-30" style="zoom:60%;" />

**AIP**

`org.apache.zookeeper.recipes.queue.DistributedQueue` 提供了队列的常用方法

- offer   入队
- element   获取队列中队首的元素，但是不出队
- remove   出队



### 分布式锁

> zk 提供的分布式实现是 zooekkper-recipes  项目下的 org.apache.zookeeper.recipes.lock.WriteLock
>
> 这个锁是公平锁

**设计**

当应用程序第一次尝试获取锁时，会向 zk 的某个分布式锁父节点下创建一个临时顺序节点。同时这个节点代表这个应用程序

`org.apache.zookeeper.recipes.lock.WriteLock`中分布式锁节点的的命名格式为`x-sessionId-serialNumber` （`x-会话号-序列号`，序列号是zk 创建顺序节点时维护的自增变量）



尝试获取锁：分布式锁父节点下序列号最小的节点代表持有分布式锁的节点，同时也代表节点对应的应用程序持有分布式锁

- 如果当前应用程序的节点的序列号是最小的，表示当前应用程序成功获取分布式锁
- 如果当前应用程序的节点的序列号不是最小的，表示获取锁失败。同时此应用程序监听它的前一个节点，以便前一个节点被删除时再次尝试获取锁

尝试释放锁：如果应用程序 A 现在持有锁，当 A 释放锁时会删除自己的节点

- A 在 zk 里对应的节点的后一个节点会尝试获取锁（因为非锁持有有者的节点会向前一个节点注册监听器）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-04_17-15-29.png" alt="Snipaste_2021-02-04_17-15-29" style="zoom:60%;" />



使用这种链式监听而不是所有节点都监听持有锁的节点的原因

- 为了避免羊群效应，每个锁请求者 watch 它前面的锁请求者。每次锁被释放，只会有一个锁请求者会被通知到。这样做还让锁的分配具有公平性，锁定的分配遵循先到先得的原则



`org.apache.zookeeper.recipes.lock.WriteLock` 对外 API 主要有

- lock    尝试获取锁（同步方法）
- unlock    释放锁（同步方法）
- isOwner    当前应用程序是否持有锁。持有锁就返回 true；否则返回 false
- getId    获取当前应用程序在 zk 中用于请求锁的节点的节点名（节点名格式之前提到过）



## 分布式选举

> zk 提供的分布式实现是 zooekkper-recipes  项目下的 org.apache.zookeeper.recipes.leader.LeaderElectionSupport

使用临时顺序 znode 来表示选举请求，创建最小后缀数字 znode 的选举请求成功。在协同设计上和分布式锁是一样的，不同之处在于具体实现。不同于分布式锁，选举的具体实现对选举的各个阶段做了监控

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-04_17-56-51.png" alt="Snipaste_2021-02-04_17-56-51" style="zoom:67%;" />



## Apache Curator

> Apache Curator 是 Apache ZooKeeper 的 Java 客户端库。Curator 项目的目标是简化 zk 客户端的使用
>
> Curator 为常见的分布式协同服务提供了高质量的实现
>
> PS：搭建 Apache Curator 源码时，只是用 maven 作为构建工具
>
> 官网[在这里](http://curator.apache.org/)

Curator 的组成部分

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-04_18-39-00.png" alt="Snipaste_2021-02-04_18-39-00" style="zoom:80%;" />

- Client：封装了 ZooKeeper 类，管理和 zk 集群的连接，并提供了重建连接机制（轻量封装了 zk Java API）
- Framework：为所有的 ZooKeeper 操作提供了重试机制，对外提供了一个Fluent 风格的 API （封装了 Client）
- Recipes：使用 framework 实现了大量的 zk 协同服务 （使用 Client 和 Framework 构建了 zk 协同服务工具，例如分布式队列，分布式锁，分布式选举）
- Extensions：扩展模块



- 初始化一个 client 分成两个步骤

  1. 创建 client
     - 创建 client 有两种方式。工厂模式创建和构造者模式创建
  2. 启动 client

- Curator 提供 Fluent 链式编程的 API，可以优雅地完成开启异步方法，创建监听等操作

- Curator 的 recipes 包下提供了这些现成的协同服务工具

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-04_18-47-48.png" alt="Snipaste_2021-02-04_18-47-48" style="zoom:80%;" />

# zk 运维

> - zk 生产环境的安装配置和 zk 的监控
> - zk 自带的监控工具和其他监控工具的集成



## 安装配置 zk 生产环境

zk 使用 log4j 做日志

log4j 的配置文件在 conf 文件夹下

建议使用 rollingfile 输出日志（将日志输出到文件中，而不是控制台）



## zk 监控命令

zk 提供了下列多种 4 字命令

可以通过 telnet 或 ncat （nc）使用客户端端口向 ZooKeeper 发出命令

比如：`echo rouk | nc localhost 2181`   `echo conf | nc localhost 2181`   `echo dump | nc localhost 2181` 

| 命令   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| `conf` | 输出相关服务配置的详细信息。比如端口号、`zk`数据以及日志配置路径、最大连接数，`session`超时、`serverId`等 |
| `cons` | 列出所有连接到这台服务器的客户端连接/会话的详细信息。包括"接收/发送"的包数量、`sessionId`、操作延迟、最后的操作执行等信息 |
| `crst` | 重置当前这台服务器所有连接/会话的统计信息                    |
| `dump` | 列出未经处理的会话和临时节点，这仅适用于领导者               |
| `envi` | 处理关于服务器的环境详细信息                                 |
| `ruok` | 测试服务是否处于正确运行状态。如果正常返回"`imok`"，否则返回空 |
| `stat` | 输出服务器的详细信息：接收/发送包数量、连接数、模式(`leader/follower`)、节点总数、延迟。所有客户端的列表 |
| `srst` | 重置`server`状态                                             |
| `wchs` | 列出服务器`watchers`的简洁信息：连接总数、`watching`节点总数和`watches`总数 |
| `wchc` | 通过session分组，列出watch的所有节点，它的输出是一个与`watch`相关的会话的节点信息，根据`watch`数量的不同，此操作可能会很昂贵（即影响服务器性能），请小心使用 |
| `mntr` | 列出集群的健康状态。包括"接收/发送"的包数量、操作延迟、当前服务模式(`leader/follower`)、节点总数、`watch`总数、临时节点总数 |



- zk 很好地支持了 JMX ，大量的监控和管理工作都可以使用 JMX 来做。比如：使用 jdk 提供的 jconsole 连接 zk 服务，在 MBean 面板中监控 zk 的状态



## zk Observer 观察者模式



Observer 和 ZooKeeper 机器其他节点唯一的交互是接收来自 leader 的 inform 消息，更新自己的本地存储，不参与提交和选举的投票过程。

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-05_11-49-13.png" alt="Snipaste_2021-02-05_11-49-13" style="zoom:60%;" />

PS：节点 1 是 observer，节点 2 是 leader

observer 主要用于解决

- 提高读性能
- 实现跨数据中心部署（当 zk 节点所在的服务器跨广域网时，写请求需要进行 propose ，accept ，commit 三次交互，如果将和 leader 不在同一广域网内的 follower 改为 boserver ，就能将 3 次交互变为 1 次 inform 交互）



## 动态配置实现不中断服务变更集群成员

- 使用动态配置需要执行集群的节点管理命令（不是集群的监控命令），这需要 super 用户权限。所以需要使用 Digest 认证配置一个 super 用户

  添加 super 用户的方式[看这里](D:\workspace\Typora-workspace\Zookeeper.md)

- 关于加密密码的方法，还可以使用 zk 提供的工具类

  ```java
  org.apache.zookeeper.server.auth.DigestAuthenticationProvider.generateDigest("super:yourPassword");
  ```

- 配置文件（将动态配置文件单独提取出来）

  ```sh
  # zoo.cfg
  reconfigEnabled=true
  dataDir=/data/zookeeper
  syncLimit=5
  initLimit=10
  dynamicConfigFile=config/dyn.cfg
  ```

  ```sh
  # dyn.cfg
  server.1=ali-1:2222:2223:participant;ali-1:2181
  server.2=ali-2:2222:2223:participant;ali-2:2181
  server.3=ali-3:2222:2223:participant;ali-3:2181
  ```



配置完成后，进入 zkCli.sh 客户端

- 登录 super 用户。`addauth digest super:yourPassword`
- 执行集群管理命令。如：`config`查看集群状态，`reconfig -remove 3`手动移除 server.3 节点



动态配置解决的问题

- 纯手动调整集群成员时，需要停止所有 zk 节点。动态配置不需要
- 纯手动调整集群成员时，在停止所有 zk 节点时，节点的数据可能存在不一致。重启集群后，如果数据少的 zk 被选为 leader 将会导致其他 follower 为和 leader 保证数据一致性会丢弃比 leader 多的数据，从而造成数据丢失



## zk 内部数据文件介绍



- zk 在内存中使用 data tree 存储节点数据

- zk 在磁盘中使用以下方式存储数据（文件地址由配置文件中的 dataDir 指定）

  - 事务日志文件。每一条日志文件都对应一个事务（每次更新 data tree 都会产生一个事务）

    - 使用 zkTxnLogToolkit.sh log.n 工具查看日志文件概述

  - 快照文件

    - zk 没有提供查看快照文件的脚本，但是提供了查看快照文件的工具类。所以可以自己写个脚本（以下示例脚本假设命名为 zkSnapshotFormatter.sh）

      ```sh
      #!/usr/bin/env bash
      . zkEnv.sh
      export CLASSPATH="$CLASSPATH"
      java org.apache.zookeeper.server.SnapshotFormatter "$@"
      ```

      执行`zkSnapshotFormatter.sh snapshot.x`即可查看格式化过的快照文件

    - 每次重启 zk 时都会生成快照文件

  - Epoch文件（ acceptedEpoch 文件，currentEpoch 文件）

    - 单机模式下，没有 Epoch 文件

PS：每一个对 ZooKeeper data tree 都会作为一个事务执行。每一个事务都有一个 zxid。zxid 是一个 64 位的整数（Java long 类型）。zxid 有两个组成部分，高 4 个字节保存的是 epoch ， 低 4 个字节保存的是 counter 。



# zk 开发进阶

> - zk 实现一个服务发现的实战项目
> - Kafka 如何使用 zk （视频内容缺失，暂时跳过）



## zk 实现服务发现

> 使用 Apache Curator 提供的 org.apache.curator.x.discovery 服务发现包实现一个服务发现



### 服务发现介绍

服务发现主要应用于微服务架构和分布式架构场景下。在这些场景下，一个服务通常需要松耦合的多个组件的协同才能完成。服务发现就是让组件发现相关的组件。服务发现要提供的功能有以下3点： 

- 服务注册
- 服务实例的获取
- 服务变化的通知机制。

Apache Curator 有一个扩展叫作 curator-x-discovery 。curator-x-discovery 基于 ZooKeeper 实现了服务发现



### curator-x-discovery 如何操作 zk 实现服务发现

- 使用一个 base path 作为整个服务发现的根目录
- 在这个根目录下是各个服务的目录
- 服务目录下面是服务实例
  - 实例是服务实例的 JSON 序列化数据。服务实例对应的 znode 节点可以根据需要设置成持久性、临时性和顺序性
  - 实例节点通常为临时性顺序节点。临时性保证了服务实例宕机后 zk 能删除节点，顺序节点保证了提供相同服务的实例能有一个顺序 ID

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-05_15-15-51.png" alt="Snipaste_2021-02-05_15-15-51" style="zoom:60%;" />

### Java 接口设计



#### 接口设计概括

curator-x-discovery 实现服务发现的三个主要接口

- ServiceProvider：在服务 cache 之上支持服务发现操作，封装了一些服务发现策略
- ServiceDiscovery：服务注册，也支持直接访问 ZooKeeper 的服务发现操作（这和 ServiceProvider 的功能重叠）
- ServiceCache：服务 cache

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-05_15-18-52.png" alt="Snipaste_2021-02-05_15-18-52" style="zoom:80%;" />



#### ServiceInstance

用来表示服务实例的 POJO，除了包含一些服务实例常用的成员之外，还提供一个 payload 成员让用户存自定义的信息

```java
private final String        name;
private final String        id;
private final String        address;
private final Integer       port;
private final Integer       sslPort;
private final T             payload; // 用于保存用户自定义信息的泛型变量
private final long          registrationTimeUTC;
private final ServiceType   serviceType;
private final UriSpec       uriSpec;
private final boolean       enabled;
```



#### ServiceDiscovery

接口提供了以下几类方法

- 服务注册，取消注册，更新服务实例
- 查询服务实例
- 创建 ServiceProviderBuilder 和 ServiceCacheBuilder 。它们都是构造者，用于构造 ServiceProvider 和 ServiceCache

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-05_15-25-14.png" alt="Snipaste_2021-02-05_15-25-14" style="zoom:80%;" />



#### ServiceProvider

接口提供了以下几种方法

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-05_15-31-05.png" alt="Snipaste_2021-02-05_15-31-05" style="zoom:80%;" />

ServiceProvider 提供服务发现 high-level API （即接口提供的方法已经足够全面，对象的声明类型为此接口类型即够使用）

客户端通常使用此接口实例来获取服务实例并调用实例提供的服务。如果服务实例出现异常，可使用 noteError 方法提醒 zk 处理出现异常的服务实例节点



ServiceProvider 是封装 ProviderStraegy 和 InstanceProvider 的 facade

- InstanceProvider 的数据来自一个服务 Cache 。服务 cache 是 ZooKeeper数据的一个本地 cache ，服务 cache 里面的数据可能会比 ZooKeeper 里面的数据旧一些
- ProviderStraegy 提供了三种策略： 轮询, 随机和 sticky 。

ServiceProvider 除了提供服务发现的方法( getInstance 和 getAllInstances )以外，还通过 noteError 提供了一个让服务使用者把服务使用情况反馈给 ServiceProvider 的机制。



#### ServiceCache

> Return the current list of instances. NOTE: there is no guarantee of freshness. This is merely the last known list of instances. However, the list is updated via a ZooKeeper watcher so it should be fresh within a window of a second or two.
>
> 译：返回当前的实例列表。注意：不能保证新鲜。这仅仅是已知的最后一个实例列表。然而，这个列表是通过 ZooKeeper 观察者更新的，所以它应该会在一两秒内刷新。

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-05_15-38-59.png" alt="Snipaste_2021-02-05_15-38-59" style="zoom:80%;" />



**总结**

ServiceDiscovery、ServiceCache、ServiceProvider 说明

- 都有一个对应的 builder。这些 builder 提供一个创建这三个类的 fluent API
- 在使用之前都要调用 start 方法
- 在使用之后都要调用 close 方法。close 方法只会释放自己创建的资源，不会释放上游关联的资源
  - 例如 ServiceDiscovery 的 close 方法不会去调用 CuratorFramework 的 close 方法



#### 一个简单的服务发现实例



ServiceDiscovery 提供的服务注册方法是对 znode 的更新操作，服务发现方法是 znode 的读取操作。同时它也是最核心的类，所有的服务发现操作都要从这个类开始。

另外服务 Cache 会接受来自 ZooKeeper 的更新通知，读取服务信息（也就是读取 znode 信息）

示例代码[见这里](D:\workspace\IDEA-workspace\source\apache-curator-4.2.0\curator-x-discovery\src\test\java\com\cx\curator\x\discovery\ServiceDiscovery.java)



### curator-x-discovery 服务发现内部核心代码实现



#### NodeCache

NodeCache 是 curator 的一个 recipe ，用来本地 cache 一个 znode 的数据

NodeCache 通过监控一个 znode 的 update / create / delete 事件来更新本地的 znode 数据。用户可以在 NodeCache 上面注册一个 listener 来获取 cache 更新的通知



#### PathCache

PathCache 和 NodeCache 一样，不同之处是在于 PathCache 缓存一个 znode 目录下所有子节点。



#### container 节点

container 节点是一种新引入的 znode ，目的在于下挂子节点。当一个 container 节点的所有子节点被删除之后，ZooKeeper 会删除掉这个 container 节点。服务发现的 base path 节点和服务节点就是 containe 节点

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_12-43-20.png" alt="Snipaste_2021-02-07_12-43-20" style="zoom:67%;" />



#### ServiceDiscoveryImpl

服务发现的实现类，并结合 NodeCache 实现节点缓存 



#### ServiceCacheImpl

ServiceCacheImpl 使用一个 PathChildrenCache 来维护一个 instances 。这个 instances 也是对 znode 数据的一个 cache 



#### ServiceProviderImpl

下述是 ServiceProviderImpl 的成员变量，所以这个类是多个对象的 facade （门面）

```java
private final ServiceCache<T> cache; // 提供 cache 功能
private final InstanceProvider<T> instanceProvider;  // 提供获取服务实例节点和过滤cache功能
private final ServiceDiscoveryImpl<T> discovery; // 提供服务发现和创建，修改cache功能
private final ProviderStrategy<T> providerStrategy;  // 提供获取所有服务实例节点功能
private final DownInstanceManager<T> downInstanceManager;  // 提供向 zk 通知节点出现异常功能
```



### curator-x-discovery-server 扩展

- curator-x-discovery-server 是基于 curator-x-discovery 实现的对外提供服务发现的 HTTP API
- curator-x-discovery 在系统质量和影响力和 ZooKeeper 相比还是有很大差距的，但是提供的服务发现的功能还是很完备的
- curator-x-discovery-server 本身实现的功能很少，不建议使用
- 建议以下的 SDK 使用优先顺序：curator recipes > curator framework > ZooKeeper API 



## Kafka

暂时跳过。B站缺失教程



# 问题

- https://www.bilibili.com/video/BV1XK4y1r7X3?p=9中涉及到了 Java 和classpath 啥的。看不懂（这是对Java命令行操作和Java环境变量配置的不了解）

- 单机模式下，使用 zkClient.sh 连接 zk服务端时连接成功后会报 SessionExpiredException 并尝试重连，重连成功。为什么第一次连上后都会重新连接一次

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-04_17-20-48.png" alt="Snipaste_2021-02-04_17-20-48" style="zoom:60%;" />

  使用的配置文件如下

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-04_17-21-32.png" alt="Snipaste_2021-02-04_17-21-32" style="zoom:33%;" />



# 比较 zk、etcd 和 Chubby



## 分布式一致性算法：Paxos

> Paxos 算法是一个一致性算法，作用是让 Asynchronous non-Byzantine Model 的分布式环境中的各个 agent 达成一致
>
> Asynchronous non-Byzantine Model （异步非拜占庭模型）
>
> 一个分布式环境由若干个 agent 组成，agent 之间通过传递消息进行通讯（分布式下不同主机间不能用共享内存通信，只能用网络协议通信）： 
>
> - agent 以任意的速度速度运行，agent 可能失败和重启。但是 agent 不会出 Byzantine fault
> - 消息需要任意长的时间进行传递，消息可能丢失，消息可能会重复。但是消息不会 corrupt （消息不会被别人篡改）

> **拜占庭将军问题**：是指拜占庭帝国军队的将军们必须全体一致的决定是否攻击某一支敌军。问题是这些将军在地理上是分隔开来的，只能依靠通讯员进行传递命令，但是通讯员中存在叛徒，它们可以篡改消息，叛徒可以欺骗某些将军采取进攻行动；促成一个不是所有将军都同意的决定，如当将军们不希望进攻时促成进攻行动；或者迷惑某些将军，使他们无法做出决定。
> Paxos算法的前提假设是不存在拜占庭将军问题，即：信道是安全的（信道可靠），发出的信号不会被篡改，因为Paxos算法是基于消息传递的。此问题由Lamport提出，它也是 Paxos算法的提出者。
> 从理论上来说，在分布式计算领域，试图在异步系统和不可靠信道上来达到一致性状态是不可能的。因此在对一致性的研究过程中，都往往假设信道是可靠的，而事实上，大多数系统都是部署在一个局域网中，因此消息被篡改的情况很罕见；另一方面，由于硬件和网络原因而造成的消息不完整问题，只需要一套简单的校验算法即可。因此，在实际工程中，可以假设所有的消息都是完整的，也就是没有被篡改。

分布式一致性问题原型

场景：

- 节点**N_1**和**N_2**上存放着数据X的拷贝
- 客户端**A**更新节点**N_1**上的数据**X**
- 一段时间之后，客户端**B**从节点**N_2**上读取数据**X**

客户端**B**从节点**N_2**上是否可以读取到客户端**A**在节点**N_1**上的数据更新取决于系统的实现，而这便是分布式一致性问题

> Paxos 算法参考博客
>
> - [Paxos最基本的思路](https://blog.csdn.net/weixin_44296862/article/details/95277801)（易懂。一次只能通过一个提议，这就是Basic Paxos，是Paxos中最基础的协议）
>   - [Paxos共识算法详解](https://segmentfault.com/a/1190000018844326)（更详细些。包括了 Basic Paxos 和 Multi-Paxos，还引用参考了其他博客，在此博客末尾有引用其链接）
> - https://blog.csdn.net/wolf_love666/article/details/92832811（弃用）

Paxos 算法的伪代码

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-20_10-20-31.png" alt="Snipaste_2021-02-20_10-20-31" style="zoom: 67%;" />

Paxos 算法的一个实例处理请求的时序图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-20_10-22-57.png" alt="Snipaste_2021-02-20_10-22-57" style="zoom:50%;" />

Leslie Lamport在论文《The Part-Time Parliament》中提出Paxos算法。由于论文使用故事的方式，没有使用数学证明，起初并没有得到重视。直到1998年该论文才被正式接受。后来2001年Lamport又重新组织了论文，发表了[《Paxos Made Simple》](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)。作为分布式系统领域的早期贡献者，Lamport获得了2013年图灵奖。



Paxos 一致性算法是理论实现，其工程实现有 Zab，Chubby，Raft



## 比较 Chubby 和 zk

> Chubby 是一个分布式锁系统，广泛应用于 Google 的基础架构中，例如知名的 GFS 和 Bigtable都用 Chubby 来做协同服务。
>
> ZooKeeper 借鉴了很多 Chubby 的设计思想，所以它们之间有很多相似之处

PS：Chubby 使用 Paxos 数据一致性协议，ZooKeeper 使用 Zab 数据一致性协议



## Raft 协议

> 参考博客[图解 Raft 算法](https://mp.weixin.qq.com/s?__biz=MzU3NDY4NzQwNQ==&mid=2247484187&idx=1&sn=1fd6e18c99ff845816c223c89f486312&chksm=fd2fd2d9ca585bcf664f61e3c1285b91ff8b7800aabc5e888dbf82cf611fe14c8283b277c98f&scene=21#wechat_redirect)，[Raft 一致性算法](https://blog.csdn.net/phantom_111/article/details/79095824)
>
> SpringCloud 的分布式注册中心 Consul 集群中就用了 Raft 这种分布式一致性算法





## etcd

> etcd 是一个高可用的分布式 KV 系统，可以用来实现各种分布式协同服务。etcd 采用的一致性算法是 raft，基于 Go 语言实现
>
> 为什么叫 etcd：etc来源于 UNIX 的 /etc 配置文件目录，d 代表 distributed system
>
> etcd 和 zk 覆盖基本一样的协同服务场景。zk 因为需要把所有的数据都要加载到内存，一般存储几百MB的数据。etcd使用bbolt存储引擎，可以处理几个GB的数据
>
> 典型应用场景：
>
> - Kubernetes 使用 etcd 来做服务发现和配置信息管理
> - Openstack 使用 etcd 来做配置管理和分布式锁
> - ROOK 使用 etcd 研发编排引擎
>
> etcd 持久化的数据使用 B+-tree 保存 ，在内存中的数据使用 B-tree 保存



# zk 实现原理和源码解读



## 存储数据结构



### B+-tree

B-tree和B+-tree



### LSM

Log Structured Merge-tree（LSM）是另外一种广泛使用的存储引擎数据结构

LSM架构图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-20_12-59-51.png" alt="Snipaste_2021-02-20_12-59-51" style="zoom: 67%;" />

> Multi-Version Concurrency Control 多版本并发控制，MVCC 是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问；在编程语言中实现事务内存

- LSM 写操作：一个写操作首先在日志中追加事务日志，然后把新的 key-value 更新到 Memtable。LSM 的事务是 WAL 日志

- LSM 读操作：在由 Memtable 和 SSTable 合并成的一个有序 KV 视图上进行 Key 值的查找

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-20_14-58-55.png" alt="Snipaste_2021-02-20_14-58-55" style="zoom:50%;" />

LSM 的优化手段

- LSM 还可使用布隆过滤器优化读操作，快速判断某条数据是否在库中

- 压缩手段（如果一直对 memtable 进行写入，memtable 就会一直增大直到超出服务器的内部限制。所以需要把 memtable 的内存数据放到 durable storage （外存）上去，生成 SSTable 文件，这叫做 minor compaction）

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-20_15-02-02.png" alt="Snipaste_2021-02-20_15-02-02" style="zoom:67%;" />



PS：Chrome 的 IndexedDB，Kafka Streams 使用的存储引擎都采用了 LSM



### 存储引擎的放大指标（Amplification Factors） 

- 读放大（read amplification）：一个查询涉及的外部存储读操作次数。如果查询一个数据需要做 3 次外部存储读取，那么读放大就是 3
- 写放大（write amplification）：写入外部存储设备的数据量和写入数据库的数据量的比率。如果我们对数据库写入了 10MB 数据，但是对外部存储设备写入了 20BM 数据，写放大就是 2
- 空间放大（space amplification）：数据库占用的外部存储量和数据库本身的数据量的比率。如果一个 10MB 的数据库占用了 100MB，那么空间放大就是 10。



## 本地存储技术



暂略



## zk 本地存储源码解析





## zk 的客户端网络通信源码



## zk 的服务端网络通信源码

