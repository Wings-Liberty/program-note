#还没有复习 

# Redis的主从复制

> redis的主从复制解决的是读取分离的问题。或者说解决的是数据备份的问题。
>
> redis主从复制常用方式
>
> - 一仆二主。一个主机有两个从机
> - 薪火相传。从机也可以有从机
> - 反客为主。使当前数据库停止和主机的同步，并由从机转为主机（从机执行`SLAVEOF no one`命令后，会从`slave`转为`master`）


## 介绍

运行redis的主机挂了，迅速将redis的数据复制到从机上。这样就有修主机的时间了

只有主机能修改数据，从机只能读数据，不能改数据

从机使用主机的`.rdb`文件覆盖自己的文件从而做到和主机数据保持一致


**搭建流程：**

主机不需要修改，正常运行即可

从机需要修改配置文件`redis.conf`，指定主机的ip和端口。一旦主机的redis挂掉，从机马上补上


## 单机启动多个Redis

1. 创建多个`redis.conf`配置文件
2. 使用不同的`redis.conf`文件（这些配置文件中配置的端口号必须不同）启动`Redis`
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



## 单机启动多个Redis遇到的问题

1. 主机的`redis-server`执行`SHUTDOWN`关闭了服务。从机会怎么办？主机重启后，从机又怎么办？

   主机关闭后，从机还是从机（不会上位变主机），从机中数据也不会丢失。主机启动后，主机依旧能存数据，从机依旧能复制主机数据

2. 从机关闭了服务。会发生什么？从机重新启动后又会发生什么？

   其他从机正常工作。从机启动后，如果没有指定主机，这个从机将不再是从机，而是一个独立的`redis-server`。需要重新在`redis-cli`中指定主机或在配置文件中就指定主机


## 哨兵模式

> 反客为主的自动版，原反客为主是要手动令从机转为主机

哨兵监控主机状态，如果主机挂掉，哨兵将选举新的主机

场景：A（master），B（slave），C（slave），D（哨兵）

A挂掉，B转为master。A重新启动，哨兵监控到A上线，A变为从机

实现步骤：

1. 写哨兵的配置文件`sentinel.conf`（配置文件的内容只需要下述一行配置即可）
2. 启动哨兵`redis-sentinel ./sentinel.conf`

哨兵的配置文件的内容

```shell
# sentinel monitor 被监控数据库名字(自己起名字) 127.0.0.1 6379 1
sentinel monitor iammaster 127.0.0.1 6379 1
```

上面最后一个数字1，表示主机挂掉后salve投票看让谁接替成为主机，得票数多少后成为主机


`sentinel.conf`的其他可配置项在这里

```
# Example sentinel.conf
 
# 哨兵sentinel实例运行的端口 默认26379
port 26379
 
# 哨兵sentinel的工作目录
dir /tmp
 
# 哨兵sentinel监控的redis主节点的 ip port 
# master-name  可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
# quorum 当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
  sentinel monitor mymaster 127.0.0.1 6379 2
 
# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
 
 
# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000
 
# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，
这个数字越小，完成failover所需的时间就越长，
但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。
可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1
 
 
 
# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#3.当想要取消一个正在进行的failover所需要的时间。  
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
# 默认三分钟
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000
 
# SCRIPTS EXECUTION
 
#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
#对于脚本的运行结果有以下规则：
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
 
#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，
这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，
一个是事件的类型，
一个是事件的描述。
如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
#通知脚本
# sentinel notification-script <master-name> <script-path>
  sentinel notification-script mymaster /var/redis/notify.sh
 
# 客户端重新配置主节点参数脚本
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
# 以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# 目前<state>总是“failover”,
# <role>是“leader”或者“observer”中的一个。 
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
 sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```


# Redis集群

> 相对于Redis的主从复制。Redis解决的是内存问题。将使用者对Redis的操作分摊到各个节点中
>
> Redis集群解决内存不够和承受并发压力的问题
>
> 下面实现的是单机创建的集群


## 介绍

![[../../020 - 附件文件夹/Pasted image 20230402103557.png|500]]

由上图可知，`redis-cli`操作的是集群。当`redis-cli`向集群中插入一对新的键值对时，数据具体存放到哪台服务器中由集群决定。所以使用者只需要像操作单个`Redis`服务一样，操作集群就可以了。

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
   pidfile /var/run/redis_6389.pid
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

其中有关键字==slots==（插槽）

上述打印信息显示，集群的插槽范围为0~16383。，每个插槽可以存放一对键值对

前三行显示了，每个节点管理的插槽范围


## 关于集群的常用命令


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
$ redis-cli --cluster add-node 127.0.0.1:6386 127.0.0.1:6379 --cluster-slave --cluster-master-id 22e8a8e97d6f7cc7d627e577a986384d4d181a4f
```

**为新节点分配插槽**

```shell
$ redis-cli --cluster reshard 127.0.0.1:6385
```

**删除节点**

```shell
$ redis-cli --cluster del-node 127.0.0.1:6386 
```

