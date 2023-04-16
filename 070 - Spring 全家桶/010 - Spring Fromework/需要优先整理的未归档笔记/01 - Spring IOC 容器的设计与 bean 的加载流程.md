# Spring IOC 容器的两个顶级接口

`IoC` 容器有两个系列
- `BeanFactory` **简单容器系列**，此接口提供容器的最基本的功能。又称 "Bean工厂"

```java
boolean containsBean(); // 检查容器里是否有指定的 bean

Object getBean(); // 创建或获取 bean

String[] getAliases(); // 获取 bean 的别名

ObjectProvider getBeanProvider(); // 获取创建 bean 实例的对象提供者

boolean isPrototype(); // 检查 bean 是否是多例的

boolean isSingleton(); // 检查 bean 是否是单例的

boolean isTypeMatch(); // 检查 bean 是否是期望的类型
```

- `ApplicationContext` **高级容器系列**，是容器的高级形态。又称"应用上下文"或"上下文"。它在简单容器的基础上又增加了许多新功能

![../../91 - 静态资源/Pasted image 20220613200016.png|675](../../91%20-%20%E9%9D%99%E6%80%81%E8%B5%84%E6%BA%90/Pasted%20image%2020220613200016.png%7C675.md)


系统运行时常用的实现类是 `DefaultListableBeanFactory`，它用多个 `ConcurrentHashMap` 保存 bean 对象


# bean 的创建过程概述

首先，由编码者使用某种形式编写 `Bean` 的配置信息。例如在.xml文件中写 `<bean>` 标签配置 `Bean`，或用配置类定义 `Bean`

然后，启动容器进行初始化。容器初始化大致分三步

1.  `BeanDefinition` 的 `Resource` **定位**（定位 `Bean` 配置信息所在位置）
    
2.  **载入** `BeanDefinition`（创建 `BeanDefinition` 对象）
    
3.  向容器**注册** `BeanDefinition`对象
    

在第一次调用容器的 `getBean()` 方法，获取`Bean`对象时，如果对象还未被实例化，就获取对应的 `BeanDefinition` 对象，并使用它作为模板创建实例对象

# IOC 容器的初始化


## 执行 BeanFactory 的后置处理器

在创建 bean 前会先进行容器的初始化，然后执行 BeanFactory 的后置处理器

```java
invokeBeanFactoryPostProcessors();
```

执行的后置处理器的目的是完成容器的初始化

1. 首先执行内置的 BeanDefinitionRegistryPostProcessor
2. 再执行容器里的 BeanDefinitionRegistryPostProcessor，执行顺序按其实现的优先级接口进行
3. 再执行内置的 BeanFactoryPostProcessor 和容器里的 BeanFactoryPostProcessor，执行顺序按其实现的优先级接口进行


其中就有扫描 xml 或 配置类并生成 BeanDefinition 的工作在这里执行

比如：扫描 `@Configuration` 注解和 `@Bean` 的工作由 BeanDefinitionRegistryPostProcessor 的唯一实现类执行

![../../91 - 静态资源/Pasted image 20220613231239.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220613231239.png)

## 注册 BeanPostProcessor

接着获取到容器里所有的 BeanPostProcessor 并**根据优先级分类注册**到容器专门管理 BeanPostProcessor 的 map 里


# Bean 的创建过程详述

bean 的创建和获取都由 `getBean()` 完成，其执行流程非常复杂

假设：现在要创建 A，A 又依赖 B

1. 如果创建过 A，就能从缓存中直接获取到 A（或进入依赖循环）
2. 如果没创建过 A，就进入 `creatBean()` 创建 A


> [!quote] 参考
> - [[../../91 - 静态资源/springioc加载流程图 1.pdf|SpringIOC 创建 bean 的流程]]



![../../91 - 静态资源/springioc加载流程图.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/springioc%E5%8A%A0%E8%BD%BD%E6%B5%81%E7%A8%8B%E5%9B%BE.png)


# 循环依赖和三级缓存

![Pasted image 20220615133910](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220615133910.png)


参考[这里](https://www.zhihu.com/question/438247718/answer/1730527725)

![../../91 - 静态资源/循环依赖课上图.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E8%AF%BE%E4%B8%8A%E5%9B%BE.png)