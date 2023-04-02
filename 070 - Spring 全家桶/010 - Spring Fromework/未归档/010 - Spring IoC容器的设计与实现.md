#还没有复习 

## 控制反转

`Spring IoC`的`IoC`全称是"Inversion of Control"，译：控制反转

"控制反转"又称"依赖反转"

`DI`全称是"Dependency Injection"，译：依赖注入



使用控制反转，依赖注入后，当创建对象时，由一个调度系统将其所依赖的对象的引用传递给它（这些对象也都来自于调度系统所管理的对象），即依赖被注入到对象中。所以，控制反转是关于一个对象如何获取它所依赖的对象的引用



上述的调度系统是应用平台，具体来说是`IoC`容器。通过使用`IoC`容器，对象依赖关系的管理被反转了，转到`IoC`容器中来了，对象之间的相互依赖关系由`IoC`容器进行管理，并由`IoC`容器完成对象的注入



## IoC容器的核心数据结构

`BeanDefinition`是`IoC`容器的**核心数据结构**。又称"Bean定义信息"。是创建`Bean`实例时所使用的**模板**

`Bean`是容器管理的对象的统称



容器在创建`Bean`实例之前会先创建`BeanDefinition`对象，它保存了`Bean`的定义信息，例如`Bean`的名字，类型，是单实例还是多实例，是不是懒加载，所依赖的其他对象有哪些等



## IoC 容器的设计与实现

`IoC`容器有两个系列

一个是实现`BeanFactory`接口的**简单容器系列**，这个系列的容器能实现容器的最基本的功能。又称"Bean工厂"

一个是实现`ApplicationContext`接口的**高级容器系列**，它是容器的高级形态。又称"应用上下文"或"上下文"。上下文在简单容器的基础上又增加了许多新功能





### Spring IoC容器的设计

下面这张图描述的`IoC`容器的接口设计图，`IoC`容器的实现类就是以容器的接口设计为基础实现的

![[../../../020 - 附件文件夹/Pasted image 20230402105933.png]]

**设计主线**

- 以`BeanFactory`为核心。从`BeanFactory`到`HierarchicalBeanFactory`再到`ConfigurableBeanFactory`。这是一条主要的`BeanFactory`类型容器的设计路线
- 以`ApplicationContext`为核心。从`BeanFactory`到`ListableBeanFactory`到`ApplicationContext`，再到`ConfigurableApplicationContext`或`WebApplicationContext`。我们常用的上下文基本都是`ConfigurableApplicationContext`或`WebApplicationContext`的实现类



我们常用的`BeanFactory`的实现类是`DefaultListableBeanFactory`。同时，以这个实现类为基础，又有许多应用于不同场景下的子类bean工厂。不过我们通常直接使用`DefaultListableBeanFactory`



我们常用的`ApplicationContext`的实现类是`AbstractApplicationContext`的子类。这个抽象类包含了**容器的初始化方法`refresh()`**。

`AbstractApplicationContext`实现的是`ConfigurableApplicationContext`接口。



### BeanFactory 的设计原理

这些接口方法定义出了一个基本的`IoC`容器

![[../../../020 - 附件文件夹/Pasted image 20230402105950.png|500]]

- `boolean containsBean(String name);`容器中是否有名为`name`的`Bean`
- `String[] getAliases(String name);`查询`Bean`的所有别名
- `getBean()`有多个重载方法，是从容器中获取`Bean`实例的方法（此方法包含如果获取不到就创建一个`Bean`实例的流程）
- `boolean isSingleton(String name);`返回`Bean`是否是单实例
- `boolean isPrototype(String name)`返回`Bean`是否是多实例（`Bean`是单实例还是多实例，这些信息保存在对应的`BeanDefinition`对象中）
- ...

这些方法提供了容器最基本的功能



### ApplicationContext 的设计原理

`ApplicationContext`除了实现了`BeanFactory`外还实现了其他接口，为容器添加了其他功能

![[../../../020 - 附件文件夹/Pasted image 20230402110006.png]]

- `MessageSource`信息源。支持国际化功能的实现
- `ResourceLoader`访问资源。用于获取 `Bean`的定义信息所在的资源，为创建 `BeanDefinition`对象做准备
- `ApplicationEventPublisher`发布事件。可以在上下文中，在`Bean`的生命周期中发布事件
- ...



## 容器初始化流程

首先，由编码者使用某种形式编写`Bean`的配置信息，例如在.xml文件中写`<bean>`标签配置`Bean`

然后，启动容器进行初始化。容器初始化大致分三步

1. `BeanDefinition`的`Resource`**定位**（定位`Bean`配置信息所在位置）
2. **载入**`BeanDefinition`（创建`BeanDefinition`对象）
3. 向容器**注册**`BeanDefinition`对象（在容器内部将对象put到一个容器持有的map对象中）



在第一次调用容器的`getBean()`方法（`BeanFactory`中定义的方法）获取`Bean`对象时，如果对象还未被实例化，就获取对应的`BeanDefinition`对象，并使用它作为模板创建实例对象



再容器完成初始化后，调用`refresh()`方法刷新容器。方法执行完后容器便可以使用

ps：容器初始化的流程可能在`refresh()`方法外执行，也可能在`refresh()`方法内执行