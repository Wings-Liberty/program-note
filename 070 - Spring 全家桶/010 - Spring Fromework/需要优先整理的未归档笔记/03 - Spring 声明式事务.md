#还没有复习 

# 事务实现方式

## 手动事务

纯 JDBC 实现的手动事务方式如下

```java
Connection conn = openConnection();
try {
    // 关闭自动提交:
    conn.setAutoCommit(false);
    // 执行多条SQL语句:
    insert(); update(); delete();
    // 提交事务:
    conn.commit();
} catch (SQLException e) {
    // 回滚事务:
    conn.rollback();
} finally {
    conn.setAutoCommit(true);
    conn.close();
}
```

Spring 提供了事务管理器后，手动实现事务的方式如下

```java
TransactionStatus tx = null;
try {
    // 开启事务:
    tx = txManager.getTransaction(new DefaultTransactionDefinition());
    // 相关JDBC操作:
    jdbcTemplate.update("...");
    jdbcTemplate.update("...");
    // 提交事务:
    txManager.commit(tx);
} catch (RuntimeException e) {
    // 回滚事务:
    txManager.rollback(tx);
    throw e;
}

```

为了让事务和业务逻辑解耦，复用创建、提交、回滚事务的模板代码以及提供事务传播级别功能，Spring 提供了声明式事务

声明式事务的目的在于用注解式开发方式操作事务管理器管理事务，让事务代码和业务逻辑代码分离

## Spring 的事务管理器

JavaEE 提供了 JDBC 事务和分布式事务 JTA（Java Transaction API），所以 Spring 决定抽象出 `PlatformTransactionManager` 和 `TransactionStatus` 分别代表事务管理器和事务

# 声明式事务使用方式

## 代码实现
> [!todo] 各种事务实现方式具体代码如下
> - [ ] 用纯 JDBC 实现 curd，事务
> - [ ] Spring 事务管理器实现
> - [ ] Spring 声明式事务实现


1. 创建数据源，JDBCTemplate（无事务的可用版本）
2. 创建事务管理器
3. 在配置类上添加 `@EnableTransactionManagement` 
4. 在方法或类上添加 `@Transactional` 

配置代码如下

```java
@Configuration  
@EnableTransactionManagement  
@ComponentScan("com.demo.transaction")  
public class Config {  
  
    @Bean // 事务管理器
    public PlatformTransactionManager transactionManager(DataSource dataSource) {  
        return new DataSourceTransactionManager(dataSource);  
    }  
  
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {  
        return new JdbcTemplate(dataSource);  
    }  
  
    @Bean // 数据源  
    public DataSource dataSource(){ // 如果没有显式指定驱动，Druid 会自动选一个驱动
        DruidDataSource dataSource = new DruidDataSource();  
        dataSource.setUrl("jdbc:mysql://101.43.143.178:3306/test");  
        dataSource.setUsername("root");  
        dataSource.setPassword("1434948003Cx!");  
        return dataSource;  
    }  
  
}
```

业务调用代码如下

```java
public static void main(String[] args) {  
    ApplicationContext ioc = new AnnotationConfigApplicationContext(Config.class);  
    Service service = ioc.getBean(Service.class);  
    service.test();  
}
```

业务代码实现如下

```java
@Component  
public class Service {  
    @Autowired  
    private JdbcTemplate jdbcTemplate;
    
    @Transactional  
    public void test(){  
        // 直接调用 jdbcTemplate
        jdbcTemplate.execute(sql);
        // 把 jdbcTemplate 传到另一个方法内再调用
        f(jdbcTemplate);
    }  
  
}
```


> [!NOTE] @Transactional 的作用范围
> - `@Transactional` 作用于方法上，方法就表示一个 `事务方法`
> - `@Transactional` 作用于类上，类中所有 public 都被视为 `事务方法`
>   
> 所以 @Transactional 本质上是修饰方法的



> [!NOTE] @Transactional 执行回滚的时机
> - 默认事务方法抛出 RuntimeException 或 Error 时会执行回滚
> - 可在 @Transactional 的 rollbackFor 属性上指定异常类型覆盖掉默认值


## 定义事务所有行为

事务的行为包含：
- 是否自动提交 / 开启事务，提交，回滚
- 事务的隔离级别
- 事务方法遇到哪种异常后会执行回滚
- 事务的传播级别

这些行为都能在 @Transactional 的属性中设置


## 事务的边界

在命令行里使用事务时，能直接根据事务开始命令和提交命令划分事务的起始和结束边界

在声明式事务的代码里，事务的开始和提交代码都被隐藏起来了。

在声明式事务里，@Transactional 修饰的方法就是一个事务方法，进入方法表示开始事务，退出方法表示结束事务


Q：划分事务边界的目的是什么？

A：解决两件事：
- 事务方法 A 内调用另一个事务方法 B 是否会开启一个新的事务
- 事务方法 A 内调用事务方法 B，B 抛出了异常，A 内已经成功执行的 sql 是否需要回滚


> [!NOTE] 上述两个问题的本质
> 其实这两件事本质上是同一件事。一个事务回滚只能回滚事务内部的 sql，所以如果 B 开启了一个新事物，B 引起的回滚只会回滚 B，不会回滚 A


## 事务的传播策略

场景：事务方法 A 内调用事务方法 B

Spring 的 `Propagation` 提供了很多事务传播策略，但常用的就两三个

默认传播级别是 `Propagation.REQUIRED`：如果外围方法已经开启了事务，就不开启新的事务。如果外围方法没有开始事务，就开始一个新事务

`SUPPORTS`：表示如果有事务，就加入到当前事务，如果没有，那也不开启事务执行。这种传播级别可用于查询方法，因为SELECT语句既可以在事务内执行，也可以不需要事务；

`MANDATORY`：表示必须要存在当前事务并加入执行，否则将抛出异常。这种传播级别可用于核心更新逻辑，比如用户余额变更，它总是被其他事务方法调用，不能直接由非事务方法调用；

`REQUIRES_NEW`：表示不管当前有没有事务，都必须开启一个新的事务执行。如果当前已经有事务，那么当前事务会挂起，等新事务完成后，再恢复执行；

`NOT_SUPPORTED`：表示不支持事务，如果当前有事务，那么当前事务会挂起，等这个方法执行完成后，再恢复执行；

`NEVER`：和`NOT_SUPPORTED`相比，它不但不支持事务，而且在监测到当前有事务时，会抛出异常拒绝执行；

`NESTED`：表示如果当前有事务，则开启一个嵌套级别事务，如果当前没有事务，则开启一个新事务。

上面这么多种事务的传播级别，其实默认的`REQUIRED`已经满足绝大部分需求，`SUPPORTS`和`REQUIRES_NEW`在少数情况下会用到，其他基本不会用到，因为把事务搞得越复杂，不仅逻辑跟着复杂，而且速度也会越慢。


> [!warning] 一个事务的传播策略只能在单线程内才能生效
> 事务传播行为通过 `ThreadLocal` 实现，如果 `ThreadLocal` 里有存放能代表事务的 `TransactionStatus` 对象，说明有事务存在，在执行事务传播策略
> 
> 所以如果在新线程中，通常情况下是获取不到先前事务的 `TransactionStatus` 对象。这会导致事务传播策略执行失败




> [!quote] 参考
> [廖雪峰 - 事务边界和传播级别](https://www.liaoxuefeng.com/wiki/1252599548343744/1282383642886177#:~:text=BusinessException%20%7B%0A%20%20%20%20...%0A%7D-,%E4%BA%8B%E5%8A%A1%E8%BE%B9%E7%95%8C,-%E5%9C%A8%E4%BD%BF%E7%94%A8%E4%BA%8B%E5%8A%A1)



# 声明式事务实现方式

本质上用的是 [[02 - Spring AOP|Spring AOP]]

`@EnableTransactionManagement` 向容器中添加了 

- `InfrastructureAdvisorAutoProxyCreator`，它是 `AbstractAutoProxyCreator` 的子类
- `BeanFactoryTransactionAttributeSourceAdvisor` 作为事务的 advisor
- `TransactionInterceptper` 作为事务的 advice 被 advisor 包装


> [!warning] @EnableTransactionManagement 和 @EnableAspectJAutoProxy 的冲突
> 这两个注解都会引导创建一个 `AbstractAutoProxyCreator` 的实现类，但 @EnableAspectJAutoProxy 的优先级更高，会覆盖掉事务管理向容器中添加的 bean，所以开启事务管理后就不要再加 @EnableAspectJAutoProxy 了

创建过 `BeanFactoryTransactionAttributeSourceAdvisor` 后，在 `AbstractAutoProxyCreator` 调用 `postProcessAfterInitialization` 时会以类有无被 `@Transactional`  修饰的方法为切入点表达式的判断条件，为 bean 创建代理对象

代理对象的方法执行流程主要在 `TransactionAspectSupport` 里

```java
protected Object invokeWithinTransaction(Method method, Class<?> targetClass,  
      final InvocationCallback invocation) throws Throwable {  
  
	PlatformTransactionManager ptm = asPlatformTransactionManager(tm);   
    // 如果有必要就开启事务（配合事务传播策略，有时可能不需要开启新事务）
    TransactionInfo txInfo = createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);  
  
    Object retVal;
    try {  
	    // 调用链，最终会调用到目标的目标方法  
        retVal = invocation.proceedWithInvocation();  
	}  
    catch (Throwable ex) {  
	    // 如果遇到需要执行回滚的异常，就回滚。否则执行提交
        completeTransactionAfterThrowing(txInfo, ex);  
        throw ex;  
    }  
    finally {
        cleanupTransactionInfo(txInfo);  
    }  
    // 执行提交
	commitTransactionAfterReturning(txInfo);  
	return retVal;
}
```

> [!quote] 参考
> [[../../91 - 静态资源/08-声明式事务源码.pdf|Spring 声明式事务源码解析]]
> 
