#还没有复习 

# Spring的常用容器注解



## 注册组件

向容器中注册组件的方式由很多种，下面介绍使用注解的方式向容器中添加组件

- `@Controller`/`@Service`/`@Repository`/`@Component`
- `@Configuration` + `@Bean`
- `@Import`

| 注解名          | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| @Configuration  | 作用于类，此注解修饰的类将成为**配置类**。配置类的作用和xml配置文件的作用相同。 |
| @Bean           | 和@Configuration一起使用。作用于方法，此方法的返回值将被放入容器中。和xml中的bean标签作用相同 |
| @Import         | 这个注解能快速注入组件，不过实现方式有很多，稍后再说         |
| @Component      | 被此注解修饰的类将被加到容器中（`@Controller`/`@Service`/`@Repository`这三个注解和`@Component`除了名字不一样外，功能是一样的） |
| @ComponentScan  | 其属性value，指定要扫描的包或类，支持正则表达式。只有被扫描到的配置类和组件才能被加入到容器。 |
| @ComponentScans | 组合注解。里面可放置多个@ComponentScan（涉及注解知识，这里不做介绍） |



辅助注解（和`@Bean`，`@Component`一起使用）

| 注解名       | 作用                                                         |
| ------------ | ------------------------------------------------------------ |
| @Scope       | 设置组件的作用域。例如设置组件是单例（默认）还是多例，或在一个request/session中是单例的 |
| @Lazy        | 懒加载。延迟初始化此bean，在需要使用此bean时再做初始化       |
| @Conditional | 按照条件注册bean。组件需要指定一个实现`Condition`接口的实现类（可自己实现）。如果实现类的方法能返回true就添加此注解修饰的组件 |

在SpringBoot中`@Conditional  `有许多拓展注解。例如`@ConditionalOnBean`，`@ConditionalOnClass`等，形如`@ConditionalOnXXX`



### @Configuration + @Bean

1. 写一个配置类
2. 在配置类中使用@Bean的方式向容器中添加组件
3. 测试用ioc容器获取组件

```java
@Configuration // 此注解表明MyConfig类是配置类
public class MyConfig {

    @Bean // 此方法返回的对象将被添加到容器中
    public User getUser() {
        return new User("张三");
    }

}
```

测试

```java
@org.junit.Test
public void test(){
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MyConfig.class);
    User user = ioc.getBean(User.class);
    System.out.println(user);
}
```

输出结果（像中间打印的日志以后不再展示）

```
获取ioc容器
11:13:26.872 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@38082d64
11:13:26.892 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
11:13:27.010 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
11:13:27.012 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
11:13:27.014 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
11:13:27.015 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
11:13:27.022 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'myConfig'
11:13:27.028 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'getUser'
User(name=张三)
```

- 最后一行输出说明@Bean修饰的方法的返回值确实被放入容器

- 由中间的日志可看出。bean由`DefaultListableBeanFactory`创建，且多为单例（Creating shared instance of singleton bean）

- 之前说bean被存在`DefaultListableBeanFactory `的众多`Map`中，以kv键值对形式存在。v是bean实例，那么k是什么？

- k是bean在容器中的名字  称beanName

  由"org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'getUser"可看出，此bean的名字就是@Bean修饰的方法名"getUser"
  
- 可使用`@Bean`注解的name属性主动修改bean的beanName。如`@Bean(name="zhangsan")`



### @Component

对`User`稍作修改

```java
package com.cx.bean;
@Component // 将此类的以一个实例注入容器
@Data
@AllArgsConstructor
public class User {
    String name;

    public User() {
        this.name = "李四";
    }
}
```

对配置类稍作修改

```java
@ComponentScan("com.cx") // 指定要扫描的包下的所有组件，User类就会被扫描到
@Configuration
public class MyConfig {
}
```

测试（和上次的测试代码一样）

输出

```
获取ioc容器
...（省略无关的日志输出）
User(name=李四)
```

- 说明`@Component`确实将`User`的一个实例添加到容器中



### @Import

对`@Import`的value属性赋值，可传入要加入容器中的类 或 使用以下接口的实现类添加bean（`ImportSelector`，`ImportBeanDefinitionRegistrar`，`FactoryBean`）

接下来对这四种方式进行实验

1. 对配置类稍作修改，指定需要注入的类

   ```java
   @Import(User.class) // 将User注入容器
   @Configuration
   public class MyConfig {
   }
   ```

   测试（测试代码依然不变，所以不再展示）

2. `ImportSelector`接口的实现类

   ```java
   // 自行创建实现类
   public class MyImportSelector implements ImportSelector {
       @Override
       public String[] selectImports(AnnotationMetadata importingClassMetadata) {
           return new String[]{"com.cx.bean.User"}; // 容器会将此String数组中的所有类添加到容器中（数组的元素值必须是类的全包名）
       }
   }
   ```

   对配置类稍作修改

   ```java
   @Import(MyImportSelector.class)
   @Configuration
   public class MyConfig {
   }
   ```

   测试（测试代码依然不变，所以不再展示）

3. `ImportBeanDefinitionRegistrar`接口的实现类

   ```java
   // 自行创建的实现类
   public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
       @Override
       public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
           // BeanDefinitionRegistry。上文说过，这个类用来创建和存放BeanDefinition的对象。将来容器将根据这些对象去实例化bean
           // 使用此方法向BeanDefinitionRegistry中添加BeanDefinition对象
           // 第一个参数：声明bean的beanName；第二个参数：指定组件的类对象
           registry.registerBeanDefinition("mybean", new RootBeanDefinition(User.class));
       }
   }
   ```

   对配置类稍作修改

   ```java
   @Import(MyImportSelector.class)
   @Configuration
   public class MyConfig {
   }
   ```

4. `FactoryBean`接口的实现类（创建bean的工厂对象）

   ```java
   // 自行创建的实现类
   public class MyFactoryBean implements FactoryBean<User> {
   	// 将返回值放入容器中。使用此方式创建bean时。bean将不执行bean生命周期中的初始化方法。
       @Override
       public User getObject() throws Exception {
           return new User("王五");
       }
   	// 指定创建的组件的类对象
       @Override
       public Class<?> getObjectType() {
           return User.class;
       }
   	// 返回true表示此bean为单例，false表示bean为多例
       @Override
       public boolean isSingleton() {
           return true;
       }
   }
   ```

   对配置类稍作修改

   ```java
   @Import(MyFactoryBean.class)
   @Configuration
   public class MyConfig {
   }
   ```

   测试（测试代码不变）



### BeanFactory和FactoryBean

BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是容器中的bean。在Spring中，所有的bean都由BeanFactory来进行管理。

FactoryBean，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂bean，它的实现与设计模式中的工厂模式和修饰器模式类似。




## Bean的生命周期

> Bean生命周期的详情可参照这篇[博客](https://www.cnblogs.com/javazhiyin/p/10905294.html)

Bean的生命周期的过程：

bean创建（调用构造方法）---初始化（调用指定的初始化方法）---bean销毁（调用指定的销毁方法）

我们可以自定义bean的初始化方法（调用构造函数之后调用）和销毁方法（销毁bean时调用）

**注：这里的初始化和类加载的初始化是不同的初始化**

下面介绍三种bean的使用初始化方法和销毁方法的方式

- 使用`@Bean`注解的的属性指定初始化方法和销毁方法

  ```java
  @Bean(initMethod="init", destoryMethod="destory") // 指定了初始化方法是Car类中名为"init"的方法，销毁方法同理
  public Car getCar(){
      return new Car();
  }
  ```

- 使用`@PostConstruct`指定初始化方法，`@PreDestroy`指定销毁方法。这两个注解都在Bean类的成员方法上使用

- 使Bean类实现指定初始化接口（InitializingBean）和销毁接口（DisposableBean）




### BeanPostProcessor接口

> 本质上使用了代理模式

`BeanPostProcessor`（译：后置处理器）的方法在上述初始化方法调用前后调用

后置处理器的`postProcessBeforeInitialization`方法在调用初始化方法之前调用

后置处理器的`postProcessAfterInitialization`方法在调用初始化方法之后调用

修改`User`类，使其实现`BeanPostProcessor`接口

```java
@Data
@AllArgsConstructor
public class User implements BeanPostProcessor {
    String name;

    public User() {
        this.name = "李四";
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("调用了User对象的postProcessBeforeInitialization");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("调用了User对象的postProcessAfterInitialization");
        return bean;
    }
}
```

修改配置类

```java
@Import({User.class})
@Configuration
public class MyConfig {
}
```

测试方法依旧不变

至于同时使用三种初始化方式+实现后置处理器接口的效果，请自行实现



## 组件赋值

| 注解名          | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| @Value          | 作用于Bean类的成员变量上。其作用和xml配置文件中bean标签的子标签property的value属性的作用一样 |
| @PropertySource | 加载指定的配置文件的配置到 Spring 的 Environment 中。@Value即可使用${}表达式从配置文件中取值 |

### @Value

稍微修改`User`

```java
@Data
@AllArgsConstructor
public class User{

    @Value("赵六") // 支持字符串，SqEl表达式和${}
    String name;

    public User() {
		System.out.println("调用了User的无参构造方法");
        this.name = "李四";
    }

}
```

配置类不变

```java
@Import({User.class})
@Configuration
public class MyConfig {
}
```

测试类不变

输出结果

````
获取ioc容器
...
调用了User的无参构造方法
User(name=赵六)
````

- 调用了`User`的无参构造方法，但其成员变量的值为`@Value`指定的值



### @PropertySource

写一个配置文件（resource文件夹下，名为application.yml）

```yaml
user.username: tom
```

配置类不变

```java
@PropertySource(value = {"classpath:/application.yml"}) // 指定配置文件
@Import({User.class})
@Configuration
public class MyConfig {
}
```

在`User`中获取配置文件的值

```java
@Data
@AllArgsConstructor
public class User{

    @Value("${user.username}")
    String name;

    public User() {
		System.out.println("调用了User的无参构造方法");
        this.name = "李四";
    }

}
```

测试类不变

输出结果

````
获取ioc容器
...
调用了User的无参构造方法
User(name=tom)
````

- 获取到配置文件中的值并成功赋值



配置文件中的值也可以这样获取到

```java
@org.junit.Test
public void getPropertyInEnvironment(){
    ConfigurableEnvironment environment = ioc.getEnvironment();
    String property = environment.getProperty("user.username");
    System.out.println(property);
}
```





## 自动装配

测试环境下可使用`ioc.getBean()`的方式获取指定的bean。但是这显然很麻烦，而且每次需要取bean时都需要先获取到容器对象。

下面介绍一种更优雅的方式获取到容器中的bean

### 测试环境

```java
// 这是下文使用的测试环境
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = {SpringAnnotationApplication.class})
public class SpringTest {
    
    @Test
    public void test(){
    }
    
}
```

接下来要用到的注解

| 注解名     | 作用                                               |
| ---------- | -------------------------------------------------- |
| @Autowired | 自动注入。获取到容器中的组件。具体使用见下文的实验 |
| @Qualifier | 和@Autowired一起使用。指定bean的beanName           |
| @Primary   | 指定默认装配的bean                                 |



### @Auowired

对配置类稍作修改

```java
@Configuration // 配置类像容器中添加了一个user对象
public class MyConfig {
    @Bean
    public User getUser() {
        return new User("张三");
    }
}
```

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = {SpringAnnotationApplication.class}) // 此类是此项目的启动类
public class SpringTest {

    @Autowired // 只需声明对象，程序会从容器中获取到user对象并赋值给此引用
    private User user;

    @Test
    public void test(){
        System.out.println(user);
    }

}
```

上述实验中，容器中只有一个`User`的对象，如果容器中有多个`User`的对象，容器会取出哪个bean？

- 默认优先按照类型去容器中查找对应组件。这里会优先在容器中找`User`类的对象。（如果声明的接口，容器会找它的实现类对象）
- 如果容器中有多个`User`的对象（但是beanName不同），容器会找`User`对象中beanName=="user"的bean（因为@Autowired修饰的对象名就是user）



### @Qualifier

`@Qualifier`可指明要装配的bean的beanName。可在不修改对象名的情况下获取到指定的bean

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = {SpringAnnotationApplication.class})
public class SpringTest {

    @Qualifier("zhangsan") // 此注解的value属性默认值为""
    @Autowired
    private User user;

    @Test
    public void test(){
        System.out.println(user);
    }

}
```



### @Primary

作用于类或方法

如果没用`@Qualifier`指定beanName时，则默认先装配有`@Promary`注解的bean

```java
@Configuration // 配置类像容器中添加了一个user对象
public class MyConfig {
    @Bean
    @Primary // 当容器中有多个同类型的bean且没有用@Qualifier时，容器会忽视@Autowired修饰的对象的对象名，优先获取有此注解的bean
    public User getUser1() {
        return new User("张三");
    }
    @Bean
    public User getUser2() {
        return new User("李四");
    }
    @Bean
    public User getUser3() {
        return new User("王五");
    }
}
```



以上三个注解均是Spring提供的注解。除此以外，还有其他注解也可以实现其功能

但是上述的三个注解更好用，所以可以大胆舍弃下面介绍的注解

- **@Resource**（JSR250） 默认按照注入的对象名从容器中查找名字对应的组件

-  **@Inject**（JSR330）需要添加新的maven依赖，功能和@Autowired一样，但是不能设置request=false。可以使用name属性指定beanName



### @Autowired在其他地方的使用

`@Autowired`注解能用在类，方法，变量，构造器等多个地方

下面说说，它在构造函数，成员变量，成员方法上的使用

```java
// 一、用在方法上
@Component
public class Boss{
    private Car car;
    
    // 容器创建对象的时候，会自动调用该方法，以完成赋值
    @Autowired
    public setCar(Car car){
        this.car = car;
    }
    
    public getCar(){
        return car;
    }
}

// 二、用在构造器上或参数列表上
// 容器创建对象时，默认调用无参构造器，再进行初始化赋值
public class Boss{
    private Car car;
    
    // 调用此构造器时，注入car。如果类只有一个构造器，那构造器上的Autowired也可以省略
    @Autowired
    Boss(/*@Autowired 放这也行*/Car car){
        this.car = car;
    }
    
    public setCar(Car car){
        this.car = car;
    }
    
    public getCar(){
        return car;
    }
}
```

常用方式是在`@Bean`下的方法参数列表中

```java
// 方法参数中的Car即使没有Autowired注解，也默认是从容器中找的对象
@Bean
public Boss getBossWithCar(Car car){
    Boss boss = new Boss();
    boss.set(car);
    return boss;
}
```



**@Autowired在List中的应用**

场景：一个接口`BaseInterface`，此接口有多个实现类`BaseImpl1`，`BaseImpl2`，`BaseImpl3`，且它们被注册进容器

应用：在某处使用

```java
@Autowired
private List<BaseInterface> baseList;
```

这将会获取容器中所有`BaseInterface`接口的实现类对象，并将它们封装到这和个List中



## Aware注入Spring底层组件

Spring有一些底层的组件，比如，applicationcontext，factorybean等

如果你想使用这些组件，可以创建一个类实现XXXAware接口。

比如想使用`ApplicationContext`，就创建一个实现`ApplicationContextAware`接口的类并注入容器

实现了这些接口后，即可使用接口方法中的底层组件的对象

```java
// 这是一个使用示例
@Component
public class Bean implements ApplicationContextAware, BeanNameAware {
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("自己注入的容器 " + applicationContext);
    }
    @Override
    public void setBeanName(String s) {
        System.out.println("这个bean的beanName是 " + s);
    }
}
```

每一个xxxAware都有一个对应的xxxAwareProcessor

容器在bean的初始化前后会调用后置处理器，因为此bean实现了`ApplicationContextAware`接口，所以会调用`ApplicationContextAwarePostProcessor`的方法去执行bean所实现的方法



## AOP

> 指在程序运行期间动态地将某段代码切入到指定方法指定位置进行运行的编程方式

![[../../../020 - 附件文件夹/Pasted image 20230402112316.png]]


| 注解名          | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| @Aspect         | 声明此类是切面类                                             |
| @Before         | 前置通知。在目标方法运行前运行                               |
| @After          | 后置通知。在目标方法运行结束后，执行return前运行。无论方法是正常结束还是异常结束都会执行 |
| @AfterReturning | 返回通知。在目标方法正常返回后运行。此注解下的通知方法可修改目标方法的返回值 |
| @AfterThrowing  | 异常通知。在目标方法出现异常后运行                           |
| @Around         | 环绕通知（最灵活的通知）。动态代理，手动推进目标方法的运行   |
| @Pointcut       | 切入点声明。此注解用于指定切点                               |



**实现流程**

1. 导入aop的相关maven依赖
2. 创建目标方法，并将所在类注入容器
3. 创建切面类，创建通知方法。添加相关注解



1. 导入的aop依赖

   ```xml
   <!--在SpringBoot下使用这个依赖-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```
   
2. 创建目标方法，并将所在类注入容器

   ```java
   package com.cx.controller;
   @Component
   public class TestController {
       // 目标方法
       public String test(){
           System.out.println("这里是TestController的test()方法");
           return "我是TestController的test()方法的返回值";
       }
   }
   ```
   
3. 创建切面类，创建通知方法。添加相关注解

   ```java
   @Component // 将切面类注入容器
   @Aspect // 指定此类是切面类
   public class TestControllerAop {
   
       // 切点表达式。指定目标方法（访问权限关键字 + 返回类型 + 全包名 + 方法名 + 参数列表）
       // 可以用*表示所有，可以用..表示所有参数表。例如"execution(public * com.cx.controller.*.*(..))" 表示com.cx.controller包下所有的public方法
       @Pointcut("execution(public String com.cx.controller.TestController.test(..))")
       public void pointcut() {}
   
       @Before("pointcut()")
       public void before(){  // 通知方法的方法名随便起
           System.out.println("前置通知");
       }
   
       @After("pointcut()")
       public void after(){
           System.out.println("后置通知");
       }
   
       @AfterReturning("pointcut()")
       public void afterReturning(){
           System.out.println("返回通知");
       }
   
   }
   ```

4. 测试类

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @SpringBootTest(classes = {SpringAnnotationApplication.class})
   public class SpringTest {
   
       @Autowired
       private TestController controller;
   
       @Test
       public void test() {
           String str = controller.test();
           System.out.println("test方法的返回值：" + str);
       }
   
   }
   ```
   
5. 运行结果

   ```
   前置通知
   这里是TestController的test()方法
   后置通知
   返回通知
   test方法的返回值：我是TestController的test()方法的返回值
   ```

   

注解中定义目标方法的返回值或所抛异常的对象名并使用此名在参数列表中接收对象。

```java
@AfterReturning(value="pointCut()", returning="res")
public void afterReturning(Object res){  // 参数类型可根据实际情况而定
    // 可在此处修改目标方法的返回值。
    // 如果返回结果是值类型的，在此处修改后返回值不会改变。返回结果是引用类型的，返回值可以被修改。
    ...
}

@AfterThrowing(value="poitCut()", throwing="exception")
public void afterThrowing(Exception exception){
    ...
}
```

**注：使用new创建一个目标类的对象并调用方法是不会有aop的效果的。使用容器中的目标类的对象，aop才能生效**



**环绕通知的实现**

稍微修改切面类

```java
@Component
@Aspect
public class TestControllerAop {

    @Pointcut("execution(public * com.cx.controller.*.*(..))")
    public void pointcut() {
    }

    @Around("pointcut()")
    public String around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕通知：执行目标方法前");
        // 执行目标方法，并获取返回值
        String res = joinPoint.proceed();
        System.out.println("环绕通知：执行目标方法后");
        return res;
    }

}
```

其它代码不变。输出结果为

```
环绕通知：执行目标方法前
这里是TestController的test()方法
环绕通知：执行目标方法后
test方法的返回值：我是TestController的test()方法的返回值
```



## 声明式事务

- 编程式事务。手动写过滤器等组件，使用try-catch-finally捕捉异常，回滚事务，关闭连接。设置需要执行此过滤的接口等
- 声明式事务。Spring实现声明式事务的方式是aop（所以需要导入aop的依赖），配置一个事务管理器；使用注解标明那些方法是事务方法

事务管理器就是Spring帮你写好了的切面类

事务管理器就是`PlatformTransactionManager`接口的实现类





声明式事务需要做两件事

- 使用`@Transactional`注解修饰类或方法，使其被事务管理。效果：方法运行时抛异常且异常没有被处理时此方法中的事务将会回滚
- 自定义事务管理器。例如：



### 手动配置声明式事务

1. 配置事务管理器

```java
@Bean
public PlatformTransactionManager transactionManager(){
	return new DataSourceTransactionManager(dataSource);
}
```

2. 使用`@Transactional`标注开启事务的方法

```java
@Transactional // 如果@Transactional标注的方法在执行时抛异常，方法中对数据库的操作会执行回滚
public void test(){
    ...
}
```



### @Transactional



### 事务的传播行为

事务的传播行为 = 事务的传播 + 事务的行为

事务方法A中使用了事务方法B和事务方法C

事务方法B抛异常，事务方法C中的操作会跟着一起回滚吗？

这由事务的传播行为控制，具体配置在`@Transactional`的属性`propagation`中

![[../../../020 - 附件文件夹/Pasted image 20230402112339.png]]


# 拓展

下面介绍Spring中的三个接口



## BeanFactoryPostProcessor接口

介绍：`BeanFactoryPostProcessor`是`BeanFactory`的后置处理器

执行时机：在所有的`BeanDefinition`已经被加载到`BeanFactory`但还未创建bean实例是执行此接口的方法

能干什么：可此接口中的`postProcessBeanFactory`方法中写向容器添加额外的bean的代码



## BeanDefinitionRegistryPostProcessor接口

介绍：`BeanDefinitionRegistryPostProcessor` 是`BeanFactoryPostProcessor`的子接口

执行时机：在所有的`BeanDefinition`还未被加载到`BeanFactory`时执行此接口的`postProcessBeanDefinitionRegistry`方法。所以此接口方法的调用要早于`BeanFactoryPostProcessor`接口方法的调用

能干什么：可在此接口的方法`postProcessBeanDefinitionRegistry`中写向`BeanDefinitionRegistry`中添加额外的`BeanDefinition`的代码。



执行顺序

1. `BeanDefinitionRegistryPostProcessor` 的`postProcessBeanDefinitionRegistry`
2. `BeanDefinitionRegistryPostProcessor` 的`postProcessBeanFactory`
3. `BeanFactoryPostProcessor`的`postProcessBeanFactory`



上述两个接口均可自行创建实现类，并添加到容器中使用。



## ApplicationListener接口

监听容器中发布的事件的组件，完成事件驱动型开发

`ApplicationListener`接口。用于监听，处理事件

`ApplicationEvent`事件。`ApplicationListener`只能监听此类和此类的子类

事件发布者是`ApplicationEventPublisher`的实现类。`AbstractApplicationContext`实现了`ApplicationEventPublisher`的`publishEvent()`方法。（发布事件的方法）



下面来一个自定义监听器并主动发布事件示例

自定义一个监听器组件。实现`ApplicationListener`接口

```java
@Component
public class MyListener implements ApplicationListener<ApplicationEvent> {
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("接收到了事件：" + event);
    }
}
```

获取context，发布事件

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = {SpringAnnotationApplication.class})
public class SpringTest {

    @Autowired
    private ApplicationContext context;

    @Test
    public void test(){
        // ApplicationEvent是一个抽象类，这里以创建匿名内部类的方式创建其对象
        context.publishEvent(new ApplicationEvent(new String("我发布了事件")){});
    }

}
```

输出结果

```
接收到了事件：org.springframework.test.context.event.BeforeTestMethodEvent...
接收到了事件：org.springframework.test.context.event.BeforeTestExecutionEvent...
接收到了事件：SpringTest$1[source=我发布了事件]
接收到了事件：org.springframework.test.context.event.AfterTestExecutionEvent...
接收到了事件：org.springframework.test.context.event.AfterTestMethodEvent...
```

- 在众多日志中找到了我发布的事件...说明事件发布成功了，监听器也接收到了事件



## InitializingBean接口

此接口下有这个方法

```java
void afterPropertiesSet() throws Exception;
```

执行时机：执行`doCreateBean()`时执行`initializeBean()`——`invokeInitMethods()`（bean已经被实例化，但是还为执行依赖注入）

如果bean实现了`InitializingBean`接口，就调用`afterPropertiesSet()`方法

