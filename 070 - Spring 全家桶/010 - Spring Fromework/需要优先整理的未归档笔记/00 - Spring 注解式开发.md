#还没有复习 

[正在看的 Spring 文档](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class)


- 注册 bean 的方式
- 组件赋值方式
	- 配置文件赋值
	- ref 容器中的其他 bean 的自动装配
- 自定义 bean 创建时的行为


# 注册 bean 的方式

| 注册方式                                     |说明|
|:-------------------------------------------- |:------------------------- |
| @Configuration + @Bean                       | 指定配置类和要注入的 bean |
| @ComponentScan + @Component | 扫描指定包内的 bean       |
| @Import                                      | 指定要注入的类            |
| FactoryBean 接口                             | 工厂创建 bean             |



## @Configuration + @Bean

- `@Configuration` 作用于类，这个注解告诉 Spring 这个类是配置类
- `@Bean` 作用于方法，方法名就是 BeanName，方法返回值就是 bean。方法参数名为 BeanName，方法参数值为根据 BeanName 从容器中找到的 bean 对象

```java
@Configuration  
public class Config {  

    @Bean  
    public Car car(){ return new Car("red"); }  
  
    @Bean  
    public Person zhangsan(Car myCar) { return new Person("zhangsan", myCar); }  
  
}
```

```java
public static void main(String[] args) {  
    ApplicationContext ioc = new AnnotationConfigApplicationContext(Config.class);  
    Person zhangsan = ioc.getBean("zhangsan", Person.class);  
    System.out.println(zhangsan);  
}
```


## @ComponentScan + @Component

- `@ComponentScan` 需要和 `@Configuration` 一块修饰一个类才能生效
- `@Component` 作用于类上，类名就是 BeanName，但会进行小驼峰风格命名的修改

```java
@Configuration  
@ComponentScan({"com.my.app.pojo"}) // 指定要扫描的包
public class Config {  
  
}
```



> [!NOTE] @Component 有很多等效注解
> @Controller/@Service/@Repository 都是 @Component 的等效注解

## FactoryBean 工厂模式

```java
@Component  
public class MyFactoryBean extends AbstractFactoryBean<Car> {  
  
    private volatile Car car;  
  
    private volatile int cnt = 3; // 这辆车用三次后就创建创建一个新车  
  
    public MyFactoryBean() {  
        setSingleton(false);  
    }  
  
    @Override  
    public Class<?> getObjectType() {  
        return Car.class;  
    }  
  
    @Override  
    protected synchronized Car createInstance() throws Exception {  
        if (cnt++ < 3) {  
            return car;  
        }  
        cnt = 0;  
        return (car = new Car(UUID.randomUUID().toString()));  
    }
```

默认 bean 的名字是 FactoryBean 实现类的类名（进行过小驼峰命名修改后的）

```java
public static void main(String[] args) {  
    ApplicationContext ioc = new AnnotationConfigApplicationContext(Config.class);  
  
    for (int i = 0; i < 20; i++) {  
        System.out.println(ioc.getBean("myFactoryBean")); // 获取到的是工厂类创建的对象，而不是工厂类对象本身
    }  
}
```

FactoryBean 在容器中的名字是 '&' + BeanName，如果想获取 FactoryBean 对象，就用这个名字或直接用 Class 获取


## @Import 快速注册组件

根据类对象指定需要注册的 bean，`@Import` 注解把类对象分为两类

- 普通类对象（不能是抽象类或接口，可以是普通的 bean，也可以是用 `@Configuration` 修饰的配置类）
- 实现了 ImportSelector 或 ImportBeanDefinitionRegistrar 接口的类对象


# 定制 bean 的加载时机

## @Scope 设置组件作用域

常用的作用域有：单例，多例，仅存在于一次会话...

## @Lazy 懒加载

作用于类或方法，使指定的 bean 在容器启动的时候不被创建，需要时才被创建

## @Conditional 按照条件注册

作用于类或方法，如果满足指定条件，则类能生效 / 注册组件。不满足不干。且该注解不自带 @Component 和 @Bean


## Bean 的初始化方式

> Bean的整个生命周期也可参照这篇[博客](https://www.cnblogs.com/javazhiyin/p/10905294.html)

Bean 的生命周期的过程：

bean 创建（调用构造方法）--- 初始化（调用指定的初始化方法）---bean 销毁（调用指定的销毁方法）

我们可以自定义 bean 的初始化过程（init方法）和销毁方法（destory方法）



### 指定初始化方法和销毁方法

```java
 // 指定了初始化方法是本类中名为 "init" 的方法。同理，销毁方法也是
 @Bean(initMethod="init", destoryMethod="destory")  
 public Car getCar(){  
     return new Car();  
 }  
```

 

### 指定初始化接口和销毁接口实现类

让实体类去实现接口
- 初始化接口 - InitializingBean
- 销毁接口 - DisposableBean


### 用注解指定初始化和销毁方法

- @PostConstruct - 指定初始化方法
- @PreDestroy - 指定销毁方法

这两个注解用在实体类的方法上


# 组件赋值

为容器创建 bean 时，bean 的成员变量应该为 Java 变量默认值或 BeanDefinition 指定的值

BeanDefinition 能为 bean 的成员变量指定以下 3 种类型的值
- 静态字面值。硬编码指定成员变量的字面值
- 配置文件的值。读取配置文件赋值
- 引用容器中其他 bean，为这个 bean 的成员变量赋值

## 静态赋值和配置文件赋值


### @Value 给组件的变量赋初值

`@Value` 有两种用法。给 bean 的成员变量指定静态字面值，或指定读取配置文件中的值

`@Value` 的值可以是字面量，`${}` 表达式，EL 表达式（`#{}`）


### 用字面量设置字面值

```java
public class Car {    
    @Value("blue") // 属性可以没有 set 方法或对应的构造器也能被 @Value 绑定成功
    private String color;
}
```


> [!NOTE] 不能用字符串表示运算表达式
> `@Value` 的属性值也能是用双引号包起来的数字（但不能是运算表达式，比如 "1+1"）

### 用 ${} 表达式进行赋值

${} 表达式允许读取配置文件值，并用 `@Value` 绑定给变量

```java
@Configuration
@PropertySource("classpath:application.properties") // 读取配置文件
public class Config {
}
```

```java
@Component("otherCar")  
public class Car {  
    @Value("blue")  
    private String color;  
  
    @Value("${car.value:defaultValue}") // 读取配置文件的值，冒号后可加默认值
    private int value;
}
```


> [!warning] ${} 表达式的值不能是数字
> 
> ${} 的值可以是字符串，但不能是数字或算数运算式


### 用 EL 表达式进行赋值

EL 表达式允许用 `@Value` 设置字面量，运算表达式，逻辑表达式，三元运算符，系统运行时参数，容器中对象的属性值等

EL 表达式非常强大

使用实例可参考[这里](http://t.zoukankan.com/solverpeng-p-5682551.html)


## 引用容器中的其他 bean 赋值

- 根据类型获取容器中的 bean
- 根据 beanName 获取容器中的 bean
- 根据默认 bean 获取容器中的 bean


### @Autowired 自动注入

示例代码如下

```java
public class Controller {
    @Autowired
	Service service;
}
```

1.  默认优先按照类型去容器中查找对应组件。这里会优先在容器中找 Service 类的对象
2.  如果有多个相同类型组件，再将对象的名称作为 beanName 去容器中找

可以用 List 装配容器中所有符合指定类型的 bean，实例代码如下

```java
public class Controller {
    @Autowired
	List<Service> services;
}
```

上述自动装配代码，会获取容器中的所有 Service 类型的 bean 并放到这个 List 里

### @Qualifier 根据 beanName 注入 bean

```java
public class Controller{  
	@Qualifier("myservice") // 显式声明注入的 bean 的 beanName，方式存在多个同类型的 bean
	@Autowired  
	Service service;  
}
```


### @Primary 指定默认装配组件

作用于类或方法，用于声明此类型的 bean 中，这个 bean 是优先级最高的 bean

作用在一个实体类后，当装配一个组件时，如果没用 @Qualifier 指定组件，则默认优先注入有 @Promary 修饰的 bean



### @Autowired 在其他地方的使用

前面说，@Autowired注解能用在类，方法，变量，构造器等多个地方

接下来说说，这个注解用在不是类的地方时，该怎么用

```java
//用在方法上  
public class Boss{  
	private Car car;
	
	// 容器创建对象的时候，会自动调用该方法，以完成赋值  
	@Autowired  
	public setCar(Car car){  
		this.car = car;  
	}
}  
```

```java
// 用在构造器上或参数上  
// 容器创建对象时，默认调用无参构造器，再进行初始化赋值  
public class Boss{  
	private Car car;  
       
	// 如果类只有一个构造器，构造器上的Autowired也可以省略  
	@Autowired  
	Boss(/*@Autowired 放这也行*/Car car){  
		this.car = car;
	}
}
```

常用方式是在 @Bean 下的方法参数上加上对象

```java
//方法参数中的Car即使没有Autowired注解，也默认是从容器中找的对象  
@Bean  
public Boss getBossWithCar(Car car){  
    Boss boss = new Boss();  
    boss.set(car);  
    return boss;  
}
```

### @DependsOn 在实例化 bean 前先实例化所依赖的 bean

有一种有些特殊的场景，比如我们需要在实例化 Bean A 之前，先实例化 Bean B，这个时候就可以使用 `@DependsOn` 注解


```java
@DependsOn(value = "testService2")
@Service
public class TestService1 {
	@Autowired
	private TestService2 testService2;
}
```

在 `getBean` 时，会调用 BeanDefinition 的 `getDependsOn()` 方法获取所依赖的 bean 并进行处理

