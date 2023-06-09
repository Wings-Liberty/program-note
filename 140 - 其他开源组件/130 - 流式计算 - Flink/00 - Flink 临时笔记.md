#正在学习

# 进度

- [ ] 基础篇：基本概念，quick start，基本运行架构
- [ ] 核心篇：基本 API，时间语义，窗口
- [ ] 高级篇：处理函数，状态编程，容错机制
- [ ] SQL 篇：sql-client，查询 sql，connector，catalog，module

学习视频[来源](https://www.bilibili.com/video/BV1eg4y1V7AN/)

跳过部分

- P17-P21
- P25
- P27
- P32
- P80-P89 自定义水位线和水位线高级用法（跳过是因为已经学过水位线简单的 API，已经了解它是干什么用的了，而且态感目前没用水位线功能）
- P109-P130 容错（主要包含了检查点 -- 重启恢复上次停机时的运行时数据，精准一次消费 MQ）
- P131-P169 Flink SQL（目前算子已经够用了，而且态感里还没有用 SQL API）、

当前进度：已经看完了 P108，不再看了，如果有需求再补上上面跳过的内容

# Flink 概述



## quick start

1. 创建 idea 项目，导入 flink-1.17.0 依赖（flnk-stream-java, flink-clients）
2. 用 data set api 实现 word count

quick start 不需要搭建 flink server，直接在 demo 里写代码即可运行
	

> [!tip] 用 链式编程时，flink 的 api 用 lambda 有时候会报错，因为 lambda 的类型推断提供不了足够的信息，可能会导致运行报错。有两个方式解决问题
> - 用匿名内部类方式写
> - 链式调用 reture，显示传入类型对象
> ![[../../020 - 附件文件夹/Pasted image 20230623204500.png]]

> [!tip] IDEA 和 Maven 使用小技巧
> 生产环境下像 flink server 有的 flink-clients、flink-stream-java 这种已经有的依赖，在 Java 客户端项目的 pom 文件里设置成 scope provided 即可。他表示打包时不打进来。但这样在 IDEA 运行程序会导致找不到类，就需要在 IDEA 的 application configuration 里勾选运行时把 provided 也打包进来。至于为啥生产环境不把这些包打进来，我还不知道为啥

## flink server 作业提交方式

作业：写了一个方法就相当于定义了一个 job

**web-ui 上提交**

在 flink-web-ui 里提交 job 步骤如下

1. 项目打 jar 包
2. 在 flink-web-ui 上传 jar 包，并选择 job 所属的类，方法，方法入参，并行度等
3. 点击提交按钮

**命令行提交**

用的也不少

## 部署模式

pre-job 模式有被弃用趋势，被 application job 模式取代

# Flink 运行时架构

- jobmanager 和 jobmaster 的关系：前者是”老大“，是一个进程，后者是一个线程，前者可以只有多个后者的线程
- jobmaster 和 job 的关系：客户端提交了一个 job 后 jobmanager 就会创建一个对应的 jobmaster 线程
- 算子：客户端创建了一个 word count 的  job，job 读到数据后，第一步用空格分割出多个单词，下一步是根据 word 分组，下一步是对分组结果进行 sum 计算，下一步是打印结果。每一步都是一个算子
- 算子链：一对一和重分区的区别。在并行算子里，如果算子的输出结果给另一个算子时需要发送到指定的 task manager 的算子就是重分区。比如 word count 里，“hello” 的数量只有 1 号 task manager 知道，那么分组算子就得把统计 “hello” 的数量结果发给 1号 task manager
- 怎么区分哪些是一对一和重分区：在 web-ui 里能看到 job 的流程图，如果线条上有 forward 表示连接的两个算子是一对一的关系，其它基本都是重分区的关系
- 逻辑流图和job流图的区别：逻辑流图就是 flink 根据客户端调用算子的顺序生成的图结构，job流图是基于逻辑流图的优化，比如把多个一对一的算子合并成一个


# Stream API

获取 StreamExecutionEnvironment 的方式主要有以下三种

- createLocal 获取本地环境
- createRemote 获取远程集群环境
- getEnv 自动获取环境


- 调用转换算子 API 的时候，可以在后面 .set 手动指定并行度。但 keyby 这种聚合算子不能指定并行度。分区算子在 web-ui 上是节点之间的线条，而不是节点

## 聚合算子

分区和分组的区别

- keyby API 是对数据分区，保证相同的 key 在同一个分区
	- 分区：一个任务就是一个分区。数据被 keyby 后，让 key=aaa 的全部给到 1 号 taskmanager， key=bbb 的全部给到 2 号 taskmanager。这样就能保证 API 顺利执行，跟 mysql 的分库分表后需要有分区 key 一样。所以分区是物理上的把数据分开
	- 分组：根据一定的规则，把数据划分到多个组里。但这是逻辑上的划分，而不是物理上的。case：给每个对象上添加一个 groupNumber 表示它是第几组的，但是所有数据都还在一个集合里

分区和分组不用区分那么细。简单的说有时候根据业务需要一个分区里可能同时包含 key=aaa 和 bbb，但 ccc 在另一个分区里。这时候说分区和分组才有意义，aaa 和 bbb 属于不同的分组，因为它们的 key 不同，但它们却在同一个分区


一般 API 返回的 stream 是不能调用聚合算子的。只有 keyby 后返回的 stream 才能调用聚合算子

如果在 keyby 后调用了 reduce 方法，那么会对每个分组里的数据进行独立的 reduce


算子里的上下文对象：任务信息放在了上下文对象里，flink调用算子时需要传入一个接口实现类，通常可以传两种接口实现类，一种是常规的函数式接口，还有一种是 RichXxxxFunction，它提供了 getContext 方法，令其可以在自定义算子的逻辑里获取到上下文对象

aggregate 算子的 merge 方法好像只能用于会话窗口

## 分区算子

keyby 不能简单地认为它和 JDK8 的 Stream 的 groupbying 方法一样

分区算子的目的是按照指定方式把数据分散到不同的 taskmanager 节点处理

keyby 最好理解，因为它和 JDK8 的 Stream 的 groupbying 方法很像。每个 taskmanager 就像是 JDK8 的 Stream 的 groupbying 后得到的 `Map<String, List<Bean>>` 里的 list，keyby 算子根据返回的 key 把数据放到特定的 taskmanager 里

但还有这样的分区算子：把数据随机分散到每个 taskmanager 里（shuttle），或者轮询（reblance），缩放轮询（rescale），广播（boardcast）或者其它方式

**轮询算子**

轮询可以解决上游数据源不均匀问题（但并不是完美地解决），这个 job 有 3 个 taskmanager，上游数据源是 kafka 的 topic=aaa，aaa 有 3 个分区

理想情况下，应该有 3 个消费者消费这个 topic 的 3 个分区。但如果 topic 的 3 个分区里数据不均匀，第一个分区有 1000条数据，其它两个各有 10 条数据。那么轮询分区算子就能在一定程度上解决这个问题，执行分区算子的 taskmanager 里，有一个需要处理 1000 条数据，但它经过轮询算子后，把后续的计算任务轮询发送给其它 taskmanager 去计算了。所以轮询算子后续的计算能均匀分摊给 taskmanager 各个节点。

**缩放轮询算子**

轮询算子会无脑把数据轮询发送给所有 taskmanager，也就是说 taskmanager 的候选者是所有的 taskmanager

缩放轮询算子也是轮询，但每个 taskmanager 执行缩放轮询算子时，taskmanager 的候选者只有一部分 taskmanager。这样轮询效率会高效一些（我也不知道为啥这样就能高效了）

**广播算子**

他会把一条数据发给所有 taskmanager，这样就会让每个 taskmanager 都收到同一条数据，然后执行后续计算


## 分流

对同一个流对象调用多次算子就能实现分流。这个 JDK 的 stream 不一样，后者调用一次方法后 stream 对象就关闭并返回一个新的 stream，也就是说 JDK 的 stream 必须且只能链式调用，不能对同一个 stream 调用多次

但 flink 可以，对同一个 stream 调用多次算子，在 web-ui 里，就相当于让这个节点输出了多条分支，多条分支可以并行执行

通常来说需要并行分支的场景里并行分支都是带条件的，也就是说互斥并行分支在实际应用里更常见

但如果直接让一个 stream 调用多次算子，那就真的是创建了多个并行分支。多次调用 filter 实现带条件的并行分支的效率很差，因为他会让数据进入到所有的分支后再计算条件

侧输出流是真正的互斥并行分支

flink 其实并没有直接提供测输出流算子，只是方便称呼就起了这个名字。flink 的分层 API 里所有 API 最终都是调用了底层的 API，其核心是 `process` 方法

说白了，如果你想实现复杂的算子但没有现成的算子/API，那就调用最底层的 API process 去实现


Collector.output 的第一个参数是OutputTag，它表示测流的标签（也就是测流的名字）


## 合流

调用合流算子，表示数据在逻辑上合并到了一个流处理对象里

但是通常调用 connect 后还需要做 keyby，因为合流算子会把多条流里的数据在逻辑上合并到一条流里，但这不代表这些数据在物理角度上合并到了一条流里，因为在多并行度下，这些数据仍会被分散保存在不同节点里

比如现在调用 connect 合并两个流，这两个流相当于两张表，现在想在算子里实现 left jion 的效果

比如这两个流里的数据是

```
(1, "张三") // id，姓名
(2, "李四")
(3, "王五")

(1, "dog1", 1) // id，狗名，狗主的 id
(2, "dog2", 1)
(3, "dog3", 2)
(4, "dog4", 1)
(5, "dog5", 3)
(6, "dog6", 2)
```

算子逻辑是

```
// map1(Person person)
// 1. 把人保存到 personCache
// 2. 从 dogCache 里找这个人的狗，组装成一条数据 (人id，人名，狗id，狗名)
// 3. 输出数据

// map2(Dog dog)
// 1. 把狗保存到 dogCache
// 2. 从 personCache 里找到这个狗的狗主，组装成一条数据 (人id，人名，狗id，狗名)
// 3. 输出数据
```

如果多并行度下，调用 connect 后，没有调用分区算子，那么很有可能 `(1, "张三")` 放到 taskmanager1 里，他的狗全放到了 taskmanager2 里，导致最终 left join 的结果是张三没有狗


## 输出算子

输出算子也叫 sink，数据源算子叫 source

[Flink输出算子文档介绍](https://www.bilibili.com/video/BV1eg4y1V7AN?p=53&spm_id_from=pageDriver&vd_source=899c3b150c7fabce1cf14fe5edd7f23c)

**FileSink**

刚开始用输出算子的时候，把输出指向文件，文件名会带一个 inprocess 表示文件正在被写入且不能被读取（正在被写入且无法读取的文件，flink 会让这些文件名以 `.` 开头变成隐藏文件，注意不要因为没看见隐藏文件就以为flink没生效）

这是因为没有设置检查点，初学此处时，先根据教程写死设置一个检查点即可，后续再学习其含义和写法

**KafkaSink**

KafkaSink 的 setDeliveryGuarantee 和精准一次消费有关。它是用来设置不丢不重相关的东西

如果设置了精准一次，就必须开启检查点，设置事务超时时间，事务的名称否则启动报错。如果不是精准一次，比如最少一次，不用开检查点也行

如果要自定义序列化器，需要调用KafkaSink.setRecordserializer传入自行实现 KafkaRecordSerializationSchema


如果用了 JDBC 的算子，并且导入的 jdbc-connector 是 17 版本的，需要在 maven 的快照仓库下载，pom 文件里需要指定仓库地址，但是如果本地 maven 软件的 conf 里如果配置有镜像源，需要检查一下

通常导入阿里云 maven 仓库后会设置一个 * 表示所有依赖都从阿里云的仓库下载

```xml
<mirror>
	<id>aliyunmaven</id>
	<mirrorOf>*</mirrorOf>
	<name>阿里云公共仓库</name>
	<url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

需要把 * 改成这样

```xml
<mirror>
	<id>aliyunmaven</id>
	<mirrorOf>*,!apache-snapshots</mirrorOf>
	<name>阿里云公共仓库</name>
	<url>https://maven.aliyun.com/repository/public</url>
</mirror>
```


`!apache-snapshots` 表示排除这个仓库的依赖，意思就是所有的依赖从从阿里云下载，但是阿帕奇快照仓库的依赖除外

jdbc 输出算子有这么一个配置项

```
JdbcConnectionOptions.JdbcConnectionOptionsBuilder().withConnectionCheckTimeoutSeconds
```

MySQL 默认连接维护 8 小时，如果 8 小时后这条连接没有消息，MySQL 就会把它断开

flink 的 JdbcSink 里，每个子任务都会和 MySQL 建立一个连接，如果连接超时断开，flink 就会报错（这是个很严重的问题，因为不会自动重连，严重的话，整个 job 就崩了）

这个选项就是在设置每隔多长时间检查一次连接（相当于发送心跳包）

自定义输出算子比较费劲，就像自己实现一个通信工具就要自己实现client-server的通信协议，序列化方式，超时重传，失败重试。自定义算子比较费劲主要是因为 flink 是流处理不是批处理，一共来了 100 条数据，处理 50 条后服务器宕机，重启后需要从后50条开始处理。这要求自行实现跳过已经处理过的数据（这是一个数据一致性问题，就像文档中说的，自行保证数据一致性问题比较麻烦，所以非必要不适用自定义输出算子。source 也是，不推荐自定义）

# 窗口

举个应用场景的例子：需要统计 flink 每小时计算了多少数据，就需要以小时为单位划分时间段

有点滑窗的意思，但是课程里专门强调窗口不是滑窗，而是水桶，数据就像水龙头里不断流出的水

时间窗口：水桶无限大，每隔一段固定的时间长度就换一个桶接水

计数窗口：水桶的大小固定，只能装 10 升水，装满就换一个桶


会话窗口只有时间窗口下有，计数窗口没有会话窗口

全局窗口，是计数窗口底层的窗口，计数窗口本质上都是调用全局窗口的 API 实现的，所以只有在需要自定义计数窗口的时候才会用全局窗口

## 窗口数据计算方式

计算窗口里的数据时，分两种方式

- 增量聚合：来一条数据计算一条数据，窗口触发的时候输出计算结果
- 全窗口：数据来了不计算，存起来，窗口触发的时候再计算并输出计算结果

上面的来一条计算一条或存起来一块计算说的都是窗口的逻辑，而不是数据被处理的逻辑

比如说窗口前后调用了 map 算子，map 算子仍是来一条计算一条，因为 flink 是流式计算，所以这点没有变

只是窗口本身的聚合逻辑何时触发是一个可选项

会话窗口和时间有关，所以只有时间窗口有。如果数据来了就开一个会话窗口，一直处理数据，突然 n 秒内一条数据都没有了，就会结束会话窗口，直到新数据来。所以会话窗口的边界是：如果来数据就开窗，如果 n 秒内窗不需要处理数据就关窗

动态窗口 dynamicGap：它是会话窗口的增强，会话窗口定义时就指定了窗口空转断开的时间，动态窗口每次的等待断开时间是不断变化的，每收到一条数据就可以由用户自行处理数据，然后返回一个秒数作为等待断开时间（对是每条数据都会更新断开时间，而不是开窗的第一条数据定的等待断开时间）


计数窗口的滑动方式里，如果窗口大小是 5 ，步长是 2，那么第一组窗口是在收到 2 条数据后关闭的。如果觉得这种现象很别扭，可以这样理解，计数窗口的滑动方式相当于：收到 2 条（步长）就开一个窗。刚启动时，收到两条，就开一个窗。这个新开的窗是第二个窗，不是第一个，否则前 2 条数据就不在窗里，所以，实际上第一个窗是在收数据前就创建的。并且他的范围是 \[-3, 2\]，第二个窗口的范围是 \[-1, 4\]。负数是开始接受数据之前，所以负数那段并没有收到数据

![[../../020 - 附件文件夹/Pasted image 20230630224256.png]]

注意调用了 window 方法后可以调用 reduce，process 等方法，但这些方法都是 WindowSource 的，不是普通的 Source 的，所以这些方法都是窗口的方法，和之前的方法同名但是不一样

如果设置时间滚动窗口的间隔是 10s，那么所有区间都是是整 10s 的倍数（0~10，10~20 ... 50~60s），而不是第一条数据来的时间。这个逻辑可以在时间滚动窗口的 assignWindows 方法里看见计算 startTime 的逻辑

WindowSource 的 reduce 方法的逻辑如下

1. 入参：上一个数据，下一条数据
2. 返回结果：一条数据

要求这三个对象的类型必须都一样

flink 称这 3 个对象为 输入对象，累加对象，输出对象

aggregate 和 reduce 相比这 3 个对象的类型可以都不一样，所以前者和后者相比，前者更加通用


## 全窗口和增量聚合

增量聚合相当于来一条计算一条，全窗口是攒够一个窗口后才开始算

后者能实现前者做不到的事，但后者需要长时间保存大量数据


aggregate 重载方法能同时接受增量聚合方法和全窗口方法两个函数接口对象

其实现方式是：来一条数据，增量聚合方法就处理一条数据，然后把结果给全窗口函数。直到一个窗口结束，全窗口函数就算完了所有数据，然后调用一个方法输出结果，这时也能拿到上下文对象。所以这种方式可以同时满足两种要求，又不需要为全窗口函数保存数据

传入的全窗口方法被调用次数和数据的数量一样，其参数 iterator 里其实每次只有一条数据，且每次输出都会把上次输出覆盖，等窗口结束后输出一条数据。所以不会再保存大量数据

这本质上不就是在增量函数和一个窗口的前后加了切面吗？

> reduce 也可以这样，传两个参数

全窗口的 process 好像强制并行度改成了 1，flink 自动实现把数据都给一个节点里（好像是这样）

## 触发器和分配器

之前并没有提到过这两个东西，因为在调用窗口的 API 时并不需要传入这两个对象，这是因为用了内置的，如果内置的触发器和分配器不满足需求，就可以自定义

这两个是干什么的：

以基于时间的窗口为例

窗口什么时候触发输出？如果是滚动窗口，窗口触发输出的时间就是开始时间 + 滚动窗口的时间间隔

可以翻翻设置指定类型窗口时，设置了滚动窗口时传入的类，他里面就是创建了内置的触发器

移除器，窗口结束时用来清除窗口数据的，不用关心，一般用默认实现

> 在触发器里有枚举类，有个枚举类实例的名字叫 FIRE，在 netty 等框架里也都有方法或类有 fire 字样，它表示开火，也表示开始，开始处理，确认的意思


# 时间语义

事件时间：数据产生的时间

处理时间：开始处理数据的时间


结果准确性：如果没有明确的事件时间定义，那么接收数据的时间是今天 23:59:59，计算完数据的时间是 00:00:01，那这条数据算是今天计算的还是明天计算的（暂时不知道区分这个有什么意义）

通常业务数据按时间归档时用事件时间，因为事件时间更接近用户的操作时间，而处理时间 = 事件时间 + 延迟时间


> 课程里又提到了前端埋点这个词


# 水位线

水位线和事件时间相关

可以先认为水位线就是逻辑时钟，逻辑时钟的时间推进是靠数据里的时间戳，所以水位线表示的时间只能向后推进不能向前回退

也就是说在时间滚动窗口里如果数据携带的时间戳在窗口内，突然又没数据了，逻辑时钟就不会向后推动，窗口也就不会关闭

时间滚动窗口设置为 10s，如果收到了数据里携带的时间戳是  10s 的数据，窗口就会关闭。这样的话，迟到的数据，比如发送时间是 9s，但是在第 11s 才到 flink 的。为了能正常接受这些迟到的数据，需要让窗口晚点关。所以可以给水位线设置迟到时间，但 flink 不是通过让窗口晚点关闭实现接受迟到数据的，而是让 flink 把时间减去几秒实现的。比如水位线设置迟到时间是 2s。flink 会让这个水位线自以为这样一件事：现在实际时间是第 6s，但水位线会认为现在是第 4s

水位线会减去几秒，但时间滚动窗口不会因为 12s 的数据被 -2 后就归到 0-10s 的窗口，它还是会正常算到 10-20s 的窗口

而这个被推迟的时间就是由水位线记录的

一旦水位线真的到了时间滚动窗口边界，关掉了窗口。那么窗口就不再计算这些迟到的数据，因为这条数据的窗口已经被关了（当然普通算子仍能计算这条数据，这里说的“丢弃”只是指这条数据不进入窗口的计算逻辑了）


虽然教程里根据有序和乱序区分了不同的水位线策略，但实际上即使是单个客户端也会是乱序到达（早发的数据晚到）。升序和乱序的默认获取方法其实调的时同一个方法只是前者的等待关闭时间是 0

> 根据之前学下来的东西，基本可以确定，水位线是从数据的时间戳里取的，时间戳的值是由用户自行指定的。这个值是用户在数据里自行添加的时间信息

水位线策略由两部分组成

- 增加方式：比如单调递增（只有在遇到更大的时间戳后才会更新水位线）
- 时间提取方式：需要实现一个函数式接口，方法提供两个参数 -- 数据的类型（用户自定义的数据的 POJO 类型），时间戳（默认值一直是 long 的最小值，通常也不会用）

水位线可以是每来一条数据就更新，也可以是各一段时间再更新

这个隔一段时间可以用 env 对象设置，`env.getConfig().setAutoWatermarkInterval()`


# 定时器

richfunction 和 processFunction 都能拿到上下文对象

上下文对象能拿到定时服务

拿到定时服务后就能注册简单的定时任务：设置在第 n s 时触发定时器执行一段自定义逻辑

这个时间可以是事件时间

如果调用上下文获取事件事件，会从水位线策略里获取，如果没有设置水位线策略，那么上下文就拿不到事件时间

只能拿处理时间（就是系统时间 `System.getCurrentMillions()`）

需要注意，水位线的值 = 设置的水位线的值 - 延迟关闭时间 - 1ms。所以有时会因为多减了 1ms 导致定时器没触发

多次注册同一个时间的定时器，最终只会有一个定时器（如果是 keyByStream，则是每个 key 下，同一个时间只能有一个定时器）

> 也提供了处理时间的定时器服务，删除定时器 API

通过定时器的 API 获取到的水位线的值是上一个数据/上一次生成的水位线。所以计算第一条数据时，水位线的值是 long 的最小值，计算第二条数据时，水位线的值是 上一条数据的事件事件 - 延迟关闭事件 - 1ms

原因：

- 链式调用设置水位线和其它算子的时候，生成水位线也被画到了流程里，在调用设置水位线前的算子获取不到水位线
- 生成的水位线对象也会作为数据在流里向下传递，第一条数据生成水位线后，第一条数据送到下一个算子，它的水位线也送到下一个算子，在下一个算子处理第一条数据时，水位线还没有送到，所以水位线没更新（这是课里讲的，但是我理解不了，这tm什么逻辑）

# 处理函数


ProcessFunction 是最底层的 API，之前在分流，合流那用过

处理函数被细分为 8 类，不用专门背


# 状态机制

flink 提供两种状态机制：托管状态，原始状态。只用前者，后者不需要学

经过 keybby 或其它方式分区后的状态叫 “键控状态” 或 “按键分区状态”，其它统称 “算子状态”

之前需要“状态”时，做法通常是在调用算子需要传入接口实现类对象时，自行创建实现类，并在类里用成员变量的方式自行维护“状态”。这种做法的缺点是没入库，不能容灾恢复，程序停止后里面的数据就全丢了

键控状态是根据 key 隔离的，因为多个 key 的可能在同一个物理分区里

状态的初始化必须在算子的 open 方法里。因为初始化状态变量需要上下文，只有在 open 方法里才能拿到上下文

状态变量的 API 有点像 ThreadLocal

> print 方法貌似就是把之前收集器收集的对象和字符串打印出来


按键分区状态只能用在 keyby 后的算子里（教程里没在分区前用过，所以我也不知道按键分区状态能不能用在 keyby 前的算子里）


归约状态，就是在状态里实现 reduce 函数
聚合状态，就是在状态里实现 aggregate 函数


状态的 TTL 清理方式和 GC 有点像，当状态到达 TTL 后不会立刻被清理，而是被打上标记，等到清理阶段一并清理掉被标记的状态。所以创建状态的 TTL 设置时有设置可见性这个选项


算子状态里列表状态和联合列表状态都是 List，区别在于：如果修改了 job 或算子的并行度，那么就需要重新分配状态的物理分区。这两个状态的分配方式不同

联合列表用的不多

广播状态需要另一条广播流向主流（数据流）把广播状态传播到所有分区

数据流里只能读广播状态，不能改

广播状态可以这样用：把一些配置信息放到数据库里，然后用 JDBC 作为数据源创建广播流，流中读到会被实时更新的配置信息，然后更新到广播状态，数据流就能实时获取到被更新的配置

状态后端：说白了就是状态的配置文件。默认用哈希表状态后端 

