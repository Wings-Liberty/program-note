01

Hive 更多的是数据增，查。因为涉及大数据数据分析，所以不处理在线业务，所以改和删的操作很少

02

hive 目标是处理海量结构化数据，所以虽然 hive 的本质是用  mapreduce 但 mapreduce 能解决的问题 hive 可能解决不了，比如处理非结构化的数据时。hive 能解决的问题，mapreduce 一定能解决

hive 相当于是 hadoop 的客户端，但为了能实现分析海量结构化数据这一具体的任务，hive 的命令封装了 Hadoop 中hdfs，mapreduce，yarn 的指令，说白了就是为了用 hadoop 实现分析海量结构化数据，封装了一个 hadoop 的客户端

hive 是客户端，而客户端是完全无状态的，hive 的集群中各个节点的 hive 不需要通信。说白了就是在运行有 hadoop 的机器上加一个 hive 网关，让 hive 接收程序发送的 HQL，并解析为 hadoop 中的动作去执行

hive 和 MySQL 一样，编译过程中解析 sql 的流程相对固定，稍微不符合语法，hive 就找不到 对应的 mapreduce 模板，所以 HQL 的写法非常固定

> 02 简单地结合 mapreduce 讲了 hive 如何工作的

表名和表的位置，这些元数据放在 hive 里

03

迭代式算法，类似嵌套查询需要的算法。hive 的调优只能挑 hql，因为他是 hadoop 的客户端，所以更细粒度的优化就只能去调 hadoop 了

04

![Pasted image 20220720095025](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220720095025.png)

JDBC 传入驱动，url，username，password创建连接，用预编译语句执行语句
JDBC 是一种操作 hive 的方式，还有很多其他的方式

![Pasted image 20220720095439](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220720095439.png)

05

hive 依赖 hadoop，hadoop 的 hdfs 的更新操作其实是版本号 + 追加数据方式更新

通常不进行 hive 的改操作

06

hive 3 相对于 2 没有太大变化，但为了保证 hive 和 hadoop 版本兼容，用 hadoop 3 就用 hive 3

教程 pdf 中，安装 hive 时有一步在解决 jar 包冲突，这个改不改都可

启动 hive 前需要先启动 hadoop。hadoop 启动过程中会经过一个安全模式阶段，等过了这个阶段再启动 hive 

推荐启动 hadoop 时，启动 hdfs，yarn，history_server（mapreduce 不需要主动启动吗？）

这节稍微说了下各个服务的默认端口号

默认 hive 的日志文件在 /tmp/user_home/hive.log 里

> tail -f 查看报错日志小技巧。进入 tail -f 后打多个空行，再去执行能复现错误的动作，错误日志会在 tail -f 的空行后展示，便于肉眼查看

hive 2 好像就不推荐用 MapReduce 了，建议用 spark，但先用 mr 学 hive 不影响学习 hive

测试插入数据时，可能会因为配置低而抛异常，可以让 hive 以 本地模式启动

hive 插入的数据在 hdfs 的 WebUI 上都是可查的。在 /user/hive/warehouse/

derby 只能建立一个客户端连接，否则就报错，所以要换 MySQL

08

在 derby 下建一个表 test，插入一条测试数据

然后换到 mysql

再 show table 时，是看不到 test 的。虽然 test 的物理文件还在 hdfs 中，但 show table  会访问元数据，元数据现在换到了 MySQL 中，mysql 中元数据现在是空的，所以查不到元数据，但在重新执行和 之前相同的 建表语句后，再调用 select * from test 时还能查到数据

所以建表语句就是在想元数据里写表名和物理文件位置的映射，并建立表的物理文件

再举个例子体会一下元数据的概念

1. 随便建立一个 txt 文件，内容尽可能保证让每行的数据和表的结构相同
2. 用 hdfs -put 把文件上传到 hive（路径就是 test 物理文件夹）
3. 再调用 select * from test，还能查到 txt 文件里的数据

表名和文件的映射存到了 MySQL 的 DBS 表和 TBLS 表里

10

默认情况下，启动的 hive 不是网络进程，没有对外暴露接口，不能被远程客户端连接，只能在本机命令行里写 HQL

hive  元数据服务访问方式指，用 hive 命令行指定另一个远程 hive连接。比如 pdf 中配置 202 机器，使其能连接 201 机器上的 hive 服务。但在命令行里操作

启动 hive server 2 时会报错，无视它，等程序自行处理报错

在程序里用 jdbc 连接 hive 时，用 hive 的驱动

![[../91 - 静态资源/Hive 2022-07-20 10.56.48.excalidraw]]

13

为了减少 hive 客户端启动时间，删掉所有链接远程 hive 的配置，直接用本地 hive 链接

如果 hive 执行命令时没有打印 mr 日志，说明没有走 mr， 查到的元数据。在执行 count(colomn_name) 时会走 mr，执行 count(\*) 或 count(1) 时没走 mr ，查询结果为 0

没走 mr，访问元数据。元数据里记录有表中数据量的数量，但只有在执行 hive 的命令时才能修改这个数据量，如果从 derby 改为 MySQL，原表里就有数据，但 mysql 中的元数据是从 0 开始计数的，而且如果用 hdfs put 向 hive 的表物理文件里写数据也不能引起 MySQL 中元数据的更新，所以如果没走 mr ，查 元数据就需要保证所有指令都要从 hive 走

19

时间类型，Java 用符合时间格式的字符串（比如 "2000-01-02"），hive 能自动转为时间类型

hive 中的 struct 相当于 Java 里的 JavaBean

hive 的集合都支持嵌套集合

21

show create table test; 就能展示之前创建表 test 时的完整建表语句

> 弹幕：企业中 查出来的数据是要存放到新表里的，所以建表还是要好好学

use database_name 会进入指定库，否则就是在默认库里进行操作

建库一般用上 if not exist 和 location，其他的选项很少用到

24

外表的数据不在本 database 里，只有元数据在 database 里

建表语句中的 as select_statement 的作用是把一个查询语句的查询结果作为输入，建立一张表

表的元数据可以在 MySQL 的 TBLS 表里看

内部表就是管理表

平时用的表多是外部表，只有临时表才会用内部表

27

每个库对应一个物理文件夹

每个表也对应一个物理文件夹

表文件夹下的所有文件，无论文件后缀名如何，查询表数据时，只要文件中每行数据符合建表时字段之间的分隔符，数据行直接的分隔符，集合元素之间的分隔符，kv之间的分隔符，这些数据就算是表中数据，能参与查询任务的筛选流程

比如 create table test1(id int, name string)，字段之间的分隔符是逗号

文件里一行是 `1101,tomcat`

他就能作为 test 表中数据，并参与此表的查询

如果类型转换出错，文件中的值不能转换为指定的类型，查询出的值就会变成 NULL

28

alter table 改的是表的元数据，而没有改表的数据

29

因为直接用 hdfs put 不会修改表的元数据信息，所以不能用 put 命令执行导入数据的行为

用 load data 时，要么用 local 选项加载本地 hdfs 系统外的文件载入 hive，要么不用 local 选项，加载 hdfs 系统内的文件载入 hive

> 有人说用 mr 查询时很慢，用 spark 换掉 mr 作为新引擎会提高很多速度

33

Import 和 Export 导入导出数据命令暂时用不了，~~因为需要比较新的版本的 hive~~


38

测试数据里的 loc 是位置 location 的缩写

mgr 是 manager 上级领导

55

sort by 只能实现部分有序，每个 reduce 里的数据都是有序的。就是外排，但就差最后一次总排序

sort by 会采用某种把数据分散到多个 reduce 里，然后再在组内排序，分配规则好像是随机的

Distribute By 用于让数据按照分散规则分散到分区内，分散规则是根据 key 的哈希值取模算的

通常这两个 by 同时用，先用 distribute by 再用 sort by，这样就能做到先让数据根据哈希值取模分散到各个分区里，再在分区里排序。比如获取每个部门中所有员工的信息，并让同一个部门内的员工按工资降序排列。distribute by dept_id sort by emp_sal，这和 order by dept_id, emp_sal 相比，前者能用分区机制进行并行排序，大数据下速度更快

58

工作中经常需要分析某天或近一段时间内的数据，所以常用 hive 以时间为依据进行数据分区，比如把每天的数据放到一个物理文件夹里

如果用 select * from dept_partition 查询分区表，默认会查询所有分区，即如果没在 where 里指定分区字段，就会全表全分区扫描

59
很少用分区的 curd 命令，分区通常是跟随 load 命令一块加进来的

61

二级分区就是在表物理文件夹里创建了两层目录

假设表名为 test，向表中 load data 多个文件 tab1.txt，tab2.txt...，表和数据的物理文件层次结构为

```
L test
	L tab1.txt
	L tab2.txt
	...
```

假设表名为 test，一级分区要求每天的数据保存在自己的分区中
```
L test
	L 2022-01-02
		L tab1.txt
		L tab2.txt
	L 2022-01-03
		L tab1.txt
		L tab2.txt
	L 2022-01-04
		L tab1.txt
		L tab2.txt
```

二级分区要求每小时的数据保存在自己的分区里

```
L test
	L 2022-01-02
		L hour1
			L tab1.txt
			L tab2.txt
		L hour2
			L tab1.txt
			L tab2.txt
	L 2022-01-03
	L 2022-01-04
```

所以多分区其实就是嵌套文件夹，包含数据文件的文件夹就是最小的分区单元

62

因为数据文件的结构非常简单，且能直接阅读，且文件位置也非常明确。所以存在以下情况


- 手动调用 hdfs mkdir 创建分区目录，用 hdfs put 向分区目录上传数据文件
- 用 load data 加载文件到分区目录

只有第 4 种情况属于合规合法行为，上传数据后，新的数据能被 select 查询到

其他 3 种方式都需要对表进行修复，或者说需要让新的数据文件和分区创建关联关系（数据文件和分区目录的关系不是简简单单的让数据文件在分区目录下就行了，而是需要把这些数据作为元数据被记录到 hive 里、准确来说是元数据表里，一般就是MySQL）

单纯地越过 hive 直接更新物理文件和分区目录是不能引起 hive 元数据的更新

63

提到了 loda data 时没有指定分区时会发生什么错误，如何解决，为什么会出错

64

静态分区指，插入数据时数据被插入哪个分区需要在插入指令里显式指定，分区目录也需要手动创建

现在希望实现，比如一级分区按天归档数据。希望来到新的一天时，hive 能自行创建新分区并把数据放到新分区里

在开启动态分区后

```sql
insert into table dept_partition_dy partition(loc) select deptno, dname, loc from dept;
```

让 select_list 的最后一个字段是分区字段

65

如果 insert into ... select 没有指定数据插入哪个分区。在 hive 3 前 sql 会报错，3 后自动进行动态分区，把数据根据查询到的数据的分区字段放到合适的动态分区里

前提是开启不严格模式的动态分区，这能节省不少事

66
分桶表不是重点，企业用的也不多

创建分桶表的关键字是 clustered by 和 into ... buckets

67

hive 内置函数可[查询此处](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/LanguageManual_UDF.html#:~:text=%E6%AE%B5%E4%B8%AD%E7%9A%84%E6%95%B4%E6%95%B0%E3%80%82-,Built%2Din%20Functions,-Mathematical%20Functions)

内置函数被分为三大类 UDF - 一进一出，UDAF - 聚合函数 多进一出，UDTF - 一进多出

比如用 count(dep_id) ... group by dep_id  时，查询出的多条具有相同 dep_id 的数据会被合并为一条并加入结果集

70

目前为止，只要 hive 的列里存的值是基本值类型（字符串，数字，日期，布尔值等）HQL 和 MySQL的 SQL 语法，常用函数均无差异

即使涉及到了分区，分区字段在 HQL 中也像 普通字段一样出现在 where_list 里

如果hive 的列里存的是数组，javabean 就另说了，其实区别也不大

71

行转列函数和列转行函数的没有明确的定义

有些帖子和 pdf 上行转列和列转行的定义刚好相反，但只要理解这俩是啥意思就行了

concat 能传入字符串常量，也能用查询的字段。

比如希望把学生信息中学号和名字两列合并到一列里

```sql
select concat(id, '-', name) from student;
```

pdf 将其归为行转列

72

collect_set 和 collect_list 函数，MySQL 没有同名函数，但可能有同功能函数

用了他们后 select_list 就还能有其他字段

73

explode 函数类似，用了他们后 select_list 还能有其他字段，不过需要侧斜关键字 lateral view 的支持

74

行转列，列转行的叫法并不好理解

多列合并为一列（concat），一行展开成多行（collect_list/set）。这样的叫法更好理解

75

开窗函数有点像 group by，但有区别。开窗函数也能根据列值分组

在 MySQL 中

```sql
SELECT s_name, COUNT(*) FROM student GROUP BY s_name
```

它的结果是获取所有名字（无重复）并对 s_name 相同的数据行行数进行计数，结果集是多行，每行一个 s_name

在 hive 中

```sql
SELECT s_name, COUNT(*) AS cnt OVER() FROM student
```

MySQL 中用了 count(\*) 就必须用 group by，但 hive 用了 over 后不用 group by

hive 中，count(\*) 会读取结果集的总数据行，over() 函数把 cnt 的值赋值给结果集的所有行

在 group by 下，把数据集分为 n 组，最终会展示 n 行数据，聚合函数的返回值统计计算的是每组内的数据

在 over 下。聚合函数的返回值统计的是整个结果集中的数据（前提是 over 里啥参数也没有）

pdf 的第一个例题中，如果题目要求的是求 4月份每个客户的购买次数，那不需要 over，用 group by 就行了

```sql
select name, count(*) from tab where substring(data, 0, 7)='2017-04' group by name
```

但题目要求是求所有客户和总人数，就需要用 over 了

76

期望的结果是这样的

![Pasted image 20220721102953](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220721102953.png)


```sql
select name, `date`, sal, sum(sal) over(partition by name) from tab 
```


over(partition by name) 要求 sum 函数根据 name 分组计算累加和

77

效果图

![[../91 - 静态资源/Pasted image 20220721103800.png|500]]

写法一
![[../91 - 静态资源/Pasted image 20220721104006.png|500]]

pdf 中给了很多种写法

官方给的 over() 内的语法[在这里](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+WindowingAndAnalytics)

官方说，当指定了 order by，而没有指定窗口范围时，默认聚合函数的统计范围是组内第一个数据当当前行数据。所以写法一和加了指定范围是从第一行到当前行的效果是一样的

假设统计范围是当前行和前后一行，共计 3 行数据时，能这样指定

![Pasted image 20220721104823](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220721104823.png)

78

当排序值相同时，需要注意排序值相同的数据行的开窗范围

![Pasted image 20220721105439](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220721105439.png)

这是 `select id, sum(id) over(order by id) from tab` 的结果

79

![[../91 - 静态资源/Pasted image 20220721110329.png|425]]

弹幕有人说这些查询有些能用子查询实现，还有人说开窗比子查询效率高
![[../91 - 静态资源/Pasted image 20220721110616.png|500]]

举个实际案例

数据库里记录了用户访问站点的记录

用户会从 A 站点上的链接挑转到 B ，C，D 站点

现在计算单跳转率，比如  A 跳转到 B 的跳转率 = A -> 的次数 ➗ A 跳转到其他站点的总次数

80

思路是这样的

1. 按时间排序
2. 把数据分成 5 组，取第一组

把数据分成 5 组可以用 ntile 函数，给所有行添加伪列，列值为组号。ntile(5) 表示会把数据分成 5 组

![[../91 - 静态资源/Pasted image 20220721111654.png|475]]

效果如下

![[../91 - 静态资源/Pasted image 20220721111708.png|500]]

分区函数其实平时还是 partition by 和 order by 用的更多，其他几个用的都不多

81

> 在对分数相同的数据行进行 rownum 函数时，分数相同的数据行在结果集中逆序出现。这是 mr 的 reduce 环形缓冲区干的，如果是 spark 就没有这个问题

求各科前 3 名

![[../91 - 静态资源/Pasted image 20220721113130.png|500]]

如果没有开窗函数，怎么办？这也是 mysql 的经典面试题，比较麻烦

> mysql 8 引入了窗口函数，8 之前没有窗口函数的话很麻烦

86

用 hive 做一个 wordcount。答案在 92

数据如下

![Pasted image 20220721114916](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220721114916.png)


grouping_set 函数用于多维分析

比如：希望计算部门男女数量，计算每个部门的人数，按照不同角度进行聚合函数计算

可以参考[文档](https://cwiki.apache.org/confluence/display/Hive/Enhanced+Aggregation%2C+Cube%2C+Grouping+and+Rollup#:~:text=the%20Language%20Manual.-,GROUPING%20SETS%20clause,-The%20GROUPING%20SETS)

如果希望这些数据能出现在一个结果集里，比如计算部门总人数 ，每个部门的人数，每个部门男的人数和女的人数

![Pasted image 20220721120721](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220721120721.png)

![Pasted image 20220721120728](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220721120728.png)

87-90 跳过自定义函数，自定义函数需要在 Java 代码里写自定义函数父类的子类再打 jar 包放 hive 的 lib 里，有这个时间还不如直接在应用层暴力处理


至此，应用都讲完了，就剩压缩和优化了


95-100

压缩也是优化的一种，暂时跳过

