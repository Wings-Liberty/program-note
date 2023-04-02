#还没有复习 

参考：

[Mybatis初始化流程及其重要的几个类](https://blog.csdn.net/l454822901/article/details/51829785)

[SqlSessionFactory和SqlSession的介绍和运用](https://blog.csdn.net/u013412772/article/details/73648537)

[SqlSessionFactoryBean 源码解析](https://www.jianshu.com/p/197358dc3ef0)

[@Mapper和@MapperScan的区别](https://www.cnblogs.com/muxi0407/p/11847794.html)

[Spring整合Mybatis原理](https://www.cnblogs.com/raitorei/articles/12880617.html)





## Java中JDBC的使用



使用`JDBC`的步骤包括

1. 设置好数据库的url，用户名，密码等常量

2. 加载驱动类，例如`MySQL`的`com.mysql.cj.jdbc.Driver`。

   加载`com.mysql.cj.jdbc.Driver`时会执行此驱动类静态代码块`java.sql.DriverManager.registerDriver(new Driver());`，将驱动注册进`DriverManager`

3. 使用`Driver`或`DriverManager`创建`Connection`对象

4. 使用`Connection`对象控制事务是否自动提交，手动提交，回滚或创建`Statement`对象执行sql语句



## MyBatis



### MyBatis的初始化和简单使用



下述代码完成了 将mybatis的xml配置文件加载到`Conguration`对象中，创建`SqlSessionFactory`对象，创建`SqlSession`对象，调用指定mapper中的方法完成一次数据库查询

```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession sqlSession = sqlSessionFactory.openSession();
List list = sqlSession.selectList("com.foo.bean.BlogMapper.queryAllBlogInfo");
```



**Configuration**类的结构和mybatis的xml配置文件的结构是一一对应的



`SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);`

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
        // 此对象用于解析
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        // parser.parse()实现将mybatis的xml配置文件解析到Java中的Configuration对象，并返回此对象
        return build(parser.parse());
    } catch (Exception e) {
        throw ...
    } finally {
        ErrorContext.instance().reset();
        try {
            inputStream.close();
        } catch (IOException e) {
            // Intentionally ignore. Prefer previous error.
        }
    }
}

// 创建并返回一个DefaultSqlSessionFactory对象
public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
}
```



上述使用mybatis的过程中，将解析mybatis配置文件的流程封装到了`SqlSessionFactoryBuilder.build()`方法内部

我们也可以使用重载方法，将`Configuration`对象作为参数传入`build`方法，并显式解析配置文件

```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
// 手动创建XMLConfigBuilder，并解析创建Configuration对象
XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, null,null);
Configuration configuration=parse();
// 使用Configuration对象创建SqlSessionFactory
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
// 使用MyBatis
SqlSession sqlSession = sqlSessionFactory.openSession();
List list = sqlSession.selectList("com.foo.bean.BlogMapper.queryAllBlogInfo");
```





### SqlSessionFactory



**SqlSessionFactory的作用**

`SqlSessionFactory`是个单个数据库映射关系经过编译后的内存镜像。每一个`MyBatis`的应用程序都以一个`SqlSessionFactory`对象的实例为核心。`SqlSessionFactory`提供创建`SqlSession`对象的方法



**SqlSessionFactory的创建方式**

- 直接new一个对象，并为它配置所需要的`Configuration`对象
- 用`SqlSessionFactoryBuilder`的`build()`方法创建。这个方法被重载多次，其核心是将mybatis的xml文件解析到`Configuration`对象中，在此基础上创建一个`SqlSessionFactory`对象



**SqlSessionFactory的线程安全性**

`SqlSessionFactory`是线程安全的,`SqlSessionFactory`一旦被创建,应该在应用执行期间都存在。在应用运行期间不要重复创建多次，建议使用单例模式，`SqlSessionFactory`是创建`SqlSession`的工厂



### SqlSession



**SqlSession的作用**

`SqlSession`是执行持久化操作的对象。类似于`JDBC`中的`Connection`。



**SqlSession的实现**

`SqlSession`接口提供CURD方法和对事务的管理方法**和获取mapper代理对象的方法**，它底层封装了`JDBC`



**SqlSession的线程安全性**

它是应用程序与持久层之间执行交互的一个单线程对象。每个线程都应该有自己的`SqlSession`实例。`SqlSession`的实例不能被共享，同时`SqlSession`也是线程不安全的





## Spring整合MyBatis的工作原理



### SqlSessionFactoryBean

`SqlSessionFactoryBean`的定义信息经常这样使用

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 指定mybatis全局配置文件的位置 -->
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <!-- 指定数据源 -->
    <property name="dataSource" ref="pooledDataSource"/>
    <!-- 指定mybatis，mapper文件的位置 -->
    <property name="mapperLocations" value="classpath:mapper/*.xml"/>
</bean>
```



`SqlSessionFactoryBean`类实现了`InitializingBean`接口。调用其`afterPropertiesSet()`方法时创建`SqlSessionFactory`对象并赋给成员变量`this.sqlSessionFactory`





## Spring IoC整合MyBatis



在`Spring`项目中使用`MyBatis`后是这样使用的

```java
@Component
public class UserService {
    @Autowired
    private UserMapper userMapper;

    public User getUserById(Integer id) {
        return userMapper.selectById(id);
    }
}
```



我们关注的就是`Spring IoC`是怎么将`UserMapper`这个接口实例化并添加到容器中的



解决方案：（以`UserMapper`为例）使用`FactoryBean`为`UserMapper`接口创建一个声明类型是`UserMapper`，实际类型是动态代理的对象

并将对象添加到容器中

但是自定义的`XxxMapper`是你自己写的，是不确定的。框架不会为每个`XxxMapper`创建一个`XxxFactoryBean`，所以这个`FactoryBean`持有一个成员变量，保存它将要创建的对象的类型。以此做到创建各种`Mapper`类的代理对象



在`Spring`整合`MyBatis`的项目中，这个`FactoryBean`就是`MapperFactoryBean`



现在还有一个问题，每个`Mapper`都要有一个`MapperFactoryBean`。那`MapperFactoryBean`在哪里被创建？



由`MapperFactoryBean`和`@MapperScan`来实现



```java
@Import(MapperScannerRegistrar.class)
public @interface MapperScan {
    String[] basePackages() default {};
    ...
}
```

`@MapperScan`的作用是向容器中添加一个`MapperScannerRegistrar`对象，并记录需要被扫描的包

这些包下的`XxxMapper`将会被`MapperScannerRegistrar`扫描到并为其创建一个`MapperFactoryFactoryBean`类型的`BeanDefinition`对象）

