#还没有复习 

[笔记参照这里](https://www.cnblogs.com/liuyangfirst/tag/MybatisPlus/)


## quickly start

1. 创建实体类User.class

2. 创建dao层接口

   ```java
   @Mapper
   public interface UserMapper extends BaseMapper<User> {
   }
   ```
   
3. 创建service接口

   ```java
   public interface UserService extends IService<User> {
   }
   ```

4. 创建sercive接口的实现类

   ```java
   @Service
   public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
   }
   ```


## 注解

在使用MyBatis-Plus实现基本的CRUD时，我们并没有指定要操作的表，只是在Mapper接口继承BaseMapper时，设置了泛型User，而操作的表为user表。由此得出结论，MyBatis-Plus在确定操作的表时，由BaseMapper的泛型决定，即实体类型决定，且默认操作的表名和实体类型的类名一致。
————————————————
版权声明：本文为CSDN博主「幻清」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_47905231/article/details/125056673


### @TableId

作用在实体类的id变量上，表示则是主键。

参数type有四种值，一般用 IdType.AUTO

参数value，如果实体类变量名和表中字段名相同则不需要写，如果不同，需要在这里写上变量对应的表中字段名

### @TableName

作用在实体类上，用于指定这个实体类对应的表

name的值就是表名

### @TableField

作用在实体类的成员变量，具体功能看javadoc


## 技巧

- 在插入一条数据后，可以直接获取对象的id（哪怕再次之前你没有setId）

  ```java
  User user = new User();
  user.setName(UUID.randomUUID().toString());
  user.setAge(20);
  userService.save(user);
  
  Integer id = user.getId();
  System.out.println(id);
  ```

- XXXServiceImpl继承了SerciveImpl<A, B>

  而SerciveImpl中已经使用了@Autowired Mapper，所以这里不需要再装配mapper了

- @JsonInclude(JsonInclude.Include.NON_NULL)

  ```
  在项目中常用到的几个注解@JsonInclude、@JsonFormat、@DateTimeFormat
  ```


## 插件

> MB和MP中的插件类似于拦截器，一般用来拦截一个对象返回代理对象，执行代理方法



### 分页插件

PaginationInterceptor

- 能干什么

  原生的MP分页是逻辑分页（获取全部数据再分页），这个插件将会把 其变为物理分页（limit真正的在查询的时候就分页了）

- 怎么用

  Spring下@Bean PaginationInterceptor添加进容器中即可



### 执行分析插件

SqlExplainInterceptor

- 能干什么

  防止update / delete 全表数据

- 怎么用

  注册到容器中即可



### 性能分析插件

> 3.2.0版本后移除

- 能干什么

  监视sql语句执行时间，设置一条sql语句最长执行时间



## 自定义sql语句

> MB是通过xml写sql，MP是通过拓展，在加载MP的时候就注入一些slq语句，至于你可以像quickly start 上一样直接使用方法即可。这里要做的就是自定义那些加载MP时就注入的sql语句

**DefaultSqlInjector**

[例子](https://www.cnblogs.com/liuyangfirst/p/9744011.html)

1. 创建一个类（名为“A”）

2. 把A类注入容器

3. A继承DefaultSqlInjector，并实现其方法

`void inspectInject(MapperBuilderAssistant builderAssistant, Class<?> mapperClass);`


# 问题

`@Repository`，`@Mapper`

对于各种XXXMapper接口，这两个注解不用添加也可以

不过，在启动类上添加`@MapperScanner`是必须的

而且，对于`@Repository`，如果是在Mapper对应的ServiceImpl中使用的话，mapper接口不使用此注解也可以

如果在其他ServiceImpl中使用`@Autowired`装配mapper的话，就需要此注解了