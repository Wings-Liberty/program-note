# 手写顺序

```sql
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

# 机读顺序

```sql
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

![Snipaste_2022-01-07_19-36-09](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2022-01-07_19-36-09.png)


# SELECT 语句执行过程

SELECT 语句会被进行词法分析，语法分析，语义分析


```sql
SELECT
	a.customer_id,
	COUNT( b.order_id ) AS total_orders 
FROM
	table1 AS a
	LEFT JOIN table2 AS b ON a.customer_id = b.customer_id 
WHERE
	a.city = 'hangzhou' 
GROUP BY
	a.customer_id 
HAVING
	count( b.order_id ) < 2 
ORDER BY
	total_orders DESC;
```

## 编译阶段

根据手写的 sql，在编译阶段提取出 token，构建出 AST

得知参与查询的表，查询条件等

## 执行阶段

根据上述的机读顺序，执行各个子句

每次执行子句都会产生一个虚拟表（VT - virtual table）如果子句缺失就不执行也不产生 VT


> [!quote] 执行阶段时，各个子句的执行过程参考这里
> [sql 语句执行顺序](https://blog.51cto.com/u_15346267/5055813)

# 注意事项


## 手写顺序和机读顺序的误区

执行 ON 子句时，因为在编译阶段就已经知道 left_table 和 right_table，所以不要死板地认为必须执行 JOIN 子句时才知道其他副表


## 条件查询使用列别名的误区

由 SQL 语句的执行顺序可知，MySQL 会先处理 `where_list` 在处理 `select_list`，所以不能在 `where_list` 中识别 `select_list` 中给列定义的别名


## 大数据量下不推荐用 LIMIT 子句（此标题移到 SQL 优化里）

当数据量非常大的时候，limit 是非常低效的。因为 limit 的机制是每次都是从头开始扫描，如果需要从第 60 万行开始，读取 3 条数据，就需要先扫描定位到 60 万行，然后再进行读取，而扫描的过程是一个非常低效的过程


## GROUP BY 和 DISTINCT 的区别

`group by` 和 `distinct` 都可以实现单列去重及多列去重的功能

[官方文档](https://dev.mysql.com/doc/refman/8.0/en/distinct-optimization.html)在描述 distinct 时提到：**在大多数情况下 distinct 是特殊的 group by**，如下图所示：

![Pasted image 20220917233820](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220917233820.png)


其他区别可以参考[这篇博客](https://blog.csdn.net/sufu1065/article/details/125669918)



## ON 和 WHERE 的区别

两者本身执行流程基本相同，因为都是根据条件进行筛选

但是它们执行的时机不同。外连接中，执行过 on 条件过滤后会执行一次[添加外部行](https://blog.51cto.com/u_15346267/5055813#:~:text=%E4%B8%8A%E7%BB%A7%E7%BB%AD%E8%BF%9B%E8%A1%8C%E3%80%82-,%E6%B7%BB%E5%8A%A0%E5%A4%96%E9%83%A8%E8%A1%8C,-%E8%BF%99%E4%B8%80%E6%AD%A5%E5%8F%AA%E6%9C%89)，而 where 条件也会执行条件过滤，但它也会添加的外部行进行条件过滤，而 on 不会，因为执行 on 的时候 VT 里还没有外部行


## 外连接下不能只用 WHERE，要加一个 ON

查询 1 出现了语法问题，必须用查询 2，或用查询 3

```sql
-- 查询 1
SELECT * FROM course t1 LEFT JOIN teacher t2 WHERE t1.t_id=t2.t_id
```

```sql
-- 查询 2
SELECT * FROM course t1 LEFT JOIN teacher t2 ON t1.t_id=t2.t_id
```

```sql
-- 查询 3
SELECT * FROM course t1 LEFT JOIN teacher t2 ON 1=1 where t1.t_id=t2.t_id
```


