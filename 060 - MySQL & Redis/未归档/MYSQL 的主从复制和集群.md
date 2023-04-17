#还没有复习 

# 前言

以下实现均使用`docker`创建`mysql`服务



# MYSQL主从复制

不使用主从复制。一个`mysql`服务挂掉后，凡是使用此服务的程序连接数据库时均会出现异常

使用主从复制。一个`mysql`作为主机，多个`mysql`服务作为从机。从机中的`mysql`相当于主机`mysql`的备份。向主机中插入一条数据，从机中也会插入一条数据



## MYSQL主从复制的简介

主机执行插入或更新数据的操作将被记录到一个二进制文件中，从机不断复制此文件到从机中，并根据此文件更新从机中`mysql`的数据

`mysql`的主从复制不是实时的，在主从复制的过程中涉及多次IO操作，比较耗时

因为延时问题，所以插入数据后不要马上就去查数据（这里的延时是毫秒级的，虽然并不慢，但是也不要立即就查数据）



注意：先搭建主从复制，再建库建表。因为如果主机先建库再搭建主从，主机建表，查数据。从机会报早不到数据库的错误



## MYSQL主从复制的实现

>  运行环境：ubuntu18中使用docker的mysql5.7镜像

实现流程：

1. 创建并修改主机，从机的配置文件
2. 启动程序
3. 主机命令行创建用户（此用户用于让从机使用）
4. 从机命令行指定主机和主机上的用户
5. 从机命令行开启主从复制

### 修改配置文件

**修改主机的配置文件**

创建`my-master.cnf`。文件具体位置根据实际情况而定

1. 设置主机的ID`server-id=1`
2. 指定二进制日志文件存放路径`log-bin=mysql-bin`。此路径的最后不是目录名，而是日志文件名（不需要文件拓展名）
3. 设置不需要复制的数据库`binlog-ignore-db=mysql`。`mysql`是`MYSQL`的系统库，没有复制的必要 
4. 设置需要复制的数据库`binlog-do-db=test_db`
5. 设置`logbin`的格式`binlog_format=STATEMENT（默认）`

```properties
[mysqld]

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/

port=3306

server-id=1
log_bin=mysql-bin
# 不需要复制的数据库的名字
binlog-ignore-db=mysql
# 需要进行主从复制的数据库的名字
binlog-do-db=test
# 设置logbin的格式
binlog_format=STATEMENT
```

**修改从机的配置文件**

创建`my-slave.cnf`

1. 指定从机的ID`server-id=2`
2. 指定日志文件名`relay-log=mysql-relay`。从机将会把主机的二进制文件的内容拷贝到此文件中

```properties
[mysqld]

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/

port=3306

server-id=2
relay-log=mysql-relay
```



### 准备docker容器

主机`mysql`在`docker`宿主机上运行，从机`mysql`在`docker`容器中运行

创建主机容器（将上面创建的配置文件以数据卷的方式覆盖掉`mysql`镜像中的配置文件）

```shell
# 这里就指明root用户的密码为'root'
$ docker run -d -p 3307:3306 --name mysql-master -v /etc/mysql/my-master.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=root mysql:5.7
```

创建从机容器

```shell
# 这里就指明root用户的密码为'root'
$ docker run -d -p 3308:3306 --name mysql-slave -v /etc/mysql/my-slave.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=root mysql:5.7
```



### 主机执行命令

启动或重启`MYSQL`的主机和从机。并进入mysql客户端中执行以下所有命令

1. 进入主机的`mysql`

   ```shell
   $ docker exec -it mysql-master /bin/bash
   $ mysql -uroot -p
   ```

2. 创建一个用户，让从机登录

  ```mysql
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' IDENTIFIED BY 'slave';
  ```

2. 查看主机状态，记录下File和Position的字段值

  ```mysql
show master status;
  ```

### 从机执行命令

1. 进入从机的`mysql`

   ```shell
   $ docker exec -it mysql-slave /bin/bash
   $ mysql -uroot -p
   ```

2. 指定主机地址和用户

   ```mysql
   -- MASTER_HOST 指定MYSQL主机的ip MASTER_PORT 指定MYSQL主机服务所开放的端口（这里主机的mysql服务的端口被修改为了3307）
   -- MASTER_USER MASTER_PASSWORD 指定MYSQL主机上的用户名和密码
   -- MASTER_LOG_FILE MASTER_LOG_POS 这两个的值为上述查看主机状态时File和Position的字段值
   CHANGE MASTER TO MASTER_HOST='39.107.101.13',MASTER_PORT=3307,MASTER_USER='slave',MASTER_PASSWORD='slave',MASTER_LOG_FILE='mysql-bin.000003',MASTER_LOG_POS=438;
   ```
   
3. 启动主从复制

   ```mysql
   start slave;
   ```

4. 查看与主机的连接状态

   ```mysql
   show slave status \G;
   ```

  如果`Slave_IO_Running`和`Slave_SQL_Running`的字段值均为`Yes`，那主从复制就搭建成功了



**测试**

在主机上建库建表，插入数据。在从机上查库查表，查数据。看看主从机上的数据是否 一致。



## MySQL主从复制中存在的问题

场景：搭建一个`MySQL`的主从复制，一台主机A，两台从机为B和C

- 读写操作都在A上吗？
- A挂掉，从机会自定转为主机吗？
  - 如果A挂掉，B转为主机。C会跟随B，将B作为主机吗？
  - 如果A重新启动，B和C会自动重新认定A作为主机吗？
- B挂掉，B重启，指定主机。B宕机期间错过插入的数据能补回来吗？

==以上问题均未解决==



# MYSQL集群和MyCat


## MyCat介绍

`MyCat`：一个数据库中间件，用于数据库的集群的搭建

`MyCat`不光可以搭建`MYSQL`的集群，还可以搭建其他数据库的集群。如`Redis`

![[../../020 - 附件文件夹/Pasted image 20230402101225.png|500]]

`Spring`项目连接`MYSQL`时需要导入相关的依赖并在配置文件中指定`MYSQL`服务所在的服务器ip，端口，`MYSQL`的用户名，密码

现在`Spring`项目想连接`MyCat`时只需要添加`MyCat`的相关依赖即。配置文件的写法和以前一样，只需将`MyCat`服务看做一个`MYSQL`一样，在配置文件中指定ip，端口，用户名，密码即可

由以上描述可见，==真正执行数据增删改查的是`MYSQL`，而`MyCat`只是在做门面==

`逻辑库`	`MyCat`是逻辑上的数据库，程序表面上连接的是`MyCat`其实连接的是它后面的`MYSQL`集群

`物理库`	`MYSQL`是实际干活和存数据的数据库是物理库

**举例**

现有`MyCat`创建的逻辑库`mycat_db`，它有`table1`,`table2`,`table3`,`table4`

其实`mycat_db`背后有两个物理库`mysql-db1`，`mysql-db2`

`table1`,`table2`是物理库`mysql-db1`中的表，`table3`,`table4`是物理库`mysql-db2`中的表

这样就形成了`MYSQL`的集群



**对于MyCat的命令行操作**

在服务器上配置好`MyCat`后，像使用`MYSQL`一样使用`mysql -uname -p -P <port>`登录即可。当然，这里的端口是`MyCat`指定的

表面上你是在使用`MYSQL`的命令操作`MyCat`，实际上你执行的命令，`MyCat`都会将命令发送到它后面的`MYSQL`集群中执行并获取其返回的结果集



## MyCat的配置文件

`MyCat`有三个核心配置文件

`schema.xml`	配置逻辑库，读写库，分库，分表（分片）规则

`server.xml`	配置mycat的端口，用户等

`rule.xml`		配置分片规则



## 创建docker容器

```shell
# 创建两个容器
$ docker run -d -p 3307:3306 --name mysql-nd-1 -v /etc/mysql/mysql.conf.d/mysqld.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=root mysql:5.7

$ docker run -d -p 3308:3306 --name mysql-nd-2 -v /etc/mysql/mysql.conf.d/mysqld.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=root mysql:5.7
```



## 修改配置文件

- `server.xml`

  ```xml
  <!-- 在mycat标签中添加以下子标签 -->
  <user name="mycat">
      <!--设置密码-->
      <property name="password">mycat</property>
      <!--设置逻辑库的库名-->
      <property name="schemas">TESTDB</property>
  </user>
  ```

- `schema.xml`

  ```xml
  <!--测试-->
  <mycat:schema xmlns:mycat="http://io.mycat/">
      
      <!-- 逻辑库 -->
  	<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100">
  		<!-- 指定物理表。name是物理表的真实表名 -->
  		<table name="dpt" dataNode="dn1"></table>
  	</schema>
      
      <!-- 物理库  属性database的值为物理库名  注：虽然有子标签dataHost，但是还是要在属性中显式指定-->
      <dataNode name="dn1" dataHost="host1" database="test"/>
          <dataHost name="host1" maxCon="1000" minCon="10" balance="2" writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
  		<heartbeat>select USER()</heartbeat>
          <!-- 可以有多个写库 -->
  		<writeHost host="hostM1" url="39.107.101.13:3307" user="root" password="root"></writeHost>
          <writeHost host="hostM2" url="39.107.101.13:3308" user="root" password="root"></writeHost>
              
          <!-- <writeHost host="hostM1" url="39.107.101.13:3307" user="root" password="root"> -->
  			<!--读库（从库）的配置 -->
  			<!-- <readHost host="hosts1" url="39.107.101.13:3308" user="root" password="root"></readHost> -->
  		<!-- </writeHost> -->
              
  	</dataHost>
      
  </mycat:schema>
  ```



补充

- 在`schema`标签中写`table`子标签，即指定物理表
- 在`schema`标签的`dataNode`属性中指定一个`dataNode`的`name`，这意味着物理库将使用`dataNode`中指定的物理库中所有的表



## 启动mycat

```shell
# 两种启动方式，前者为前台启动，后者为后台启动。mycat是java写的，所以前台是java程序输出的日志
$ mycat consloe
$ mycat start

# 执行此命令，打印mycat可使用的选项
$ mycat
```

```shell
# MyCat默认开启的端口号为8066（数据窗口）和9066（后台管理窗口）。日常操作使用8066端口即可
# 请指定ip发，否则可能报Access denied
$ mysql -umycat -p -P 8066 -h ip
```

==在mycat中操作数据库时，不建议使用可视化工具（sqlyog已经出现不兼容问题，表显示不全，分库分表出现异常等）。建议在命令行下执行命令==



## 测试

分表后的查询结果将只查找一个库

```mysql
select * from dpt;
```


## 读写分离&主从复制



mycat中配置读写库，将读写的操作通过算法分配到不同的库中

主机备机：在mycat中的体现不明显。一个datanode标签中写多个writeHost即为配置一台主机和多台备机，在mysql中要搭建这些mysql的主从复制

但是由于读写库中，读库必须要和写库的数据一致，所以读写库之间也要搭建主从复制



在`schema.xml`中配置`dataNode`标签的子标签`writeHost`中配置子标签`readHost`

```xml
<!-- 物理库 注：虽然有子标签dataHost，但是还是要在属性中显式指定-->
<dataNode name="dn1" dataHost="host1" database="test"/>
	<dataHost name="host1" maxCon="1000" minCon="10" balance="2" writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
    <heartbeat>select USER()</heartbeat>
	
    <!-- 配置读写库 -->
    <writeHost host="hostM1" url="39.107.101.13:3307" user="root" password="root">
        <!--读库（从库）的配置 -->
        <readHost host="hosts1" url="39.107.101.13:3308" user="root" password="root"></readHost> 
    </writeHost>
</dataHost>
```

配置`schema.xml`中`dataHost`标签的`balance`属性

0—不开启读写分离，所有读操作都在`writeHost`中

1—所有读库和stand by writeHost写库均参与读操作（双主双从中M1-S1，M2-S2。M1是主机，M2是备机。除M1外均参与读操作）

2—读操作随机分配到所有读写库中

3—读操作随机分配到读库中，写库不参与读操作



为检测从机是否配上去了，需先设置上述`balance`的值为2（所有读操作都随机的在 writeHost、readhost 上分发）可执行以下命令

```sql
insert into user (name) values (@@hostname)
```

主从机在不同的docker容器中，所以hostname是不同的

插入成功后，多次执行查询语句，即可发现执行select的mysql是随机的



## 分库

> 分库：将一个库中的多个表分到多个库中



.用户常用信息表，用户等级信息表，用户消费信息表  放到用户库中

订单表，订单详情表  放到订单库中

将表分到不同的库中，但是相关的表放在一个库中，方便`join`多表查询



**怎么分库**

修改`schema.xml`文件，配置分库

在`schema`标签中使用多个`dataHost`或添加多个`table`子标签

效果：在`mycat`的客户端中使用`show tables;`会打印原本不在同一个库中的表。这也意味着可以在mycat的一个逻辑库中操作多个物理库中的表



## 分表

> 分表（也叫分片）：不同的库中创建同一张表。即一张表的数据分到了多个库中



A库中用户表中数据过多，在B，C等其他库中创建表结构相同的用户表。即，将用户表数据分到多个库中，从而减轻单个库的压力



**怎么分表**

修改`schema.xml`文件，配置分表

在`schema`标签中使用多个`dataHost`或添加多个`table`子标签



在`schema.xml`的schema标签的table子标签中指定属性rule的值

rule属性将根据指定的规则决定在插入数据时，数据将被插入到哪个库的表中

比如，根据插入的数据的 customer_id % 2 的值将数据插入到某个库中

```xml
<tableRule name="mod_rule">
     <rule>
         <columns>customer_id</columns>
         <algorithm>mod-long</algorithm>
     </rule>
</tableRule>

…

<function name="mod-long" class="io.mycat.route.function.PartitionByMod">
     <!-- how many data nodes -->
     <property name="count">2</property>
</function>
```

查询数据时，mycat也会根据id计算此次查询应该使用哪个库



### 跨库join

订单表要和订单详情表进行多表查询

向订单表插入一条数据 的同时 也会向订单详情表中插入多条数据

因为以后需要对订单表和订单详情表及性能夺标查询，所以一条订单数据和多条订单详情数据必须在同一个库中



```xml
<table name= "orders" dataNode="dn1,dn2" rule="mod_ _rule" >
	<childTable name= "orders_ detail" primaryKey= "id" joinKey= "order_id" parentKey="id" />
</table>
```

使用`childTable`标签，声明插入订单数据后，订单详情数据也会插入到同一个库中。





### 全局表

A库B库均有表1，且他们的数据是同步的，一致的。而不是AB两库中表的数据的和是总数据

所以全局表数据量小，更新不频繁

在`schema.xml`的schema标签的table子标签中指定属性type的值为"global"（type="global"）



## 全局序列

> 用于解决插入的数据生成的id是唯一的

向一张表插入多条数据时，多条数据可能在不同库的同名表中被创建。如果设置数据的id为自动增长。那么会出现不同库的同名表中有数据的id是相同的情况

全局序列就是用于解决这个问题的。即创建一个id生成器，以保证插入数据时，即使数据是在不同库的同名表中被创建的，数据的id也会是唯一的。



三种方式。本地文件，时间戳，数据库，

### 本地文件（不推荐）

mycat原本的作用是拦截sql语句，然后 把sql语句分派到mysql服务器上。

本地文件就是，mycat拦截sql时，创建id，id为自动增长。即所有库，所有表的所有数据的id均为mycat生成的，且所有数据的id均不同（传统方式中，A表和B表是两张不同的表，所以A表有id为3的数据，B表也有id为3的数据并不会造成冲突。本地文件方式中，所有数据的id将均不相同）id增长到多少了，只有mycat知道

不推荐理由：mycat的主机挂掉后，即使mycat的从机顶上也会出现id号不同步的情况（原mycat的主机记录的id已经创建到101了，mycat的从机顶上后其id的 记录却到100。这将会出现mycat将id-100重复使用的情况）



### 时间戳（不推荐）

使用系统的当前时间戳作为id。

不推荐（时间戳有18位，对于id来说过长）



### 数据库

在数据库中创建一张表，表中数据在此成为“数据库序列”

执行插入操作时，数据库序列递增。所有插入操作均使用“数据库序列作为id”



### 自定义

比如使用redis维护一个数据作为全局序列

