
Spark 是一种**基于内存的**快速、通用、可扩展的大数据分析计算引擎

Spark 比 Hadoop 的 Mapreduce 快

Hadoop MapReduce 由于其设计初衷并不是为了满足循环迭代式数据流处理，因此在多  
并行运行的数据可复用场景中**存在计算效率等问题**。因为 MR 不是基于内存的，多个 job 串联，一个job 的output 是另一个 job 的 input 时，就存在大量磁盘 IO

# Spark Core

03 Spark 核心模块


Spark MLlib 和 GraphX 和机器学习相关，目前不学



![Pasted image 20221001223103](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221001223103.png)

这是 scala 的 sdk 环境，需要去官网下载

这个 dmeo 项目时在 Java 项目里写的


05

讲解 word count 的过程，并输出了一个示意图，写了一个 **demo - wc**

06

Spark 有点像一个运行环境，应用程序写好一段 scala 代码后交给 spark 去运行



> [!NOTE] flatMap 扁平化操作
> 是把整体转为个体的操作，在 流式计算和正则表达式里都有体现


12

spark 如何运行指定的程序作为 job

在 Local 模式下

- bin/spark-shell 运行在命令行里写的 scala 代码
- 把 jar 包上传到服务器的 spark 里，用 bin/spark-submit 命令指定 jar 里的类和方法作为一个 job 注册到 spark

```bash
bin/spark-submit \  
--class org.apache.spark.examples.SparkPi \  # 类
--master local[2] \  # 环境，这里用 spark 的 master，local[2] 这个 2 的含义稍后解释
./examples/jars/spark-examples_2.12-3.0.0.jar \  # jar 包位置
10 # 方法参数
```

23

Executor 不保存 job 的逻辑，因为如果保存了就是写死了代码。所以是由 Driver 向 executor 计算逻辑 - job 和入参 


20-24 讲了一个 **demo - test**，讲了分布式计算中任务拆分和聚合的本质，讲的很好


Driver 有 Task，包含整个任务执行逻辑和全部参数

Task 会被拆分成 SubTask，SubTask 才是能发送给计算节点的任务对象，包含了 Task 的部分数据和整个计算逻辑

Spark 提供了一个比 demo 更完整更强大的模板代码，并提供 RDD，累加器，广播变量等数据结构做任务的拆分，计算和聚合


RDD 就是 demo 里的 Task，包含了所有逻辑和数据



25

RDD 是一个最小的计算单元，如果要定义一个复杂的计算流程，整个流程会被划分为多个子流程 - RDD，多个 RDD 串/并联起来就是一个完成的计算流程。所以 RDD 虽然不是最终发送给计算节点的 SubTask，但也是最小的计算单元

28

先讲解了一个 IO 示例，读取文件里的所有行，然后分词。用到了多层输入输出流的装饰器模式

文件流，缓冲流包文件流，字符流包缓冲流

然后讲了 demo - wc 中的 RDD，实际上 wc 的多个函数式接口调用就是每次调用函数式接口就创建一个 RDD 并装饰上一个 RDD。在最后调用 collect 方法后才开始执行，读数据，一个个 RDD 处理数据

PS：装饰器模式有时候看起来像链表，但通常这种理解是错误的。因为装饰器通常做的是增强，链式调用是其一小部分。而且链式调用和链表那种串联调用、责任链模式是不一样的


29

![Pasted image 20221002134836](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221002134836.png)

存储弹性：如果内存不够，数据放磁盘，而不是直接报内存溢出

容错弹性：如果 SeuTask 执行失败，Driver 还能知道应该从哪个文件，读哪些数据重新做一个 SubTask 给计算节点去执行（这个采用分区实现，比如用哈希算法）

对于  RDD ，接收到的数据就像是方法里的局部变量，只在计算是存在，当计算完后，把数据交给下一个 RDD，这些数据就没了，只有 output 被传递到了下一个 RDD

30

RDD 的核心属性

分区计算函数就是 job 的函数

首选位置函数是计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算

就近原则，首选位置函数会选择最合适的计算节点，把 SubTask 给他。就近原则指，如果数据离某个计算节点最近或就在它上面，那它就是首选的计算节点

35

所以创建 RDD 时就需要指定分区数量，默认分区数量 = CPU 核数



40

Q：算子是什么

A：RDD 的 Stream API 也叫算子，和 Java 的 Stream API 是一个东西，也分为中间操作和终结操作。不过在 scala 里叫转换算子和行动算子


93

每个 RDD 都保存有**前面的血缘关系链**

比如 RDD 链中，第 4 个 RDD 计算报错了。因为 RDD 不保存数据，所以不能让第 3 个RDD重新计算一次，而是让第一个 RDD 重新读数据并开始计算

这就需要第 4 个 RDD 知道第一个 RDD 是谁，所以每个 RDD 都需要保存**前面的血缘关系链**，这是为了提高系统容错性

![Pasted image 20221002150002](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221002150002.png)


94

RDD 之间的依赖关系有多种

有 One2One（窄依赖）的，有 Shuffle（宽依赖）的

- One2One 指 oldRDD 的一个分区的数据产生 out 后输入到 newRDD 的一个分区
- Shuffle 指 newRDD 一个分区的数据来自一个 oldRDD 的多个分区

oldRDD 是上游，newRDD 是下游

如果是 shuffle，需要划分阶段，第一个阶段的所有 task 完成后才能进行第二个阶段

![Pasted image 20221002152421](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221002152421.png)

Application（一个 scala 方法）->Job->Stage->Task 每一层都是 1 对 n 的关系

![Pasted image 20221002152619](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221002152619.png)


99

![Pasted image 20221002153537](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221002153537.png)

todo：学习 Spark Core 总结，课件梳理


# 工程化架构 - MVC 架构




# Spark Streaming

Spark Core 和 Spark SQL 的重点是离线数据，Spark Stream 是实时数据处理

![Pasted image 20221002155617](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221002155617.png)

Spark Streaming 是基于 Spark Core 的

Spark Streaming 用于流式数据的处理，但不是真正的流式处理，而是介于流式和批量之间的方式

![Pasted image 20221002155833](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221002155833.png)

比如公司对  syslog 的处理就是每分钟一次

Spark Streaming 的 “原语”，相当于 Spark Core 的 “算子”

被压机制：防止消费能力和数据收集能力不一致，用于进行自动调整的机制


- 2.1 Spark Streaming 的 WordCount 案例实操中，Stream 的数据来自端口
- 3.1 Spark Streaming 的 RDD 队列实操中，Stream 的数据来自 RDD，这时 RDD 作为输入的数据集合
- 3.2 自定义数据源

即 Spark Streaming 的输入数据可以来自多种数据源，如果想要让 FileBeat，LogStash，Kafka 等作为数据源，可以用官方 API 或自定义数据源


192

公司在 scala 里用了 Spark Streaming 提供的 Kfaka Direct API，也直接用了 Kafka 的客户端，redis 的客户端

貌似 scala 能直接写 Java 代码，用 Java 的 maven 依赖里的类，所以 kafka 和 redis 客户端都是 Java 的


193

状态转换。状态就是 Spark Streaming 的全局变量

举个例子：统计 "Hello" 的 wc

但目前 Spark 只能这样做：每隔一个时间周期统计一次数据，计算 “hello” 的 wc，但下次时间周期的统计结果和上次的完全独立，没有一个 sum 全局变量做累加记录


updateByKey 这个有状态的方法，pdf 里讲的不清楚，这里简单再讲下

假设 Spark Streaming 有一个大 Map，这个 map 保存了所有全局变量，key 就是全局变量名

如果要做全局 wc 统计，updateByKey 需要一个函数 A，函数需要两个参数

updateByKey 会对流种每个元素都执行一次函数 A

函数 A 需要两个参数

- 第一个参数是 stream 中的元素本身
- 第二个参数是 map 中的同名变量

即在这个函数里，能拿到元素本身和元素在 Spark Streaming 中的同名全局变量，然后对这个全局变量随便搞


全局变量其实是存在文件里，所以用 updateByKey 前需要创建一个检查点，检查点就是用来指定全局变量所属文件的目录位置。在启动程序前也需要提前声明从哪个检查点恢复数据，以便重启时能获取重启前的聚合计算结果


PS：抽时间再学习下 Java Stream 更多的 API

196

滑窗中包含若干的时间周期，一个时间周期就是 Spark Streaming 的采集周期

滑窗需要定义，滑窗会包含几个时间周期，以及滑窗每次向右移动时会右移几个时间周期的数据


199