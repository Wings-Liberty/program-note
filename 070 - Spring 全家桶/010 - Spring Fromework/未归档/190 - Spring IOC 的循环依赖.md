#还没有复习 

问题：Bean 对象 A 中持有成员变量 B b。Bean对象B中持有成员变量A a。在创建A时需要B对象，于是去创建B对象。创建B对象时又需要A对象，但是A对象还没创建完。这怎么解决

解决：三级缓存

```java
/** Cache of singleton objects: bean name to bean instance. 一级缓存*/
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Cache of singleton factories: bean name to ObjectFactory. 三级缓存*/
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** Cache of early singleton objects: bean name to bean instance. 二级缓存*/
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
```



bean 的创建包含，实例化和初始化。初始化可以调用 set 方法或构造器实现。在出现循环依赖时，set 方法能解决循环依赖，而构造器不能



创建 bean 时，bean 实例化完成后就提前暴露对象（将刚完成实例化的bean加到某级缓存中）



每次获取某个单实例bean前会先尝试从容器中查找，如果命中，直接返回查找结果。如果容器中不存在再执行创建bean的流程。以下就是尝试从容器中查找bean的方法

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 先尝试从一级缓存中查找bean
    Object singletonObject = this.singletonObjects.get(beanName);
    // 如果没查到bean且bean正在被创建，就进入方法
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            // 尝试从二级缓存中查找bean
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                // 尝试从三级缓存中查找bean
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    // 尝试调用三级缓存中k的getObject方法获取三级缓存中的bean
                    singletonObject = singletonFactory.getObject();
                    // 将bean（半成品，只执行了实例化，还没有完成初始化）从三级缓存中去除并加到二级缓存中
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```





创建bean时，在调用populateBean前会先调用

```java
// 如果bean不在一级缓存中，就将bean加到三级缓存中 
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));

protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```



B/b 创建完后，要将 b 加到一级缓存中。`addSingleton`调用时的状态是a是半成品，b是成品（因为a还未将b赋给自己的成员变量，但b已经将a赋给自己的成员变量了）

```java
protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        // 将bean加到一级缓存中
        this.singletonObjects.put(beanName, singletonObject);
        // 将bean从二、三级缓存中移除
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}
```

完成b的创建后，将b赋值给a的成员变量，调用a的`addSingleton`此时a和b都是成品，将a加到一级缓存并从二、三级缓存中去掉。

此时a和b都是成品且都只存在于一级缓存中



新的问题：

- 为什么只有一级缓存不能解决循环依赖

  因为认为规定，一级缓存中的bean是能直接使用的完整的bean，为了将成品与半成品分离开才有了多级缓存

- 为什么要有三级缓存，而不是二级缓存

  这需要思考一个核心问题。三级缓存比二级缓存多了什么东西？答：三级缓存的kv的k是包装了bean的ObjectFactory，而不是bean本身

  从三级缓存中获取bean时会调用ObjectFactory的getObject方法。显然这个方法能增强bean

  如果容器中所有的bean都不需要ObjectFactory的getObject方法对自己进行修改，那么可以只用二级缓存，并将三级缓存去掉

  ```java
  // 这是将bean加到三级缓存的操作
  addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
  
  // 如果有必要，对这个bean执行所有的SmartInstantiationAwareBeanPostProcessor.getEarlyBeanReference
  // AbstractAutoProxyCreator，AbstractAdvisorAutoProxyCreator均是其子类
  protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
      Object exposedObject = bean;
      if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
          for (BeanPostProcessor bp : getBeanPostProcessors()) {
              if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                  SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                  exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
              }
          }
      }
      return exposedObject;
  }
  ```

  使用三级缓存可以创建aop的代理

  如果使用2级缓存就不能使用aop了吗？[参考](https://www.cnblogs.com/semi-sub/p/13548479.html)

- 如果没有循环依赖，还会调用ObjectFactory的方法吗



## 启动类的父类构造方法

**AbstractApplicationnContext**

```java
public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        refresh();
    }
}
```

