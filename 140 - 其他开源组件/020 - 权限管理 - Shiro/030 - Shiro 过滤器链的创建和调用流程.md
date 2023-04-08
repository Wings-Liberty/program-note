#还没有复习 

# 过滤器链的创建和调用流程分析


Shiro以`ShiroFilterFactoryBean`为入口


1. Shiro创建过滤器


`ShiroFilterFactoryBean`是一个`FactoryBean`它会在Spring IOC创建bean是创建一个`SpringShiroFilter`对象。这个对象是执行Shiro过滤器链的入口，但是Shiro实际使用的过滤器链是下面创建的过滤器链


2. Shiro内部创建过滤器链

```java
// 设置接口权限
HashMap<String, String> map = new HashMap<>();
map.put("/login", "anon"); // 设置login使用匿名身份即可访问
map.put("/test", "user");
map.put("/**", "authc"); // 设置其他接口均需要被认证才能访问
shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
```


Shiro最总会根据上述设置，设置一个map

```java
private Map<String, NamedFilterList> filterChains = new LinkedHashMap();
```

这个map的k是url，v是一个过滤器链，链中包含此url所需要使用的过滤器


例如：根据上述配置，'/test'在map中的kv是

```
'test'  ——>  NamedFilterList  (list中包含了user和authc两个过滤器)
```


3. 过滤器链的调用

`SpringShiroFilter`对象执行过滤时会从上述的`filterChains`中根据请求的url获取一个过滤器链来执行过滤


# 过滤器链的创建

这里的内容包含上述流程的1和2


`ShiroFilterFactoryBean`是创建过滤器链的入口。作为一个`FactoryBean`它将创建一个bean，这个bean就是`SpringShiroFilter`


```java
public Object getObject() throws Exception {
    if (this.instance == null) {
        this.instance = this.createInstance();
    }

    return this.instance;
}
```


```java
protected AbstractShiroFilter createInstance() throws Exception {
    log.debug("Creating Shiro Filter instance.");
    SecurityManager securityManager = this.getSecurityManager();
    String msg;
    if (securityManager == null) {
        msg = "SecurityManager property must be set.";
        throw new BeanInitializationException(msg);
    } else if (!(securityManager instanceof WebSecurityManager)) {
        msg = "The security manager does not implement the WebSecurityManager interface.";
        throw new BeanInitializationException(msg);
    } else {
        // 创建过滤器链（下面将分析这个方法）
        FilterChainManager manager = this.createFilterChainManager();
        // 创建一个过滤器链解析器，用于调用过滤器时根据url找到合适的过滤器链
        PathMatchingFilterChainResolver chainResolver = new PathMatchingFilterChainResolver();
        chainResolver.setFilterChainManager(manager);
        return new ShiroFilterFactoryBean.SpringShiroFilter((WebSecurityManager)securityManager, chainResolver);
    }
}
```

`createFilterChainManager`方法实现了过滤器链的创建

```java
// ShiroFilterFactoryBean
protected FilterChainManager createFilterChainManager() {
    DefaultFilterChainManager manager = new DefaultFilterChainManager();
    // 获取Shiro内置的过滤器
    Map<String, Filter> defaultFilters = manager.getFilters();
    Iterator var3 = defaultFilters.values().iterator();

    while(var3.hasNext()) {
        Filter filter = (Filter)var3.next();
        this.applyGlobalPropertiesIfNecessary(filter);
    }

    // 获取自定义的过滤器。获取到的过滤器是shiroFilterFactoryBean.setFilters();中set的过滤器
    Map<String, Filter> filters = this.getFilters();
    String name;
    Filter filter;
    if (!CollectionUtils.isEmpty(filters)) {
        // 为自定义的过滤器设置name。就像内置过滤器一样，例如FormAuthenticationFilter的名字是anthc
        for(Iterator var10 = filters.entrySet().iterator(); var10.hasNext(); manager.addFilter(name, filter, false)) {
            Entry<String, Filter> entry = (Entry)var10.next();
            name = (String)entry.getKey();
            filter = (Filter)entry.getValue();
            this.applyGlobalPropertiesIfNecessary(filter);
            if (filter instanceof Nameable) {
                ((Nameable)filter).setName(name);
            }
        }
    }

    // 获取到的map是shiroFilterFactoryBean.setFilterChainDefinitionMap(map);处set的map，map的k是url，v是过滤器的name
    Map<String, String> chains = this.getFilterChainDefinitionMap();
    if (!CollectionUtils.isEmpty(chains)) {
        Iterator var12 = chains.entrySet().iterator();
        
		// 遍历map，根据map中的kv创建过滤器链
        while(var12.hasNext()) {
            Entry<String, String> entry = (Entry)var12.next();
            String url = (String)entry.getKey();
            String chainDefinition = (String)entry.getValue();
            // 根据url和过滤器的name将过滤器put到Map<String, NamedFilterList> filterChains
            // filterChains是DefaultFilterChainManager的成员变量
            manager.createChain(url, chainDefinition);
        }
    }

    return manager;
}
```

添加过后filterChains长这样。url就是过滤器链的名字（chainName）

![[../../../020 - 附件文件夹/Pasted image 20230402231118.png|725]]

至此就完成了过滤器链的创建


# 过滤器链的调用


下面将讲述`ShiroFilterFactoryBean`创建创建的`SpringShiroFilter`对象是怎么选择并执行过滤器链的

**`SpringShiroFilter`的继承体系图**

![[../../../020 - 附件文件夹/Pasted image 20230402231143.png|500]]

其中`AbstractShiroFilter`实现了`doFilterInternal`方法

**`doFilterInternal`方法的第三个参数chain是Servlet容器中已经存在的过滤器链，现在只探究Shiro的过滤器链，所以在以下流程中chain可以忽略不管**

```java
protected void doFilterInternal(ServletRequest servletRequest, ServletResponse servletResponse, final FilterChain chain) throws ServletException, IOException {
    Throwable t = null;

    try {
        final ServletRequest request = this.prepareServletRequest(servletRequest, servletResponse, chain);
        final ServletResponse response = this.prepareServletResponse(request, servletResponse, chain);
        Subject subject = this.createSubject(request, response);
        subject.execute(new Callable() {
            public Object call() throws Exception {
                AbstractShiroFilter.this.updateSessionLastAccessTime(request, response);
                // 这个方法中执行了获取过滤器链并执行过滤器链的流程
                AbstractShiroFilter.this.executeChain(request, response, chain);
                return null;
            }
        });
    } catch (ExecutionException var8) {
        t = var8.getCause();
    } catch (Throwable var9) {
        t = var9;
    }
    // 省略了下面处理异常的代码
}
```


假设当前请求的url是 /test，下面分析`executeChain`方法中获取并执行过滤器链的流程

```java
protected void executeChain(ServletRequest request, ServletResponse response, FilterChain origChain) throws IOException, ServletException {
    // 获取过滤器链
    FilterChain chain = this.getExecutionChain(request, response, origChain);
    // 执行过滤器链
    chain.doFilter(request, response);
}
```


```java
protected FilterChain getExecutionChain(ServletRequest request, ServletResponse response, FilterChain origChain) {
    FilterChain chain = origChain;
    // 获取过滤器链解析器，这个对象就是在创建过滤器链流程中创建的PathMatchingFilterChainResolver对象
    // 如果忘了可以回看上一步创建过滤器链的流程
    FilterChainResolver resolver = this.getFilterChainResolver();
    if (resolver == null) {
        return origChain;
    } else {
        // 获取过滤器链，如果不为空就赋值给chain
        FilterChain resolved = resolver.getChain(request, response, origChain);
        if (resolved != null) {
            chain = resolved;
        } else {
            log.trace("No FilterChain configured for the current request.  Using the default.");
        }
        return chain;
    }
}
```


`getChain`方法将根据url从filterChains（DefaultFilterChainManager的成员变量）中获取一个NamedFilterList

```java
public FilterChain getChain(ServletRequest request, ServletResponse response, FilterChain originalChain) {
    FilterChainManager filterChainManager = this.getFilterChainManager();
    if (!filterChainManager.hasChains()) {
        return null;
    } else {
        // 获取到请求中的url
        String requestURI = this.getPathWithinApplication(request);
        // 获取到filterChains的k的集合的迭代器
        Iterator var6 = filterChainManager.getChainNames().iterator();

        String pathPattern;
        // 遍历filterChains，如果有k的值与请求的url相同就继续；如果没有就返回null
        do {
            if (!var6.hasNext()) {
                return null;
            }

            pathPattern = (String)var6.next();
        } while(!this.pathMatches(pathPattern, requestURI));

		// 根据url从filterChains中获取NamedFilterList并创建FilterChain类型的代理对象
        return filterChainManager.proxy(originalChain, pathPattern);
    }
}
```


```java
public FilterChain proxy(FilterChain original, String chainName) {
    // 根据url从filterChains中获取过滤器链
    NamedFilterList configured = this.getChain(chainName);
    if (configured == null) {
        String msg = "There is no configured chain under the name/key [" + chainName + "].";
        throw new IllegalArgumentException(msg);
    } else {
        // 创建代理对象
        return configured.proxy(original);
    }
}
```


创建代理对象的过程比较简单

```java
// 将过滤器链filters赋值给this.filters。Servlet的其他过滤器链orig赋给this.orig
public ProxiedFilterChain(FilterChain orig, List<Filter> filters) {
    if (orig == null) {
        throw new NullPointerException("original FilterChain cannot be null.");
    } else {
        this.orig = orig;
        this.filters = filters;
        this.index = 0;
    }
}
```


在执行完Shiro的过滤器链后再执行Servlet中的其他过滤器链

```java
public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
    if (this.filters != null && this.filters.size() != this.index) {
        // 先执行完Shiro的过滤器链
        if (log.isTraceEnabled()) {
            log.trace("Invoking wrapped filter at index [" + this.index + "]");
        }

        // 由于doFilter方法传的chains对象是this，所以下一个doFilter方法的执行还在这里，再回到这里是index已经递增了。
        // 待index==this.filters.size()后就进入下面的else执行Servlet中其他的过滤器链
        ((Filter)this.filters.get(this.index++)).doFilter(request, response, this);
    } else {
        // 再执行Servlet中的其他过滤器链
        if (log.isTraceEnabled()) {
            log.trace("Invoking original filter chain.");
        }

        this.orig.doFilter(request, response);
    }

}
```

