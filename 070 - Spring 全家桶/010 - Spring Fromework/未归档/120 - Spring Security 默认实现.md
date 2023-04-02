#还没有复习 

3. 更多的配置 （重写configure方法）实现的自定义认证和授权（重写用户名密码认证和模仿eladmin的token认证）和自定义授权
4. Spring Security在SpringBoot中的自动装配原理
5. SecurityContext

ps：自定义授权，包含注解式开发和java-configure开发

参考

[Spring Security用户认证和权限控制（默认实现）](https://blog.csdn.net/weixin_44516305/article/details/87860966)

[Spring Security用户认证和权限控制（自定义实现）](https://blog.csdn.net/weixin_44516305/article/details/88868791)



## 认证和鉴权

认证和鉴权分别对应`authentication`/`authorization`，`Spring Security`和`Shiro`都涉及到了这两个词

通俗点说

- 认证就是登录的过程。服务器根据你传递的用户名，密码或者其他形式的参数查询是否存在这个用户，如果存在说明认证成功

- 鉴权就是判断你是否由权力访问资源的流程。你想看某个视频，只有vip用户才能看，但是你当前登录的账户不是vip，所以你不能看。判断能不能看这个视频（是否有权限访问指定的资源）的过程就是鉴权





## Spring Security的过滤器链

`Spring Security`集成了`Spring MVC`，``Spring Security``通过创建一系列`Filter`（过滤器）来实现认证和授权



## 集成Spring Security

1. 导入`Maven`依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

2. 创建`WebSecurityConfigurerAdapter`的子类并添加`@EnableWebSecurity`注解

   ```java
   @EnableWebSecurity
   public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
   }
   ```

至此，启动项目后，`Spring Security`为程序添加了许多过滤器用于认证和鉴权



## Spring Security中内置的过滤器



`Spring Security`中有许多内置的过滤器（这里只简单介绍两个）

在尝试自定义认证和授权流程之前应该先掌握内置的过滤器是如何工作的



其中我认为比较重要的内置过滤器是`UsernamePasswordAuthenticationFilter`和`FilterSecurityInterceptor`（它是过滤器，不是拦截器）



### Authentication



在了解过滤器之前先了解一下`Authentication`类

此类是“认证主体”，这个类的包含了用户是否认证成功，用户的信息等信息

```java
public interface Authentication extends Principal, Serializable {

    // 获取权限
	Collection<? extends GrantedAuthority> getAuthorities();

	Object getCredentials();

	Object getDetails();

	Object getPrincipal();
	
    // 返回认证主体是否已经被认证过
	boolean isAuthenticated();

    // 这只认证主体是否成功通过认证
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}

```





### UsernamePasswordAuthenticationFilter

这个过滤器提供用户名，密码方式认证服务

这个过滤器拦截了`/login`请求并校验用户名，密码。如果认证成功（用户名，密码都对了）就创建一个`Authentication`对象；认证失败就抛`AuthenticationException`



1. 默认只拦截 `POST`方式的`/login`请求，如果不符合直接执行下面的过滤器（`chain.doFilter(request, response);`）
2. 获取请求中的用户名和密码，并创建一个`Authentication`对象（实际类型是`UsernamePasswordAhtienticationToken`），这个对象是没有被认证的“认证主体”
3. 使用成员变量`AuthenticationManager`的`authenticate()`方法进行认证
4. `AuthenticationManager`遍历持有的`AuthenticationProvider`对象，使用能认证此`Ahthentication`的`AuthenticationProvider`进行认证
5. `AbstractUserDetailsAuthenticationProvider`先使用缓存获取user对象，如果获取不到就用持有的`UserDetialsService`和`PasswordEncoder`从`DB`中获取`User`对象并加密密码再封装起来
6. 拿到user对象后校验对象是否过期，密码是否正确等。没问题后创建一个新的`AuthenticationToken`作为已经被认证的“认证主体”
7. 将`AuthenticationToken`放到`SecurityContext`中（`SecurityContext`以后再说）



上述流程中重要的部分是

- 过滤器本身和其认证逻辑
- `AuthenticationManager`对象。它持有各种`AuthenticationProvider`，可以完成对不同类型`Authentication`的认证
- `AuthenticationProvider`持有的`UserDetialsService`和`PasswordEncoder`。他们的作用是创建`UserDetials`对象为以后创建被认证的`Authentication`做铺垫



```java
/* AbstractAuthenticationProcessingFilter 是UsernamePasswordAuthenticationFilter的父类 */
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {

    HttpServletRequest request = (HttpServletRequest) req;
    HttpServletResponse response = (HttpServletResponse) res;

    // 如果请求不符合要求就直接放过
    if (!requiresAuthentication(request, response)) {
        chain.doFilter(request, response);
        return;
    }
    Authentication authResult;
    try {
        // 认证流程在这里，如果认证失败会抛异常
        authResult = attemptAuthentication(request, response);
        if (authResult == null) {
            return;
        }
    }
    catch (AuthenticationException failed) {
        return;
    }

    if (continueChainBeforeSuccessfulAuthentication) {
        chain.doFilter(request, response);
    }

    // 将Authentication放进SecurityContext中
    successfulAuthentication(request, response, chain, authResult);

}

/* UsernamePasswordAuthenticationFilter */
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
    // 如果请求不是POST方式就抛异常
    if (postOnly && !request.getMethod().equals("POST")) {
        throw new AuthenticationServiceException(
            "Authentication method not supported: " + request.getMethod());
    }

    String username = obtainUsername(request);
    String password = obtainPassword(request);

    // 创建一个未认证的AuthenticationToken
    UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
        username, password);
	// 调用AuthenticationManager进行认证
    return this.getAuthenticationManager().authenticate(authRequest);
}

/* ProviderManager */
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    Class<? extends Authentication> toTest = authentication.getClass();

    Authentication result = null;

    // 遍历持有的AuthenticationProvider，如果有AuthenticationProvider能认证Authentication就认证
    for (AuthenticationProvider provider : getProviders()) {
        // AuthenticationProvider是否能认证当前类型的Authentication
        if (!provider.supports(toTest)) {
            continue;
        }
        try {
            // 认证
            result = provider.authenticate(authentication);
            if (result != null) {
                break;
            }
        } catch (AuthenticationException e) {
        }
	}
    if (result != null) {
        return result;
    }
}

/* AbstractUserDetailsAuthenticationProvider */
public Authentication authenticate(Authentication authentication) throws AuthenticationException {

    String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED"
        : authentication.getName();

    // 尝试从缓存中获取User对象
    boolean cacheWasUsed = true;
    UserDetails user = this.userCache.getUserFromCache(username);

    // 缓存中获取不到就重新获取并缓存起来
    if (user == null) {
        cacheWasUsed = false;

        try {
            // 调用UserDetailsService获取user对象并使用PasswordEncoder加密密码
            user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
        }
        catch (UsernameNotFoundException notFound) {
        }
    }

    try {
        // 前置检查
        preAuthenticationChecks.check(user);
        // 检查密码是否正确
        additionalAuthenticationChecks(user,
                                       (UsernamePasswordAuthenticationToken) authentication);
    }
    catch (AuthenticationException exception) {
    }

    // 后置检查。前后置检查都在检查user是否已经过期
    postAuthenticationChecks.check(user);

    // 将user缓存起来
    if (!cacheWasUsed) {
        this.userCache.putUserInCache(user);
    }

    Object principalToReturn = user;

	// 创建一个被认证的Authentication（能执行到这里还没抛异常说明user认证成功了）
    return createSuccessAuthentication(principalToReturn, authentication, user);
}
```



### FilterSecurityInterceptor



这个过滤器被放在过滤器链的最后面，控制鉴权流程，用于检查此次请求是否有权限访问资源，如果不能就抛`AccessException`



1. 从`SecurityContext`中获取`Authentication`对象，如果没有认证就认证一下（调用`AuthenticationManager`认证）
2. 获取当前请求资源所需要的权限
3. 调用`AccessDecisionManager`持有的投票器进行投票，根据投票结果判定请求携带的`Authentication`是否有权限访问资源。判定标准根据`AccessDecisionManager`不同的实现类决定。例如一票否决，一票通过，多数服从少数等标准



上述流程中重要的部分是

- 过滤器本身
- `AccessDecisionManager`和投票器



ps：通常鉴权流程交给这个过滤器就足够了,无需创建新的鉴权过滤器



### AnonymousAuthenticationFilter

匿名的“权限主体”过滤器

1. 如果你没有经过认证就访问被保护起来的资源，这个过滤器会检测到`SecurityContext`中没有`Authentication`对象，并为其创建一个`AnonymousAuthenticationToken`类型的`Authentication`并放进`SecurityContext`
2. 匿名的“认证主体”只有`ROLE_ANONYMOUS`权限。在访问被保护起来的资源时会被`FilterSecurityInterceptor`判定为没有权限访问资源的



