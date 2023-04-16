```
bin/hive -help  

usage: hive  
-d,--define <key=value> Variable subsitution to apply to hive commands. e.g. -d A=B or --define A=B  
--database <databasename> Specify the database to use  
-e <quoted-query-string> SQL from command line  
-f <filename> SQL from files  
-H,--help Print help information  
--hiveconf <property=value> Use value for given property  
--hivevar <key=value> Variable subsitution to apply to hive  
commands. e.g. --hivevar A=B  
-i <filename> Initialization SQL file  
-S,--silent Silent mode in interactive shell  
-v,--verbose Verbose mode (echo executed SQL to the console)
```

[[folder/创建数据库]]


[[folder/获取数据库信息]]


[[folder/删除数据库]]

```
bin/hive -help  

usage: hive  
-d,--define <key=value> Variable subsitution to apply to hive commands. e.g. -d A=B or --define A=B  
--database <databasename> Specify the database to use  
-e <quoted-query-string> SQL from command line  
-f <filename> SQL from files  
-H,--help Print help information  
--hiveconf <property=value> Use value for given property  
--hivevar <key=value> Variable subsitution to apply to hive  
commands. e.g. --hivevar A=B  
-i <filename> Initialization SQL file  
-S,--silent Silent mode in interactive shell  
-v,--verbose Verbose mode (echo executed SQL to the console)
```

## 创建数据库
name=create database
type=DDL
desc=CREATE DATABASE [IF NOT EXISTS] database_name 创建数据库

**simple example**

```sql
create database db_hive;
```

创建一个数据库，数据库在 HDFS 上的默认存储路径是/user/hive/warehouse/\*.db

创建一个数据库还要包括：if not exists，指定数据库物理文件地址，加注释，添加额外的 kv 信息

**complete grammar**

```sql
CREATE DATABASE [IF NOT EXISTS] database_name  
[COMMENT database_comment]  
[LOCATION hdfs_path]  
[WITH DBPROPERTIES (property_name=property_value, ...)];
```

举个例子

```sql
create database db_hive2 if not exists location '/db_hive.db';
```


## 获取数据库信息
name=show, desc databases
type=DDL
desc=查看数据库列表，查看数据库的详细信息

**simple example**

查看有哪些数据库

```sql
hive> show databases;

hive> show databases like 'db_hive*';

OK  
db_hive  
db_hive_1
```

查看数据库的详细信息

```sql
hive> desc database db_hive;

hive> desc database extended db_hive;

OK  
db_hive hdfs://hadoop102:9820/user/hive/warehouse/db_hive.db  
```


## 删除数据库
name=drop database database_name
type=DDL
desc=删除数据库

删除空数据库  

```sql
hive> drop database db_hive2;
```

如果删除的数据库不存在， 最好采用 if exists 判断数据库是否存在
```sql
hive> drop database db_hive;

FAILED: SemanticException [Error 10072]: Database does not exist: db_hive  

hive> drop database if exists db_hive2;
```


如果数据库不为空，可以采用 cascade 命令，强制删除

```sql
hive> drop database db_hive;

FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask.InvalidOperationException(message:Database db_hive is not empty. One or more tables exist.)  

hive> drop database db_hive cascade;
```

