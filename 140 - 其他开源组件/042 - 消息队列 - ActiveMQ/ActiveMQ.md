#不再有学的价值了，准备删除

# 消息中间件的基本概念


## 消息中间件和`ActiveMQ`的关系

`ActiveMQ` 是消息中间件的一种落地实现


## 谈谈`消息中间件`，`消息`和`中间件`

`消息`：举例如微信，短信，语音

`消息中间件`也称`消息队列`


## 应用举例

**技术向举例**

dubbo有一个三角形`消费者`，`服务者`，`注册中心（一般是zookeeper）`

SpringCloud也有一个三角形`客户端`，`服务端`，`注册中心eureka`

其中注册中心就像是中间件一样连接起了客户端和服务端

**实际生活向举例**

A向B发送短信，A发送短信的条件是消息中间件健在，即A不需要关心B到底有没有收到，B的手机是不是没电了（异步）A并不需要等待B真的接收到短信后才能做其他事情


## 队列与主题

`队列Queue`，`主题Topic`

这两个词时消息中间件中的词，是两种发送接收消息的方式，或者说是存储消息的方式

`队列`就像A给B发消息

`主题`就像A，B订阅了CodeSheep的公众号，公众号发送消息，所有订阅此公众号的人均可收到推送，而C因为没有订阅所以没收到推送

`队列`一对一，`主题`一对多


## ActiveMQ的结构

> 和jdbc操作数据库类似

activemq也有连接工厂，也有connection，session（JDBC中有`Connection`，`Mybatis`中有`SqlSession`和`SqlSessionFactory`）

session用于创建生产者和消费者，前者用于发送消息，后者用于接收消息

activemq的消息又分为队列与主题


# ActiveMQ操作

> `ActiveMQ`是java写的程序，相对来说好查错

## quickstart实操

1. 下载

2. 运行

   ```shell
   # 在activemq的bin目录下
   $ ./activemq start
   # 检查端口占用情况的三种命令 lsof -i  ; netstat -anp ; ps -ef
   # activemq默认使用端口61616
   ```

3. 停止

   ```shell
   # 在activemq的bin目录下
   $ ./activemq stop
   ```

带日志记录的启动

```shell
# 在activemq的bin目录下，指定一个.log文件(名字随便)，将日志追加到此文件中
$ ./activemq start > run_actice.log
```

activemq的http客户端默认使用8161端口

在浏览器地址栏输入`ip:8681/admin`即可，前提是8681端口可远程访问

**拓展**

```shell
# 使用此命令后可启动多个activemq，或者说可使用不同的配置文件在同一台机器上同时运行多个activemq
$ ./activemq start xbean:配置文件路径
```


# Java整合AvtiveMQ

> JMS > MQ > ActiveMQ

先介绍java底层代码操作mq，再介绍Spring操作mq


1. 搭建项目，导入maven依赖
2. 写代码


1. 搭建项目，导入maven依赖

   ```xml
   <!--整合web-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-activemq</artifactId>
       <exclusions>
           <exclusion>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-log4j12</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   
   <!--SpringBoot2.1+后使用此依赖取代activemq-pool的依赖，否则装配template时会失败-->
   <dependency>
       <groupId>org.messaginghub</groupId>
       <artifactId>pooled-jms</artifactId>
   </dependency>
   ```
   

## java底层代码操作activemq

> 和java使用jdbc操作数据库类似，java操作activemq有以下几个步骤
>
> `创建activemq工厂`，`用工厂创建connection``
>
> ``用connection创建session`，`用session创建目的地（队列或主题）`
>
> `用session创建消费者或生产者`，`用session创建的消费者或生产者接收或生产Message（消息）`
>
> `使用.close()方法关闭上述对象`（close的顺序和jdbc操作数据库一样，先创建的后关闭，后创建的先关闭）

下面将举例`向队列发送消息`，`消费队列消息`（有两种实现，接收消息-同步，监听消息-异步）

### 发送队列的消息

```java
public class SendQueueMessage {
    public static String brokerURL = "tcp://39.107.101.13:61616"; // mq的地址
    public static String queueName = "queue01"; // 队列的名字
    public static void main(String[] args) throws JMSException {
        // 1. 创建工厂
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(brokerURL);
        // 2.创建connection
        Connection connection = factory.createConnection();
        connection.start();
        // 3. 创建session 第一个参数，是否开启事务；第二个参数，消息签收策略（这两个参数在后面会讲）
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4. 创建消息队列queue
        Queue queue = session.createQueue(queueName);
        // 5. 创建消息生产者provider
        MessageProducer producer = session.createProducer(queue);
        // 6. 创建消息message，并使用provider发送消息
        for (int i = 1; i <= 3; i++) {
            producer.send(session.createTextMessage("message->" + i));
        }
        // 7. 关闭
        producer.close();
        session.close();
        connection.close();
        System.out.println("消息发送成功");
    }
}
```

### 消费队列的消息

1. 接收消息（同步/阻塞式接收消息）

```java
public class ReceiveQueueMessage {
    public static String brokerURL = "tcp://39.107.101.13:61616";
    public static String queueName = "queue01";
    public static void main(String[] args) throws JMSException {
        // 1. 创建工厂
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(brokerURL);
        // 2.创建connection
        Connection connection = factory.createConnection();
        connection.start();
        // 3. 创建session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4. 创建消息队列queue
        Queue queue = session.createQueue(queueName);
        // 5. 创建消费者consumer
        MessageConsumer consumer = session.createConsumer(queue);
        // 6. 接收消息
        while (true){
            TextMessage message = (TextMessage) consumer.receive(); // 不为receive方法设置参数（等待时间），默认会一直等待接收消息
            if (message == null) {
                break;
            }
            System.out.println("接收到了消息：" + message.getText());
        }
        // 7. 关闭
        consumer.close();
        session.close();
        connection.close();
    }
}
```

2. 监听消息（异步/非阻塞式接收消息）

```java
public class ListenQueueMessage {

    public static String brokerURL = "tcp://39.107.101.13:61616";
    public static String queueName = "queue01";

    public static void main(String[] args) throws JMSException, IOException {
        // 1. 创建工厂
        ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(brokerURL);

        // 2.创建connection
        Connection connection = factory.createConnection();
        connection.start();

        // 3. 创建session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        // 4. 创建queue
        Queue queue = session.createQueue(queueName);

        // 5. 创建consumer
        MessageConsumer consumer = session.createConsumer(queue);

        // 6. 接收消息
        consumer.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                if (message != null && message instanceof TextMessage) {
                    TextMessage textMessage = (TextMessage) message;
                    try {
                        System.out.println("接收到的消息：" + textMessage.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        System.in.read(); // 阻塞式输入，这句话会一直等待控制台输入一个字符。在这里的作用是，不让程序直接结束，等待监听器接收消息，防止监听器还没接收消息就执行下面的.close

        // 7. 关闭
        consumer.close();
        session.close();
        connection.close();
    }
}
```

### 多个消费者同时获取队列中的消息

场景：已经启动了多个消费者，消息队列中暂时没有消息

执行：生产者发送多个消息

问题：多个消费者将怎么分配接收消息

答：平均分


### 发送主题的消息

> Topic和Queue还有一个区别是，Topic不保存消息。只要生产者发送消息，将直接发送到订阅此Topic的所有消费者手中。如果没有消费者订阅，这条消息就是条“废消息”，即使消费者以后再订阅也接受不到之前发的消息

### 接收主题的消息


关于发送，接收主题消息的代码不再描述。只需将发送，接收队列消息的代码中的Queue对象改为Topic即可

其代码除了队列使用Queue对象，主题使用Topic对象外，其他代码都是一样的

`todo`：看看`Queue`和`Topic`是不是实现了同一个接口


## Spring操作activemq

> 此节是在学习完ActiveMQ的Broker后补充的，因为笔记结构原因没有讲此内容追加在Broker后面，插入到了java底层代码操作activemq笔记的后面

1. 添加maven依赖（上述java操作activemq的依赖其实就是spring/springboot所使用的依赖）

2. 向容器中注入需要的组件`factory`，`connection`，`session`，`JmsTemplate`等等
3. 使用`JmsTemplate`操作avtivemq


## SpringBoot整合activemq

1. 添加maven依赖（如上spring整合acivemq）

2. 在配置文件中配置activemq

   ```yaml
   spring:
     activemq:
       user: admin
       password: admin
       broker-url: tcp://39.107.101.13:61616
       pool:
         enabled: true
         max-connections: 10
     jms:
     	# 此配置false-操作topic，true-操作queue
       pub-sub-domain: false
   ```

3. 注入topic或queue

4. 添加注解`@EnableJms`，开启jms（不过添加此注解，也能执行下述5中的测试代码）

5. 在业务逻辑中装配`JmsMessagingTemplate`和第3步注入的对象，使用其对象操作activemq

   ```java
   @Autowired
   private JmsMessagingTemplate jmsMessagingTemplate;
   @Autowired
   private ActiveMQQueue queue;
   
   @Test
   public void test() {
       System.out.println(jmsMessagingTemplate);
       jmsMessagingTemplate.convertAndSend(queue, "SpringBootTest");
   }
   ```

除直接在接口的业务逻辑中使用消费者和生产者为，还可以实现

`生产者定时发送消息`，`消费者持续监听消息`

### 生产者定时发送消息

使用SpringBoot整合定时任务实现定时发送消息

`@EnableScheduling`写在启动类上

`@Scheduled`写在方法上，程序会根据注解的元素定时执行方法的业务逻辑。写cron表达式，指定执行方法/任务的时间，可用cron在线生成器生成


### 消费者持续监听消息

`@JmsListener`

```java
@JmsListener(destination = "myQueueName") // 此元素的值为队列名或主题名
public void consumerMsg(TextMessage textMessage) throws JMSException {
    String text = textMessage.getText();
    System.out.println("***消费者收到的消息:    " + text);
}
```


### Topic的持久化订阅

> 要实现Topic的非持久化，只需将上述代码稍加修改即可。但是由于SpringBoot无法通过配置文件的方式设置Topic的持久化订阅，所以其实现需要自己手动配置

需要做的事：自定义

# Message的结构


jms的message由以下三部分组成：`消息头`，`消息体`，`消息属性`

`消息头`携带jms的目的地，持久模式，过期时间，ID等

`消息体`携带信息主体，可以字符串，map，字节，java流，Object（对象）等方式存储消息

`消息属性`存储`消息头`除字段以外的其他属性，简单地说就是存储自定义的拓展kv键值对（要和`消息体`区分开，消息体是信息主体，消息属性是辅助属性。例如可用消息属性做标识，去除，重点标注等功能）

用java操作activemq时，以上三部分均可操作。其中，设置`消息属性`时可使用各种常用的基本类型（String，int，boolean等都可以用）这里不做实现


# JMS的可靠性

三部分组成：`持久性`，`事务`，`签收`

> 以下实现，均有java底层代码（未整合Spring）操作activemq实现的

## 持久化

未持久化：关闭activemq后，消息队列中未持久化的消息丢失

持久化：关闭activemq后，重启activemq，那些持久化且未被消费的消息不会丢失

```java
producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT); // 生产者设置持久化模式
/**
 * 有两个可选参数（其实是两个int常量）
 * DeliveryMode.PERSISTENT		持久化
 * DeliveryMode.NOT_PERSISTENT 	不持久化
**/
```

默认值：默认使用持久化


### Queue持久化

生产者需要这么做

```java
// 生产者的这句要移动到设置持久化的语句后面，其实默认就是持久化，尝试后发现不移动也能实现
connection.start();
```

消费者无需修改



### Topic持久化

即`持久订阅`，对应了上述不持久化的`非持久订阅`

生产者需要这么做

```java
// 生产者的这句要移动到设置持久化的语句后面
connection.start();
```

消费者需要这么做

```java
// 创建connection后，设置订阅者的信息（给订阅者一个名字/id）
connection.setClientID("cx");

// 将原第5步，用session创建的consumer接收消息；修改为
// 5. 订阅topic，用TopicSubscriber接收消息
TopicSubscriber subscriber = session.createDurableSubscriber(topic, "aliasName"); // 这里需要一个别名，暂时随便顶一个别名即可（因为不知道别名是干啥用的）
Message message = subscriber.receive();
```

这么修改的理由：修改后

启动一次消费者（启动再关闭也算启动过一次），即可完成消费者订阅指定的Topic（这里称A订阅了t）

场景：消费者订阅Topic，生产者生产Topic消息

结果：所有订阅此Topic的消费者将会受到消息，A订阅过t。但是A已经关闭。当A重新启动后还是能获取到t先前发送的消息。因为A之前订阅过了t

对比：修改前不能做到，重启消费者后获取宕机期间漏收的Topic消息。修改后，消费者启动过一次，就代表指定的client订阅上了指定的Topic。就算消费者宕机，等消费者重启后还是能收到消息


> 事务偏生产者，签收偏消费者
>
> 字面意思，怎么发送消息是生产者的事。怎么签收消息是消费者的事

## 事务

> 教程上并没有对事务将过多的介绍，只讲了操作方法。因为mysql和redis已经讲过事务了

```java
// 创建session时设置是否开启事务，第一个参数为是否开启事务的参数
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
```


### 关闭事务

调用send方法后立即将消息发送至队列或主题


### 开启事务

调用send方法后需要再调用commit方法提交事务

生产者不commit，消息提交不上去

消费者不commit，获取到消息后，本应该被消费的消息，却还会存在队列中（由于没有提交事务，activemq认为消息并未被消费）


## 签收

> 设置签收方式。就像设置收快递，是本人从快递员手中亲手签收，还是快递员将快递送至快递站，本人去快递站认领即可
>
> 以下实现均在消费者处修改

```java
// 创建session时设置签收方式，第二个参数为设置签收方式的参数
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
```

可选参数有4个，常用的是前两个

`SESSIOn.AUTO_ACKNOWLEDGE`（译：自动确认），`SESSION.CLIENT_ACKNOWLEDGE`（译：客户确认）

`SESSION.DUPS_OK_ACKNOWLEDGE`（译：傻瓜确认），`SESSION.SESSION_TRANSACTED`（译：会话交易）

签收又分两种情况，未开启事务和开始事务

### 未开始事务时

`SESSION.CLIENT_ACKNOWLEDGE`在获取到消息对象（message）后，需调用message.acknowledge()方法，表示签收到此消息。如果不调用，activemq会认为消息没有被消费，继续消息保存在队列中

`SESSIOn.AUTO_ACKNOWLEDGE`自动签收消息


### 开启事务时

```java
// 创建session时设置签收方式，第二个参数为设置签收方式的参数
Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
```

`SESSION.CLIENT_ACKNOWLEDGE`在获取到消息对象（message）后，需调用message.acknowledge()方法，表示签收到此消息。如果不调用，activemq会认为消息没有被消费，继续包存在队列中

使用commit，不使用acknowledge，提交事务时也会自动签收消息。（即用不用acknowledge都会签收消息）

没有使用commit，那么使用acknowledge才会签收消息（消费消息）

# ActiveMQ的Broker

用java代码实现的一个小型activemq

作为了解，不需要使用


# 待续

