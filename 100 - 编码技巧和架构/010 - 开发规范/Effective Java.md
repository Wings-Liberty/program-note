# 引言

本书目的在于能更有效地使用 Java 编程语言及其基本类库`java.lang`、`java.util`、`java.io`及其子包`java.util.current`和`java.util.function`

本书共 12 章，每章都涉及软件设计的一个主要方面



此外还需掌握以下几点

- 组件的概念：组件指可任何可复用的软件元素。小到一个方法，大到一个框架都可以叫组件
- 本书目的不在于构建能高效执行的代码，而增注重代码的清晰，简洁，可读性，可复用性，扩展性，可维护性等
- 每条规则尽可能以：什么场合下使用什么方法编码完成什么任务更合适，这样做有哪些优点和缺点，以及标准 0 分的做法（错误做法）及其缺点



# 第 1 章：创建和销毁对象

> 回答了
>
> - 何时以及如何创建对象
> - 何时以及如何避免创建对象
> - 如何确保它们能适时地销毁
> - 如何管理对象销毁前必须执行的各种清理动作



## 第 1 条：用静态工厂方法代替构造器

> 此静态工厂方法和设计模式中的工厂方法模式不同



- `from(instant)`。类型转换方法，经静态工厂方法，把 A 类型的实例，转为 B 类型的实例并返回
- `of(obj1, obj2 ...)`。聚合方法，把多个对象合并到一个对象中，并返回
- `valueOf(obj)`。是 from 和 of 的替代方法，基本类型的包装类提供此法作为避免重复创建对象的方式
- `getInstance(options)`。把参数作为创建对象用的描述信息，返回一个定制的对象。可能返回一个单例对象
- `newInstance(options)`、`create`、`createInstance`。把参数作为创建对象用的描述信息，返回一个定制的对象，一定返回一个新创建的对象
- `type`、`getType`、`newType`。工厂方法在不同的类中时使用



> 服务提供者框架是什么：多个服务提供者实现一个服务
>
> 服务提供者框架包含 4 大组件：服务接口，提供者注册 API，服务访问 API，服务提供者接口（可选）
>
> 以 JDBC 为例
>
> - 服务接口：系统定义的接口，需要服务提供者实现，比如：Connection 接口需要 MySQL 的驱动实现
> - 提供者注册 API：比如：DriverManager.registerDriver 用于注册 MySQL 驱动
> - 服务访问 API：比如：DriverManager.getConnection
> - 服务提供者接口：
>
> 关于服务提供者框架（service provider framework），可参考其变体 [Java 的SPI 机制](https://zhuanlan.zhihu.com/p/28909673)



## 第2条：构建者模式创建对象（Builder）

```java
ClassABuilder builder = ClassABuilder.setXxx().setXxx().build();
```





```java
public class NutritionFacts {
    private final int servingSize;
    private final int calories;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        // Optional parameters - initialized to default values
        private int calories      = 0;

        public Builder servingSize(int servingSize) {
            this.servingSize = servingSize;
            return this;
        }
        public Builder calories(int val) {
            this.calories = val;
            return this;
        }
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        this.servingSize  = builder.servingSize;
        this.calories     = builder.calories;
    }

    public static void main(String[] args) {
        Builder b = new NutritionFacts.Builder(240)
                	.calories(100)
            		.sodium(35);
        NutritionFacts cocaCola1 = b.build();
        NutritionFacts cocaCola2 = b.build();
        NutritionFacts cocaCola3 = b.build();
    }
}
```



## 第 3 条：用私有构造器或者枚举类型强化 Singleton 属性

> 介绍如何创建单例对象

- 懒汉模式
- 饿汉模式
- 枚举实现

还需要注意防止通过序列化机制破坏掉单例模式



## 第 4 条：通过私有构造器强化不可实例化的能力

设置私有构造器 + final 修饰 class，**防止仅提供静态变量和静态方法的工具类被实例化或被子类继承后实例化**



PS：在 JDK 中 Math，Arrays，Collections 类都是这样的类



## 第 5 条：优先考虑依赖注入来引用资源



对于必须需要依靠底层资源才能工作的类来说，不建议用单例模式，而应该采用依赖注入方式为目标类设置底层资源

依赖注入：目标类对象持有的底层资源的引用的声明类型是**比较抽象的**（接口或抽象类或顶级父类），只有在创建目标类对象时才会**动态**传入底层资源对象。同时目标类持有的底层资源（依赖注入的资源对象）通常是**不可变的**



> SpringIOC 在 SpringBoot 搭建的 web 项目中都是这样用的
>
> 以 Spring Cloud 搭建的微服务为例，客户端持有的是服务的抽象接口，而具体实现只有等到程序运行时才会确定
>
> 或以 Dubbo 为例，Dubbo 中有大量的同级组件（比如序列化策略），到底用哪些组件只有运行时才知道



## 第 6 条：避免创建不必要的对象

避免创建不必要的对象的目的是在于剩内存



- 尽量用基本数据类型，不用包装类

- 当类提供静态工厂时，应尽量使用静态工厂

- 不要轻易使用对象池

- 复用不可变的对象

  

> 比如避免调用 String.match，因为它每次都会创建一个 Pattern 对象，创建这个对象的开销很大。所以，应该把用依赖注入方式来复用他



## 第 7 条：消除过期的对象引用

> JVM 会回收多数明显不再被需要的垃圾对象，但 JVM 很难发现人为创建后，但忘了使用的垃圾对象



- 内存泄漏可以尝试用 WeakHashMap 或 LinkedHashMap 的 removeEldestEntry 解决（这些工具都是`java.lang.ref`包里的）
- 对于提供注册事件的 API，应该同时提供取消注册功能。防止被注册的事件数量无限膨胀
- 主动把不再被需要的对象的引用设为 null



## 第 8 条：避免使用终结方法和清除方法

Java 自带的终结方法（finalizer）和清除方法（cleaner）就是垃圾

因为不能保证它们能被及时执行，所以不建议用

如果没有被及时执行，对象回收就会被延迟，极大增加了内存泄漏的可能



## 第 9 条：try-with-sources 优先于 try-finally

```java
try(InputStream in = FileInputStream(src)){
    ...
}
```

在使用多层 try-finally 时，如果处理不当就会出现第二个异常覆盖掉第一个异常出现过的记录，导致查错时根本注意不到第一个异常

> 比如第一个异常没被 catch 捕捉到，且 final 块中抛出了第二个异常，待退出 try-final 块时，只会抛出第二个异常

try-with-sources 能实现自动调用 try 的参数列表中实现了 AutoCloseable 接口的实例的 close 方法，同时还能正常处理异常



· 二、对于所有对象都通用的方法

o 第10条：覆盖 equals 方法时请遵守通用约定

o 第11条：覆盖 equals 方法时总要覆盖 hashCode

o 第12条：始终要覆盖 toString

o 第13条：谨慎地覆盖 clone

o 第14条：考虑实现 Comparable 接口



# 第 4 章：类和接口



## 第 15 条：使类和成员的可访问性最小化

只有包中的 API 才需要被设为 public 供外部访问，其他类和成员的访问，控制，修改细节应该放在黑盒里

调用者无需知道 API 内部执行细节，工具开发者也能仅需要对少量的 API 进行兼容维护，却能对 API 内部的实现细节进行任意修改

工具开发者对被导出的 API 不能轻易修改，否则会造成调用者在升级依赖时出现严重的不兼容现象



## 第 16 条：要在共有类而非共有域中使用访问方法

尽可能把公有类的成员变量设为 private。如果实在有必要才为成员变量设置 get，set 方法

但这也意味着外界能直接获取到类 的内部结构。所以如果公有类的域是可变的，可以尝试用 get 方法返回一个不可修改的视图或数据副本





## 第 17 条：使可变性最小化
