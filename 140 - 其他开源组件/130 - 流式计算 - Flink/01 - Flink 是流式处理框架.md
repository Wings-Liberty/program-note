Flink 是流式计算框架，其功能和使用方式很简单

# 它是怎么计算数据的？

1. 指定数据的来源，从来源处不断获取一条条数据
2. 让数据进入提前绘制好的流程（就像画流程图一样，不过绘制的方式是调用 API）
3. 把数据发送到目的地（比如数据库，MQ 等地方）

# 它提供什么功能？

- 实时计算，提供流处理和批处理（流处理和批处理能复用同一套代码，而 Spark 就不能，后者需要维护两套代码）
- 支持处理有界流和无界流（可以先对有界流做排序之类的行为后再进行后续计算）
- 提供状态机制 -- 能保存自定义变量
- 事件驱动 -- 来数据才处理，没来数据就进行低消耗方式空转
- 支持搭建集群，且管理集群方式多样化 -- 能集成 Yarn，K8s，或独立运行
- 支持容灾 -- 宕机后重启时能从宕机前的状态继续运行
- 直接提供精准一次（exactly-one）功能


# 它和 Spark 的区别是什么？


| 功能     | Flink                  | Spark                                      |
| -------- | ---------------------- | ------------------------------------------ |
| 流批处理 |流处理为根本。支持流处理和批处理|批处理为根本，支持微批处理，不支持流处理|
| 状态机制 | 支持                   | 不支持                                     |
| 精准一次 | 支持，通过修改配置即可 | 不支持                                     |
| 时间语义 | 提供事件时间和处理时间 | 只提供处理时间                             |
| 窗口     | 支持                   | 支持，但不够灵活（窗口必须是批次的整数倍） | 


# 它的分层 API 是什么意思？

Flink 提供了供用户调用的 API，但有的 API 类似写 sql 语句，有的 API 类似通过实现接口方法来自定义处理数据的逻辑

Flink 根据其提供的目的和实现方式对这些 API 进行了分层

| API 名称            | 提供的目的                      | 实现方式                     |
| ------------------- | ------------------------------- | ---------------------------- |
| SQL                 | 像写 sql 语句一样的调用         | 封装了 Table API             |
| Table API           | 像操作数据库表一样的 API        | 封装了底层 API 和 Stream API |
| Stream API          | 提供更易用的 API                | 封装了底层 API               |
| 底层 API - 处理函数 | 提供最底层 API 用于实现任何功能 | 有状态流处理                 |

越顶层的 API 越易用，但是越不灵活

抄作业：只调用 Stream API 和 底层 API

# Flink 集群

> 因为 Flink 的分区只有在集群模式下才会真正的做物理分区，所以学习阶段也需要搭建集群了

## 集群里都有什么节点组成

Flink 是 CS 架构，client 指定义并提交 job 的地方

server 端有两类节点：

- JobManager：用于管理 TaskManager 集群并解析 client 提交过来的 job 的地方
- TaskManager：用于执行 job，是真正“干活”的工作节点

![[../../020 - 附件文件夹/Pasted image 20230716121314.png|700]]

## 怎么启动 Fink

这里简单说一下启动单机 Flink 和集群 Flink

**单机版**

下载 `flink-1.17.0-bin-scala_2.12` 后，改一下配置文件 `conf/flink-conf.yaml`

```yaml
# 如果不改这个配置，就不能远程访问 flink web ui
rest.bind-address: 0.0.0.0
```

然后调用启动脚本即可

```bash
bash bin/start-cluster.sh
```

**集群版**

用 docker 方式启动，调用 `ssa_scrpit` 后输入命令

```
install flink_cluster
start flink_cluster
```

docker 配置方式参考 `ssa_scrpit` 的 `xxx_flink_cluster` 函数和 `docker-compose.yaml` 文件

## 部署模式

这里说的部署模式不是单机部署或者集群部署方案，而是 `JobManager` 和 `TaskManager` 的工作方式，不同的方式会影响 Flink 的如下部分

- 集群的生命周期以及资源的分配方式
- job 的 `main` 方法在哪执行（`Client` 还是 `TaskManager`）

会话模式，单作业模式，应用模式

#有待了解  暂时没看明白这三个模式的区别

> pre-job 模式有被弃用趋势，被 application job 模式取代

## 历史服务器

顾名思义，这个服务只用来记录并查看历史，防止Flink 挂掉后只能打开磁盘文件查看历史日志定位问题

开启历史服务器需要修改历史服务相关配置，然后调用脚本启动历史服务

#有待了解 暂无需求，先跳过去

# Flink 的运行时架构是什么样的？

之前说过了 Flink 有 `Client`，`JobManager` 和 `TaskManager`

现在说一下它们是怎么工作的

## 系统架构

![[../../020 - 附件文件夹/Pasted image 20230716123621.png|950]]

这里提到了几个新名词 `JobMaster`，`Task Slot`

**JobMaster**

`JobManager`（进程）收到客户端每提交一个 job 就会创建一个 `JobMaster` 线程去解析 job，根据其调用的算子之间的调用关系，将其解析图数据结构

**Task Slot**

`Slot` 是资源调度的最小单位，`slot` 的数量限制了 `TaskManager` 能够并行处理的任务数量

比如

1. `Client` 交了一个 job，里面有 3 个可以并行执行的算子，并且每个算子的并行度都是 3
2. `JobManager` 在请求 slot 时，请求到了 1 号 `TaskManager` 的 3 个 `slot`
3. 执行算子时，1 号 `TaskManager` 就会启动 3 个线程并行执行

如果第 2 步各请求到了 3 个 `TaskManager` 中的一个 `Slot`，那么就是 3 个 `TaskManager` 各启动一个线程执行算子

> `TaskManager` 之间是可以交换数据的，这是为了满足有些并行度只能是 1 的算子的功能，这种算子需要先收集各个并行算子返回的数据（需要多个 `TaskManager` 把数据传输到一个 `TaskManager` 后在执行并行度是 1 的算子）

## 核心概念


### 并行度，算子和子任务

当要处理的数据量非常大时，我们可以把一个算子 “复制” 多份到多个节点，数据来了之后就可以到其中任意一个执行。这样一来，一个算子任务就被拆分成了多个并行的“子任务”（subtasks），再将它们分发到不同节点，就真正实现了并行计算

在 `Flink` 执行过程中，每一个算子（operator）可以包含一个或多个子任务（operator subtask），这些子任务在不同的线程、不同的物理机或不同的容器中完全独立地执行

算子和子任务是不同阶段对同一段逻辑或代码的称呼

简单来说，调用 Flink API 时就是调用了一个算子，如下

```java
public static void main(String[] args) throws Exception {  
        // 1. 准备执行环境  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        // 2. 从 socket 里读取数据  
        DataStreamSource<String> source = env.socketTextStream("192.168.10.198", 7777);  
        // 3. 映射
        SingleOutputStreamOperator<Tuple2<String, Long>> flatMapOperator = source.flatMap((String line, Collector<Tuple2<String, Long>> collector) -> {  
            String[] words = line.split(" ");  
            for (String word : words) {  
                collector.collect(Tuple2.of(word, 1L));  
            }  
        }).returns(Types.TUPLE(Types.STRING, Types.LONG));  
        // 4. 分组
        KeyedStream<Tuple2<String, Long>, String> keyByOperator = flatMapOperator.keyBy(data -> data.f0);  
        // 5. 聚合
        SingleOutputStreamOperator<Tuple2<String, Long>> sumOperator = keyByOperator.sum(1);  
        // 6. 打印结果
        sumOperator.print();  
        // 7. 开始执行  
        env.execute();  
    }
```

上述代码分别调用了 `source`，`flatMap`，`sum`，`print` 算子

在 `Client` 提交 job  给 `JobManager` 时，`JobMaster` 会根据上述代码生成图数据结构

首先生成一个逻辑流图，里面包含了算子的调用关系，如下图（这是一个 word count demo）

![[../../020 - 附件文件夹/Pasted image 20230716135713.png]]

再根据逻辑流图（逻辑执行图）和其它信息生成 job 流图（物理执行图），比如说为了提高速度 `flatMap` 的并行度设为了 3

那么 `flatMap` 节点实际上会被分成 3 个并行节点，并行节点就被称为了“并行子任务”，这 3 个并行节点在逻辑流图里其实只有一个节点，且被称为“算子”

![[../../020 - 附件文件夹/Pasted image 20230716135924.png|700]]

每个算子的并行度可以继承执行环境的统一设置，也可以单独给算子设置并行度

```java
// 给执行环境设置并行度（一些并行度只能是 1 的算子不受这里的影响）
env.setParallelism(2);
// 给指定算子设置并行度
stream.map(word -> Tuple2.of(word, 1L)).setParallelism(2);
```

> 在开发环境（不用 flink server，直接用本地环境）中，没有配置文件，默认并行度就是当前机器的 CPU 核心数


> flink server 里有多个地方都能设置并行度：配置文件，命令行，代码

### 分区，分组

参考刚才提到的 job 实例，如下（这是一个 word count demo）

![[../../020 - 附件文件夹/Pasted image 20230716135713.png]]

一行字符串会在 `flatMap` 里被分割成多个单词并把每个单词转为 `Tuple2.of(word, 1L)` 对象

但是经过 keyBy 后，单词字面量相同的 `Tuple2` 对象会被放到一个地方，就像是把一个 `List<Tuple2>` 变成了一个 `Map<String, List<Tuple2>>`，数据就像是下面的步骤一样被处理

1. 接收一个字符串（socket source）并把字符串送给下一个算子
2. 收到一个字符串把字符串分割成多个单词，并把每个单词都放到算子的输出结果里（flatMap）
3. 根据单词字面量把对象放到不同的区里（keyBy）
4. 统计每个分区里的单词个数

也就是说一条数据会被放到不同的分区里处理。如果 socket 把数据先放到了分区1里，flatMap 从分区 1 里取数据，然后把结果放到分区 1 里。keyBy 会把数据分发到各个分区里，sum 会以分区为隔离单位，分别计算每个分区里的数据

也就是说算子之间传输数据的方式有两种

- 一对一（One-to-one，forwarding）
- 重分区（Redistributing），比如 keyBy

keyBy 会对数据进行分区，实际上这样说并不准确。分区是物理上的，分组是逻辑上的

case1：现在的并行度是 1，也就是只有一个 `Task Slot`，那么数据经过 keyBy 后相当于进入了一个线程的 `Map<String, List<Tuple2>>`

case2：现在的并行度是 10，也就是只有一个 `Task Slot`，那么数据经过 keyBy 后会把数据分散到 10 个 `Task Slot` 里（key 相同的在一个 `Slot` 里）

如果单词有 10 种，那么 case1 中，因为 key 有 10 中，所以分组数量有 10 个；因为 `Slot` 只有 1 个，所以物理存储空间也只有 1 个，分区数量也就只有 1 个

case2 中，因为 key 有 10 种，所以分组数量仍为 10 ；因为 `Slot` 有 10 个，所以物理存储空间有 10 个，分区数量也就有 10 个

### 算子链

引子问题：`Slot` 里有什么东西？

在说 Flink 运行时架构时第一次提到了 `Slot` ，它是调度资源的最小单位。提交一个 job 时，会根据生成物理流图向 `TaskManager` 申请 `Slot`

还是以 word count demo 为例

![[../../020 - 附件文件夹/Pasted image 20230716135713.png]]

如果 `socket` 和 `flatMap` 的并行度都是 2，那么会为 `socket` 申请 2 个 `Slot`，为 `flatMap` 申请 2 个 `Slot`

那 `socket[0]` 收到的数据会发送给 `flatMap[0]` 还是 `flatMap[1]` 呢？

如果 `socket[0]` 和 `flatMap[1]` 不在同一个 `TaskManager` 节点上，那就需要把 `socket` 的结果经过网络传输传给 `flatMap`

为了优化掉这个消耗，在提交 job 时把 `socket[0]` 和 `flatMap[0]` 这两个算子合并成一个**算子链**放到一个 `Slot` 里，这样  `socket[0]` 收到的数据就直接发给 `flatMap[0]` 了

结果就成了这样

![[../../020 - 附件文件夹/Pasted image 20230716143833.png|700]]

这样，这个数据流图所表示的作业最终会有5个任务，由5个线程并行执行。

将算子链接成 task 是非常有效的优化：可以减少线程之间的切换和基于缓存区的数据交换，在减少时延的同时提升吞吐量

Flink 默认会按照算子链的原则进行链接合并，如果我们想要禁止合并或者自行定义，也可以在代码中对算子做一些特定的设置：

```java
// 禁用算子链
.map(word -> Tuple2.of(word, 1L)).disableChaining();

// 从当前算子开始新链
.map(word -> Tuple2.of(word, 1L)).startNewChain()
```

### 任务槽和并行度是什么关系

假如一个 `TaskManager` 有三个 `slot`，那么它会将管理的内存平均分成三份，每个 `slot` 独自占据一份这样一来，在 `slot` 上执行一个子任务时，相当于划定了一块内存专款专用”，就不需要跟来自其他作业的任务去竞争内存资源了

所以现在只要 2 个 `TaskManager`，就可以并行处理分配好的 5 个任务了

![[../../020 - 附件文件夹/Pasted image 20230716152940.png|700]]

`conf/flink-conf.yaml` 配置文件中，可以设置 `TaskManager` 的 `slot` 数量，默认是 1

```
taskmanager.numberOfTaskSlots: 1
```

`slot` 目前仅仅用来隔离内存，不会涉及 CPU 的隔离。在具体应用时，可以将 `slot` 数量配置为机器的 CPU 核心数，尽量避免不同任务之间对 CPU 的竞争。这也是开发环境默认并行度设为机器 CPU 数量的原因

不同的任务可以共享同一个任务槽的共享，也就是说 `Slot` 里可以有多个算子的子任务，但是同一个 job 的并行算子子任务不能在同一个 `Slot` 里

Flink 默认是允许 `slot` 共享的，如果希望某个算子对应的任务完全独占一个 `slot`，或者只有某一部分算子共享 `slot`，我们也可以通过设置 “slot共享组” 手动指定：

```java
.map(word -> Tuple2.of(word, 1L)).slotSharingGroup("1");
```

## 逻辑流图/作业图/执行图/物理流图

之前简单说过逻辑流图和物理流图，但细分下来对 job 的图数据结构解析其实一共有 4 个阶段，每个阶段都有一个图数据结

![[../../020 - 附件文件夹/Pasted image 20230716154506.png]]![[../../020 - 附件文件夹/Pasted image 20230716154604.png]]

**逻辑流图（StreamGraph）**

这是根据用户通过 DataStream API 编写的代码生成的最初的 DAG 图，用来**表示程序的拓扑结构**。这一步一般在客户端完成。

**作业图（JobGraph）**

StreamGraph 经过优化后生成的就是作业图（JobGraph），这是提交给 `JobManager` 的数据结构，确定了当前作业中所有任务的划分

主要的优化为：将多个符合条件的节点链接在一起合并成一个任务节点，**形成算子链**，这样可以减少数据交换的消耗。JobGraph 一般也是在客户端生成的，在作业提交时传递给 `JobMaster`

我们提交作业之后，**Flink 自带的 Web UI 展示的就是作业图**

**执行图（ExecutionGraph）**

`JobMaster` 收到 JobGraph 后，会根据它来生成执行图（ExecutionGraph）。ExecutionGraph 是 JobGraph 的并行化版本，是调度层最核心的数据结构。与 JobGraph 最大的区别就是**按照并行度对并行子任务进行了拆分**，并明确了任务间数据传输的方式

**物理图（Physical Graph）**

**JobMaster** 生成执行图后，会**将它分发给 `TaskManager`**

**各个 `TaskManager` 会根据执行图部署任务，最终的物理执行过程也会形成一张“图”**，一般就叫作物理图（Physical Graph）。这只是具体执行层面的图，并不是一个具体的数据结构

物理图主要就是在执行图的基础上，进一步确定数据存放的位置和收发的具体方式。有了物理图，`TaskManager` 就可以对传递来的数据进行处理计算了



# 名词解释

- 任务，子任务：任务就是 job，是定义信息。子任务是算子，千万认为处理一条数据就是执行了1个任务|
- 时间
	- 时间时间：数据产生的时间
	- 处理时间：开始处理数据的时间
- 输入源和输出源