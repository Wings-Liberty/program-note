
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

[来自菜鸟的教程](https://www.runoob.com/sql/sql-distinct.html)

# 复杂查询条件

- 条件查询
	- 单表查 - where，多表查 - on，join
	- 等值查询，模糊查询，比较查询 - =，like，<，>，!=
	- 范围查询 - between，in
	- 逻辑查询 - and，or
	- 空值查询 - is null，is not null
- 分组，聚合函数 - group by，having
- 排序 - order by
- 去重 - distinct

 
![Snipaste_2020-06-19_17-00-43](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2020-06-19_17-00-43.png)



> [!NOTE] 内连接语法问题
> - 用 [inner] join 时需要用 on 进行多表查询的过滤
> - 用 [table1, table2 ...] 时不能用 on 进行多表查询，只能用 where


# 空值查询注意事项



## =null 和 is null 的区别

| id  | status |
| --- | ------ |
| 1   | null   |
| 2   | null   | 

```sql
SELECT id, status FROM test WHERE status = NULL;
```

返回的结果集包含 0 条数据


```sql
SELECT id, status FROM test WHERE status!=1;

SELECT id,status FROM test WHERE status='';
```

上述两种查询的返回的结果集统统包含 0 条数据

`!=1` 不能筛选 `NULL` 这种特殊类型的。应改成 `WHERE IFNULL(status, '') != 1;`


> [!NOTE] IFNULL
> 由两个参数，第一个参数是列名，第二个参数是如果列为 null 则用这个作为缺省值。函数返回列值或缺省值


必须用如下 sql 才能查出值为 null 的数据

```sql
SELECT id,status FROM test WHERE status is NULL;
```

## 聚合函数对 null 的处理

- count 会计算空字符串，不计算 null


## null 参与的逻辑运算，算数运算和比较运算

null 表示 unknown

MySQL 中，逻辑值有 `true`，`false`，`unknown`。null 就是 unknown


![Pasted image 20220611150120](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220611150120.png)

