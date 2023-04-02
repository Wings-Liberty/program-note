#还没有复习 

## @EnableAspectJAutoProxy注解的介绍



使用`@EnableAspectJAutoProxy`注解开启AOP后，此注解会像容器中添加一个`AnnotationAwareAspectJAutoProxyCreator`类型的bean

此bean的继承体系如图

![[../../../020 - 附件文件夹/Pasted image 20230402110458.png]]

`AnnotationAwareAspectJAutoProxyCreator`是一个`BeanPostProcessor`

对`BeanPostProcessor`接口方法的实现在`AbstractAutoProxyCreator`中

将此bean添加到容器中后，在创建其他bean时调用后置处理器时如果有需要，`AnnotationAwareAspectJAutoProxyCreator`会调用`postProcessAfterInitialization`方法为bean创建并返回代理对象



## 后置处理器创建bean的代理对象


```java
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) throws BeansException {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (!this.earlyProxyReferences.contains(cacheKey)) {
            // 如果有需要就为bean创建一个代理对象并返回
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```


```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
        return bean;
    }
    if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
        return bean;
    }
    if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
        return bean;
    }

    // 获取此bean需要的通知器
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    // 如果specificInterceptors!=null就为bean创建代理对象
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        // 为bean创建代理对象
        Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }

    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}
```



`createProxy()`方法将会调用`Proxyfactory`类型的对象创建代理对象



## AopProxy代理对象

Spring AOP创建的代理对象是`AopProxy`类型的

![[../../../020 - 附件文件夹/Pasted image 20230402110511.png|500]]

`AopProxy`的实现类分两种

`Jdk`实现的动态代理和`Cglib`实现的动态代理


## Proxyfactory的设计

![[../../../020 - 附件文件夹/Pasted image 20230402110526.png|275]]

- `ProcyConfig` 提供默认配置
- `AdvisedSupport` 用于保存，管理通知器
- `ProxyCreatorSupport`拥有创建代理对象的功能，且持有一个`ProxyFactory`类型的成员变量


## ProxyFactory创建代理对象

`createProxy()`方法调用者是`AnnotationAwareAspectJAutoProxyCreator`

方法实现在其父类`AbstractAutoProxyCreator`中

```java
protected Object createProxy(Class<?> beanClass, @Nullable String beanName, @Nullable Object[] specificInterceptors, TargetSource targetSource) {

    if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
        AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
    }

    // 由代理工厂创建代理对象
    ProxyFactory proxyFactory = new ProxyFactory();
    // 将AnnotationAwareAspectJAutoProxyCreator作为ProxyConfig赋给ProxyFactory做配置
    proxyFactory.copyFrom(this);

    if (!proxyFactory.isProxyTargetClass()) {
        if (shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        }
        else {
            evaluateProxyInterfaces(beanClass, proxyFactory);
        }
    }

    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    // 将所有通知器添加到proxyFactory中（添加到父类AdvisedSupport的this.advisors变量中）
    proxyFactory.addAdvisors(advisors);
    proxyFactory.setTargetSource(targetSource);
    customizeProxyFactory(proxyFactory);

    proxyFactory.setFrozen(this.freezeProxy);
    if (advisorsPreFiltered()) {
        proxyFactory.setPreFiltered(true);
    }
	
    // 用ProxyFactory创建代理对象并返回
    return proxyFactory.getProxy(getProxyClassLoader());
}
```


```java
public Object getProxy(@Nullable ClassLoader classLoader) {
    // createAopProxy()创建代理对象并将AdvisedSupport传递给创建的AopProxy对象
    // getProxy()将通知器织入到目标类中
    return createAopProxy().getProxy(classLoader);
}

protected final synchronized AopProxy createAopProxy() {
    if (!this.active) {
        activate();
    }
    // this是ProxyFactory对象，同时也是AdvisedSupport，对象中保存了通知器
    return getAopProxyFactory().createAopProxy(this);
}

// 此方法创建代理对象
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw ...
        }
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        return new ObjenesisCglibAopProxy(config);
    }
    else {
        return new JdkDynamicAopProxy(config);
    }
}
```



