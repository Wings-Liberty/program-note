# 索引的概念

索引是什么：索引(Index)是帮助MySQL高效获取数据的**数据结构**



`Mysql`默认使用的索引结构式`B+Tree`

`B+Tree`是`B-Tree`（多路平衡查找树）的加强版



[二叉树，平衡二叉树，B-Tree和B+Tree](https://blog.csdn.net/weixin_41948075/article/details/100180136)



# 索引在MySQL中的表现形式



## 查看表所使用的存储引擎



存储引擎作用于表，你可以查看某张表的建表语句从而得知这张表由那个存储引擎管理

例如：`show create table user \G;`

![[../../020 - 附件文件夹/Pasted image 20230402101458.png|500]]

建表时默认使用的存储引擎是`InnoDB`



## 索引文件



进入`mysql`执行命令，获取到数据库的数据存放在哪个目录下

`SHOW VARIABLES LIKE '%datadir%';`

数据库中的数据和索引分布如下

```
L dataDir data的实际路径
  L database1 数据库1
  L database2 数据库2
    L tableName1.frm 表结构文件
    L tbaleName1.idb 使用InnoDB引擎的表会有此文件，文件包含表数据和索引
    L tableName2.frm
    L tableName2.MYD 使用Myisam引擎的表会有此文件，文件包含表中数据
    L tableName2.MYI 使用Myisam引擎的表会有此文件，文件包含索引
  ...
```



## Myisam中的索引

![[../../020 - 附件文件夹/Pasted image 20230402101513.png|625]]

`Myisam`中的索引无主次之分，索引的`B+Tree`数据结构的叶子节点存放key值对应的数据的地址。地址指向`xxx.MYD`文件



## InnoDB中的索引

![[../../020 - 附件文件夹/Pasted image 20230402101524.png|700]]


图中左边是主键索引的`B+Tree`，右侧是辅助索引的`B+Tree`

主键索引的叶子节点存的是数据的地址，辅助索引的叶子节点存的是主键的值、

当用到辅助索引时，会用搜索结果命中主键值 在主键索引中找数据的地址



## 聚簇索引和非聚簇索引



**聚簇索引**

聚簇索引并不是一种单独的索引类型，而是一种数据存储方式
术语‘聚簇’表示数据行和相邻的键值聚簇的存储在一起



如果主键索引的 key 是有顺序的id，那么主键索引存数据的方式就是聚簇索引。具体表现为，表中数据行的id是有序的



这样做的好处：按照聚簇索引排列顺序，查询显示一定范围数据的时候，由于数据都是紧密相连，数据库不不用从多个数据块中提取数据，所以节省了大量的 IO 操作



[主键和索引的关系](https://blog.csdn.net/kangsenkangsen/article/details/51234600)



**非聚簇索引**

非聚簇索引又称辅助索引，二级索引

聚簇索引是一级索引



# 索引的种类

`主键索引`	`MYSQL`的表中的主键即为主键索引。`InnoDB`会自动为主键创建主键索引

`唯一索引`	索引的key是`unique`的就是唯一索引。唯一索引对应的字段的字段值无重复，可为 null

`普通索引`

`全文索引`



根据一个索引包含的 key 的个数，又可以将索引分为

`单值索引`	只包含一个字段的索引

`复合索引`	包含多个字段的索引



# 索引的增删查命令

**创建索引**

```sql
CREATE [UNIQUE] INDEX [索引名] ON 表名(字段列表)
-- 例如：创建单值索引和复合索引
CREATE INDEX name ON emp(`name`)
CREATE INDEX idx_age_deptid_name ON emp(id, age, deptid, name) 
```

**删除索引**

```sql
DROP INDEX 索引名 ON 表名
-- 例如：
DROP INDEX idx_age_deptid_name ON emp
```

**查询表中有哪些索引**

```sql
SHOW INDEX FROM 表名
-- 例如：
SHOW INDEX FROM emp
```

其查询结果的返回[字段解释](https://blog.csdn.net/u010286027/article/details/100763781)

ps：索引命名。通常索引名使用`idx_tableName_xx_xx`，'xx'为字段名



## 单值索引

```sql
CREATE INDEX idx_name ON `user`(`name`)
```

`MYSQL`将创建一个`key`为`name`的 B+Tree 树

当sql语句中使用的< where_condition >包含有`name`字段时，`MYSQL`会使用B+Tree树进行查询



## 复合索引

```sql
CREATE INDEX idx_name_age ON `user`(`name`, age)
-- CREATE INDEX idx_age ON `user`(age, `name`) 创建索引的顺序不同，区别很大的
```

`MYSQL`将创建一个`key`为`name`的 B+Tree 树，再根据此树的每一个节点创建一个索引为`age`的 B+Tree 树。（请注意两个`key`的创建顺序）

当 sql 语句中使用的 < where_condition > 中包含有`name`和`age`字段或只有`name`字段时

`MYSQL`会先使用`key`是`name`的树查询，再使用查询结果所在节点的`key`为`age`的树进行查询





# 索引的使用规则



## 全值索引

全值索引指一个索引中将表中所有字段作为 key

```sql
select * from `user` where < where_condition >
```

建立一个全值索引

```sql
CREATE INDEX idx_name_age_dptid ON `user`(`name`, age, dpt_id)
```



## 最左匹配原则

```sql
-- 索引可用
select * from `user` where `name`='tom' and age=12
```

```sql
-- 使用到了name索引，没有使用到dpt_id索引
select * from `user` where `name`='tom' and dpt_id=12
```

这和创建的索引的结构有关



## 索引中关键字的离散性

字段的离散性越好，选择性越好，越适合做索引的关键字



那么，什么字段的离散性算好，什么字段的离散性算差？

性别，sex。取值范围为0 或 1。多行数据中很容易出现sex的值相同的数据，这样的字段的离散型差

名字，name。取值的随机性强，且相比于sex，name的值不易重复。离散性优于sex

电话号码，phone。不会出现重复。通常每个人持有的电话号码都不同。离散性很好



## 覆盖索引

使用覆盖索引时，不需要执行 "回表"，这将提高性能

通常不推荐 < select_list > 使用'*'，因为这样需要获取数据行中所有数据

如果 < select_list > 中只指定了部分字段，刚好这些字段在< where_list >中也都有，那么不需要访问数据行也可以获取到查询（因为`B+Tree`的非叶子节点的 key 的值就是< select_list >中需要的数据）

直接在树的节点中就能获取到数据，而不是用获取到的数据的地址去访问数据。这种使用索引的方式叫做覆盖索引。它避免访问数据行，减少了IO操作



**理解方式一：**索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引[叶子节点](https://baike.baidu.com/item/叶子节点/9718763)存储了它们索引的数据；当能通过读取索引就可以得到想要的数据，那就不需要读取行了。一个索引包含了（或覆盖了）满足查询结果的数据就叫做覆盖索引

**理解方式二：**是非聚集复合索引一种形式，它包括在查询里的 Select、Join 和 Where 子句用到的所有列（即建索引的字段正好是覆盖查询条件中所涉及的字段，也即，索引包含了查询正在查找的数据）



# EXPLAN 查询 sql 的执行计划



执行查询语句，根据查询所用时间判断 sql 性能。这种做法过于粗略

可使用`EXPLAN`命令更细致地了解查询语句的执行计划。例如此次查询用到了索引没有，用到了哪个索引，查询扫描了多少数据行



将`EXPLAN`关键字放在 sql 语句前。使用此命令后，查询语句不会真的执行，而是模拟查询。例如：

```mysql
EXPLAIN SELECT * FROM `user` WHERE id=5;
-- 最好加上SQL_NO_CACHE 表示查询时不使用缓存
EXPLAIN SELECT SQL_NO_CACHE * FROM `user` WHERE id=5;
```

会打印如下字段和字段值

`id`,`select_type`,`table`,`partitions`,`type`,`possible_keys`,`key`,`ref`,`rows`,`filtered`,`Extra`



`id`一条 select 语句可能需要多次查询，每个`id`号码代表一次独立的查询。执行顺序为，id 不同，从下到上（从大到小）执行；id 相同，从上到下执行

`select_type`查询类型（举几个例子）

	- `SELECT` 简单查询类型
	- `PRIMARY` 子查询中外部的查询类型
	- `DERIVED` from 关键字后的子查询
	- `SUBQUERY` select 或 where 后的子查询

`type`访问类型（system > const > eq_ref > ref > range > index > ALL）

	- `system`	表中只有一条数据
	- `const`   使用索引，匹配到一行数据
	- `eq_ref`  使用唯一性索引，索引的值在表中都只有一条记录匹配（常见于使用主键索引或唯一索引的查询）
	- `ref`     使用非唯一性索引，索引的查找结果都匹配到多行数据
	- `range`   在`where`中对索引使用了`between`，`>`，`<`，`in`等范围查询
	- `index`   读全表，但是使用到了索引，且都只在树上执行查询和取数据（比`ALL`好）
	- `ALL`     读全表，在硬盘中遍历数据，在源数据中遍历查询和取数据

`possible_keys`可能使用到的索引

`key`实际用到的索引

`key_len`索引使用的字节数。此值为最大的可能长度，而非实际使用长度。一般来说使用的条件越多，此值越大

`ref`显示索引的哪些 key 被使用了，其值为`const`或索引的哪些字段被使用了

`rows`MySQL 认为它查询时必须检查的行数，越小越好

`filtered`表示存储引擎返回的数据在服务层过滤后剩下的数据与过滤前的数据的数量比值

`Extra`重要的提示信息。以下是不同的取值结果

	- `Using filesort`    	表示此次查询结果的数据排序方式无法用索引完成。出现此信息表示该做优化了
	- `Using temporary`   	使用了临时表保存中间结果
	- `Using index`			用到了覆盖索引，避免了访问表的数据行
	- `Using where`			用到了where过滤
	- `Using join buffer`	使用了连接缓存
	- `impossible where`	where 子句的值全为 false
	- `select tables optimized away` 在没有 GROUP BY 自居的情况下，基于索引优化 MIN/MAX 操作或者对于 Myisam 存储引擎优化 COUNT(*) 操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化



# 索引失效情况



索引失效：在 < select_list >  中用到了索引中的关键字，且符合最左匹配原则，但用`EXPLAN`查询后发现完全没有用到或用到部分索引



- `where`后使用了`like`。'xx%'，'%xx'，%xx%'。三者从左到右降低查询效率效果递增，且只有使用'xx%'才能用到索引，后面两种都用不到索引
- `where`中包含对字段进行`null`值的判断
- 使用了`!=`或`<>`。使用了`is not null`（使用`is null`不影响索引）
- 使用字符串时没有加单引号
- 使用了`or`但没有为每个条件创建索引
- 使用了`>`等比较运算符。使用了比较运算符的索引可以生效，但其后面的索引会失效
- 在 < where_condition > 中使用了函数，计算，自动或手动的类型转换（如自动将没使用单引号的字符串转为字符串）





# 多表查询优化

驱动表（主表，全扫描的表），被驱动表

驱动表建立索引意义不大，因为就算是建立索引也需要被全扫描。被驱动表建立索引才有明显的效果。所以被驱动表是大表（数据多），驱动表是小表

当使用外连接时，给被驱动表创建索引。当使用内连接时，mysql会自动选择哪张表作为被驱动表（A表没有索引，B表有索引时，mysql自动选择B表作为被驱动表）

也可以手动指定驱动表和被驱动表（防止mysql将有索引的小表作为被驱动表）。`STRAIGHT_JOIN`（关键字前是驱动表，后面是被驱动表）取代`inner join`



# 优化



## order by优化

当`EXPLAN`的`Extra`提示`Using filesort`时意味着你该优化你的`order by`了

1. 使用最左匹配原则
2. < where_list > 与 < order_by_condition > 的组合满足最左匹配原则

满足上述条件后`order by`后的索引即可生效。如果`where`或`select`中都没使用到索引，那么`order by`使用的索引失效

如果`order by`的字段不在索引中，mysql 将会从单路排序和双路排序中选一种方式进行排序

单路排序比双路排序快。单路排序使用的是内存



## group by 优化

分组`group by`和排序`order by`的区别唯一的区别是`group by`在没有筛选条件（where < where_list >）的情况下也能使用索引，索引不会失效



## or 优化

`or` 的查询尽量用 `union`或者`union al`代替(在确认没有重复数据或者不用剔除重复数据时，`union all`会更好) 



为什么要用`union`代替`or`？`union`需要查询两次，`or`需要查询一次，为什么`union`查询还要比`or`快？

举例：

`SELECT * FROM emp WHERE id=2 or id=5`

`SELECT * FROM emp WHERE id=2 UNION SELECT * FROM emp WHERE id=5`

使用`or`后，执行计划的`type`会变成`range`（范围查询）

使用`union`后，执行计划包含两条的`type`为`const`的执行计划

这导致了`union`的两次查询快于`or`的一次查询

