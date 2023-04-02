#还没有复习 

# 常用命令

> 关键字忽略大小写，不知道的现查[redis中文网](https://www.redis.net.cn/order/)

redis中常用的数据类型 `string`, `map`, `list`, `set`, `sortedset`

常用命令

**String** 

`set key value`, `get key`, `del key` （set命令集增改命令于一体）

**list**

`lpush key value`, `rpush key value`, `lrange key start end`, `lpop key`, `rpop key `

**hashmap** （注：下面的field是字段名）

`hset key field value`, `hget key field`, `hdel key field`, `hgetall key` 

**set** （无需集合，元素不重复。key是一个set的名字，一个key中可以有好多元素）

`sadd key value`, `smembers key`， `srem key value`

**sortedset**（score是排序的依据，默认从小到大排序）

`zadd key score value`, `zrange key start end [withscores]`, `zrem key value`

**全局命令**

`keys *`, `type key`, `del key`

`select dbId` dbId是数据库的id（默认0~15）

`dbsize` 当前库中key数量

`fulshdb` 清空当前库

`fulshall` 清空所有库



# 配置文件



# 持久化



## RDB

> Redis DataBase

`dump.rdb`

可以理解为将数据备份到磁盘，通过使用该文件就可以将磁盘中的数据恢复到redis中

相关配置

`save` 后跟时间和操作数，表示指定时间内执行操作达到目标值就保存一次数据。后跟""或不配置此项均表示关闭功能 

在redis客户端使用`save`或`bgsave`均可立即保存数据（无需达到指定操作数，且这两个命令按字面意思即可知道区别）



## AOF

> Append Only File

`appendonly.aof`

文件中存着用户使用过的除读操作外的历史命令的日志，重启redis后，redis会读取该文件，达到恢复数据的效果

相关配置有

`appendonly`值为yes，表示开启aof

`appendfsync` 是啥自己看配置文件的注释



使用方式：将他们的文件放在redis目录下时，重启redis即可

两种持久化可同时使用，默认优先使用aof文件恢复数据

==注：fulhall操作（快乐删库）也会被举例到aof文件中==



# Redis的事务

---

## 是什么

一个事务是一条或多条命令的集合，这一条或者多条命令要么全部执行成功，要么全部执行失败。



## 用关事务的命令

`MULTI` 开始事务

`DISCARD` 取消事务

`EXEX` 执行事务中的命令

`WATCH key` 监视指定的key，如果==事务开始前==（在事务中key的值可以被修改）key的值被修改了，事务执行的时候会被打断，事务中所有命令执行失败

`UNWATCH` 取消对所有key的`WATCH`



## Redis事务的特点

- 全体连坐
- 冤头债主

rdis的事务执行分两种异常。就像是java中的`RunTimeException`和普通的异常（普通异常在编码阶段就要处理，运行时异常只能在运行时才会发现）

reids开始事务后，一些命令会不检查对错，先入队，在执行事务是再执行命令。另一些命令是入队前就会检查，如有异常直接打印



当执行有“普通异常”的事务时，全体连坐，所有命令不生效

当执行有“运行时异常”的事务时，冤头债主，抛异常的命令不能执行，而能正常执行的命令产生的结果将会生效

即`redis`对事务的支持是部分支持，不是完全支持



**单独的隔离操作**：事务中所有的命令多会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断

**没有隔离级别的概念**：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际的执行，也就是不存在 “ 事务内的查询要看到事务里面的更新，在事务外查询不能看到 ” 这个是让人万分头痛的问题

**不保证原子性**：Redis 同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚



## Watch监控

> watch在上述事务命令中已经简单介绍过，这里不再赘述

| 锁的名称             | 描述                                           |
| -------------------- | ---------------------------------------------- |
| 乐观锁               | 乐观。每次取数据的时候，会认为数据没有被修改过 |
| 悲观锁               | 悲观。每次取数据的时候，会认为数据都被修改过   |
| CAS（Check And Set） | 乐观锁的实现                                   |



加悲观锁，就是锁表。A在用表1，那B就不能用表1。因为悲观锁为防止取的数据是被修改过的而将表全锁上

加乐观锁，像git一样。A在改数据，结果B在A改之前改过了，那么A就要先拿到最新的数据然后再修改，提交。和git的pull，push的功能一样。因为乐观锁不会锁表，而会在改数据前检查一下能不能改



# Redis的发布和订阅（了解）

其功能就像微信公众号，订阅公众号后的用户可在公众号发送推文时接收到推文。公众号只会把推文发给订阅了它的用户

消息中间件都有这个功能，redis也能做，但是一般用redis做缓存而不做订阅和发布。一般消息中间件做，例如`activemq`

## 案例

场景：有两个redis客户端，redis-cli1（发布者），redis-cli2（订阅者）

订阅者执行

```shell
> SUBSCRIBE c1 c2 c3
```

发布者执行

```shell
> PUBLISH c2 hello-redis
```

订阅者阻塞式接收消息



订阅时也可使用通配符

```shell
> PSUBSCRIBE new*
```

发布者

```shell
PUBLISH new1 redis2015
```



# Redis的主从复制（master/slave）

> redis的主从复制解决的是读取分离的问题
>
> redis主从复制常用方式
>
> - 一仆二主。一个主机有两个从机
> - 薪火相传。从机也可以有从机
> - 反客为主。使当前数据库停止和主机的同步，并由从机转为主机

---

## 是什么

是主从复制

运行redis的主机挂了，迅速将redis的数据复制到从机上。这样就有修主机的时间了

只有主机能修改数据，从机只能读数据，不能改数据

从机使用主机的`.rdb`文件覆盖自己的文件从而做到和主机数据保持一致



## 怎么用

主机不需要修改，正常运行即可

从机需要修改配置文件`redis.conf`，指定主机的ip和端口。一旦主机的redis挂掉，从机马上补上



## 单机启动多个Redis

1. 创建多个`redis.conf`配置文件
2. 使用不同的`redis.conf`文件启动`redis`
3. 在`slave`从机中指定`master`

完成上述步骤即可完成单机集群



1. 复制多个`redis.conf`

   修改port，修改日志文件（logfile）的名字为“redis_port.log”，修改生成的rdb文件的名字（dbfile）修改pidfile文件名

2. 使用不同的配置文件启动redis

   ```shell
   $ redis-server ./redis6379.conf
   $ redis-server ./redis6380.conf
   $ redis-server ./redis6381.conf
   ```

3. 启动多个`redis-cli`连接不同的`redis-server`

   ```shell
   $ redis-cli -p 6379
   $ redis-cli -p 6380
   $ redis-cli -p 6381
   ```

4. 查看redis的集群状态，以下命令会打印当前redis的身份，主从机的地址和端口号

   ```shell
   > info replication
   ```

   ```shell
   # Replication
   role:master
   connected_slaves:0
   master_replid:9dbcba9b7769c48547860bda99e9464b717a0ef3
   master_replid2:0000000000000000000000000000000000000000
   master_repl_offset:0
   second_repl_offset:-1
   repl_backlog_active:0
   repl_backlog_size:1048576
   repl_backlog_first_byte_offset:0
   repl_backlog_histlen:0
   ```

   可见，三个`redis-server`均为`master`

5. 在两个`redis-server`中执行命令，指定主机地址（ip+端口）

   ```shell
   > SLAVEOF 127.0.0.1 6379 
   ```

   此后这两个`redis-server`均变为`slave`，且所存数据和`master`一致

即，`slave`的原数据全部丢弃，数据和`master`同步

### 单机启动多个Redis遇到的问题

1. 主机的`redis-server`执行`SHUTDOWN`关闭了服务。从机会怎么办？主机重启后，从机又怎么办？

   主机关闭后，从机还是从机（不会上位变主机），从机中数据也不会丢失。主机启动后，主机依旧能存数据，从机依旧能复制主机数据

2. 从机关闭了服务。会发生什么？从机重新启动后又会发生什么？

   其他从机正常工作。从机启动后，如果没有指定主机，这个从机将不再是从机，而是一个独立的`redis-server`。需要重新在`redis-cli`中指定主机或在配置文件中就指定主机

   

**薪火相传**

有从机的从机还是从机

**反客为主**

从机执行`SLAVEOF no one`命令后，会从`slave`转为`master`



## 哨兵模式

> 反客为主的自动版，原反客为主是要手动令从机转为主机

哨兵监控主机状态，如果主机挂掉，哨兵将选举新的主机

场景：A（master），B（slave），C（slave），D（哨兵）

A挂掉，B转为master。A重新启动，哨兵监控到A上线，A变为从机

实现步骤：

1. 写哨兵的配置文件`sentinel.conf`
2. 启动哨兵

哨兵的配置文件的内容

```shell
sentinel monitor 被监控数据库名字(自己起名字) 127.0.0.1 6379 1
sentinel monitor iammaster 127.0.0.1 6379 1
```

上面最后一个数字1，表示主机挂掉后salve投票看让谁接替成为主机，得票数多少后成为主机



# Redis集群

> 相对于Redis的主从复制。Redis解决的是内存问题，将对Redis的操作分摊到各个节点中
>
> Redis集群解决内存不够和承受并发压力的问题
>
> 下面实现的是单机创建的集群（没有使用docker）。目前还未实现单机使用docker或多机搭建集群

==下面说的主节点和主机是一个东西==

## 搭建集群流程

1. 修改配置文件，开启集群
2. 启动一堆redis
3. 使用redis的集群命令，将redis连接起来形成集群
4. 使用`redis-cli`实操



1. 修改配置文件。主机的配置文件需要开启集群，从机不需要开启集群，但是需要指定主机的地址

   ```shell
   port 6379
   daemonize yes
   pidfile /var/run/redis_6379.pid
   # 主机需要开启集群
   cluster-enabled yes
   cluster-node-timeout 15000
   cluster-config-file  nodes-6379.conf
   ```

   ```shell
   port 6389
   daemonize yes
   pidfile /var/run/redis_6379.pid
   # 从机需要指定主机地址，并关闭集群
   cluster-enabled no
   slaveof 127.0.0.1 6379
   ```

   配置`redis`集群需要至少三个节点，每个节点再配置一个从机。如果配置集群的话需要在启动`redis`前删除所有`.rdb`文件

2. 启动一堆redis。这里启动的前三个是主机，后三个是从机。

   ```shell
   $ redis-server redis6379.conf
   $ redis-server redis6380.conf
   $ redis-server redis6381.conf
   $ redis-server redis6389.conf
   $ redis-server redis6390.conf
   $ redis-server redis6391.conf
   
   $ ps -ef | grep redis
   ```

3. 使用命令将多个`redis`服务合并为集群

   ```shell
   # 主从节点的地址全都写上
   $ redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6389 127.0.0.1:6390 127.0.0.1:6391
   
   # 教程使用的是后者，但是实操时使用的是前者，且执行前者命令并没有出现奇怪的问题
   $ redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6389 127.0.0.1:6390 127.0.0.1:6391 --cluster -replicas 1
   ```

4. 使用`redis-cli`实操

   ```shell
   $ redis-cli -c -p 6379
   ```

   `-c`选项表示使用集群启动，如果没有此选项，以下操作均不可实现

   
   
   在`6379`端口下的`redis-cli`使用命令存入数据时，因为集群的原因，数据可能存到不同的节点。成功存入数据的同时当前终端也会自动跳转到存入数据的节点的`redis-cli`
   
   但是，正因如此。`redis-cli`无法执行批量操作。因为存取数据时，可能不同数据会存入不同节点或从不同的节点取数据。所以在一个节点下的`redis-cli`无法执行批量操作



## 插槽

在执行如下命令时，会打印一些集群的信息

```shell
$ redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381
```

```
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
M: 5df2139bfae322a623dfeaa7e31f8663548a9fb2 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
M: 4f358a2baf74f490e14093cdcf6422f17b95b67e 127.0.0.1:6380
   slots:[5461-10922] (5462 slots) master
M: 4efd186e801cfb226fd29215f98c1927eb035a01 127.0.0.1:6381
   slots:[10923-16383] (5461 slots) master
```

其中有关键字==Slots==（插槽）

上述打印信息显示，集群的插槽范围为0~16383。，每个插槽对应一个键值对

前三行显示了，每个节点管理的插槽范围



## 关于集群的常用命令

> 以下命令除了第一个，基本都没用过



**查看节点列表**

```shell
127.0.0.1:6379> cluster nodes
```

还有很多redis客户端执行的`cluster`命令，例如：查所有节点中键值对总数，插槽总数等，查询key所在的插槽，这里不赘述

**添加主机节点**

```shell
$ redis-cli --cluster add-node 127.0.0.1:6385 127.0.0.1:6379
```

**添加从机节点**

```shell
redis-cli --cluster add-node 127.0.0.1:6386 127.0.0.1:6379 --cluster-slave --cluster-master-id 22e8a8e97d6f7cc7d627e577a986384d4d181a4f
```

**为新节点分配插槽**

```shell
redis-cli --cluster reshard 127.0.0.1:6385
```

**删除节点**

```shell
redis-cli --cluster del-node 127.0.0.1:6386 
```

Alter user 'root'@'localhost' INDENTIFIED by 'root';

## 集群中的故障恢复

集群中也是有主节点的

场景：主节点挂掉怎么办？主节点挂掉后又重启，原主节点还是主节点吗？

主机挂掉，从机顶上（主从复制的内容）

某段插槽的`master`和`slave`全挂掉，那就是真挂了，这段插槽不能用了，用了会报error

在`redis.conf`配置文件中，可以设置如果有某段插槽不可用，集群中的节点全部变为不可用



## todo

集群中，不配置从机的配置文件，直接启动6个开启集群的redis。好像会自动变为3个主机3个从机

