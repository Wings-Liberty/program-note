#还没有复习 

## Web环境下的MVC

`Spring IoC`是一个独立的模块，并不能直接在Web容器中发挥作用。

所以要在Web容器中使用IoC容器，需要为`Spring IoC`设计一个启动过程，并把IoC容器导入进来

Web容器的启动过程一方面处理Web容器的启动，另一方面将IoC容器载入到web环境中并将其初始化

而`Spring MVC`是建立在IoC容器的基础上的，在导入IoC容器后才能建立MVC


## 上下文在Web容器中的启动过程


在常见的web.xml中需要配置一个`DispatcherServlet`类型的servlet和一个`ContextLoaderListener`类型的listener

`Spring MVC`通过这两个类在Web容器中建立MVC，并将创建好的容器放到`ServletContext`中

`ContextLoaderListener`用于实现`Spring IoC`的启动，创建IoC容器作为"根容器"

`DispatcherServlet`创建另一个IoC容器，并与根容器搭建双亲容器，完成MVC的建立

`ContextLoaderListener`调用方法`contextInitialized()`实现IoC容器的启动

`DispatcherServlet`调用父类的`init()`方法创建IoC容器和搭建双亲容器，以搭建好的IoC容器为基础建立MVC


### 创建根上下文

此`initWebApplicationContext()`方法的实现在`ContextLoaderListener`的父类`ContextLoader`中

```java
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
    // 如果ServletContext已经存在根容器，说明已经创建过根容器，就不需要再执行下面的流程
    if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
        throw ...
    }

    try {
        if (this.context == null) {
            // 创建容器
            this.context = createWebApplicationContext(servletContext);
        }
        if (this.context instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
            // 如果容器没有被初始化过就执行以下流程
            if (!cwac.isActive()) {
                if (cwac.getParent() == null) {
                    // Spring5以后此方法直接返回null值
                    ApplicationContext parent = loadParentContext(servletContext);
                    cwac.setParent(parent);
                }
                // 执行容器的refresh()方法刷新容器
                configureAndRefreshWebApplicationContext(cwac, servletContext);
            }
        }
        // 将容器作为根容器。以指定常量为key，容器为value将其添加到ServletContext中
        servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

        ClassLoader ccl = Thread.currentThread().getContextClassLoader();
        if (ccl == ContextLoader.class.getClassLoader()) {
            currentContext = this.context;
        }
        else if (ccl != null) {
            currentContextPerThread.put(ccl, this.context);
        }

        return this.context;
    }
    catch (Error err) {
        throw err;
    }
}
```


### 在DispatcherServlet中创建IoC容器

`init()`  —— `initServletBean()` —— `initWebApplicationContext()`

此`initWebApplicationContext()`方法的实现在`DispatcherServlet`的父类`FrameworkServlet`中

```java
protected WebApplicationContext initWebApplicationContext() {
    // 从ServletContext中获取根容器
    WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    
    WebApplicationContext wac = null;

    // 默认this.webApplicationContext==null
    if (this.webApplicationContext != null) {
        wac = this.w ebApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {               
                if (cwac.getParent() == null) {
                    // 将根容器作为父容器
                    cwac.setParent(rootContext);
                }
                // 启动容器
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        // 如果wac还是为null就创建容器，并将根容器设置为父容器
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        // 搭建MVC，初始化Spring MVC的九大组件
        onRefresh(wac);
    }

    // 获取 由DispatcherServlet创建的容器名，以kv方式放进ServletContext中
    if (this.publishContext) {     
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
    }

    return wac;
}
```


```java
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
    Class<?> contextClass = getContextClass();

    if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
        throw ...
    }
    
    ConfigurableWebApplicationContext wac =
        (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);

    wac.setEnvironment(getEnvironment());
    
    // 设置双亲容器
    wac.setParent(parent);
    String configLocation = getContextConfigLocation();
    if (configLocation != null) {
        wac.setConfigLocation(configLocation);
    }
    
    /** 配置并刷新容器
      * 在刷新容器时，会为容器创建一个BeanFactory对象。
      * 调用DefaultListableBeanFactory的构造方法时会尝试获取父容器并将父容器本身或父容器持有的BeanFactory作为此BeanFactory的parentBeanFactory
      这个parentBeanFactory会在getBean中被用到（在获取bean前，先尝试从父工厂中获取bean）
    **/
    configureAndRefreshWebApplicationContext(wac);

    return wac;
}
```

