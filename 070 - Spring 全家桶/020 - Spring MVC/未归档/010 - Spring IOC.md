#还没有复习 

学的是 Spring Framework，所以以下用法均仅在纯`spring-context`依赖中就能使用（AOP 需要引入新的依赖）

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring.version}</version>
</dependency>
```



# 配置文件式用法

1 个`ClassPathXmlApplicationContext`+ 1 个`application.xml`配置文件

```
spring-ioc-appcontext
├── pom.xml
└── src
    └── main
        ├── java
        └── resources
            └── application.xml
```

```java
public static void main(String[] args) {
    ApplicationContext application = new ClassPathXmlApplicationContext("application.xml");
}
```

创建对象时，对象的成员变量如果是基本数据类型，及其包装类和 String，就用属性 value 指定

如果是 object 就用 ref，其值为其他 bean 的 id

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="baoma" class="com.example.bean.Car">
        <property name="color" value="blue"/>
    </bean>

    <bean id="zhangsan" class="com.example.bean.User">
        <property name="name" value="zhangsan"/>
        <property name="age" value="12"/>
        <property name="car" ref="baoma"/>
    </bean>
</beans>
```

标签通过`constructor-arg`指定构造器，默认用无参构造器

或通过`property`调其 setter 方法为其赋值



# 注解式用法

1 个`AnnotationConfigApplicationContext`+ 1 个带有`@ComponentScan`的类 A

`@ComponentScan`的扫描范围是 A 类所在的包及其所有子包下的 bean，bean 名默认为类名，但首字母小写（bean 名大小写敏感）

A 也会被扫进去（即使`@ComponentScan`没有`@Component`加成），因为容器的构造函数默认会直接把 A 类当成 bean 注册进去

````java
@ComponentScan
public class AppConfig {
    public static void main(String[] args) {
        ApplicationContext application = new AnnotationConfigApplicationContext(AppConfig.class);
    }
}
````

```java
@Component
public class User {
    private String name;
    private Integer age;
    @Autowired
    private Car car;
    // 省略构造函数和 getter，setter 方法
}
```

`@Autowired`可以写在 User 的成员变量上，setter 方法上，构造函数上，`@Bean`加成的方法的参数列表上

`@ComponentScan`的扫描标准是有`@Component`加成的类

> 通过`@ComponentScan`获取扫描范围并进行扫描 bean 的工作是`ConfigurationClassPostProcessor`做的



# 注解定制 bean

- `Scope`注解指定 bean 的声明周期
- `Autowired`注入 List。IOC 会寻找容器中所有实现`Validator`接口的 bean，并加到此 list 中

```java
@Autowired
List<Validator> validators;
```

- `Order`控制 bean 的注入顺序。也可以控制`Autowired`把 bean 注入 List 时的顺序
- `@Autowired(required = false)`找不到 bean 就不找了
- `@Configuration`+`@Bean`创建第三方包中的类的 bean
- `@PostConstruct`和`@PreDestroy`为 bean 绑定初始化方法和销毁方法
- `@Qualifier`在创建 bean 或注入 bean 时指定 bean 的别名（也是 bean 的 id）
- `@Primary`表示在注入 bean 时，如果找到多个同类型的 bean，优先注入有此注解的 bean
- FactoryBean 工厂创建 bean

# Resource 读取文件

在 Spring 下读取文件非常简单

```java
@Component
public class AppService {
    @Value("classpath:/logo.txt") // 也可用绝对路径 "file:/path/to/logo.txt"
    private Resource resource;
}
```

调用`Resource.getInputStream()`就可以获取到输入流



使用 Maven 的标准目录结构时 classpath 包含了`src/main/resources`，所以所有资源文件放入`src/main/resources`即可



# 从文件或 bean 中读取配置

从文件中读取配置

```java
@PropertySource("user.properties") // 表示读取 classpath 的 user.properties
public class User {
    @Value("${people.name:Z}") // 表示把 people.name 的值注入 name。默认值为 Z。如果没有默认值，且找不到此配置则报错
    String name;
}
```

从 bean 中读取配置

```java
@Value("#{service.data}")
private String name;
```

`#{}`中是 beanName.filedName



`@Value`可以用在成员变量的声明或`@Bean`加成的方法的参数列表中



# 条件装配

条件装配指判断给定条件后决定是否创建 bean

- `@Profile`不常用
- `@Condition`常用



Spring 规定了 3 中环境：native - 开发，test - 测试，production - 生产

并通过 JVM 参数`-Dspring.profiles.active=`指定

```java
@Bean
@Profile("!test") // 表示在非 test 环境下会创建此 bean
ZoneId createZoneId() {
    return ZoneId.systemDefault();
}
```

此 JVM 参数和`@Profile`的值均为 环境名[,分支]。比如`-Dspring.profiles.active=test,master`或`@Profile({"test", "master"})`



```java
@Component
@Conditional(MyCondition.class)
public class User{...}
```

MyCondition.class 必须实现 Condition 接口，并实现其 matches 方法。如果方法返回 true，就创建 bean

Spring Framework 仅提供了`@Condition`接口。SpringBoot 基于`@Condition`提供更多种类的注解





# AOP

> 指在程序运行期间动态地将某段代码切入到指定方法指定位置进行运行的编程方式

引入新依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>${spring.version}</version>
</dependency>
```

在`@Configuration`类上标注`@EnableAspectJAutoProxy`

1. 创建目标方法，并将所在类注入容器

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

2. 创建切面类，创建通知方法。添加相关注解

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
       }
   
       @After("pointcut()")
       public void after(){
       }
   
       @AfterReturning("pointcut()")
       public void afterReturning(){
       }
   }
   ```

   

注解中定义目标方法的返回值或所抛异常的对象名并使用此名在参数列表中接收对象。

```java
@AfterReturning(value="pointCut()", returning="res")
public void afterReturning(Object res){
    // 如果返回结果是值类型的，在此处修改后返回值不会改变。返回结果是引用类型的，返回值可以被修改。
    ...
}

@AfterThrowing(value="poitCut()", throwing="exception")
public void afterThrowing(Exception exception){
    ...
}
```

**注：用 new 创建一个目标类对象并调用方法是不会有 aop 的效果的**



**环绕通知的实现**

```java
@Component
@Aspect
public class TestControllerAop {
    @Pointcut("execution(public * com.cx.controller.*.*(..))")
    public void pointcut() {}

    @Around("pointcut()")
    public String around(ProceedingJoinPoint joinPoint) throws Throwable {
        // 执行目标方法，并获取返回值
        String res = joinPoint.proceed();
        return res;
    }

}
```



`@Pointcut`的属性也可以是注解，表示拦截有此注解的方法

比如：`@Around("@annotation(metricTime)")`。拦截所有有`@MetricTime`的方法

这种方法，因为需要在 bean 的方法上标注，所以可以很清楚地知道哪些方法被 ”切“ 了，可读性更高



> Spring 对接口类型使用 JDK 动态代理，对普通类使用 CGLIB 创建子类。如果一个 Bean 的 class 是 final，Spring 将无法为其创建子类



**Spring 通过 CGLIB 创建的代理类的坑**

AOP 的本质是动态代理，创建代理 proxy A，它会继承 B，并持有 B 的实例

但 A 的 class 字节码文件是直接生成的，不是由 JVM 经过用源文件经过正规编译生成的字节码，所以 A 继承了 B，但不会调用`super();`执行 B 的初始化方法

Java 编译器在编译时，默认把成员变量的赋值从声明处转移到构造方法里，所以 A 访问 B 的成员变量时可能获得 null 值