#还没有复习 

# Shiro简介

Shiro使用Servlet的过滤器实现认证和鉴权

[官网传送门](http://shiro.apache.org/web.html)（自己并没有看过...）


# Shiro的核心类

`Subject`						认证主体。每个线程都有一个这个类的对象，对象表示当前用户。对象包含了用户是否经过认证，用户持有什么权限

`Realm`							认证器。在查db后获取到用户数据后，将用户的数据封装到此对象中，用于和请求中传来的用户名密码做比对。如果要自定义认证流程，应该创建一个此类的实现类

`SecurityManager`		安全管理器。Shiro的核心。包含各种`Realm`的实现类以及各种过滤器


# 认证流程

1. 使用`Subject currentUser = SecurityUtils.getSubject();`获取当前线程的认证主主体（Subject对象由ThreadLocal保存）
2. 取出请求中的认证信息，如用用户名密码。将认证信息封装到`AuthenticationToken`中
3. 调用`currentUser.login(token);`完成认证。（如果认证出错会抛出异常。且可以调用`currentUser.isAuthenticated()`随时判断Subject是否通过了认证）
4. 在`currentUser.login(token);`中会使用SecurityManager对象获取到所有的Realm并遍历它们的认证方法。如果有一种Realm支持当前AuthenticationToken且通过认证，Subject就能通过认证


# Spring整合Shiro实现流程


Spring整合Shiro需要完成两件事

- 实现认证：创建登录接口，并实现上述认证流程
- 实现鉴权（授权）：向`SecurityManager`中添加过滤器，设置指定的接口要用指定的过滤器进行过滤，设置指定接口需要有某些权限才能访问

Spring整合XXX时，基本都是将XXX的核心类作为Spring Bean注入到Spring IOC容器中

Spring整合Shiro时，也需要这样做


## 实现认证

以下对象均需要注入到Spring IOC容器中

ps：假设认证方式是用户名密码登录

1. 创建`Realm`的实现类。如果有必要还可以创建一个自定义的`AuthenticationToken`
2. 创建`SecurityManager`，并将自定义的`Realm`的实现类set到SecurityManager中
3. 实现登录接口
   1. 将接口方法的方法参数中的用户名密码并将其封装到AutjenticationToken中
   2. 获取Subject对象，并执行`subject.login(token);`方法

认证成功后，再从其他地方获取Subject对象并调用`isAuthenticated()`方法时就会返回`true`

这个Subject对象会在鉴权时用到


## 实现鉴权

授权也可以称为鉴权

提前设置好在访问指定接口前需要先执行指定的过滤器。过滤器通常会检查Subject对对象是否完成了认证，以及Subject对象是否拥有访问接口的权限

所以，Shiro实现鉴权就是

- 将自定义的过滤器放到ShiroFilterFactoryBean对象中（ShiroFilterFactoryBean会自行将自定义的过滤器放到SecurityManager中）或直接使用Shiro内置的过滤器

  ```java
  // 将自定义的过滤器添加到ShiroFilterFactoryBean对象中
  shiroFilterFactoryBean.setFilters();
  ```


- 指定哪些接口使用哪些过滤器，例如

  ```java
  // 创建一个map，k是接口的url，v是过滤器的名字。此处的url支持ant风格
  HashMap<String, String> map = new HashMap<>();
  map.put("/login", "anon"); // 设置login使用匿名身份即可访问，anon是Shiro一个内置过滤器的名字
  map.put("/**", "authc"); // 设置其他接口均需要被认证才能访问
  shiroFilterFactoryBean.setFilterChainDefinitionMap(map); // 将上述配置set到ShiroFilterFactoryBean对象中
  ```

为了能让Shiro的过滤器生效需要将Shiro的过滤器添加到Servlet容器中

使用web.xml添加过滤器，使用SpringMVC的`WebMvcConfigurationSupport`实现类添加过滤器或使用SpringBoot提供的注解式方式添加过滤器均可。下面使用的是SpringMVC的`WebMvcConfigurationSupport`实现类添加过滤器

1. 创建`ShiroFilterFactoryBean`，将容器中的SecurityManager对象set到ShiroFilterFactoryBean对象中（稍后解释ShiroFilterFactoryBean）

2. 将`ShiroFilterFactoryBean`添加到Servlet容器中

   ```java
   @Configuration
   public class WebMVCConfiguration extends WebMvcConfigurationSupport {
       @Bean
       public FilterRegistrationBean setLogServiceFilter(ShiroFilterFactoryBean shiroFilter) {
           // DelegatingFilterProxy是代理对象
           // 它会从IOC容器中找filter-name中同名的Filter的实现类作为过滤器
           FilterRegistrationBean<DelegatingFilterProxy> registrationBean = new FilterRegistrationBean<>();
           registrationBean.setFilter(new DelegatingFilterProxy());
           registrationBean.setName("shiroFilter"); // 自定义的过滤器名
           registrationBean.addUrlPatterns("/*");
           return registrationBean;
       }
   
   }
   ```

下面解释一下`ShiroFilterFactoryBean`和`DelegatingFilterProxy`

`ShiroFilterFactoryBean`

此类`FactoryBean`接口。它会在容器调用`getBean();`时创建一个Shiro的过滤器链


`DelegatingFilterProxy`

此类是一个Filter，从名字上看就知道它时一个代理类，其`doFilter()`方法实际会调用成员变量`Filter delegate`的`doFilter()`方法

delegate是在Servlet初始化Filter时，DelegatingFilterProxy根据过滤器名（上述代码中的过滤器名叫'shiroFilter'）调用容器的`getBean()`方法获取到。具体代码如下

```java
// 方法执行完后将返回值赋值给delegate
protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
    String targetBeanName = getTargetBeanName();
    Assert.state(targetBeanName != null, "No target bean name set");
    Filter delegate = wac.getBean(targetBeanName, Filter.class);
    if (isTargetFilterLifecycle()) {
        delegate.init(getFilterConfig());
    }
    return delegate;
}
```


## 登出

1. 获取Subject对象（`SecurityUtils.getSubject();`）
2. 调用`subject.logout();`方法

即可完成subject对象的清理工作

登出后，subject会回到未认证过的状态