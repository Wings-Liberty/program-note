#还没有复习 

> 运行环境：JDK8



# 重要组件



## UserDetailsService

用于处理密码是否匹配的接口

需要自己创建实现类并添加到容器中



## AuthorizationServerConfigurerAdapter认证服务器

能干什么？

- `public void configure(AuthorizationServerEndpointsConfigurer endpoints)`设置认证所用的UserDetailsService对象，authenticationManager对象。设置生成jwt所用的签名和增强器。设置处理认证异常处理器
- `public void configure(ClientDetailsServiceConfigurer clients)`设置认证所需要的clientid和clientsecret和认证方式（刷新令牌，密码模式，授权码模式）和令牌时间和刷新令牌有效时间



## ResourceServerConfigurerAdapter资源服务器

能干什么？

- `public void configure(HttpSecurity http)`设置接口的权限。设置哪些接口需要认证，哪些需要权限，哪些直接放过





# 授权流程

登录/获取令牌时所经过的流程

包括校验用户名和密码，创建令牌和刷新令牌的流程（创建令牌时使用增强器对jwt进行增强）

![[../../020 - 附件文件夹/Pasted image 20230402224431.png]]

```java
@RequestMapping(value = "/oauth/token", method=RequestMethod.POST)
	public ResponseEntity<OAuth2AccessToken> postAccessToken(Principal principal, @RequestParam
	Map<String, String> parameters) throws HttpRequestMethodNotSupportedException {

		...

		String clientId = getClientId(principal);
        // 使用ClientDetailsSercice获取到ClientDetails
		ClientDetails authenticatedClient = getClientDetailsService().loadClientByClientId(clientId);

		TokenRequest tokenRequest = getOAuth2RequestFactory().createTokenRequest(parameters, authenticatedClient);

		if (clientId != null && !clientId.equals("")) {
			if (!clientId.equals(tokenRequest.getClientId())) {
				...
			}
		}
		if (authenticatedClient != null) {
			oAuth2RequestValidator.validateScope(tokenRequest, authenticatedClient);
		}
		...

		if (isRefreshTokenRequest(parameters)) {
			tokenRequest.setScope(OAuth2Utils.parseParameterList(parameters.get(OAuth2Utils.SCOPE)));
		}

        // 使用TokenGranter创建OAuth2Request和Authentication 整合这两个对象后再做一些处理返回OAtuhentication对象
		OAuth2AccessToken token = getTokenGranter().grant(tokenRequest.getGrantType(), tokenRequest);
		if (token == null) {
			...
		}

		return getResponse(token);

	}
```


```java
// OAuth2AccessToken token = getTokenGranter().grant(tokenRequest.getGrantType(), tokenRequest);
// 这里使用的TokenGranter是一个CompositeTokenGranter类型的匿名类
tokenGranter = new TokenGranter() {
    private CompositeTokenGranter delegate;

    @Override
    public OAuth2AccessToken grant(String grantType, TokenRequest tokenRequest) {
        if (delegate == null) {
            delegate = new CompositeTokenGranter(getDefaultTokenGranters());
        }
        return delegate.grant(grantType, tokenRequest);
    }
};
```


````java
// delegate.grant(grantType, tokenRequest);

public OAuth2AccessToken grant(String grantType, TokenRequest tokenRequest) {
    // 下面一张图讲了tokenGranters里面都有什么
    for (TokenGranter granter : tokenGranters) {
        // 如果granter不支持当前的认证方式，返回null。如果支持就生成令牌（执行签名加密，自定义增强器的增强都是在这里执行的）
        // 当前使用的认证方式为名密码模式，使用的是ResourceOwnerPassword
        OAuth2AccessToken grant = granter.grant(grantType, tokenRequest);
        if (grant!=null) {
            return grant;
        }
    }
    return null;
}
````

![[../../020 - 附件文件夹/Pasted image 20230402224451.png]]

由名字可以知道这里的XXXTokenGranter是各种认证方式所使用的创建令牌的令牌生成器

授权码，刷新令牌，隐含，客户端证书，资源持有者密码  模式token生成器


```java
public OAuth2AccessToken grant(String grantType, TokenRequest tokenRequest) {

    if (!this.grantType.equals(grantType)) {
        return null;
    }

    String clientId = tokenRequest.getClientId();
    ClientDetails client = clientDetailsService.loadClientByClientId(clientId);
    validateGrantType(grantType, client);


    return getAccessToken(client, tokenRequest);

}

protected OAuth2AccessToken getAccessToken(ClientDetails client, TokenRequest tokenRequest) {
    // getOAuth2Authentication(client, tokenRequest) 校验密码，生成OAuth2Authentication
    // tokenServices.createAccessToken 创建refreshToken，使用jwt增强器或jwt增强器链中的增强器对令牌进行增强
    // 其中还包含了使用tokenStore存储刚创建的令牌的流程。使用的默认的tokenStore会将令牌存到内存中。你也可以自己写一个tokenStore将toekn存到Redis中。暂时不整，有时间可以尝试一下
    return tokenServices.createAccessToken(getOAuth2Authentication(client, tokenRequest));
}

protected OAuth2Authentication getOAuth2Authentication(ClientDetails client, TokenRequest tokenRequest) {

    Map<String, String> parameters = new LinkedHashMap<String, String>(tokenRequest.getRequestParameters());
    String username = parameters.get("username");
    String password = parameters.get("password");
    // Protect from downstream leaks of password
    parameters.remove("password");

    Authentication userAuth = new UsernamePasswordAuthenticationToken(username, password);
    ((AbstractAuthenticationToken) userAuth).setDetails(parameters);
    try {
        // 这里使用的authenticationManager就是认证服务器那指定的authenticationManager对象
        // 这里会调用自定义的UserDetailsService的实现类进行认证
        userAuth = authenticationManager.authenticate(userAuth);
    }
    catch (XXException ase) {
        ...
    }
    
    ...

    // 使用OAuth2Request和Authentication创建一个OAuth2Authentication令牌
    OAuth2Request storedOAuth2Request = getRequestFactory().createOAuth2Request(client, tokenRequest);		
    return new OAuth2Authentication(storedOAuth2Request, userAuth);
}
```


创建令牌的流程就是这些，下面详细讲讲自定义的UserDetailService实现类是怎么进行密码校验和用户账号是否可用的

关于自定义的UserDetailService实现类对密码的校验，在上述的getAccessToken方法中又调用了几个方法栈。

```
authenticate:166, AbstractUserDetailsAuthenticationProvider (org.springframework.security.authentication.dao)
authenticate:175, ProviderManager (org.springframework.security.authentication)
authenticate:195, ProviderManager (org.springframework.security.authentication)
authenticate:501, WebSecurityConfigurerAdapter$AuthenticationManagerDelegator (org.springframework.security.config.annotation.web.configuration)
getOAuth2Authentication:71, ResourceOwnerPasswordTokenGranter (org.springframework.security.oauth2.provider.password)
getAccessToken:72, AbstractTokenGranter (org.springframework.security.oauth2.provider.token)
```

具体的校验在`AbstractUserDetailsAuthenticationProvider`的authenticate方法中

```java
public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
    ...

    String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED"
        : authentication.getName();

    boolean cacheWasUsed = true;
    
    // 使用用户名从缓存中获取user对象
    UserDetails user = this.userCache.getUserFromCache(username);

    if (user == null) {
        cacheWasUsed = false;

        try {
            // 调用了自定义的UserDetailService实现类并catch了UsernameNotFoundException和InternalAuthenticationServiceException（内部身份验证服务异常，没见过这个玩意）
            user = retrieveUser(username,
                                (UsernamePasswordAuthenticationToken) authentication);
        }
        catch (UsernameNotFoundException notFound) {
            ...
        }

    }

    try {
        // 对user的user.isAccountNonLocked()，user.isEnabled()，user.isAccountNonExpired()进行了检查
        preAuthenticationChecks.check(user);
        // 执行校验密码。密码不匹配就抛BadCredentialsException
        additionalAuthenticationChecks(user,
                                       (UsernamePasswordAuthenticationToken) authentication);
    }
    catch (AuthenticationException exception) {
        ...
    }

    // 检查用户的证书是狗是否国过期 user.isCredentialsNonExpired()
    postAuthenticationChecks.check(user);

    // 如果这个user没有被缓存，就缓存起来。在调用自定义的UserDetailService实现类前会根据username在缓存中找user。如果找不到再调用自定义的UserDetailService实现类
    if (!cacheWasUsed) {
        this.userCache.putUserInCache(user);
    }

    Object principalToReturn = user;

    if (forcePrincipalAsString) {
        principalToReturn = user.getUsername();
    }
	
    return createSuccessAuthentication(principalToReturn, authentication, user);
}
```


# 过滤器

认证流程涉及到大量的过滤器，就先说说过滤器

- 在springboot中怎么创建过滤器
- 过滤器使用的是动态代理吗？用的是aop，不是动态代理。因为过滤器是在执行dispatcherServlet的doDispatch前执行的
- 过滤器的执行时机


## SpringBoot中创建过滤器

两种创建方式

1. 使用`FilterRegistrationBean`创建

   ```java
   // 由名字可知FilterRegistrationBean是一个RegistrationBean
   @Bean
   public FilterRegistrationBean myFilter(){
       FilterRegistrationBean registrationBean = new FilterRegistrationBean();
       registrationBean.setFilter(new MyFilter()); // 传入Filter接口的实现类
       registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet")); // set过滤器需要过滤的url
       return registrationBean;
   }
   ```

2. 使用`@WebFilter`和`@ServletComponentScan`

   ```java
   @WebFilter(urlPatterns = "/*") // 指定过滤器要过滤的url
   public class MyFilter  implements Filter {
       @Override
       public void init(FilterConfig filterConfig) throws ServletException {
           System.out.println("执行MyFilter的init方法");
       }
   
       @Override
       public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
           System.out.println("执行chain.doFilter前执行MyFilter的doFilter方法");
           chain.doFilter(request,response);
           System.out.println("执行chain.doFilter后执行MyFilter的doFilter方法");
       }
   
       @Override
       public void destroy() {
           System.out.println("执行MyFilter的destroy方法");
       }
   }
   ```
   
   ```java
   @SpringBootApplication
   @ServletComponentScan("com.cx.filter") // 将创建的过滤器扫描进来
   public class BootApplication {
   
       public static void main(String[] args) {
               SpringApplication.run(BootApplication.class ,args);
       }
   }
   ```


   这种做法是Servlet3.0规范的，不是SpringBoot规范的

   既然这样，还是用第一种办法吧，书写简单而且是springboot支持的方式

​      

   ==过滤器将形成一条过滤器链，如果不在`doFilter`方法中执行`chain.doFilter`方法。请求将不会发送到接口方法==


   当过滤器链中由多个过滤器时，将会这样执行

   例如有A、B、C、D四个过滤器   

   A 执行chain.doFilter前 执行MyFilter的doFilter方法
   B 执行chain.doFilter前 执行MyFilter的doFilter方法
   C 执行chain.doFilter前 执行MyFilter的doFilter方法
   D 执行chain.doFilter前 执行MyFilter的doFilter方法

   D 执行chain.doFilter后 执行MyFilter的doFilter方法
   C 执行chain.doFilter后 执行MyFilter的doFilter方法
   B 执行chain.doFilter后 执行MyFilter的doFilter方法
   A 执行chain.doFilter后 执行MyFilter的doFilter方法


## 过滤器的执行时机


### init
   

过滤器的`init`方法的执行是在`AbstractApplicationContext`的`refresh`的 `onRefresh`方法中执行的

`onRefresh`是空方法，实现留给了子类。


### doFilter

在请求进入`DispatcherServlet`的`doDispatch`方法前就执行了doFilter



# 认证流程

使用令牌访问那些需要token的接口时所经过的流程



在资源服务器在SpringSecurity过滤器链中添加了一个过滤器`OAuthenticationProcessingFilter`用于校验token

   ```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException,
        ServletException {

    final boolean debug = logger.isDebugEnabled();
    final HttpServletRequest request = (HttpServletRequest) req;
    final HttpServletResponse response = (HttpServletResponse) res;

    try {

        // 从请求头中获取到Authorization。request.getHeaders("Authorization")
        // Bearer asdafhmxxx...xxx... 再将'Bearer '这个前缀去掉 String authHeaderValue = value.substring(OAuth2AccessToken.BEARER_TYPE.length()).trim();
        // 将token等信息并将其封住到Authentication中
        Authentication authentication = tokenExtractor.extract(request);

        if (authentication == null) {
            if (stateless && isAuthenticated()) {
                if (debug) {
                    logger.debug("Clearing security context.");
                }
                SecurityContextHolder.clearContext();
            }
            ...
        }
        else {
            request.setAttribute(OAuth2AuthenticationDetails.ACCESS_TOKEN_VALUE, authentication.getPrincipal());
            if (authentication instanceof AbstractAuthenticationToken) {
                AbstractAuthenticationToken needsDetails = (AbstractAuthenticationToken) authentication;
                needsDetails.setDetails(authenticationDetailsSource.buildDetails(request));
            }
            // 校验token。下面将分析这个方法。（这个authenticationManager是不是认证服务器中指定的authenticationManager？）
            Authentication authResult = authenticationManager.authenticate(authentication);

            ...

            SecurityContextHolder.getContext().setAuthentication(authResult);

        }
    }
    catch (OAuth2Exception failed) {
        ...
    }

    chain.doFilter(request, response);
}
````



```java
public Authentication authenticate(Authentication authentication) throws AuthenticationException {

    if (authentication == null) {
        throw new InvalidTokenException("Invalid token (token not found)");
    }
    // 获取到token
    String token = (String) authentication.getPrincipal();
    
    // 校验令牌的流程在这个方法中。下面会继续分析这个方法的执行
    // 	private ResourceServerTokenServices tokenServices;
    // 这个是tokenServices的声明。由此看出认证流程由资源服务器来做。但是自定义的认证服务器并不是这个
    // @todo 整明白自定义的认证服务器和这个对象是啥关系
    OAuth2Authentication auth = tokenServices.loadAuthentication(token);
    
    ...

    Collection<String> resourceIds = auth.getOAuth2Request().getResourceIds();
   
    ...

    checkClientDetails(auth);

    if (authentication.getDetails() instanceof OAuth2AuthenticationDetails) {
        OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails) authentication.getDetails();
        // Guard against a cached copy of the same details
        if (!details.equals(auth.getDetails())) {
            // Preserve the authentication details from the one loaded by token services
            details.setDecodedDetails(auth.getDetails());
        }
    }
    auth.setDetails(authentication.getDetails());
    auth.setAuthenticated(true);
    return auth;

}
```



```java
public OAuth2Authentication loadAuthentication(String accessTokenValue) throws AuthenticationException,
    InvalidTokenException {
       
    // 使用tokenStore将token转为令牌对象
    OAuth2AccessToken accessToken = tokenStore.readAccessToken(accessTokenValue);
        
    // 判空，查令牌是不是过期
    if (accessToken == null) {
        throw new InvalidTokenException("Invalid access token: " + accessTokenValue);
    }
    else if (accessToken.isExpired()) {
        tokenStore.removeAccessToken(accessToken);
        throw new InvalidTokenException("Access token expired: " + accessTokenValue);
    }

    // 获取令牌中的权限
    OAuth2Authentication result = tokenStore.readAuthentication(accessToken);
    if (result == null) {
        // in case of race condition
        throw new InvalidTokenException("Invalid access token: " + accessTokenValue);
    }
    if (clientDetailsService != null) {
        String clientId = result.getOAuth2Request().getClientId();
        try {
            clientDetailsService.loadClientByClientId(clientId);
        }
        catch (ClientRegistrationException e) {
            throw new InvalidTokenException("Client not valid: " + clientId, e);
        }
    }
    return result;
}
```

