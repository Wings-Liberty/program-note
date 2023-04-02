#还没有复习 

> 运行环境：jdk8，springboot-2.2.2

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
</parent>
```



# 创建Spring IOC容器对象

容器在哪里创建以及容器的运行时类型

```java
// 启动类
@SpringBootApplication
public class BootApplication {
    public static void main(String[] args) {
            SpringApplication.run(BootApplication.class);
    }
}
```

```java
// 执行run方法时，创建Spring IOC容器。这里只是创建了容器对象，容器还未执行refresh方法
context = createApplicationContext();
```



`SpringBoot`默认使用的容器类型是`AnnotationConfigServletWebServerApplicationContext`



# Spring IOC容器结构分析

接下类分析下`AnnotationConfigServletWebServerApplicationContext`

下图的继承关系比较“吓人”，但是重点只有几个

![[../../../020 - 附件文件夹/Pasted image 20230402122224.png]]

- `GenericApplicationContext`

  这个类是`AnnotationConfigServletWebServerApplicationContext`的父类，`GenericApplicationContext`持有成员变量

  `private final DefaultListableBeanFactory beanFactory;`

  此成员变量的初始化是在调用`GenericApplicationContext`的无参构造器时执行的

  ```java
  public GenericApplicationContext() {
      this.beanFactory = new DefaultListableBeanFactory();
  }
  ```

- `DefaultListableBeanFactory`

这个类是创建和管理bean实例的bean工厂

![[../../../020 - 附件文件夹/Pasted image 20230402122243.png]]
  
持有成员变量

`private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);`

保存最初的`BeanDefinition`对象

  

其父类`AbstractBeanFactory`持有成员变量

`private final Map<String, RootBeanDefinition> mergedBeanDefinitions = new ConcurrentHashMap<>(256);`

保存最终的`BeanDefinition`对象，之后会调用其中的`BeanDefinition`对象作为模板创建bean实例



​其父类`DefaultSingletonBeanRegistry`持有成员变量

​`private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);`

​保存创建完成的单实例bean

​它还持有其他成员变量保存其他种类的bean实例



有了上述的继承关系`AnnotationConfigServletWebServerApplicationContext`的对象将非常强大，这个对象将可以创建`BeanDefinition`和bean实例，管理所有的`BeanDefinition`和bean实例



# 补充

`DefaultListableBeanFactory`的 `beanDefinitionMap`和`AbstractBeanFactory`的`mergedBeanDefinitions`都保存了`BeanDefinition`对象，那么创建bean实例时用谁做模板？

在`AbStractApplicationContext`执行`finishBeanFactoryInitialization(beanFactory)`方法创建剩余的所有非懒加载的单实例bean时

调用`AbstractBeanFactory`的`getMergedBeanDefinition`方法获取`BeanDefinition`对象

会先调用抽象方法`getBeanDefinition`（由`DefaultListableBeanFactory`实现）

此方法会从`DefaultListableBeanFactory`的`beanDefinitionMap`中找`BeanDefinition`

找不到，抛异常；找到，对其加工，将其put到`mergedBeanDefinitions`

再从`mergedBeanDefinitions`中获取并返回`RootBeanDefinition`

将其作为模板创建bean实例