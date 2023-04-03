#还没有复习 

[参考这篇博客](https://www.cnblogs.com/LeeScofiled/p/10511948.html)

# Shiro内置过滤器介绍


Shiro的内置过滤器可以在`DefaultFilter`枚举类中看到，它包含了11个过滤器

```java
// 枚举类元素的名字就是过滤器的名字
anon(AnonymousFilter.class),
authc(FormAuthenticationFilter.class),
authcBasic(BasicHttpAuthenticationFilter.class),
logout(LogoutFilter.class),
noSessionCreation(NoSessionCreationFilter.class),
perms(PermissionsAuthorizationFilter.class),
port(PortFilter.class),
rest(HttpMethodPermissionFilter.class),
roles(RolesAuthorizationFilter.class),
ssl(SslFilter.class),
user(UserFilter.class);
```



在设置鉴权的过滤器和接口关系时用到过这些过滤器的名字

````java
// 创建一个map，k是接口的url，v是过滤器的名字。此处的url支持ant风格
HashMap<String, String> map = new HashMap<>();
map.put("/login", "anon"); // anon对应上述的AnonymousFilter
map.put("/**", "authc"); // authc对应上述的FormAuthenticationFilter
shiroFilterFactoryBean.setFilterChainDefinitionMap(map); // 将上述配置set到ShiroFilterFactoryBean对象中
````


下面将根据`FormAuthenticationFilter`的实现分析Shiro的内置过滤器


# Shiro内置过滤器的继承体系

![[../../020 - 附件文件夹/Pasted image 20230402230714.png]]

`AbstractFilter`实现了`Filter`接口，过滤器执行过滤的方法是`doFilter()`。只要摸清`FormAuthenticationFilter`是怎么执行此方法即可

目标：根据上述继承体系图，掌握`FormAuthenticationFilter`的过滤流程


## AbstractFilter

实现了`init`和`destory`方法，没有实现`doFilter`方法


## NameableFilter

没有实现`doFilter`方法

`NameableFilter`持有成员变量`String name;`，值为过滤器的名字。这个过滤器的作用和它的名字一样，只是给过滤器添加命名的功能


## OncePerRequestFilter

实现了`doFilter`方法，其功能和名字一样，保证每个请求只被当前过滤器对象处理一次

```java
public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {
    String alreadyFilteredAttributeName = this.getAlreadyFilteredAttributeName();
    // 如果过滤器能获取到指定属性值说明这个过滤器被执行过
    if (request.getAttribute(alreadyFilteredAttributeName) != null) {
        filterChain.doFilter(request, response);
    } else if (this.isEnabled(request, response) && !this.shouldNotFilter(request)) {
        request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);
        try {
            // 如果过滤器没有被执行过就执行这个方法
            this.doFilterInternal(request, response, filterChain);
        } finally {
            request.removeAttribute(alreadyFilteredAttributeName);
        }
    } else {
        filterChain.doFilter(request, response);
    }
}
```

**总结：`OncePerRequestFilter`实现的`doFilter`让过滤流程的实现转移到`doFilterInternal`方法中，而这个方法是抽象方法，等待子类实现**

```java
protected abstract void doFilterInternal(ServletRequest var1, ServletResponse var2, FilterChain var3) throws ServletException, IOException;
```


## AdviceFilter

实现了`doFilterInternal`方法

Advice这个词在Spring AOP中出现过，和Spring AOP类似这个过滤器的功能和它的名字一样，提供类似通知的功能

```java
public void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
    Exception exception = null;

    try {
        // 在chain.doFilter(request, response);执行前执行
        boolean continueChain = this.preHandle(request, response);

        if (continueChain) {
            // 执行chain.doFilter(request, response);
            this.executeChain(request, response, chain);
        }

        // 在chain.doFilter(request, response);执行后执行
        this.postHandle(request, response);
    } catch (Exception var9) {
        exception = var9;
    } finally {
        this.cleanup(request, response, exception);
    }
}
```

`preHandle`和`postHandle`方法都是抽象方法，等待子类实现

- `this.preHandle(request, response);`的返回值将决定是否执行`chain.doFilter(request, response);`
- `this.executeChain(request, response, chain);`直接执行`chain.doFilter(request, response);`

**总结：`AdviceFilter`又将过滤流程的实现从`doFilterInternal`方法转移到`preHandle`方法**


## PathMatchingFilter

实现了`preHandle`方法

这个过滤器的功能和PathMatchingFilter的名字一样，匹配路径。`preHandle`方法判断当前请求是否需要进行过滤

判断条件是`PathMatchingFilter`的成员变量`Map<String, Object> appliedPaths`中的key是否含有当前请求所请求的接口url

```java
protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
    if (this.appliedPaths != null && !this.appliedPaths.isEmpty()) {
        Iterator var3 = this.appliedPaths.keySet().iterator();

        String path;
        do {
            // 如果没有找到匹配的url就return
            if (!var3.hasNext()) {
                return true;
            }

            path = (String)var3.next();
        } while(!this.pathsMatch(path, request));

        Object config = this.appliedPaths.get(path);
        // 需要过滤就执行这个方法
        return this.isFilterChainContinued(request, response, path, config);
    } else {
        return true;
    }
}
```

如果需要过滤就执行`onPreHandle`方法，如果此方法返回`false`，`preHandle`方法也会返回`false`

```java
private boolean isFilterChainContinued(ServletRequest request, ServletResponse response, String path, Object pathConfig) throws Exception {
    if (this.isEnabled(request, response, path, pathConfig)) {
        return this.onPreHandle(request, response, pathConfig);
    } else {
        return true;
    }
}
```

**总结：`PathMatchingFilter`将过滤流程的实现由`preHandle`转移到了`onPreHandle`方法**

ps：`onPreHandle`方法默认实现是返回`true`


## AccessFilter

重写了`onPreHandle`方法

这个过滤器的功能和它的名字一样，访问过滤器，用于判断是否允许请求访问接口

```java
public boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
    return this.isAccessAllowed(request, response, mappedValue) || this.onAccessDenied(request, response, mappedValue);
}
```

`onAccessDenied`方法的实现调用了重载方法

```java
protected boolean onAccessDenied(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
    return this.onAccessDenied(request, response);
}
```

`isAccessAllowed`和`onAccessDenied`均是抽象方法，等待子类实现

**总结：`AccessFilter`将过滤流程的实现交给了`isAccessAllowed`和`onAccessDenied`，如果有一个方法返回`true`就能执行后面的过滤器**


## AuthenticationFilter

这个过滤器实现了`isAccessAllowed`方法

```java
protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
    Subject subject = this.getSubject(request, response);
    return subject.isAuthenticated();
}
```

**总结：这个方法比较简单，如果当前Subject对象认证过就返回`true`；否则返回`false`**


## AuthenticatingFilter

这个过滤器没有实现`onAccessDenied`方法，但是重写了`isAccessAllowed`方法

```java
protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
    return super.isAccessAllowed(request, response, mappedValue) || !this.isLoginRequest(request, response) && this.isPermissive(mappedValue);
}
```

- 调用了父类`AuthenticationFilter`的`isAccessAllowed`方法。也就是检查了一下当前Subject对象是否认证过
- 调用了`isLoginRequest`方法。如果请求不是登录请求这个方法就返回`true`。判断条件是请求的url是否是"/login.jsp"（此值由`AccessFilter`的成员变量loginUrl的默认值），由于我们不用jsp，所以方法默认返回`false`
- 调用`isPermissive`方法。给这个方法传递的参数是访问当前接口所需要的权限，类型是String数组。参数由`PathMatchingFilter`的成员变量提供。判断条件是参数中是否包含"permissive"元素，如果包含返回`true`。参数默认为空，所以如果你没有自定义过接口的权限方法返回`false`


虽然这个过滤器看上去没有提供和过滤有关的比较有用的实现，但是它提供了`executeLogin`方法。这个方法会帮你实现`subject.login(token)`的流程，但是`this.createToken(request, response);`是一个抽象方法，需要子类实现，子类`FormAuthenticationFilter`实现了它

```java
protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
    AuthenticationToken token = this.createToken(request, response);
    if (token == null) {
        String msg = "createToken method implementation returned null. A valid non-null AuthenticationToken must be created in order to execute a login attempt.";
        throw new IllegalStateException(msg);
    } else {
        try {
            Subject subject = this.getSubject(request, response);
            subject.login(token);
            return this.onLoginSuccess(token, subject, request, response);
        } catch (AuthenticationException var5) {
            return this.onLoginFailure(token, var5, request, response);
        }
    }
}
```

**总结：AuthenticatingFilter提供了一些如登录，和重定向跳转的方法，并没有onAccessDenied方法的实现，那么这个实现就只能在最后一个子类FormAuthenticationFilter中完成了。**


## FormAuthenticationFilter

这个 过滤器实现了`onAccessDenied`方法

```java
protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
    // 当请求的接口url和loginUrl相同就返回true
    if (this.isLoginRequest(request, response)) {
        // 如果请求是POST方式就返回true
        if (this.isLoginSubmission(request, response)) {
            return this.executeLogin(request, response);
        } else {
            return true;
        }
    } else {
        // 重定向到loginUrl指定的接口
        this.saveRequestAndRedirectToLogin(request, response);
        return false;
    }
}
```

**总结：FormAuthenticationFilter实现了认证失败后的处理逻辑**


# 总结

`AbstractFilter`实现了`init`和`destory`方法，没有实现`doFilter`方法

`NameableFilter`给过滤器添加命名的功能

`OncePerRequestFilter`保证每个请求被过滤一次；将过滤流程的实现从`doFilter`转移到`doFilterInternal`方法中，而这个方法是抽象方法

`AdviceFilter`将过滤流程的实现从`doFilterInternal`方法转移到`preHandle`方法

`PathMatchingFilter`只过滤指定的请求；将过滤流程的实现由`preHandle`转移到了`onPreHandle`方法

`AccessFilter`将过滤流程的实现交给了`isAccessAllowed`和`onAccessDenied`，这两个方法还都是抽象方法。


上述总结中的类都没有实现过滤流程，只是定义了过滤所使用的方法。方法的实现在它们的其他子类中。


Shiro提供了一个过滤器的继承体系（可以根据此继承体系自定义过滤器并添加到Shirio的过滤器链中），还提供了一些实现过滤流程的类（除了DefaultFilter中提供了11个过滤器还有例如`SslFilter`等其他过滤器）


ps：`AccessFilter`的子类有`AuthenticationFilter`和`AuthorizationFilter`。前者是认证过滤器，后者是鉴权过滤器

