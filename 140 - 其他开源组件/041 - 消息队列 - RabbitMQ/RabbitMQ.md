#还没有复习 

# RabbitMQ 的四个核心组件和工作原理 



生产者：生产消息并发送给 MQ

消费者：消费者消费指定队列里的消息

交换机：负责接收生产者的消息；用 routingKey 绑定若干个队列

队列：保存消息的队列



消息发送模式有：简单模式，工作队列模式，发布订阅模式，路由模式，主题模式等



- 工作队列模式下，用的是工作队列交换机，交换机收到 msg 后，根据 msg 的 routingKey 把 msg 分发给交换机的某个队列
- 发布订阅模式下，用的是扇出交换机，交换机收到 msg 后，把 msg 复制并分发给交换机绑定的所有队列中

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220623094738132.png" alt="image-20220623094738132" style="zoom:50%;" />

此外，RabbitMQ 还有其他几个组件，如下图所示

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220623094936765.png" alt="image-20220623094936765" style="zoom:80%;" />

- virtual host 是 RabbitMQ 客户端面向的是 RabbitMQ 服务端的一个虚拟主机或者说 namespace

  每个 virtual host 和 RabbitMQ 服务端的关系就像 mysql 一个 database 和 mysql 服务器整个进程的关系

- broker 是接收和分发消息的应用，RabbitMQ Server 就是 Message Broker



# RabbitMQ 的模式



模式指，交换机收到 msg 后，如何分发 msg 到队列里



常用的模式有：简单模式，工作队列模式，发布订阅模式，路由模式，主题模式。不同模式使用不同类型的交换机

比如：发布订阅模式 - 扇出交换机，路由模式 - 直接交换机，主题模式 - 主题交换机



# 简单模式和工作队列模式

发送者发送消息到 “默认交换机”，rouotingKey 和 queueName 保持一致

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220623095004907.png" alt="image-20220623095004907" style="zoom:80%;" />



工作队列模式下，交换机会绑定多个 rouotingKey 和 queue



```xml
<dependency>
	<groupId>com.rabbitmq</groupId>
	<artifactId>amqp-client</artifactId>
</dependency>
```



**生产者**

```java
public class Producer {
	private final static String QUEUE_NAME = "hello";
	public static void main(String[] args) throws Exception {
        // 创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("admin");
        factory.setPassword("123");
        // channel 实现了自动 close 接口 自动关闭 不需要显示关闭
        try(Connection connection = factory.newConnection(); Channel connection.createChannel()) {
            /**
            * 生成一个队列
            * 1.队列名称
            * 2.队列里面的消息是否持久化 默认消息存储在内存中
            * 3.该队列是否只供一个消费者进行消费 是否进行共享 true 可以多个消费者消费
            * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
            * 5.其他参数
            */
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message="hello world";
            /**
            * 发送一个消息
            * 1.发送到那个交换机
            * 2.路由的 key 是哪个
            * 3.其他的参数信息
            * 4.发送消息的消息体
            */
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("消息发送完毕");
        }
	}
}
```

**消费者**

```java
public class Consumer {
	private final static String QUEUE_NAME = "hello";
	public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setUsername("admin");
        factory.setPassword("123");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        System.out.println("等待接收消息.........");
        // 消费回调接口 这里写处理消息的业务逻辑代码
        DeliverCallback deliverCallback=(consumerTag,delivery)->{
            String message= new String(delivery.getBody());
        	System.out.println(message);
        };
        // 取消消费回调接口 如在消费的时候队列被删除掉了
        CancelCallback cancelCallback=(consumerTag)->{
            System.out.println("消息消费被中断");
        };
         /**
        * 消费者消费消息
        * 1.消费哪个队列
        * 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
        * 3.消费者未成功消费的回调
        */
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
}
```



# 消息应答

消息应答指消息发送到消费者后，消费者需要发送 msg ack 给 MQ，让 MQ 知道这条 msg 已经被成功消费了



设置消息应答的原因在于：消费者收到 msg 并不代表 msg 被消费了。通常当消费者收到 msg 并在业务逻辑层面处理完 msg 后才算消费完消息



消息应答有以下 3 种方式

- 自动应答：消费者收到 msg 后立即发送 ack 给 MQ

  ```java
  // 第二个参数：消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
  channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
  ```

- 手动应答（在上述 `channel.basicConsume()` 上设置 `false` 即开启手动应答）
  
  - 独立应答：消费者发送的 ack 只能针对某一个条 msg
  
    ```java
    channel.basicAck(); // 用于肯定确认
    channel.basicNack(); // 用于否定确认
    channel.basicReject(); // 拒绝处理
    ```
  
  - 批量应答：类似 TCP 的累计确认。消费者对 msg5 发送应答后，MQ 会认为 msg5 及其之前的消息都被应答
  
    ```java
    channel.basicAck(deliveryTag, true); // true 表示对之前的消息进行累计确认
    ```



自动应答和手动批量应答都可能会导致消息丢失，因为可能存在消费者没有处理完 msg 就宕机，但 MQ 已经收到了 msg 的应答



**队列里的一条消息**只能被一个消费者处理，在消费者签收消息/发送 ack 后，消息就算被消费过了，MQ 就不会让其他消费者消费了



如果消息被消费但长时间没有被应答，消息就会重新入队，再次等待被消费

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220623105155421.png" alt="image-20220623105155421" style="zoom:80%;" />



# 消息分发策略



## 轮训分发

如果不是广播 msg，MQ 的交换机会始终以轮训方式分发消息给队列



所以默认情况下，比如工作队列模式下，msg 会被轮训发送到所有队列里



##  不公平分发 - 预取值

为了使能者多劳，给消费能力强的消费者分发更多 msg，消费能力弱的消费者分发较少的 msg



消费者可自定义预取值 n，表示消费者最多能持有几条 msg。如果消费者持有 msg 数量达到 n，就会先拒绝获取 msg。但是 MQ 分发 msg 的策略还是轮训分发，只不过是消费者用预取值拒绝了接收 msg

```java
// 设置预取值
channel.basicQos(prefetchCount);
```




# 消息持久化

消息持久化，指发送者发送到 MQ 的消息能被持久化，使得那些未被消费的消息不会因为 MQ 宕机而永久性丢失



消息持久化包括**队列持久化**和**消息持久化**



**队列持久化**

默认创建的队列都是非持久化的，MQ 关闭后队列就会被删除

```java
// 第二个参数就是是否持久化队列
channel.queueDeclare(QUEUE_NAME, true, false, false, null);
```



> 如果想把已存在的非持久化队列变为持久化的，需要把原队列先删除再重新创建
>
> 凡是需要修改队列配置的行为，都需要先删除原队列再创建新的，否则启动报错



**消息持久化**

要想让消息实现持久化需要在消息生产者修改代码，发送 msg 时添加 `` 属性

```java
// 设置第三个参数为 MessageProperties.PERSISTENT_TEXT_PLAIN
channel.basicPublish("ExchangeName", routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN, msgBytes);
```



实现生产者感知消息是否被成功持久化，参考**消息发布确认**



# 发布确认原理

发布不确认用于消息是否被投递到匹配的队列中（如果设置了消息持久化，那么此时消息一定被持久化过了）

生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，所有在该信道上面发布的消息都将会被指派一个唯一的 ID，一旦消息被投递到所有匹配的队列之后，broker 就会发送一个确认给生产者 ack，这就使得生产者知道消息已经正确到达目的队列了

```java
channel.confirmSelect();
```



如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出



发布确认也有批量确认模式，通过设置通道的 `multiple` 属性实现累计确认。但这和消息应答一样，在需要保证消息 0 丢失时不能使用此模式



消息发布确认分为同步确认和异步确认



## 同步确认

同步确认会调用 `channel.waitForConfirms(long);` 进行阻塞，但此方法会在所有已发送的 msg 收到 ack 后才会被唤醒线程

所以这个方法可以实现单个 msg 确认或批量确认（这个批量确认能保证消息 0 丢失，但一旦发现有消息没有被持久化，会无法感知是哪条消息没有被持久化）



## 异步确认

MQ 根据消息持久化情况，**MQ 通知消费者**调用回调方法。持久化成功调 success，失败调 fail



```java
public static void publishMessageAsync() throws Exception {
    try (Channel channel = RabbitMqUtils.getChannel()) {
        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, false, false, false, null);
        // 开启发布确认
        channel.confirmSelect();
        // 保存哪些发送了，但还没有收到发布确认的 msg
        ConcurrentSkipListMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();
        // 收到 ack，删除缓存
        ConfirmCallback ackCallback = (sequenceNumber, multiple) -> {
            if (multiple) { // 批量删除
                outstandingConfirms.headMap(sequenceNumber, true).clear();
            } else {
                outstandingConfirms.remove(sequenceNumber);
            }
        };
        ConfirmCallback nackCallback = (sequenceNumber, multiple) ->{
            String message = outstandingConfirms.get(sequenceNumber);
        	System.out.println("发布的消息"+message+"未被确认，序列号"+sequenceNumber);
        };
        // 添加异步确认的监听器
        channel.addConfirmListener(ackCallback, nackCallback);
        
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            String message = "消息" + i;
        // channel.getNextPublishSeqNo() 获取下一个将被发送的消息的序列号
            outstandingConfirms.put(channel.getNextPublishSeqNo(), message);
            channel.basicPublish("", queueName, null, message.getBytes());
        }
    }
}
```



如果消息确认失败，需要把确认失败的消息保存起来以便重新发送



这种方式的缺点在于它需要保证 MQ 不宕机。而且如果 MQ 没有宕机，通常也不会出现确认失败问题



# 交换机和交换机模式



##  Exchanges 的类型

交换机总共有以下类型：直接 - direct，主题 - topic，标题 - headers，扇出 - fanout

标题类型已经不常用了。无名类型就是默认类型，用空字符串表示无名交换机



交换机的类型由客户端声明交换机时指定

```java
channel.exchangeDeclare(EXCHANGE_NAME, "fanout"); // 第二个参数是交换机的类型
```



> RabbitMQ 启动后自动创建各种类型的交换机各一个



##  临时队列

客户端可以创建有指定名称的队列，也能创建临时队列，一旦断开了消费者的连接，临时队列将被自动删除

```java
String queueName = channel.queueDeclare().getQueue();
```



## 绑定 - Binding

交换机有 ExangeName，队列有 QueueName

- 消费者消费消息是面向 QueueName 的，建立和 MQ 的连接后，指定 QueueName 消费消息
- 生产者生产消息面向的是 ExangeName + RoutingKey。生产者发送消息时，需要指定交换机和路由 key，而不是指定 QueueName

```java
channel.queueBind(queueName, EXCHANGE_NAME, routingKey);
```



生产者发送消息的直接目的地不是 QueueName，是 ExangeName + RoutingKey

交换机维护了一个 Map<RoutingKey, Queue>。RoutingKey 可以重复，但 RoutingKey + Queue 不能重复。这个 map 就是 Binding 集合

交换机收到 msg 后，根据 msg 携带的  RoutingKey 找到并分发给若干个 Queue



## 路由模式 - 直接交换机（Direct）

直接交换机模式下，消息会被交换机发送给指定 RoutingKey 下的所有队列

Binding 中，一个 Queue 可以有多个 RoutingKey。向 routingkey="black" or "green" 发送 msg 都会发到 Q1 里

```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");
```

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220623140254530.png" alt="image-20220623140254530" style="zoom:80%;" />

一个 RoutingKey 也可以有多个 Queue。向 routingkey="black" 发送 msg 会发到 Q1，Q2 里

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220623140435682.png" alt="image-20220623140435682" style="zoom:80%;" />



## 发布订阅模式 - 扇出交换机（Fanout）

发布订阅模式下，交换机收到消息后会忽略 RoutingKey，直接把消息复制并分发给 map 中的所有队列

> 所以扇出模式下，所有 Binding 的 RoutingKey 不同也没有任何关系



> 每次使用交换机或队列时，不管是生产者还是消费者都调 `channel.exchangeDeclare()`  或 `channel.queueDeclare()` 或 `channel.queueBinding()`，这样能有效防止没有指定的队列或交换机。说白了就是 `createIfAbsent()`



## 主题模式 - 主题交换机（Topics）

交换机绑定队列时，用的 RoutingKey 是主题表达式，主题表达式被要求必须是一个单词列表，以点号分隔开

比如：\"stock.usd.nyse\", \"nyse.vmw\"



在发送 msg 时，msg  携带的 RoutingKey 是主题匹配式。匹配式仍被要求必须是一个单词列表，以点号分隔开。此外，\*(星号)可以代替一个单词，#(井号)可以替代零个或多个单词



举例：

现在主题交换机的 Binding 有

| RoutingKey   | Queue |
| ------------ | ----- |
| \*.orange.\* | Q1    |
| \*.\*.rabbit | Q2    |

向这个交换机发送 msg 时，携带如下 RoutingKey  

- quick.orange.rabbit，被队列 Q1Q2 接收到
- lazy.orange.elephant，被队列 Q1Q2 接收到
- quick.orange.fox，被队列 Q1 接收到



不匹配任何绑定不会被任何队列接收到会被丢弃



> - 当一个队列绑定键是#，那么这个队列接收所有数据，就有点像 fanout 了
> - 如果队列绑定键当中没有 # 和 \* 出现，那么该队列绑定类型就是 direct 了



#  死信队列

死信，顾名思义就是无法被消费的消息

某些时候由于特定的原因导致 queue 中的某些消息无法被消费，这样的消息如果没有后续的处理，就变成了死信，有死信自然就有了死信队列



死信消息主要来源于以下 3 个地方

- 消息 TTL 过期
- 队列达到最大长度（队列满了，无法再添加数据到 MQ 中）
- 消息被拒绝（`basic.reject()` 或 `basic.nack()`）并且 `requeue=false`  



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220623152408503.png" alt="image-20220623152408503" style="zoom:80%;" />

RabbitMQ 没有直接实现死信交换机这种特殊的交换机

死信的实现方式是：在声明普通队列时，声明普通队列被绑定到了哪个交换机和这个交换机的哪个 RoutingKey 上，将这个交换机和 Binding 作为死信队列

```java
// 声明死信和普通交换机 类型为 direct
channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

//声明死信队列
String deadQueue = "dead-queue";
channel.queueDeclare(deadQueue, false, false, false, null);
//死信队列绑定死信交换机与 routingkey
channel.queueBind(deadQueue, DEAD_EXCHANGE, "lisi");

//正常队列绑定死信队列信息
Map<String, Object> params = new HashMap<>();
// 正常队列设置死信交换机 参数 key 是固定值
params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
// 正常队列设置死信 routing-key 参数 key 是固定值
params.put("x-dead-letter-routing-key", "lisi");
String normalQueue = "normal-queue";

// 声明正常队列
channel.queueDeclare(normalQueue, false, false, false, params);
channel.queueBind(normalQueue, NORMAL_EXCHANGE, "zhangsan")
```



如果普通队列中的消息消费失败就把消息放到刚才绑定的交换机上，这个交换机在逻辑上就是死信交换机。因为死信交换机仅仅是逻辑上的，而不是真正实现了一个死信交换机类型，所以死信交换机也能声明各种类型 - 直接交换机，广播交换机，主题交换机...。不讨论死信交换机/队列无限链式绑定死信交换机/队列



==注意死信交换机的声明和使用应该在消费者侧。因为生产者只关心发送消息，不关心消息如何被消费，所以死信机制作为消息消费机制理应在消费端==



## 消息的 TTL

每条消息都有 TTL 存活时间



消息的 TTL，可以用在队列设置本队列中所有消息的 TTL，也可以给每个消息独立设置 TTL

- 声明队列时可以设置队列里每条消息的默认 TTL，在声明队列时传入 properties，关键词为 `x-message-ttl` 

- 向 MQ 发送消息时，可以设置针对单个消息的 TTL，在调用 basicPuhlish 时传入 properties（利用 new BasicProperties 链式创建）

  ```java
  channel.basicPublish(NORMAL_EXCHANGE, 
                       routingKey, 
                       AMQP.BasicProperties().builder().expiration("10000").build(),
                       message.getBytes()
                      );
  ```

消息的 TTL 理应由发送方控制



> 为每个消息独立设置 TTL 存在消息到达 TTL 后不被删除的问题。因为此种方式下，消息是在出队时才被检查消息 TTL 是否归零



## 队列达到最大长度

队列的最大长度只能在声明队列时设置

```java
param.put("x-max-length", 6); // 设置正常队列长度的限制
channel.queueDeclare(QUEUE_NAME, false, false, false, param);
```



# SpringBoot 整合 RabbitMQ

1. 导入依赖
2. 配置参数
3. 向容器中加入 bean - 交换机，队列，绑定
4. 写生产者，消费者的代码

**依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**配置**

```properties
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123
```

**声明交换机，队列和绑定**

```java
// 声明交换机
@Bean
public DirectExchange exchangeA(){
	return new DirectExchange("E1");
}
// 声明队列
@Bean
public Queue queueA(){
    return QueueBuilder.durable("Q1").build();
}
// 声明队列 A 绑定 交换机 A
@Bean
public Binding queueaBindingX(Queue queueA, DirectExchange exchangeA){
	return BindingBuilder.bind(queueA).to(exchangeA).with("RoutingKeyA");
}
```

**生产者和消费者**

```java
// 生产者
@Autowired
private RabbitTemplate rabbitTemplate;

public void sendMsg(String msg){
    rabbitTemplate.convertAndSend("E1", "Q1", msg);
}
```

```java
// 消费者
@Slf4j
@Component
public class DeadLetterQueueConsumer {
    @RabbitListener(queues = "Q1")
    public void receive(Message message, Channel channel) throws IOException {
        String msg = new String(message.getBody());
        log.info("当前时间： {},收到死信队列信息{}", new Date().toString(), msg);
    }
}
```



# 延迟队列

RabbitMQ 的延迟队列就是定时任务。Rabbit MQ 延迟队列可以用死信队列 + TTL 实现，但通常不这么做，因为[社区](http://www.rabbitmq.com/community-plugins.html)提供了延迟队列的插件（rabbitmq_delayed_message_exchange）



> 不用死信队列 + TTL，因为定时时间要么用多个队列设置不同的 TTL 表示不同的定时时间，要么为每个消息设置 TTL。前者会导致队列数量激增，后者存在消息到达 TTL 后不会被即时删除的问题



> RabbitMQ 的插件机制：把插件放到 MQ 的插件目录（/usr/lib/rabbitmq/lib/rabbitmq_server/plugins）下，重启 MQ，再激活插件（`rabbitmq-plugins enable 插件名`）即可使用



RabbitMQ 的延迟队列插件通过提供延迟类型的交换机实现

```java
// 自定义一个延迟交换机
@Bean
public CustomExchange delayedExchange() {
    Map<String, Object> args = new HashMap<>();
    // 自定义交换机的类型
    args.put("x-delayed-type", "direct");
    return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, args);
}
```



分布式定时任务的实现方式有很多种：Quartz，Netty，Kafka，JDK 自带定时任务



# 发布确认高级

前面讲的发布确认是MQ没有宕机的情况下，消息不管成功失败都会给生产者这边回应，现在高级发布确认就是MQ宕机了无法给回应。可能是MQ整个宕机了，也可能是交换机崩了或不见了，队列崩了或不见了

在生产环境中由于一些不明原因，导致 RabbitMQ 重启，在 RabbitMQ 重启期间生产者消息投递失败，导致消息丢失，需要手动处理和恢复。在 RabbitMQ 集群不可用的时候，无法投递的消息该如何处理呢



## confirm-type 处理投递失败的消息

```properties
spring.rabbitmq.publisher-confirm-type=correlated
```

- NONE。禁用发布确认模式，是默认值
- CORRELATED。发布消息成功到交换器后会触发回调方法
- SIMPLE。有两种效果，其一效果和 CORRELATED 值一样会触发回调方法，其二在发布消息成功后使用 rabbitTemplate 调用 `waitForConfirms` 或 `waitForConfirmsOrDie` 方法

```java
rabbitTemplate.setConfirmCallback(myCallBack); // 设置回调方法
```

```java
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback {
    /**
    * 交换机不管是否收到消息都会执行的回调方法（如果交换机没有收到消息，可以在这里设置重发逻辑）
    * CorrelationData - 消息相关数据
    * ack - 交换机是否收到消息
    */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id=correlationData!=null?correlationData.getId():"";
        if(ack){
        	log.info("交换机已经收到 id 为:{}的消息",id);
        } else {
        	log.info("交换机还未收到 id 为:{}消息,由于原因:{}",id,cause);
        }
    }
}
```



## Mandatory 处理不可路由消息

在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息**，**如果发现该消息不可路由，那么消息会被直接丢弃，此时生产者是不知道消息被丢弃这个事件的

那么如何让无法被路由的消息帮我想办法处理一下？最起码通知我一声，我好自己处理啊。通过设置 mandatory 参数可以在当消息传递过程中不可达目的地时将消息返回给生产者。

```java
/**
* true：交换机无法将消息进行路由时，会将该消息返回给生产者
* false：* 如果发现消息无法进行路由，则直接丢弃
*/
rabbitTemplate.setMandatory(true);
// 设置回退消息交给谁处理
rabbitTemplate.setReturnCallback(returnCallback); // 实现 RabbitTemplate.ReturnCallback 的 returnedMessage
```



## 备份交换机处理不可路由消息

如果给交换机设置了备份交换机，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220623170010309.png" alt="image-20220623170010309" style="zoom:80%;" />



有了 mandatory 参数和回退消息，我们获得了对无法投递消息的感知能力，有机会在生产者的消息无法被投递时发现并处理

但有时候，我们并不知道该如何处理这些无法路由的消息，最多打个日志，然后触发报警，再来手动处理

而通过日志来处理这些无法路由的消息是很不优雅的做法，特别是当生产者所在的服务有多台机器的时候，手动复制日志会更加麻烦而且容易出错。而且**设置 mandatory 参数会增加生产者的复杂性，需要添加处理这些被退回的消息的逻辑**。

现在希望暂时不处理回退消息，但又希望把他们保存起来不丢失。死信交换机能存储处理失败的消息，但不能存储回退消息，因为这些不可路由消息根本没有机会进入到队列

备份交换机能接收这些回退消息，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。当然，我们还可以建立一个报警队列，用独立的消费者来进行监测和报警。

```java
// 声明备份 Exchange
@Bean
public FanoutExchange backupExchange(){
	return new FanoutExchange(BACKUP_EXCHANGE_NAME);
}
// 声明警告队列。被消费者消费，用于获取不可路由消息
@Bean
public Queue warningQueue(){
	return QueueBuilder.durable(WARNING_QUEUE_NAME).build();
}
// 声明报警队列绑定关系
@Bean
public Binding warningBinding( Queue warningQueue, FanoutExchange backupExchange){
	return BindingBuilder.bind(queue).to(backupExchange);
}
// 声明备份队列。仅用于备份不可路由消息
@Bean("backQueue")
public Queue backQueue(){
	return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
}
// 声明备份队列绑定关系
@Bean
public Binding backupBinding(Queue backQueue, FanoutExchange backupExchange){
	return BindingBuilder.bind(queue).to(backupExchange);
}
```



mandatory 参数与备份交换机可以一起使用的时候，如果两者同时开启，备份交换机优先级高。即如果设置了备份交换机，mandatory 机制将不再生效



# 幂等性

消息幂等保证可以用 redis 实现

1. 生产消息前，把消息的唯一标识符 set 到 redis 里
2. 消费消息时，根据消息的唯一标识符 del 掉同名 key
   - 如果能删除，说明此消息没有被消费过
   - 如果不能删除，key 不存在，说明消息被消费过



# 优先级队列

有些队列中的消息需要有优先级，消费者优先消费重要的消息

RabbitMQ 用优先队列（大根堆）实现

- 当声明队列时传入 "x-max-priority" 参数，声明了此队列是优先队列，val 就是此优先队列的最高优先级。消息的优先级不能大于 val

  ```java
  Map<String, Object> params = new HashMap();
  params.put("x-max-priority", 10);
  channel.queueDeclare("hello", true, false, false, params);
  ```

- 发送消息时，需要传入消息的优先级

  ```java
  AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
  ```

  

> 官方推荐 `x-max-priority` 值为 5~10，太大会增加性能开销

# 惰性队列

RabbitMQ 从 3.6.0 版本开始引入了惰性队列的概念。惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储



而默认情况下，当生产者将消息发送到 RabbitMQ 的时候，队列中的消息会尽可能的存储在内存之中，这样可以更加快速的将消息发送给消费者。即使是持久化的消息，在被写入磁盘的同时也会在内存中驻留一份备份



队列具备两种模式：default 和 lazy。默认的为 default 模式。在队列声明的时候可以通过 "x-queue-mode" 参数来设置队列的模式，取值为"default"和"lazy"

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false, false, false, args);
```



> 内存开销对比（测试数据来源未知）：在发送 1 百万条消息，每条消息大概占 1KB 的情况下，普通队列占用内存是 1.2GB，而惰性队列仅仅占用 1.5MB



# RabbitMQ 集群



## 通过软负载控制集群

集群下在一台机器上的指令，会作用于整个集群



RabbitMQ 的集群中，每个节点的交换机，队列都是节点独有的。且 RabbitMQ 没有提供连接集群的客户端



不仅 RabbitMQ，很多开源软件都是如此，需要用软负载软件控制集群。比如 Nginx，Haproxy+Keepalive



[nginx,lvs,haproxy 之间的区别](http://www.ha97.com/5646.html)



## 镜像队列

类似 “复制” / “主从复制”。但通常一个队列只镜像一份，能节省点空间资源



通过修改集群 Policy 设置镜像队列/交换机。设置指定数量的镜像交换机，防止单点故障导致服务不可用



源 + 镜像数量 = ha-params = 2，假设现在源是 node1，镜像是 node2。如果 node1 宕机，node2 变成源，node3 变成镜像。总之 源 + 镜像数量 = ha-params = 2

引入镜像队列（Mirror Queue）机制，可以将队列镜像到集群中的其他 Broker 节点之上，如果集群中的一个节点失效了，队列能自动地切换到镜像中的另一个节点上以保证服务的可用性

