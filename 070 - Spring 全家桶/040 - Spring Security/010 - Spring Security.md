#还没有复习 

 b基于认证，授权和鉴权的知识可知



用户的行为是这样

- 用户登录前只用 “匿名用户” 权限
- 用户登录时，后端经过**认证**校验输入信息无误后，返回**授权**凭证（用户用此凭证即代表持有更多的权限）
- 携带凭证访问接口
  - 如果凭证含访问此接口的权限，通过**鉴权**，可访问接口
  - 如果凭证不包含访问此接口的权限，无法通过**鉴权**，访问会被拒绝



一个权限框架的顶层设计应该长这样

- 认证组件
  - 提供 Authentication interface ，其实现类提供多种认证方式
- 授权组件
  - 提供 AuthenticationProvider，其实现类用于生成凭证返回给客户端
- 鉴权组件（filter）
  - 提供 Authorization interface，其实现类用于校验请求中携带的凭证是否有权限访问当前接口

> 以下内容均是基于 Servlet Web 的。但 Spring Security 也支持 Reactive Web


# 认证 - Authentication


## 用户信息

- UserDetailsService 接口。用于向 DB 查询用户信息，以便校验和客户端输入的信息是否一致
- UserDetails 接口。仅用于保存 user 身份信息，是 user 的 Domain
- Authentication 接口。封装 UserDetails，相当于 UserDetails 的状态机，记录此账号是否经过认证，是否过期，是否被锁定
- UserDetailsManager 用于保存和维护 UserDetails 的集合。自定义

>  开启 Spring Security 时如果没有自定义的 UserDetailsManager 组件就会创建一个 InMemoryUserDetailsManager
>
> 并为其维护的 UserDetails 集合里创建一个 user，这就是 Spring Security Hello World 里用的那个 “内置” user



## 用户密码

用户的密码是被加密过的，所以请求中客户端输入的密码和从 DB 中获取的密码都应该是被加密过的

- PasswordEncoder 接口。其实现类用于对密码的加密和校验
- DelegatingPasswordEncoder 代理。其维护多个 PasswordEncoder，并选一个合适的对客户端输入的密码进行校验，以应对不同的加密方式



> MD5 和 sha 不是加密算法，是哈希摘要算法。极易被破解，现已不适用于做密码加密算法

> Spring Security 默认采用 bcrypt 算法作为加密算法
>
> bcrypt 加密时需要到 salt，但进行校验时却不再需要 salt，所以其连 salt 都不需要保存。其特点是慢



# Spring Boot 下的 Spring Security 自动配置

Spring Security 自动装配的入口类在 WebSecurityConfiguration

经过 SpringBoot 的各种自动装配，完成了以下事情

- 创建了默认的 WebSecurityConfiguration，用于保存 Security 下的配置信息
- 创建了 FilterChainProxy，是 Spring Security 的核心过滤器链（这里的代理指一个 filter 代表了一个 FilterChain）
- 创建了 DelegatingFilterProxy，其代理了上面这个真正的 Filter
- 创建了 AuthenticationManager



# WebSecurityConfigurerAdapter 自定义配置类

WebSecurityConfigurerAdapter 是定制 Spring Security 的重要入口，通过继承它实现定制 Spring Security 的所有内容

它是定制 Spring Security 的重要入口，其通过 3 个 configure 同名方法操作 3 个核心类实现定制

```java
void configure(WebSecurity web);
void configure(HttpSecurity http);
void configure(AuthenticationManagerBuilder auth);
```



## 认证管理器配置

AuthenticationManagerBuilder 用于配置 AuthenticationManager

AuthenticationManager 用来管理 UserDetails 和 PasswordEncoder



## 核心过滤器配置

WebSecurity 用于配置 springSecurityFilterChain

但不涉及定制过滤器链的方法，而是核心过滤器的整体配置。比如 ignore 方法用于忽略 Spring Security 对静态资源的限制，配置 HTTPFireWall

所以定制化 Spring Security 通常很少用到它



## 安全过滤器配置方法

HttpSecurity 用于添加自己的过滤器，配置拦截地址和接口权限等

是定制化 Spring Security 的重点



# 解析 UsernamePasswordAuthenticationFilter

> UsernamePasswordAuthenticationFilter 支持很常见的用户名密码方式登录，所以掌握它和它的父类很重要

其 doFilter 做了这些事

- 根据 request 和 response 封装了一个 UsernamePasswordAuthenticationToken
- 用 AuthenticationManager 校验这个 Token 并返回一个 Authentication



通过继承并修改 UsernamePasswordAuthenticationFilter 就能实现以下事情

- 定义 UsernamePasswordAuthenticationFilter 的拦截地址。其默认之拦截 post 方式的 /login 请求
- 定制获取 username 和 password 的方式。这样就能实现从不同格式的参数中获取 username 和 password，比如前端用 json 或 表单或其他格式的数据传参

> UsernamePasswordAuthenticationFilter 如同字面意思，它的目的仅为支持用户名密码登录方式。定制它能从不同格式的参数中获取用户输入，而非定义或支持不同登录方式

# AuthenticationManager

首先 UsernamePasswordAuthenticationFilter 使用的 AuthenticationManager 的对象是`void configure(AuthenticationManagerBuilder auth);`定制的



ProviderManager 是 AuthenticationManager 的实现类，其持有一个 AuthenticationProvider 集合，每种 AuthenticationProvider  仅支持一种 Authentication 的认证

ProviderManager 轮询 AuthenticationProvider 认证。通常只要有一种 AuthenticationProvider 支持认证当前的 Authentication 且认证成功即为认证成功

通常一个 UserDetailsService 配一个 AuthenticationProvider



因为 AuthenticationManager 是处理多方式认证的核心，所以 AuthenticationManager 是实现多种认证方式的核心。学好它很重要



# SharedObject

用于存放共享对象，便于从任何地方获取那些需要的对象。省的到处乱传参数




# 自定义验证码登录

实现自定义方式登录就要实现

拦截器拦截请求后，认证管理器提供认证方式认证

- Filter 拦截指定登录接口
- AuthenticationManager 和 AuthenticationProvider 校验指定 AuthenticationToken
- 校验过程需要结合 UserDetailsService 获取用户真实信息和 AuthenticationToken 中的信息是否匹配



用户的行为包括 1. 获取验证码 2. 校验验证码

所以后端应该提供这两个接口

1. 获取验证码接口，Controller 发送验证码并 return 发送成功或失败
2. 校验验证码接口，其就是验证码登录接口，所以地址可以是 /codeLogin

> PS：登录接口也可以没有对应的 Controller，比如 formLogin 下可以配置 loginProcessingUrl 作为逻辑上的登录接口，但实际上没有此接口。访问此接口时其认证和授权都是在 UsernamePasswordAuthenticationFilter 里做的
>
> 其返回内容可直接写在 response 里（实现跳转到另一个 Controller 或直接返回 json 数据都行）

通常创建 JWT 并返回给用户（直接写 response 里）的流程在 AuthenticationSuccessHandler 里实现

filter 的实现可以是基于 AbstractAuthenticationProcessingFilter 创建的子类，其登录接口就是逻辑接口，实际上并没有对应的 Controller 和接口方法

发送给客户端的响应数据由 filter 的 AuthenticationSuccessHandler 和 AuthenticationFailHandler  

# Spring Security 内置的 Filter

共 33 个内置过滤器，但只有仅有少数 filter 直接被注入 FilterChain。下面列举几个使用的 filter

- RequestCacheAwareFilter。用于用户认证成功后，重新恢复因为登录被打断的请求
- CorsFilter。用于处理跨域
- RememberMeAuthenticationFilter。实现 “记住我” 功能
- AnonymousAuthenticationFilter。匿名
- FilterSecurityInterceptor。**如果你要实现动态权限控制就必须研究该类**

> Spring Security 中有很多过滤器，为了不让代码膨胀，且实现鉴权逻辑统一处理，在鉴权的 filter 中通常不做权限校验的事务，而是这样做
>
> 不管是基于 Session 还是基于 JWT 方式保存用户状态。每个请求必须携带点东西作为自己的凭证
>
> 鉴权 filter 的实现就是校验凭证是否合法，如果合法，将其封装到 AuthenticationToken 并放到 SecurityContext 里
>
> 其权限校验由兜底的 FilterSecurityInterceptor 校验
>
>
> 当然，如果是校验用户名密码是否正确，如果不正确就直接抛异常，而不是由兜底 FilterSecurityInterceptor 处理（具体见 AuthenticationManager 的 authentic 方法）

Spring Security 为这些过滤器定义了加载顺序（如果过滤器链里有这些对象的话就照这个顺序加载）

这个顺序由 FilterComparator 的 filterToOrder 维护

通常实现自定义的登录 filter 就要把自定义的 filter 放在 UsernamePasswordAuthenticationFilter 前



# 过滤器链体系

后端收到客户端发送来的请求后

应用根据请求的 URI 的路径来确定该请求的过滤器链（Filter）以及最终的具体 Servlet 控制器（Controller）  

![[../../020 - 附件文件夹/Pasted image 20230402224149.png|500]]

- GenericFilterBean。Spring Web 中的 Filter，其和 Servlet 标准中的 Filter 不同
- DelegatingFilterProxy。在 Spring 体系中定义、添加 filter 就是向 Servlet 中添加 DelegatingFilterProxy，它是原生 Servlet Filter 和 Spring Filter 的代理
- SecurityFilterChain。一个 SecurityFilterChain 对应处理一个接口要用到的过滤器链
- FilterChainProxy。是 Spring Filter Bean，是 GenericFilterBean 的实现类。对外是一个 filter，其实用于保存并代理 SecurityFilterChain - 过滤器链

FilterChainProxy 根据 url 挑选一个匹配的 filterChains 后再执行过滤流程

```java
private List<Filter> getFilters(HttpServletRequest request) {
    for (SecurityFilterChain chain : filterChains) {
        if (chain.matches(request)) {
            return chain.getFilters();
        }
    }

    return null;
}
```

```java
private List<SecurityFilterChain> filterChains;
```

![[../../020 - 附件文件夹/Pasted image 20230402224201.png|600]]


# 自定义退出登录



实现自定义对出登录，现需要了解服务端为登录成功的客户端做了什么，客户端登录成功后又做了什么

只有了解上述两件事后，才能知道退出登录时，客户端和服务端需要 clear 哪些数据



**一般登录后，服务端会给用户发一个凭证。常见有以下的两种：  **

- 基于 Session 客户端会存 cookie 来保存一个 sessionId ，服务端存一个 Session
- 基于 token 客户端存一个 token 串，服务端会在缓存中存一个用来校验此 token 的信息  

**退出登录需要做这些事**

- 当前的用户登录状态失效。这就需要我们清除服务端的用户状态
- 退出登录接口并不是 permitAll ， 只有携带对应用户的凭证才退出  



自定义退出流程通常用 LogoutConfigurer 配置一个 LogoutFilter，而是不是自己实现一个 Filter



# Spring Security 中的两大异常

- AuthenticationException - 认证异常
- AccessDeniedException - 鉴权 / 拒绝访问异常



这两个异常通常是 Filter 抛出的，Spring Security 用两个异常处理器处理这两个异常

其异常处理其的配置由 HttpSecurity 的 exceptionHandling 提供的 ExceptionHandlingConfigurer 配置

- AuthenticationEntryPoint - 统一处理 AuthenticationException 异常
- AccessDeniedHandler - 统一处理 AccessDeniedException 异常



```java
httpSecurity.exceptionHandling()
	.accessDeniedHandler(new SimpleAccessDeniedHandler())
	.authenticationEntryPoint(new SimpleAuthenticationEntryPoint())
```



执行流程是在 ExceptionTranslationFilter 的 doFilter 中

```java
try {
    chain.doFilter(request, response);
}
catch (Exception ex) {
    // 异常处理器处理两大异常
}
```



# Spring Security 常用工具

- RequestMancher - filter 常用其判断 filter 是否需要拦截当前 url 的访问
- ObjectMapper 和 response.getWriter() - 常用于 filter 中向 response 写数据。比如请求被 filter 拦截且不能向下执行就直接在 response 中写 objectMapper 作为相应数据，并 return，而不再继续执行过滤器链



# 鉴权 - Authentication

Spring Security 默认采用 Session 方式保存用户会话状态。但目前通常用 JWT，所以以下内容均是基于 JWT 实现的鉴权方式

> 使用 JWT 这种无状态方案时，最好在 httpSecurity 里把 Session 状态策略改为无状态
>
> ```java
> // session 生成策略用无状态策略
> .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)  
> ```

鉴权过滤器也放在 UsernamePasswordFilter 前（原因稍后再说）



自定义一个 JwtAuthenticationFilter ，其应该继承 OncePerRequestFilter

鉴权过滤器通常都应该执行这样的逻辑

- 如果 SecurityContext 中已经有 AuthenticationToken，说明请求的凭证已经被提出来了，直接执行下一个过滤器
- 如果上述 if 不成立，就按照自己的逻辑提取出凭证，并校验凭证本身是否合法（而不是校验凭证是否有权限访问此接口）
- 如果凭证本身合法就将其封装到 AuthenticationToken 并放到 SecurityContext 中



上述 filter 是处理 access_token 的逻辑

如果传来的是 refresh_token 就应该再设设置一个处理 refresh_token 的 filter



# 动态访问控制



## permitAll 与 anonymous

- permitAll 不需要任何 Authentication，会放行所有用户的请求
- anonymous 需要当前的 Authentication 是 AnonymousAuthenticationToken，通常登录过的用户是没有这个的



## 动态访问控制的两种实现方式

Spring Security 实现动态控制访问的方式有两种：基于接口，基于注解

> 个人认为基于接口的方式更好，因为可以先把接口许可关系放在 DB，程序运行时动态加载
>
> 如果是基于注解的，就会造成硬编码问题（虽然注解方式更好写）



> 基于注解的配置也可以很复杂，控制粒度可能比基于接口的配置还细。但是是硬编码




# 安全上下文 SecurityContext

- SecurityContext 是一个存放 Authentication 的容器
- SecurityContextHolder 是 SecurityContext 的工具类

其默认用 ThreadLocal 保存 SecurityContext，所以可以在任何地方获取 SecurityContext

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
```


## SecurityContext 的生命周期

准确来说是讨论 Authentication 的生命周期

- filter 处理请求时校验 request 中的 token，校验成功后创建一个 Authentication 并放在 SecurityContext 中
- 返回响应后，Spring Security 自行把 SecurityContext 中的 Authentication 给 clear 掉


## SecurityContext 的存储策略

共有 3 种存储方式

- ThreadLocal 存储。也是默认存储方式
- InheritableThreadLocal 存储。用于多线程下
- MODE_GLOBAL - 静态存储机制。不常用


# 动态访问控制执行流程



## FilterSecurityInterceptor

其处于过滤器链的倒数第 2 个位置。是校验此 Authentication（认证主体）是否有权限访问此接口的入口

它拦截请求后，会

- 获取访问接口需要的权限集合
- 获取 Authentication 中包含的权限
- 采用指定投票策略，检查 Authentication 是否有权限访问此接口



## FilterInvocationSecurityMetadataSource

是标记接口。用于获取访问当前接口所需要的权限集合



## AccessDecisionManager

用于检查当前 Authentication 是否有权限访问当前接口

通常一个接口可能需要多种权限才能访问，所以检查策略有以下几种

- 一票否决：Authentication 缺少一种权限就不能访问接口
- 一票通过：Authentication 只要有一种需要的权限就能访问接口
- 共识通过：Authentication 持有所需要的权限比缺少的资源多就能访问接口

其使用哪种策略取决于当前 AccessDecisionManager 决策器是哪种实现类



# 在一个应用中实现多个安全策略

可以构建多个 WebSecurityConfigurerAdapter  的子类，并为每个 WebSecurityConfigurerAdapter  配置 httpSecurity

使用 http.antMatcher 方法，定义 httpSecurity 对哪写路由生效。这样就实现了从多个 httpSecurity 中选择某个配置

> @Order 指定 WebSecurityConfigurerAdapter  的优先级

```java
// app 接口安全策略
http.antMatcher("/app/v1");
```

```java
// 后台接口安全策略
http.antMatcher("/admin/v1");
```

