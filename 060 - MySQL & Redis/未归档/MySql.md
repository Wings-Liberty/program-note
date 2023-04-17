#还没有复习 

#  安装

> 这里使用`apt install`简易安装

`apt install mysql-server mysql-client `

安装好的`mysql`的配置文件在`/etc/mysql/`目录下。以下是其他文件所在文职

```
/usr/bin                 客户端程序和脚本  
/usr/sbin                mysqld 服务器  
/var/lib/mysql           日志文件，数据库  ［重点要知道这个］  
/usr/share/doc/packages  文档  
/usr/include/mysql       包含( 头) 文件  
/usr/lib/mysql           库  
/usr/share/mysql         错误消息和字符集文件  
/usr/share/sql-bench     基准程序  
```



源码安装：在[清华大学镜像源](https://mirror.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-5.7/)找安装包，安装后需要[修改MySQL密码](https://www.jb51.net/article/142025.htm)

设置允许远程登陆

- 修改 /etc/mysql/mysql.conf.d/mysqld.cnf 的 bind-address 为 0.0.0.0
- 修改 mysql 表，设置 host 字段为 % 或 特定 ip



1. 切换数据库

use mysql;

2. 执行如下指令 %代表所有主机

update user set host='%' where user='root';

3. 指令生效

 flush privileges;



# DDL（数据定义语句）

1. 创建数据库
   `create database <databaseName> charset utf8;`
2. 显示所有数据库
   `show databases;`
3. 使用数据库
   `use <databaseName>`
4. 创建表
   `create table <tableName> (file type, file type, ``....);`
5. 显示所有的表
   `show tables`
6. 查表所含的字段
   `desc 表名;`
7. 给表加字段
   `alter table 表名 add 字段名 字段类型`
8. 修改字段的类型
   `alter table 表名 modify 字段名 字段类型;`
9. 删除字段
   `alter table 表名 drop 字段名;`
10. 修改表名
    `rename table 原名字 to 新名字;`
11. 查看建表的细节
    `show create table 表名;`
12. 修改表的字符集（不常用）
    `alter table 表名 character set 字符集名称`
13. 修改列/字段名
    `alter table 表名 change 原始列/字段名 新列/字段名 字段类型;`
14. 删除表
    `drop table 表名;`

索引（加快查询效率）
创建索引
`create index 索引名 on 表名(字段)`
删除索引
`drop index 索引的名称 on 表名`
查看索引
`show index from 表名`

事务开始于DML语句
结束语DDL语句或DML语句执行失败或DCL语句或.......



约束（规定某个字段的数据不能重复出现，如id）
primary key 主键 唯一而且不为空
unique
auto_increment （自增）
not null （不为空）
例：
`create table teacher (id int primary key auto_increment, name varchar(10));`



# DML（管数据的语句）
增删改查

1. 查询所有的数据
   `select * from <tableName>` \G;	//    \G可有可无它表示另一种显示方式

2. 插入数据
   insert into 表名 (字段1,字段2) values (字段1的值,字段2的值);

3. 更新数据/修改数据
   update 表名 set 列名=列值 where 列名=列值;

4. 删除记录
   delete from 表名 where 列名 = 值；	// 删除数据可以找回
   truncate table 表名			// 删除整个表，再重新建立一个和原表结构一样的表

# DQL（数据查询语句）

> 以下是常用查询语句和常用关键字
>
> [来自菜鸟的教程](https://www.runoob.com/sql/sql-distinct.html)



## 手写顺序

```mysql
SELECT DISTINCT
	< select_list >
FROM
	< left_table > < join_type >
JOIN < right_table > ON < join_condition>
WHERE
	< where_condition >
GROUP BY
	< group_by_list >
HAVING
	< having_condition >
ORDER BY
	< order_by_condition >
LIMIT < limit_number >;
```



## 机读顺序

```mysql
FROM < left_table >
ON < join_condition >
< join_type > JOIN < right_table >
WHERE < where_condition >
GROUP BY < group_by_list >
HAVING < having_condition >
SELECT
DISTINCT < select_list >
ORDER BY < order_by_condition >
LIMIT < limit_number >;
```

![[../../020 - 附件文件夹/Pasted image 20230402002218.png|600]]

## 常用查询语句

1. 查询所有的数据
   `select * from 表名;`	

   `select * from 表名 \G;`  不加`\G`字段名在第一行，加`\G`字段在第一列显示。这样可以避免字段过得多显示不清楚
   
   `select 字段名 DISTINCT from 表名`去重查询，去掉所选字段值全部相同的数据行。
   
2. 查询指定字段
   `select 字段名 from 表名;`

3. 条件查询（根据条件查询）
   新：`<>`（也可以表示不等于）和`!=`一样
   `select * from 表名 where 字段名 条件;`
   例：select * from user where age >= 19;

4. 条件查询
   新：between  ... and ...
   `and`	`or`	`is null`	`is not null`，`and`优先级高于`or`
   `select * from user where age between 15 and 18;`	// 数字是从小到大写，闭区间
   `select * from user where age = 18 and sid = 1;`
   `select * from user where age is null;`
   `select * from user where age is not null;`

5. 模糊查询
   like _ 表示一个字符	% 表示多个字符
   `select * from user where username like '小%';`

6. 排序
   排序	order by
   升序	asc;
   降序	desc;
   例：
   `select * from user order by age;`	//默认升序，即从上到下是从小到大
   `select * from user order by age desc;`

7. 聚合函数
   count ()	不统计是 null 的数据
   `max ()`，`min ()`，`sum ()`	
   
   avg ()		求平均值
   
   例：
   `select count(sid) from user;`
   `select max(age) from user;`
   
8. 分组
   group by 字段名。GROUP BY 语句用于结合聚合函数，根据一个或多个列对结果集进行分组
   
   只显示符合条件的第一个（有两个 age=11 的数据则只显示第一个）
   
9. having + 条件/聚合函数
   
   where 后面不能加聚合函数
   
   `select * from user having score=avg(score)` 查询成绩和平均年成绩相同的学生信息
   
10. 分页
    limit int(起始数据的下标x)， int(最后数据的下标y)
    
    数据的下标从0开始计算

    注：[x,y) 这是左闭右开区间
    
11. 查重

    ```sql
    SELECT DISTINCT `username` FROM `user`;
    ```

    查询结果中`username`将不会用重复数据，例：如查询结果只包含查到的第一个'李四'，第二个李四会忽略不做展示

关键字的执行顺序和书写顺序：

select(5) from(1) where(2) group by(3) having(4) order by(6) limit(7);



## 常用关键字

[on, where 和 having 的区别](https://www.cnblogs.com/ajianbeyourself/p/9836954.html)

`on`，要区别于`where`

- on 只能用于外连接查询，其作用是和过滤不符合条件的数据行，但保留完整的主表

> select * form tab1 left join tab2 on (tab1.size = tab2.size) where tab2.name=’AAA’
>
> on 用于 tab1 left join tab2 on (tab1.size = tab2.size) 先生成中间表 tmp，然后执行 select * form tmp where tab2.name=’AAA’

on 在保留主表的前提下生成两张表的笛卡尔积中间表 1，where 对中间表 1 进行过滤再生成中间表 2，用中间表 2 执行聚合函数的计算生成中间表 3，having 对中间表 3 进行过滤。3 个关键字的执行速度递减





`as`，用于给字段起别名

`select name as username, password from member`

`as`关键字也可以省略



原本名为`name`现在改为`username`

`case when`和`if`为条件判断语句，如果条件成立就...不成立就....。（例如可实现根据条件判断结果添加不同的临时字段和字段值）至今没用过，所以暂时不写介绍和实现



## 常用函数

查询静态值 / 常用函数的使用
`select 'some string';`，`select 1+1;`，`select now();`

`select curdate();`，`select curtime();`，`select pi();`

`select mod(45,7);`，`select sqrt(25);`
可以在查询的时候通过as 修改这一列的列名

还要其他一堆聚合函数

round() 四舍五入
round(columnname,x)		四舍五入保留x位小数
floor()								直接舍
ceiling()							直接入
concat 								合并多个字符串
left 										取字符串的前几位



# 多表联查



## 多表之间的关系

1. 一对一

人和身份证。一个人只能有一张身份证

2. 一对多

员工和部门。一个部门有多个员工，一个员工只能有一个部门

3. 多对多

学生和课程表。一个学生苦役选多门课程，一个课程也可以被多个学生选择

多对多关系中需要借助第三张中间表





## 多表查询分类

> 数据库在通过连接两张或多张表来返回记录时，都会生成一张中间的临时表，然后再将这张临时表返回

### 内连接查询

**隐式内连接**

语法： `select 字段列表 from 表名1 a, 表名2 b on 条件`

**显式内连接**

其效果和隐式内连接相同，只显示符合条件的语句

语法： `select 字段列表 from 表名1 inner（inner可省略） join 表名2 on 条件`

使用 where 条件消除无用数据，例：

`select user.u_id, user.u_username, country.c_countryname from user, country` // 查询结果会有重复数据

`select user.u_id, user.u_username, country.c_countryname from user, country where country.c_id = user.u_cid` // 查询结果没有重复数据



### 外连接查询

> 一般只用左外链接而不是用右外连接

**左外链接**

`select 字段列表 from 表1 left outer(可省略) join 表2 on 条件`

查询的是左表的所有数据以及其交交集。主体在左，右表字段为空的补`null`

**右外连接**

`select 字段列表 from 表1 right outer(可省略) join 表2 on 条件`

查询的是右表的所有数据以及其交交集，主体在右，左表字段为空的补`null`

==外连接除左外链接和右外连接外还有，自然连接和全连接==



#### 全连接

`mysql`不支持全连接，`oracle`支持全连接

要想用`mysql`实现全连接，需要使用`union`关键字。

使用左外链接查询 + 'union' + 右外连接查询。

注意，这里的左外连接和右外连接select的字段必须一致，例如查A，B表时，两个连接的查询字段应该一致`select a.*, b.*`

`union`自带去重功能，`union all`不带去重功能

[多表查询教程](https://www.bilibili.com/video/BV12b411K7Zu?p=191)

**常用的7中多表查询**

![[../../020 - 附件文件夹/Pasted image 20230402002305.png]]

`mysql`不支持`FULL OUTER`关键字，但又其他解决方案（`union`关键字）。具体方案见下文

`select * from emp e left jion tmp t on e.tmp_id=t.id where e.id is null  `

`union `

`select * from emp e right jion tmp t on e.tmp_id=t.id where t.id is null`



#### 子查询

> 不推荐用子查询，推荐用多表查询

概念：查询中嵌套查询，成嵌套查询为子查询

下面将比较多个查询和子查询的区别

查询最高工资是多少假设查询结果是9000

`select Max(salary) from emp;`

查询员工信息，并且工资等于 9000 的员工

`select * from emp where emp.salary = 9000;`

子查询，一步到位

`select * from emp where emp.salary = select Max(salary) from emp;`

子查询的不同情况：

1. 子查询的结果是单行单列的

   查询员工工资小于平均工资的人
   `select * from emp where emp.salary < (select avg(salary) from emp);`

2. 子查询的结果是多行单列的
   
   查询'财务部'和'市场部'所有员工信息
   `select id from dept where name = '财务部' or name = '市场部';`
   `select * from emp where dept_id = 3 or dept_id = 2;`
   
   子查询
   
   `select * from emp where dept_id in (select id from dept where name = '财务部' or name = '市场部');`
   
3. 子查询的结果是多行多列的
   
   子查询可以作为一张虚拟表
   
   查询员工入职日期是 2011-11-11 日后的员工信息和部门信息
   
   `select * from dept t1, (select * from emp where emp.join_date > '2011-11-11') t2 where t1.id = t2.dept_id;`



99 查法：

`select student.name, teacher.name from student, teacher where student.id = teacher.id`



# DCL（事务）

一个事务是由一条或者多条 sql 语句构成，这一条或者多条sql语句要么全部执行成功，要么全部执行失败



## 事务的操作

`开启事务`，`回滚`，`提交事务`

```sql
-- 开启事务
START TRANSACTION;

-- 回滚。当执行事务出现异常时需要回滚以保证数据安全
ROLLBACK;

-- 提交事务。将事务的执行结果持久化，在提交之前数据都只是暂时改变而不是持久化地改变
COMMIT;
```



## 事务的四大特性

`原子性`，`一致性`，`隔离性`，`持久性`

### 原子性（Atomicity）

事务中所有操作是不可再分割的原子单位。事务中所有操作要么全部执行成功，要么全部执行失败

### 一致性（Consistency）

事务执行后，数据库状态与其它业务规则保持一致。如转账业务，无论事务执行成功与否，参与转账的两个账号余额之和应该是不变的。

### 隔离性（Isolation）

隔离性是指在并发操作中，不同事务之间应该隔离开来，使每个并发中的事务不会相互干扰。

下面的事务的隔离级别将详细介绍隔离性

### 持久性（Durability）

一旦事务提交成功，事务中所有的数据操作都必须被持久化到数据库中，即使提交事务后，数据库马上崩溃，在数据库重启时，也必须能保证通过某种机制恢复数据。





## 事务的隔离级别

| 隔离级别                     | 脏读（Dirty Read） | 不可重复读（NonRepeatable Read） | 幻读（Phantom Read） |
| ---------------------------- | ------------------ | -------------------------------- | -------------------- |
| 未提交读（Read uncommitted） | 可能               | 可能                             | 可能                 |
| 已提交读（Read committed）   | 不可能             | 可能                             | 可能                 |
| 可重复读（Repeatable read）  | 不可能             | 不可能                           | 可能                 |
| 可串行化（Serializable ）    | 不可能             | 不可能                           | 不可能               |

### 概念

多个事务之间的隔离，是相互独立的。但是如果多个失事务操作同一批数据，则会引发一些问题，设置不同的隔离级别就能解决这些问题

### 存在的问题

- 脏读：一个事务读取到另一个事务中没有提交的数据

- 不可重复读（虚读）：同一个事务中，两次读取到的数据不一样

  事务A做更新数据的操作。事务B在事务A执行更新操作前后查询了两次数据，结果两次数据不一样（因为读到了事务A更新数据后的数据和更新前的数据）

- 幻读：一个事务操作（DML）数据表中的所有记录，另一个事务添加了一条数据，则第一个事物查询不到自己的修改

### 隔离级别

- `read uncommitted`：读未提交的事务

  可能出现的问题：脏读，不可重复读，幻读

- `read committed`：读已提交（Oracle默认级别）

  可能产生的问题：不可重复读，幻读

- `repeatable read`：可重复读（MySql默认级别）

  可能产生的问题：幻读

- `serializable`：串行化

  可以解决所有问题。类似于加锁，一个事务使用一张表时另一个事务不能用这张表

注意：隔离级别从小到大安全性越来越高，但是效率越来越低

数据库查询隔离级别

```sql
SELECT @@transaction_isolation;
```

数据库设置隔离级别

```sql
-- 这里的<隔离级别的字符串>可用选项为上述隔离界别中的四个选项（read uncommitted，read committed等四个选项）
SET GLOBAL TRANSACTION SELECT isolation LEVEL <隔离级别的字符串>;
```





# 数据库设计的范式

> 了解即可

---

设计数据库时，需要遵循一些规范。要遵循后面的范式要求，必须先遵循前面的所有范式的要求



设计关系型数据库时，遵从不同的规范要求，设计出合理的关系型数据库，这些不同的要求被称为不同的范式，各种范式呈递增规范，越高的范式，数据库冗余越小

目前关系型数据库有六种范式：第一范式（1NF）、第二范式（2NF）、第三范式（3NF）、巴斯-科德范式（BCNF）、第四范式（4NF）和第五范式（5NF，又称完美范式）

==数据库范式了解即可，是设计表的基本原则，不是应用开发的规范==



# 导入导出sql文件

---

## 导出sql文件

```shell
$ mysqldump -u<username> -p <databaseName> > file.sql
$ mysqldump -u<username> -p -d <databaseName> > file.sql
```

前者导出表结构和数据，后者只导出表结构

导出文件后，sql文件存放地在`/usr/lib/mysql/`，根据情况不同，可使用`find / -name file.sql`命令查询文件所在地

## 导入sql文件

一般 sql 文件中没有建库命令，这样的话需自行建库后再执行sql文件

1. 在mysql客户端执行sql文件

   ```sql
   mysql> source /home/abc/abc.sql;
   ```

2. 在shell中执行sql文件

   ```shell
   $ mysql -u<username> -p <databaseName> < file.sql
   ```



# mysql架构介绍

> 这里说的配置指的是堆mysql的用户啊，配置文件啊，锁表，锁列等的操作



## 用户权限问题

`mysql`的用户信息在`mysql`库的`user`表中 

`user`表的主要字段大致分为

`Host`	用户登录方式（如："localhost"该用户只能在本机登录，"%"该用户可远程登录）

`User`	用户名

`._priv`	有多个以此结尾的字段	和用户权限有关的字段（priv指权限）

`authentication_string`	密码（被加密过）



### 创建用户

```mysql
create user zhang3 identified by '123123';
```

用户名'zhang3'，密码'123123'。默认可远程登录

远程登录后，只能看见一张表（即：zhang3用户权限不够，不能看到和操作其他表）

### 修改用户的普通信息

修改当前用户的密码:
`set password =password('123456')`

修改某个用户的密码:
`update mysql.user set password=password('123456') where user='li4';`
`flush privileges;`   # 所有通过user表的修改，必须用该命令才能生效。

修改用户信息

`update mysql.user set user='li4' where user='wang5';`

删除用户

`drop user li4;`

不要通过`delete from  user u where user='li4';` 进行删除，系统会有残留信息保留。 

### 修改用户权限

更多实现见脑图



## 杂项配置

`mysql`中有一个变量`sql_mode`

```sql
show  variables like 'sql_mode';
```

变量存的是sql语法的规则。如果不同的mysql服务使用的`sql_mode`的值不同，可能会导致sql语句在不同的mysql上无法执行





# MYSQL用户(tmp)



## 修改用户的普通信息

```mysql
-- 修改当前用户的密码。官方推荐方式。在mysql客户端执行即可
ALTER USER dev@'%' IDENTIFIED BY 'dev';

-- 通过修改user表中数据方式修改密码。修改mysql表需要有修改系统库表数据的权限
update mysql.user set authentication_string=password('root') where user='dev';
-- 通过update修改user表中用户密码的操作，必须用该命令后新密码才能生效 
flush privileges;
```



## 杂项配置

`MYSQL`中有一个变量`sql_mode`

```sql
show variables like 'sql_mode';
```

变量存的是sql语法的规则。如果不同的mysql服务使用的`sql_mode`的值不同，可能会导致sql语句在不同的mysql上无法执行





# 索引及其优化分析

> 创建表的主键时会生成索引，但是主键和索引是不同的

[主键和索引的关系](https://blog.csdn.net/kangsenkangsen/article/details/51234600)

主键一定是索引，但是索引不一定是主键



> 以下内容均为自己的小总结，有待修改

索引和数据

学《算法》的二叉树，平衡二叉树，红黑树等的时候，节点的值都是索引，每个节点的索引又对应着数据。不过数据放在别的地方

二叉树的查找算法使得使用索引进行查询的速度飞起

如果没用索引，那就是遍历所有数据，挨个检查数据和sql语句里的条件是否一致

使用索引后，速度飞起

1. 创建索引

   创建索引后，会以指定的字段作为索引创建 B+Tree（mysql中的算法，就像红黑树一样，是一种每个节点有一个索引的查找算法）

2. 执行sql语句

   mysql会解析sql语句，查看sql语句中的 条件有没有可以作为索引进行查找的字段

   如果有，那么mysql会用该字段作为索引，先在索引所在的树中进行查找，直接缩小所要查找的范围（核心）

## 单表索引优化

索引分两种`单值索引`，`复合索引`

`单值索引`一个索引中包含一个字段。用索引生成==一个==树

`复合索引`一个索引中包含多个字段，例如：索引名`idx_age_deptid_name`，索引包含的字段`age, deptid, name`

会生成==一堆==树

`SELECT SQL_NO_CACHE * FROM emp WHERE emp.age=30 and deptid=4 AND emp.name = 'eMxcTm';`

mysql 解析出此 sql 会使用到的索引，然后用 age=30 作为索引去 age 生成的树中查询，然后拿查询到的结果范围去`deptid`生成的条件是`age=30`的树中查询，同理，在 name 的树中去查

即`age`生成一个树，`deptid`生成一堆树，`age`的每个节点对应`deptid`中的一个树。同理，`deptid`的每个节点对应`name`的一个树

**最佳左前缀法则**一个索引有是三个字段，先用第一个字段作为索引检查，再用第二个，再用第三个。

如果 sql 中根本没用到第一个字段，那么查询时根本用不了此索引。原因见上述`复合索引`的原理

如果 sql 的条件中用到了索引的第一个字段和第三个字段，没用到第二个字段，那么在 mysql 真正查询时只会用第一个字段做索引在第一个字段的树中查找，不会用第三个字段做索引。原因如上

sql 的 where 条件后尽量不要使用计算，函数等。如果作为索引的字段在函数，计算中被使用，索引会失效（查询的时候不使用索引），like 可以用（他不算函数什么的）

上述的计算，指比较大小（等于不算在内。大小于，大于等于，小于等于，不等于算在内）。如果使用了，那么其右面的作为索引的字段失效。即，使用计算后，部分字段失效。当然，你也可以在创建索引时，把用于比较大小的字段放在后面

is null 索引可生效，is not null 索引失效

like 使用前置 %，索引失效

where 后的字段顺序和索引中的字段顺序不一致不会导致索引失效。因为 mysql 的优化器会对查询结果一致的sql语句进行调整。但是若 order by 中的字段顺序和索引中的顺序不一致，索引可能会失效，因为 order by 中字段顺序不同会导致查询结果不同，所以 mysql 的优化器不会对 order by 后的字段顺序进行调整

类型转换（无论自动转换还是手动转换），在 where 后对 varchar 类型的字段使用不同类型的数据作为条件例如`where name = 123`，mysql 会自动进行类型转换，但是将不再使用索引



**总结**

建索引就能提高查询速度

where 后面尽量不使用函数，计算



## 多表查询优化

驱动表和被驱动表

两张表关联查询时，有一张表一定会被全扫描，这张表叫驱动表，另一张就叫被驱动表

驱动表建立索引意义不大，因为就算是建立索引也需要被全扫描，所以给驱动表建立索引意义不大。被驱动表建立索引才有明显的效果

当使用外连接时，给被驱动表创建索引即可（驱动表是主表）。当使用内连接时，mysql 会自动选择哪张表作为被驱动表（当A表没有索引，B 表有索引时，mysql 自动选择 B 表作为被驱动表）

也可以手动指定驱动表和被驱动表（防止mysql将有索引的小表作为被驱动表）。`STRAIGHT_JOIN`（关键字前是驱动表，后面是被驱动表）取代`inner join`

mysql5.7 后 mysql 会对 sql 自动进行优化，所以同一个多表查询的不同sql语句，本应该性能有所区别，但是mysql优化后，性能可能会是一样的

虚拟表无法创建索引

**小结**

多表查询尽量不要使用子查询，因为子查询不能使用索引

可使用多表查询的左右外连接的==7种使用方式==解决问题



## 排序/分组优化

对于使用了`order by`的 sql 语句，如果使用过滤条件，那么索引将会失效

where，limit 都算是过滤条件，要是没有说明过滤条件可以添加的话，使用 limit 也是可以的

如果 where 后面有比较大小的条件，那么order by后面的字段即使存在于索引中也会失效



如果order by的字段不在索引中，mysql将会从单路排序和双路排序中选一种方式进行排序

单路排序比双路排序快。单路排序使用的是内存

单路排序的优化策略见脑图

**问题**不需要手动指定使用单路排序还是双路排序吗？mysql会自动选择吗？



分组group by和排序order by的区别唯一的区别是group by在没有筛选条件的情况下也能使用索引，索引不会失效



## 覆盖索引

当过滤条件中无法使用索引时，覆盖索引是最后的手段

select * 换为具体的字段。select 字段，能用到索引，但是没有在条件里使用索引的搜索速度快

所以少用 select *



## 小结

在一条查询语句中，当有多个索引都可以被使用时，mysql会自动选择最合适的一个索引

**sql编程跳过不学**

# 查询截取分析

> 此节将讲解mysql的监控日志。mysql将指定级别的sql操作写进日志
>
> 提高查询速度的方法：创建合适的索引
>
> 但是并不是所有sql都要见索引，或者说项目运行几年后，应该检查索引的使用情况，重新创建新的索引或修改索引。但是这并不意味这需要检查所有sql的执行速度



## 慢查询日志

> 慢查询日志：记录了执行时间较长的sql的日志。方便定位需要优化的对象

默认不开启，开启后会对性能有较小的影响，所以如果不是为了调优的话，一般不开启

生产环境下，慢查询日志文件都比较大，需要使用日志分析工具

**全局日志**所有sql均记录到日志中，大伤性能。除非极端情况，否则不会使用



## SHOW PROCESSLIST

此命令用于列出 mysql 客户端连接的相关进程

不想让谁连的话，就像杀进程一样 kill 掉它就行了



# 工具和技巧拾遗

> 此节讲解mysql的视图操作技巧

视图：view。是一段sql代码

视图是一张虚拟表，类似函数

```sql
CREATE VIEW view_name 
AS
SELECT column_name(s)
FROM table_name
WHERE condition
```

如上述，将一大段sql封装到一个名为‘view_name’的视图中。

但凡想使用此sql，利用视图即可

```sql
select * from view_name
```

据上述简单用例。视图就像一张虚拟表，又像函数

需要了解，掌握。虽然我只想做到了解水平



# 主从复制

> 据学习`redis`的主从复制经验来看。主从复制，一个主机，多个从机。主机能写数据，从机不能使用mysql客户端写数据，从机是用来做备份的，其数据时刻与主机保持一致。当主机挂掉，从机选出一个作为新的主机顶上
>
> 在开发环境和测试环境中不需要使用主从复制和集群环境。生产环境下才需要使用
> 

mysql的主从复制不是实时的，在主从复制的过程中涉及多次IO操作，比较耗时，所以存在延时问题

mysql主从复制是使用一个二进制的`.log`日志文件进行的

因为延时问题，所以插入数据后不要马上就去查数据（这里的延时是毫秒级的，虽然并不慢，但是也不要立即就查数据）



注意：搭建主从复制时，要先搭建好组从关系后，再建库建表。因为如果主机先建库再搭建主从，主机建表，查数据。从机会报早不到数据库的错误

mysql的主从复制中，从机可以做写操作

# MyCat

> 一个数据库中间件，用于数据库的集群

java可以把mycat当做mysql一样，配置一个数据源即可连接到mycat后面的集群数据库。这一点有点像nginx

此外，mycat不仅可以连接mysql，还能连接oracle，redis等

mycat是一个逻辑库（表面上是一个数据库，实际上它并不存数据），mysql才是物理库（真正存数据的库）。



## 运行mycat



### 配置逻辑库

逻辑库：将mycat当做一个mysql。java程序只需要像连接mysql一样连接mycat即可

`server.xml`此配置用于设置mycat的用户，以及用户所操作的逻辑库，和用户的权限 



### 配置物理库

在`schema.xml`中配置物理库

配置上述配置逻辑库时对应的物理库，指定物理库中的读机器和写机器。设置读数据时物理库的负载 方式

设置心跳sql（设置一条sql语句，只要是正常情况下能够成功执行并有有意义的返回值的sql就行。mycat定时发送sql，如果能获取到远程mysql服务返回的有效响应即算成功维持心跳）等



### 运行mycat

配置好上述后，单独 测试 mysql能否运行。

启动所有mysql后，运行mycat

mycat有两种运行方式

前台启动（阻塞式运行，推荐 ，因为mycat是java写的，日志由java打印，便于查错）`mycat console`

后台运行（守护进程运行）`mycat start`

启动成功后，像连接mysql一样连接mycat即可

`mysql -uroot -p -P8066`	数据窗口，日常使用此窗口足以满足需求

`mysql -uroot -p -P9066`	后台窗口，开发一般不用

==在mycat中操作数据库时，不建议使用可视化工具（sqlyog已经出现不兼容问题，表显示不全，分库分表出现异常等）。建议在命令行下执行命令==





## 读写分离

设置`schema.xml`文件中的dataHost标签的balance属性值即可设施负载轮询方式

balance常用1或3模式



## 分库和分表

分表：将一张表的字段分成若干份，每份都可称为一张独立的表

分库：将一张表的数据分到不同库的同名表中





## 分库

>  垂直拆分：将原一个库中的一堆表拆分到若干个库中
>
> mysql每个库的数据达到500万后速度 会有明显的下降，所以需要分库

数据库A有很多表，A库中有6张表数据过多，需要分库。即创建库B，C。将表1，2放在A库。表3，4放在B库。表5，6放在C库。

虽然java配置文件里只能写一个库的连接信息，但是连接mycat时，mycat只有==一个==逻辑库。这个逻辑库背后有很多库



一堆表的分库原则

同一个库中的不同表能使用jion，所以会使用到多表查询的表放在同一个库里。



客户表A，订单表B，订单详情表C。显然A不会与B或C使用多表查询，B和C可能会出现多表查询

所以，在mycat中配置，两个mysql（mysql1和mysql2）将B和C放在mysql1中，A放在mysql2中（mysql1和mysql2是）

这样的话，创建表A时mycat只会在mysql2中创建，mysql1中不会创建

注：分库操作（执行创建表的操作）应该在mycat中执行，而不是在物理库中执行

## 分表

>  水平拆分：A库中一张表的数据过多时，需要在其他库中建立同样的表，将A库中表的数据分散到其他库中的同名表中

### 分表依据

分表时应该用什么字段去分？

id？前50%的id分到A库的表中，后50%的id分到B库的表中

创建时间？数据按时间排序后，前50%放A库，后50%放B库

都不合理，应该根据实际情况中用户的访问情况分析

例如：订单表，根据id或创建时间去分库就不合理。因为用户一般很少去访问很久之前的订单信息，而会经常访问近期下的订单的信息

应该根据用户id去分，50%用户的订单数据放A库，50%用户的订单数据放B库



### 跨库join

表1和表2需要进行多表查询时，如果

A库，B库均有订单表（表1）和订单详情表（表2）



## 全局序列

> 用于解决插入的数据生成的id是唯一的

向一张表插入多条数据时，多条数据可能在不同库的同名表中被创建。如果设置数据的id为自动增长。那么会出现不同库的同名表中有数据的id是相同的情况

全局序列就是用于解决这个问题的。即创建一个id生成器，以保证插入数据时，即使数据是在不同库的同名表中被创建的，数据的id也会是唯一的。



三种方式。本地文件，时间戳，

### 本地文件（不推荐）

mycat原本的作用是拦截sql语句，然后 把sql语句分派到mysql服务器上。

本地文件就是，mycat拦截sql时，创建id，id为自动增长。即所有库，所有表的所有数据的id均为mycat生成的，且所有数据的id均不同（传统方式中，A表和B表是两张不同的表，所以A表有id为3的数据，B表也有id为3的数据并不会造成冲突。本地文件方式中，所有数据的id将均不相同）id增长到多少了，只有mycat知道

不推荐理由：mycat的主机挂掉后，即使mycat的从机顶上也会出现id号不同步的情况（原mycat的主机记录的id已经创建到101了，mycat的从机顶上后其id的 记录却到100。这将会出现mycat将id-100重复使用的情况）



### 时间戳（不推荐）

使用系统的当前时间戳作为id。

不推荐（时间戳有18位，对于id来说过长）

### 数据库

### 自定义

