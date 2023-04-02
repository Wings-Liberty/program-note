#还没有复习 

## Bean的生命周期

bean 的生命周期比较复杂。简化其生命周期后大致分为

- 实例化。创建实例对象
- 初始化。依赖注入，对 bean 执行初始化方法，执行后置处理器
- 销毁。销毁 bean


## getBean的设计与实现

- `getBean()`方法是`BeanFactory`对外暴露的方法。在需要使用bean时一般都会调用此方法获取bean实例
- 在`AbstractBeanFactory`中`getBean()`的所有重载方法都会调用`doGetBean()`方法
- `doGetBean()`会先尝试从容器中查找bean，如果找不到就调用`createBean()`和`doCreateBean()`创建bean并将其添加到容器中


```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
	
    // 如果name是&开头的，方法的返回值会去掉name的&前缀。
    // FactoryBean的实现类的name会被加上'&'前缀
    final String beanName = transformedBeanName(name);
    Object bean;

    // 尝试从容器中获取单实例bean，如果获取不到再尝试创建bean
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        // 如果name以'&'开头，方法返回值为sharedInstance
        // 如果name不以'&'开头，且sharedInstance时FactoryBean，方法返回值为FactoryBean创建的对象
        // FactoryBean创建的对象会被缓存到 专门存放FactoryBean生成的对象的map中（FactoryBeanRegistrySupport-的成员变量 factoryBeanObjectCache）
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }

    else {
        ...
        // 尝试获取父容器，如果有父容器，尝试调用父容器的getBean方法获取bean。在Spring MVC时再说这个父子容器/双亲容器
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            String nameToLookup = originalBeanName(name);
            if (parentBeanFactory instanceof AbstractBeanFactory) {
                return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
                    nameToLookup, requiredType, args, typeCheckOnly);
            }
            else if (args != null) {
                return (T) parentBeanFactory.getBean(nameToLookup, args);
            }
            else {
                return parentBeanFactory.getBean(nameToLookup, requiredType);
            }
        }
		...
        try {
            // 根据beanName，获取BeanDefinition。以此为模板创建bean实例
            final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            checkMergedBeanDefinition(mbd, beanName, args);

            // 获取此bean所依赖的其他bean
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
                for (String dep : dependsOn) {
                    if (isDependent(beanName, dep)) {
                        throw ...
                    }
                    registerDependentBean(dep, beanName);
                    try {
                        getBean(dep);
                    }
                    catch (NoSuchBeanDefinitionException ex) {
                        throw ...
                    }
                }
            }

            // 创建单实例bean的入口
            if (mbd.isSingleton()) {
                // 在方法中会调用lambda表达式实现的方法创建bean。下一段代码分析将围绕此方法
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                    catch (BeansException ex) {
                        ...
                    }
                });
				// 如果bean是FactoryBean那么方法返回值为FactoryBean创建的对象，如果不是，返回值为sharedInstance
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }
			// 创建多实例bean的入口
            else if (mbd.isPrototype()) {
                ...
            }
			// 创建其他类型实例的入口
            else {
               ...
            }
        }
        catch (BeansException ex) {
            ...
        }
    }
	...
    return (T) bean;
}
```


`getSingleton()`方法做了哪些事？

- 调用`singletonFactory.getObject()`创建bean对象
- 调用`addSingleton()`将bean添加到容器中

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    synchronized (this.singletonObjects) {
        // 再次尝试从容器中获取bean，如果能获取到就直接return
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet<>();
            }
            try {
                // 创建bean实例，此对象的方法实现由doGetBean中调用处使用lambda表达式实现。下面分析此方法的实现
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            catch (Exception ex) {
                ...
            }
            finally {
               ...
            }
            if (newSingleton) {
                // 将创建好的bean添加到缓存中
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}
```



## createBean的设计与实现

- `createBean()` 在执行一些准备工作后会调用`doCreateBean()`（创建bean的方法）
- `doCreateBean()`创建bean的流程包含实例化bean，初始化bean



```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
    RootBeanDefinition mbdToUse = mbd;
	...
    try {
        // 给它一个机会返回一个代理对象（一般返回值都是null），所以通常船舰bean实例的方法是doCreateBean
        // 此处用于执行容器中InstantiationAwareBeanPostProcessor类型的后置处理器的applyBeanPostProcessorsBefore/AfterInstantiation方法
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) {
            return bean;
        }
    }
    catch (Throwable ex) {
        throw ...
    }

    try {
        // 创建bean实例。下面分析这段代码
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        return beanInstance;
    }
    catch (Throwable ex) {
        throw ...
    }
}
```



`doCreateBean()`包含实例化bean，初始化bean

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
	// 创建的bean实例会暂时被包装在此对象中
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        // 创建bean实例，并将bean包装在BeanWrapper中
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    // 从包装类中取出bean
    final Object bean = instanceWrapper.getWrappedInstance();
    Class<?> beanType = instanceWrapper.getWrappedClass();
    if (beanType != NullBean.class) {
        mbd.resolvedTargetType = beanType;
    }

    synchronized (mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            try {
                // 遍历容器中的MergedBeanDefinitionPostProcessor并调用postProcessMergedBeanDefinition
                applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            }
            catch (Throwable ex) {
               ...
            }
            mbd.postProcessed = true;
        }
    }

    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                      isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        ...
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }

    Object exposedObject = bean;
    try {
        // 依赖注入的入口
        // 除了依赖注入外，还会遍历容器中的InstantiationAwareBeanPostProcessor调用postProcessPropertyValues
        populateBean(beanName, mbd, instanceWrapper);
        // 初始化bean
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    catch (Throwable ex) {
        ...
    }

	...

    return exposedObject;
}
```





```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    // 如果bean实现了以下任意接口，BeanFactoryAware，BeanClassLoaderAware，BeanNameAware。那么执行接口方法
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
            invokeAwareMethods(beanName, bean);
            return null;
        }, getAccessControlContext());
    }
    else {
        invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    // 遍历容器的BeanPostProcessor执行postProcessBeforeInitialization方法
    // 其中包含执行@PostConstruct指定的初始化方法
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    try {
        // 1. 如果bean实现了InitializingBean接口，调用其afterPropertiesSet方法
        // 2. 如果在@Bean注解上指定了初试化方法，调用初始化方法
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
        throw ...
    }
    // 遍历容器的BeanPostProcessor执行postProcessAfterInitialization方法
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}
```

