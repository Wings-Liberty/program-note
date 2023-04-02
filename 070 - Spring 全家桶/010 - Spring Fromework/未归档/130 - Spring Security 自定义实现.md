#还没有复习 

## 如何实现自定义的认证和授权

`Spring Security`重写`WebSecurityConfigurerAdapter`方法可以实现自定义的认证和授权，有两种方式可用

- 在`configure(HttpSecurity http)`方法种调用`HttpSecurity `提供的3种认证方法开启不同的认证方式

- 在`configure(HttpSecurity http)`方法种调用`HttpSecurity `的`apply()`方法，将自定义的`SecurityConfigurerAdapter`（包含了你自定义的认证和授权）导入`Spring Security`。


ps：`HttpSecurity `提供了

- `openidLogin()`     基于 OpenId 的验证
- `formLogin()`         基于表单的身份验证
- `oauth2Login()`     基于外部OAuth 2.0或OpenID Connect 1.0提供程序配置身份验证



上述两种自定义方式本质上都是调用了`HttpSecurity `对象的`apply()`方法。第一种方式其实是将提前写好的`SecurityConfigurerAdapter`实现类作为参数传到`apply()`方法种



关于`WebSecurityConfigurerAdapter`和`SecurityConfigurerAdapter`以后再说



## WebSecurityConfigurerAdapter



这里简单说一下`WebSecurityConfigurerAdapter`



通过重写`WebSecurityConfigurerAdapter`的三个`configure()`方法实现自定以认证和授权

`void configure(HttpSecurity http);`

`void configure(AuthenticationManagerBuilder auth);`

`void configure(WebSecurity web);`

通常重写第一个方法



## 开启并自定义表单登录认知功能

重写`WebSecurityConfigurerAdapter`的`configure(HttpSecurity http);`方法，调用`http.formLogin()`

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    // 关闭csrf
    http.csrf().disable();

    // 创建一个SecurityConfigurerAdapter的子类，并将其应用到http中
    http.apply(customUsernamePasswordAuthenticationConfig);

    http.formLogin() // 开启表单登录认证方式
        .loginPage("/login") // 如果检查到没有登录，就调用这个接口
            .loginProcessingUrl("/login") // UserNamePasswordAuthenticationFilter默认拦截的url是login，这里可以自定义它拦截的url
            .defaultSuccessUrl("/successLogin") // 成功登录后调用的接口（成功通过）
            // .successForwardUrl("/hello") 
            // defaultSuccessUrl和successForwardUrl的区别参考 https://www.xttblog.com/?p=4994
        	// .successHandler()  // 设置一个成功处理器，认证成功会执行此对象的方法
            // .failureHandler()  // 设置一个失败处理器，认证失败会执行此对象的方法
            .permitAll() // 放行登录接口
        	.and()
        .logout() // 开启登出功能，创建DefaultLogoutPageGeneratingFilter
            .logoutUrl("/logout")// 设置执行登出服务的url
            .logoutRequestMatcher(getLogoutRequestMatcher()) // 默认登出只支持get方式的"/logout"，这里设置使用POST和GET方式的/logout都能响应登出
            .logoutSuccessUrl("/successLogout")
            .and()
        //                .rememberMe() // 创建相关过滤器，开启记住我功能，但是需要表单提交一个name为'remember-me'的单选框
        //                .tokenRepository() // 自定义token存储业务
        //                .tokenValiditySeconds() // 自定义token存储时长
        //                .and()
        .authorizeRequests() 
            .antMatchers("/hello") 
            .permitAll() // 放行/hello接口
            .anyRequest()
            .authenticated(); // 其他接口都保护起来
}
```



## 自定义认证流程

下面将模仿`UsernamePasswordAuthenticationFilter`创建一套认证使用的组件



- 写一个`Filter`过滤器，可以继承`AbstractAuthenticationProcessingFilter`（默认只拦截`POST`方式的'/login'的请求，具体见`doFilter()`方法）也可以继承`GenericFilterBean`
- 写`AuthenticationProvider`的实现类，这里使用的”认证主体“实现类是内置的`UsernamePasswordAuthenticationToken`，你也可以自定义一个
- 写一个`UserDetialsService`的实现类并注入到自定义的`AuthenticationProvider`的实现类中
- 写一个`SecurityConfigurerAdapter`的子类，将自定义的`Filter`和`AuthenticationProvider`加入到`HttpSecurity`对象中



```java
@Component
public class CustomUsernamePasswordAuthenticationConfig extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {

    @Autowired
    private CustomUsernamePasswordAuthenticationProvider customUsernamePasswordAuthenticationProvider;

    // 这个方法用到的HttpSecurity对象和自定义的WebSecurityConfigurerAdapter的子类使用的HttpSecurity对象是同一个
    @Override
    public void configure(HttpSecurity http) throws Exception {

        CustomUsernamePasswordAuthenticationFilter filter = new CustomUsernamePasswordAuthenticationFilter();

        // 设置AuthenticaionManager，这里获取到的对象是http中保存的对象
        filter.setAuthenticationManager(http.getSharedObject(AuthenticationManager.class));

        // 设置记住我服务
        //        filter.setRememberMeServices();
        // 设置成功失败处理器
        //        filter.setAuthenticationSuccessHandler();
        //        filter.setAuthenticationFailureHandler();

        http
            // 将自定义的AuthenticationProvider加到http中，http会后会将其加入到AuthenticationManager中
            // 既然是自定义的Filter其实也可以不通过调用AuthenticationManager进行验证，可以直接将自定义的AuthenticationProvider注入到Filter中写创建并校验Authentication的流程
            .authenticationProvider(customUsernamePasswordAuthenticationProvider)
            // 把自定义的过滤器放在UsernamePasswordAuthenticationFilter前
            .addFilterBefore(filter, UsernamePasswordAuthenticationFilter.class);
    }
}
```



```java
public class CustomUsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

    private static final String username = "username";
    private static final String password = "password";

    protected CustomUsernamePasswordAuthenticationFilter() {
        // 设置拦截器拦截/login
        super(new AntPathRequestMatcher("/login", "POST"));
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {

        // 获取用户名，密码
        String username = this.obtainUsername(request);
        String password = this.obtainPassword(request);

        // 判空
        if(StringUtils.isEmpty(username) || StringUtils.isEmpty(password)){
            throw new UsernameNotFoundException("用户名或密码为空");
        }

        // 这里可以使用自定义AuthenticationToken
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(username, password);

        return this.getAuthenticationManager().authenticate(authenticationToken);
    }

    private String obtainPassword(HttpServletRequest request) {
        return request.getParameter(password);
    }

    private String obtainUsername(HttpServletRequest request) {
        return request.getParameter(username);
    }

}
```



```java
@Component
public class CustomUsernamePasswordAuthenticationProvider implements AuthenticationProvider {

    // 自定义的UserDetailsService
    @Qualifier("customUserDetailsService")
    @Autowired
    private UserDetailsService userDetailsService;

    // 自定义的PasswordEncoder
    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {

        // 获取用户名和密码
        String username = authentication.getPrincipal().toString();
        String password = authentication.getCredentials().toString();

        // 获取user，通常自定义的UserDetailsService实现类会包含类似用username查询user对象的逻辑
        UserDetails user = userDetailsService.loadUserByUsername(username);

        if (user == null) {
            throw new InternalAuthenticationServiceException("找不到用户");
        }

        // 省略校验user账户是否过期的流程

        // 直接校验密码
        if (!passwordEncoder.matches(password, user.getPassword())) {
            throw new BadCredentialsException("用户名或密码出错");
        }

        // 到这里还没有出错，就创建一个已经被认证的Authentication
        UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken(user, user.getPassword(), user.getAuthorities());

        return result;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```



## 自定义权限



通常将鉴权流程交给`FilterSecurityInterceptor`就够了，我们要做的是赋予用户不同的权限和为接口设置权限



为用户赋予权限的流程在`UserDetailsService`的`UserDetails loadUserByUsername(String username)`方法中

创建一个`User`对象，并将权限赋值给它，在用户认证成功后即可完成权限赋值



为接口设置权限有两种方式

- Java-configure  写Java代码配置
- 注解配置



## Java-configure配置



```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        .antMatchers("/user").hasRole("ROLE_USER") // 有ROLE_USER权限的用户才能访问'/user'接口
        .antMatchers("/admin").hasRole("ROLE_ADMIN")  // 有ROLE_ADMIN权限的用户才能访问'/admin'接口
        .anyRequest()
        .authenticated();
}
```





### 注解配置

1. 在自定义的`WebSecurityConfigurerAdapter`的子类上添加`@EnableGlobalMethodSecurity`开启注解式配置
2. 在接口方法上使用注解并指定访问接口所需要的权限



```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
}
```



```java
@PreAuthorize("hasAnyRole('ROLE_USER')")
@GetMapping("/user")
@ResponseBody
public UserDetails user(@AuthenticationPrincipal UserDetails userDetails){
    return userDetails;
}

@PreAuthorize("hasAnyRole('ROLE_ADMIN')")
@GetMapping("/admin")
@ResponseBody
public String admin(){
    return "admin";
}
```


ps：关于`hasRole()`方法和	`hasAuthority()`方法的区别[看这里](https://blog.csdn.net/cauchy6317/article/details/85162225?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param)

