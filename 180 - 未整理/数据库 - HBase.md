# HBase 概述

HBase 是分布式、面向列（列族）的开源数据库。HBase 依赖 HDFS 提供可靠的数据存储服务， Zookeeper 为 HBase 集群提供稳定服务



适用于随机，即时读取大数据时使用。能轻松保存数十亿行，数百万列数据



# HBase 的数据存储模型

HBase 的设计理念依据 Google 的 BigTable 论文，Bigtable 是一个稀疏的、 分布式的、 持久的多维排序 map



如果说 MySQL 是一个简单的二维表格，那么 Hbase 就是多维表格，可以保存非关系型数据，并尽可能节省空间



HBase 是 kv 键值对数据库，在 HBase 的多维 map 中的映射是这样的

```
        键         ->     值
(行键, 列键, 时间戳) -> 未解释的字节数组
```





## 逻辑存储模型

假设有以下 json 格式的数据

```json
{
    "row_key1": {
        "personal_info": {
            "name": "zhangsan",
            "city": "北京",
            "phone": "131********"
        },
        "office_info": {
            "tel": "010-1111111",
            "address": "atguigu"
        }
    },
    "row_key11": {
        "personal_info": {
            "city":"上海",
            "phone":"132********"
        },
        "office_info": {
            "tel": "010-1111111"
		}
    },
    "row_key2": {
    ......
}
```

把这些数据放到二维表格中，数据就是这样的

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220625202234740.png" alt="image-20220625202234740" style="zoom:80%;" />

在二维表格中，会存在大量的空单元格。所以用 MySQL 保存这些数据会造成大量空间浪费



以上述表格为参考，HBase 根据 RowKey 把若干个行划分为一个 Region，根据列族，把若干干个列族分为一个 Store

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220624190159183.png" alt="image-20220624190159183" style="zoom: 80%;" />



## 数据模型

- namespace：相当于 MySQL 中的一个 database。一个 MySQL Server 能有多个 database；一个 HBase 能有多个 namespace
- table：一个 namespace 能有多个 table
- column 和 column family：HBase 是面向列族的数据库，在定义表结构时必须定义列族，不需要定义列。列在插入数据行时动态添加
- Time Stamp：时间戳，作为版本号存在。一条数据可以有多个版本，不同版本见用此属性标识
- cell：最小数据存储单元/最小数据读取单元，kv 对中，v 就是 cell，一个 `rowkey, column Family:column Qualifier, timestamp` 能定位到一条数据（因为有版本号问题存在，所以一个 k 可以定位到多个 cell，但通常以最新版本的 cell 作为有效数据）



默认有两张 namespace："hbase" 系统空间，"default" 默认空间

如果创建表时没有指定 namespace，默认表被放到这里



## 物理存储模型

HBase 存储数据的物理文件被称为 StoreFile（StoreFile 也叫 HFile）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220625203539316.png" alt="image-20220625203539316" style="zoom:80%;" />

在 StoreFile 文件中，每行都是一个 cell，但 cell 仅对应一个值

一个完整的数据行由多个 cell 组成



此外由于数据在 HDFS 中保存，数据仅支持追加，不支持随机修改，所以

- 数据的修改通过创建新 cell 实现，并用可用时间戳作为 版本号 标记最新的有效 cell
- 数据删除为逻辑删除，在标记位进行 “假删除”



# HBase 基本架构

了解到 HBase 的存储模型后才能展开架构的介绍

HBase Server 运行时有多个进程：master 和 region server。但 master 和 region server 不保存数据，只负责管理数据



- Master，通常和 hadoop 的 namenode 部署在同一台机子上。负责通过 ZK 监控 Region Server 进程状态，同时是所有元数据变化
  的接口。能对 region 进行的故障转移和拆分
- Region Server：通常和 hadoop 的 datanode 部署在同一台机子上。主要负责数据的处理。同时在执行 Region 的**拆分和合并**



集群中 HBase master 的监控故障转移和拆分指：

- 拆分：防止 Region Server 管理的 Region 数量不均衡，master 合理拆分任务给 Region Server
- 故障转移：有 Region Server 宕机后，master 把宕机节点的任务交给其他 Region Server 接收



> - 为了实现 master 的高可用，可以为 master 设置 backup-master
>
>   master 宕机后，原本不保存 Region Server 的 backup-master 成为 master，宕机的 master 重启后自动变成 backup-master
>
> - 为了实现 region server 的高可用，用 zk 作为服务注册中心

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220625205209039.png" alt="image-20220625205209039" style="zoom:80%;" />

> hbase 需要依赖 zookeeper。为了实现开箱即用，hbase 自带有 zk。所以默认 hbase 会自定启动自带的 zk，如果需要用自己搭建的 zk 集群，需要把这个配置禁掉
>
> 很多软件都是这样干的

region server 负责管理若干个 Region 中的数据

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220625205251157.png" alt="image-20220625205251157" style="zoom:80%;" />





# Master 架构

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220625221021945.png" alt="image-20220625221021945" style="zoom:80%;" />

- MasterWAL 会记录 master 还未执行的操作，这个日志用于备份未执行的操作，防止 master 宕机后丢失操作
- hbase:meta 表是元数据表，记录了 region 被哪些 Region Server 管理



# Region Server 架构

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220625221534275.png" alt="image-20220625221534275" style="zoom:80%;" />

- MemStore：写缓存， 由于 HFile 中的数据要求是有序的， 所以数据是先存储在 MemStore 中，排好序后，等到达刷写时机才会刷写到 HFile，每次刷写都会形成一个新的 HFile，写入到对应的文件夹 store 中
- WAL：数据要经 MemStore 排序后才能刷写到 HFile， 但把数据保存在内存中会有很高的概率导致数据丢失，所以数据会先写在一个叫做 Write-Ahead logfile 的文件中，然后再写入 MemStore 中。所以在系统出现故障的时候，数据可以通过这个日志文件重建
- BlockCache：读缓存，每次查询出的数据会缓存在 BlockCache 中， 方便下次查询  



# 数据写流程



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220625221828081.png" alt="image-20220625221828081" style="zoom:80%;" />



1. 首先访问 zk，获取 hbase:meta 表位于哪个 Region Server

2. 访问对应的 Region Server，获取 hbase:meta 表，将其缓存到连接中，作为连接属性 MetaCache，由于 Meta 表格具有一定的数据量，导致了创建连接比较慢；

   之后使用创建的连接获取 Table，这是一个轻量级的连接，只有在第一次创建的时候会检查表格是否存在

3. 调用 Table 的 put 方法写入数据，解析RowKey，对照缓存的 MetaCache，查看具体写入的位置有哪个 RegionServer

4. 将数据顺序写入（追加）到 WAL

5. 根据写入命令的 RowKey 和 ColumnFamily 查看具体写入到哪个 MemStory，并且在 MemStory 中排序

6. 向客户端发送 ack

7. 等达到 MemStore 的刷写时机后，将数据刷写到对应的 story 里



# 读数据流程



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20220625222455214.png" alt="image-20220625222455214" style="zoom:80%;" />

1. 创建 Table 对象发送 get 请求
2. 优先访问 Block Cache，查找是否之前读取过，并且可以读取 HFile 的索引信息和布隆过滤器
3. 不管读缓存中是否已经有数据了（可能已经过期了），都需要再次读取写缓存和 store 中的文件
4. 最终将所有读取到的数据合并版本，按照 get 的要求返回即可  

