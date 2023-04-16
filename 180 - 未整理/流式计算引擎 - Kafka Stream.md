
# Kafka 概述

Kafka 是一个分布式的**基于发布/订阅模式的消息队列**（Message Queue），主要应用于大数据实时处理领域


# 消息队列的两种模式

**点对点模式**（一对一，消费者主动拉取数据，消息收到后消息就被删除）

![Pasted image 20220912104251](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912104251.png)


**发布/订阅模式**（一对多，消费者消费数据之后不会清除消息）

也叫广播模式，消息生产者（发布）将消息发布到 topic 中，同时有多个消息消费者（订阅）消费该消息，发布到 topic 的消息会被所有订阅者消费


![Pasted image 20220912104409](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912104409.png)


# Kafka 基础架构

![Pasted image 20220912104707](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912104707.png)

主要包含：生产者，消费者，Kafka 集群，zk（Kafka 2.8.0 及其以后的版本不在用 zk 作为注册中心）

## 基础架构的重要组成部分

- Producer：消息生产者，是向 broker 发消息的客户端

- Consumer：消息消费者，是向 broker 取消息的客户端

- Consumer Group（CG）： 消费者组，由**多个 consumer 组成**，所有的消费者都属于某个消费者组，**逻辑上是一个订阅者**。消费者组内**每个消费者负责消费 Topic 不同分区的数据**，一个分区只能由一个组内消费者消费；消费者组之间互不影响

- Broker：一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker 可以容纳多个 topic

- Topic：逻辑上一个 Topic 是一个队列， **生产者和消费者面向的都是一个 Topic**

- Partition：分区。为了实现扩展性，一个非常大的 topic 可以分布到多个 broker（即服务器）上，**一个 topic 可以分为多个 partition**，**每个 partition 是一个有序的队列**

- Replica：**分区的副本**，集群中节点发生故障时，副本顶上。一个 topic 的每个分区都有若干个副本，副本里也有 leader 和 follower 之分


## 消费者组，主题和分区

现在 broker 有一个 TopicA，但如果**生产者很多，一个消费者就消费不过来**

所以 **TopicA 可以被多个消费者消费**，这些消费者消费同一个 Topic，所以逻辑上属于同一个**消费者组**

多个消费者消费同一个队列的并发性能不高，因此，可以把一个 Topic 拆分成多个**分区**，每个分区保存一部分数据，消费者组每个消费者消费一些分区，防止消息积压，这一切看起来都很合理

![Pasted image 20220912143802](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912143802.png)

Topic 是逻辑上的概念，而 **Partition 是物理上的概念**，**每个 Partition 对应于一个 log 文件**，该 log 文件中存储的就是 Producer 生产的数据

消费者组中的每个消费者，都会实时记录自己消费到了哪个 offset，以便出错恢复时，从上次的位置继续消费


![Pasted image 20220912144239](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912144239.png)


逻辑上每个 Paration 有一个 log，物理上会把一个 log 进行分片，这样能防止 log 文件过大不好检索



## 文件存储机制


生产者生产的**消息会被顺序追加到 log**，为防止 log 文件太大，实现快读定位数据，Kafka 采用分片和索引机制，将每个 partition 分为多个 segment

每个 segment 对应两个文件 “.index” 文件和 “.log” 文件

这些文件位于一个文件夹下， 该文件夹的命名规则为：`topic 名称 + 分区序号`

例如， first 这个 topic 有三个分区，则其对应的文件夹为 `first-0`, `first-1`, `first-2`

“.index”文件存储大量的索引信息，“.log”文件存储大量的数据，索引文件中的元  
数据指向对应数据文件中 message 的物理偏移地址

![Pasted image 20220912144631](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912144631.png)


Q：如何根据 offset 找 message？

A：先找到其所属的 segement，再从 index 文件里找对应的 offset，因为 index 文件里的 offset 是递增的，且每条数据定长，所以能很快找到。找到 offset 后就能找到物理地址，在 log 文件里根据这个物理地址找 message 即可

所以这个查找过程很快


## 分区备份机制

防止分区宕机，有需要给每个分区进行备份

![Pasted image 20220712172707](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220712172707.png)

正常情况下消费者和生产者对消息的**生产和消费都只针对作为 leader 的分区**，**follower 作为副本自行复制 leader 里的消息**。**leader 宕机后 follower 顶上**

老版 Kafka 用 zk 完成**主从分区的选举**，元数据存储等任务


![Pasted image 20220712173038](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220712173038.png)



> [!NOTE] Kafka 是多语言开发的
> Kafka 的生产者和消费者是 Java 写的，Broker 是 Scala 写的


# Kafka 集群注意事项


## 关闭集群的正确方式

- 在用 zk 管理集群的情况下，kafka 关机前会操作 zk。所以关闭整个集群时，先关 kafka 再关 zk

否则 kafka 会因为关机时找不到 zk 而导致一些错误，比如不能即时把自己在 zk 的状态改为关机/下线


## 脚本连接集群的正确方式

- kafka 由很多部分组成，为了独立控制各个部分，kafka 提供了很多脚本

用脚本指定 kafka 地址时，可以只指定一个 a，如果 a 没宕机，执行的命令也能用 a 获取到整个集群的数据，如果 a 宕机，脚本就报错了。所以生产环境下用脚本时会指定多个 kafka 的地址，测试开发环境指定一个就够了

![Pasted image 20220912110924](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912110924.png)

生产者和消费者的控制台方式生产和消费者脚本仅用于测试，一般情况下还是用客户端实现生产者和消费者

## 管理分区数量的正确方式

- **分区数只能增加不能减少**。因为多个分区组成一个 topic，每个分区都保存了有效数据而不是副本。所以不能减少，但理论上当某些分区里没有消息时就可以被安全地删除，但不能用命令行执行这种减少分区的行为


分区数量需要在声明时就定义好。如果后增加或减少了分区数量或直接新增或减少 broker 节点导致分区数量减少，就需要做数据迁移。不能用 create 或 alter 命令修改已存在的主题的副本数

不管是希望减少分区还是添加分区，数据迁移的操作方式简单来说就是，编写 json 文件，用 json 文件生成数据迁移计划，执行数据迁移计划


## 组件管理和处理消息的默认参数

- kafka 拦截器用的不多

- kafka 的 kv 序列化器是自行实现的，不需要自己再实现

- kafka 的数据的校验是很简单的，因为这些工作通常由 hadoop 做

- 生产者的双端队列是双端队列组，每个队列映射一个分区

- linger.ms **默认值为 0**，**所以生产一条消息就向 topic 发送一条，此时 batch size 会失效**

- Sender 和 im 的发送端一样需要进行消息确认机制，失败重发，幂等。sender 的等待 ack 的滑窗大小默认是 5，如果滑窗满，就必须等收到 ack 后能发第 6 个消息

- 消息以 kv 格式发送，通常都是 String，且 k 为空.key。key 可以用来做分区策略

- 分区后，生产者可以多线程发送，topic 的某个分区宕机不会引起整个 topic 的宕机


## 如何提高生产者吞吐量

- 修改 linger 和 batch size

- 开启压缩，提高每个批次能容纳的消息数量



# Producer 的数据可靠性保证


生产者对数据的可靠性保证指：生产者生产的消息被希望发送到指定的分区并感知消息被 Broker 接收（ACK）


这需要**消息应答**和**超时重传**保证。如果是集群，就涉及到 **leader 和 follower 数据同步**和**何时发送 ACK** 问题

![Pasted image 20220912145238](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912145238.png)


## 副本数据同步策略

- 半数以上完成同步，就发送 ack。延迟低

- 全部完成同步，才发送 ack。延迟高

Kafka 用方案二，原因是方案二需要的副本数量少

举个例子

如果有 1 个 leader，3 个副本。当 leader 宕机，重新选举 leader 时，方案一只容许 1 个节点故障，方案二容许 2 个节点故障



> [!NOTE] 半数以上投票选举策略
> 半数以上投票选举策略中，基数是写死的，是集群启动时集群中预期的健康节点总数。半数是基数的一半，在集群启动后半数就是常量了


## Leader 选举

采用的方案二后，想象以下情景：

1. leader 收到数据，所有 follower 都开始同步数据
2. 但有一个 follower，因为某种故障，迟迟不能与 leader 进行同步
3. 那 leader 就要一直等下去，直到它完成同步，才能发送 ack。这个问题怎么解决呢？


Leader 维护了一个动态的 in-sync replica set (**ISR**)，意为**和 leader 保持同步的 follower 集合**

![Pasted image 20220712223522](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220712223522.png)

ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。如果 follower 长时间未向 leader 同步数据 ，则该 follower 将被踢出 ISR

这个时间阈值由 `replica.lag.time.max.ms` 参数设定。 Leader 发生故障之后，就会从 ISR 中选举新的 leader

如果宕机的 leader 又重新上线，自行变成 follower，并被放在 isr 的最后一个位置，但选 leader 是按照 AR 里的顺序来的，AR 里的顺序是固定的

## 消息应答策略

就是 ACK 机制

ack = 0。表示不开启消息应答。生产者不等待 ACK，不重发消息。这种策略可靠性差，生产者不知道消息是否落盘，不知道该不该重发消息

ack = 1。开启消息应答。生产者等待 Leader 把消息落盘后返回的 ACK

ack = -1。开启消息应答，Leader 会等到 Follower 都完成同步后才会发送 ack


Q：但是如果**在 follower 同步完成后， Broker 发送 ACK 之前**， leader 发生故障，那么会造成数据重复

A：这需要幂等性/消息去重解决。**幂等性保证下，重复消息不会被落盘**


## Exactly Once

下游数据**消费者要求数据既不重复也不丢失**，即 Exactly Once 语义

**At Least Once + 幂等性 = Exactly Once**


ack=-1 能实现 At Least Once 保证数据不丢失，一定能发到目的地

要启用幂等性，只需要将 Producer 的参数中 enable.idompotence 设置为 true 即可

Kafka 的幂等性实现其实就是将原来下游需要做的**去重放在了数据上游**，**由 Producer 和 Broker 保证消息不丢失，不重复**

开启幂等性的 Producer 在初始化的时候会被分配一个 PID，**发往同一 Partition 的消息会附带 Sequence Number**。而 **Broker 端会对 <PID, Partition, SeqNumber> 做缓存，当具有相同主键的消息提交时，Broker 只会持久化一条**，以此实现消息不重复。  

问题：

Q1：消费者重启 PID 就会变化

Q2：不同的 Partition 也具有不同主键，所以幂等性无法保证跨分区跨会话的 Exactly Once

所以如果**生产者不重启**，且**失败重发的消息只会发往 Broker 的同一个分区**就能保证生产者端的 Exactly Once


# 消费者端的数据可靠性保

> [!NOTE] 控制台执行消费者
> ```bash
> bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic first
> ```



## 拉模式的消费策略

consumer 采用 pull（拉） 模式从 broker 中读取数据

push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的


pull 模式不足之处是，如果 kafka 没有数据，消费者可能会陷入循环中， 一直返回空数  
据

针对这一点， Kafka 的消费者在消费数据时会传入一个时长参数 timeout，如果当前没有数据可供消费， consumer 会等待一段时间之后再返回，这段时长即为 timeout


## 消费者选择哪个分区消费？


一个 consumer group 中有多个 consumer，一个 topic 有多个 partition，所以必然会涉及到 partition 的分配问题，即确定**哪个 partition 由哪个 consumer 来消费**


Kafka 有两种分配策略，一是 RoundRobin，一是 Range


## 消费者如何维护 offset


Q：消费者消费过程中重启了，重启后会从哪继续消费？

A：Kafka 0.9 版本之前， consumer 默认将 offset 保存在 Zookeeper 中

![Pasted image 20220912152308](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912152308.png)

从 0.9 版本开始，consumer 默认将 offset 保存在 Kafka 一个内置的 topic 中，该 topic 为 `__consumer_offsets`

## 消息消费的有序性

**单个分区内的数据也不一定是有序的**

- 因为**发送者按序发送数据，broker 不一定按序接收**

- **消费者按序获取的数据，数据不一定能按序到达消费者**


1.X 以后 kafka 开启幂等后，窗口内允许至多由 5 个数据。只有前一个数据落盘后，后一个数据才能落盘，否则在内存中等待，直到前一个数据落盘


0.9 版本前，消费者消费消息，偏移量保存在 zk 里，Kafka 就需要和 zk 频繁通信，zk 压力大，且 Kafka 和 zk 不在同一台服务器上时，通信延迟会更长。0.9 版本后，偏移量保存到了 Kafka 里



## 消息的高效读写数据策略

- 顺序写

日志 .log 文件方式记录，物理上以块为单位写数据（默认一个块是 1G，所以日志文件大小总是 1G 的整倍数），从 1G 里找数据不方便，所以用 .index 文件辅助他


- 零拷贝


# 事务

生产者和消费者都有事务机制，生产者可能会批量发送消息，消费者可能会批量消费消息，这时会用到事务。


此外，之前提过，生产者发送消息时 Broker 在消息落盘后，返回 ACK 前重启了会导致生产者重发消息。生产者发送消息时携带 <PID, Partation, SequeNumber> 实现幂等性，但生产者重启会重新分配 PID，所以事务还能帮助生产者重启服务时找回原来的 PID


## 生产者事务


为了实现跨分区跨会话的事务，需要引入一个全局唯一的 Transaction ID，并将 Producer PID 和Transaction ID 绑定。这样当 Producer 重启后就可以通过正在进行的 Transaction ID **获得原来的 PID**。


为了管理 Transaction， Kafka 引入了一个新的组件 Transaction Coordinator

Transaction Coordinator 负责将事务所有写入 Kafka 的一个内部 Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行


## 消费者事务

对于 Consumer 而言，事务的保证就会相对较弱，尤其无法保证 Commit 的信息被精确消费。这是由于 Consumer 可以通过 offset 访问任意信息，而且不同的 Segment File 生命周期不同，同一事务的消息可能会出现重启后被删除的情况



# Kafka API

两套 API，一套是 kafka 官方的

```xml
<dependency>  
	<groupId>org.apache.kafka</groupId>  
	<artifactId>kafka-clients</artifactId>  
</dependency>
```

另一套是 Spring 的，KafkaTemplate 封装了 kafka 官方的客户端

所以这里聊官方的 API

## 发送消息

不管是同步发送还是异步发送，它们都是这样发的

![Pasted image 20220912164029](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912164029.png)


生产者把生产的消息交给 `RecordAccumulator`，再由发送线程发送消息到 Broker

对于发送端，有两个重要参数

- batch.size：只有数据积累到 batch.size 之后， sender 才会发送数据

- linger.ms：如果数据迟迟未达到 batch.size， sender 等待 linger.time 之后就会发送数据


```java
public static void main(String[] args) throws Exception {

    Properties props = new Properties();  
    props.put("bootstrap.servers", "127.0.0.1:9092"); //kafka 集群，broker-list  
    props.put("acks", "all");
    props.put("retries", 1);//重试次数  
    props.put("batch.size", 16384);// 批次大小  
    props.put("linger.ms", 1);// 等待时间  
    props.put("buffer.memory", 33554432);//RecordAccumulator 缓冲区大小  
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer"); // key 序列化器
    props.put("value.serializer",  "org.apache.kafka.common.serialization.StringSerializer"); // val 序列化器

    Producer<String,String> producer = new KafkaProducer<>(props);

    for (int i = 0; i < 100; i++) {  
        producer.send(
            new ProducerRecord<String, String>(
                "first",
                Integer.toString(i),
                Integer.toString(i)
            ),
            new Callback() {  
                //回调函数， 该方法会在 Producer 收到 ack 时调用，为异步调用  
                @Override  
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception == null) {
                        System.out.println("success->" + metadata.offset());  
                    } else {  
                        exception.printStackTrace();  
                    }
                }  
            }
        );  
    }  
    producer.close();
}
```



**生产者分区发送方式**


生产者生产的**消息会被发到哪个分区里**，路由方式很简单

- 在发送消息时直接指定
- 给消息一个 key，对 key 进行哈希求余得到分区号
- 随机选择一个分区


## 消费消息

消费者消费消息涉及到**手动提交**，**批量提交**，**同步提交和异步提交**，以及**何时提交**的问题

需要重点关注这些参数

- enable.auto.commit：是否开启自动提交 offset 功能（当然要手动提交）

- auto.commit.interval.ms：自动提交 offset 的时间间隔（既然手动提交，就不要设置）


以手动同步批量提交为例

```java
public static void main(String[] args) {

    Properties props = new Properties();
    props.put("bootstrap.servers", "hadoop102:9092"); //Kafka 集群
    props.put("group.id", "test"); // 只要 group.id 相同，就属于同一个消费者组
    props.put("enable.auto.commit", "false"); // 关闭自动提交 offset
    props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
  
    KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

    consumer.subscribe(Arrays.asList("first"));//消费者订阅主题

    while (true) {
        // 消费者拉取 100 条数据
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records) {
            System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
        }
        // 同步提交，当前线程会阻塞直到 offset 提交成功
        consumer.commitSync();
    }
}
```


**无论是同步提交还是异步提交 offset，都有可能会造成数据的漏消费或者重复消费**。先  
提交 offset 后消费，有可能造成数据的漏消费；而先消费后提交 offset，有可能会造成数据的重复消费


# Kafka 监控平台

kafka-eagle 默认端口号是 8048

访问 http://127.0.0.1:8048/ke 即可


# Kafka Stream

[介绍一位分布式流处理新贵：Kafka Stream](https://cloud.tencent.com/developer/article/1031210)

