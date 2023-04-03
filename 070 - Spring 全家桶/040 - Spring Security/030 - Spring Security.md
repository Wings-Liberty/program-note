#还没有复习 

# 浅学

## 主要用于

- 身份验证和访问控制（认证和授权）
- 简化过滤器，拦截器等

> 安全应该在程序设计之初就需要考虑的


## 使用方式

> 使用aop思想进行横切编程

**重要的几个类**

- WebSecurityConfigurerAdapter  自定义Security策略类
- AuthenticationManagerBuilder  自定义认证策略
- @EnableWebSecurity 开启WebSecurity模式


### 认证和授权

```java
//开启安全模式
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter{
    //认证
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //链式编程
        //设置请求的权限规则
        http.authorizeRequests()
                .antMatchers("/**").permitAll() //匹配路径，全部放行
                .antMatchers("/vipUserInfo").hasRole("vip"); //匹配路径。值放行有vip权限的人

        //没有权限会跳转到"/login"
        //开启表单登陆
        http.formLogin();
    }
    //授权  直接执行这段代码会报错！！！修改方式在下面
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //从内存中读取数据并授权
        //连接数据库比较繁琐，暂时使用从内存中读取的方法
        auth.inMemoryAuthentication()
                .withUser("user1").password("123").roles("vip1", "vip2")
                .and() //用来拼接用户的方法
                .withUser("user2").password("123").roles("vip1")
                .and()
                .withUser("root").password("123").roles("vip1", "vip2", "vpi3");
    }
}
```

报错，密码没有进行加密（框架默认密码不能是明文）

修改如下

```java
@Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {

        //从内存中读取数据并授权
        //连接数据库比较繁琐，暂时使用从内存中读取的方法
        //添加加密，加密方式有很多种，PasswordEncoder接口下的所有实现类都可以使用
        auth.inMemoryAuthentication().passwordEncoder( new BCryptPasswordEncoder() )
                .withUser("user1").password(new BCryptPasswordEncoder().encode("123")).roles("vip1", "vip2")
                .and() //用来拼接用户的方法
                .withUser("user2").password(new BCryptPasswordEncoder().encode("123")).roles("vip1")
                .and()
                .withUser("root").password(new BCryptPasswordEncoder().encode("123")).roles("vip1", "vip2", "vpi3");

    }
```

### 注销 记住我

```java
//开启注销
//清空cookie和session
http.logout();

//设置注销成功后的首页
http.logout().logoutSuccessUrl("/index");
//它还有些设置清除session和cookie等方法

//关闭csrf功能，登陆失败的可能原因之一，这个功能是用来防止网站受到攻击的
http.csrf().disable();
```



### 连接数据库

[给数据库中的数据权限]( https://docs.spring.io/spring-security/site/docs/5.2.2.BUILD-SNAPSHOT/reference/htmlsingle/#jc-authentication-jdbc )


模板引擎中

> sec:authxxx 都是关于权限的语句，是模板引擎和security的整合包中的东西，需要下载依赖

在前端页面中使用模板引擎可以控制显示有权限的用户才能显示


# Security

**前言**

记录部分笔记后才发现，课程部分开发分为功能实现和代码重构两部分，代码重构目的在于，将代码写成像springboot源码一样，具有可扩展性（当别人对于已有的功能不满足时，支持其他人重写代码）具有可配置性（就像在yml里配置数据库信息一样，对于一些可修改的代码参数，实现可以直接在yum配置里修改，而不是在代码里修改参数的值）


## 开始开发
### 代码结构
1. security-study：主模块/父工程
2. security-study-core：核心业务逻辑
3. security-study-browser：浏览器安全特定代码
4. security-study-app：app相关特定代码
5. security-study-demo：样例程序

demo依赖brower，app

browser，app依赖、core


### hello word案例

先关闭session

关闭redis（我自己遇到的问题，需要把依赖注释）

暂时关闭security
> 由于security:
      basic:
        enabled: false 已经过时，所以使用以下代码来代替

```java
//@Configuration //下面这个注解里自带有Configuration注解
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/**").permitAll();
    }

    /**
     * 配置一个userDetailsService Bean不在生成默认security.user用户
     * @return
     */
    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        return super.userDetailsService();
    }

}
```

目的：能让项目正常访问到/hello

## SpringMVC开发RestFul API


### RestFul API

| 传统风格       | restful风格                        |
| -------------- | ---------------------------------- |
| /user/add?id=1 | /user/1（不应该有动词）method=post |

熟练度层级

1. 使用http协议传输
2. url即资源
3. 使用http方法进行操作，使用http状态码表示处理结果
4. 在资源中带有链接


### 使用mock进行测试

**略，课程里使用的mock感觉不好用，直接使用postman**

**新的注解**

@JsonView

作用：返回User对象时，并不想把user对象中的password变量返回，在user类的getpassword方法上使用该注解就可以在返回user对象时不返回password

使用方法：

1. 使用接口来声明多个视图

2. 在pojo的get方法上指定视图

3. 在Controller方法上指定视图

具体使用方法百度

@Valid  校验注解

和BlindingResult类经常一块使用

因为一旦校验失败就会直接返回错误页面，所以使用BlindingResult可以达到带着错误信息继续执行java代码的目的


## 基于表单开发Security

### hello

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin() //使用表单登陆
                .and()
                .authorizeRequests() //对请求做授权
                .anyRequest() // 所有请求
                .authenticated(); //都需要被认证
    }

}
```



### 自定义用户认证逻辑

简单的登陆，因为密码没有加密，所以无法运行

```java
@Component
public class UserService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        // 还有一种构造方法有7个参数，其余四个是用户是否过期，用户是否被冻结，用户密码是否过期等
        //这里返回的user里的用户名密码（密码是从数据库里查出来的）和前端传过来的用户名密码是否一致决定登录是否成功
        return new User("admin",
                "111",
                AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));

    }
}
```


### 加密解密

可执行加密的对象

```java
//这里是security的配置类
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin() //使用表单登陆
                .and()
                .authorizeRequests() //对请求做授权
                .anyRequest() // 所有请求
                .authenticated(); //都需要被认证
    }

}
```


```java
@Component
public class UserService implements UserDetailsService {
    @Autowired
    PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        System.out.println("登陆的用户名是：" + username);

        String passwrd = passwordEncoder.encode("111");

        System.out.println("数据库的密码是：" + passwrd);
		
        return new User("admin",
                passwrd,
                AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));

    }
}
```


### 个性化认证流程

- 自定义登录页面

  - 处理不同类型的请求（判断是不是电脑端的请求）

    略


自定义登录路径

自定义处理请求的路径

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {

//        Logger logger = LoggerFactory.getLogger(getClass());

        http.formLogin() //使用表单登陆
                .loginPage("/myLogin.html") //自定义的登录页面的url
                .loginProcessingUrl("/myLoginController") //修改处理登录请求的url，默认是/login，修改后只是名字改了，其实还是跳转到那了，就像设置了视图跳转器一样
                .and()
                .authorizeRequests() //对请求做授权
                .antMatchers("/myLogin.html").permitAll() //匹配到指定的页面并放行/或者说选择指定的页面放行
                .anyRequest() // 所有请求
                .authenticated() //都需要被认证
                .and()
                .csrf().disable();
    }
```



- 自定义登录成功处理 实现AuthenticationSucceHandler 或继承SavedRequestAwareAuthenticationSuccessHandler
- 自定义登录失败处理 实现AuthenticationFailureHandler 或继承SimpleUrlAuthenticationFailureHandler

AuthenticationSucceHandler和SavedRequestAwareAuthenticationSuccessHandler的区别在于

前置直接实现自己的方法后者可以覆盖父类的这个方法也可以使用父类的方法，父类的方法是用来进行重定向或转发的

继承的这些父类也都实现了前面提到的接口

![[../../020 - 附件文件夹/Pasted image 20230402225011.png]]

登录成功处理

```java
@Component
public class authenticSuccess extends SavedRequestAwareAuthenticationSuccessHandler {

    private Logger log = LoggerFactory.getLogger(getClass());
    
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {

        log.info("登录成功");

        //执行跳转
        super.onAuthenticationSuccess(request,response,authentication);
        
    }
}
```

登录失败处理

```java
@Component
public class authenticFail extends SimpleUrlAuthenticationFailureHandler {

    private Logger log = LoggerFactory.getLogger(getClass());

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {

        log.info("登录失败");

        //执行跳转
        super.onAuthenticationFailure(request,response,exception);

    }
}
```

在配置类里需要装配这两个处理对象，还需要设置成功/失败处理器（叭叭叭叭叭叭）

```java
    //装配处理器
	@Autowired
    AuthenticationSuccessHandler successHandler;

    @Autowired
    AuthenticationFailureHandler failureHandler;

	@Override
    protected void configure(HttpSecurity http) throws Exception {

//        Logger logger = LoggerFactory.getLogger(getClass());

        http.formLogin() //使用表单登陆
                .loginPage("/myLogin.html") //自定义的登录页面的url
                .loginProcessingUrl("/myLoginController") //修改处理登录请求的url，默认是/login，修改后只是名字改了，其实还是跳转到那了，就像设置了视图跳转器一样
                .successHandler(successHandler) //设置成功处理器
                .failureHandler(failureHandler) //设置失败处理器
                .and()
                .authorizeRequests() //对请求做授权
                .antMatchers("/myLogin.html").permitAll() //匹配到指定的页面并放行/或者说选择指定的页面放行
                .anyRequest() // 所有请求
                .authenticated() //都需要被认证
                .and()
                .csrf().disable();
    }
```



###  认证流程小结

目的：将前面的零碎知识穿起来

![[../../020 - 附件文件夹/Pasted image 20230402225028.png]]

#### 认证流程

1. 登陆页面输入用户名

2. UsernamePasswordAuthenticationFilter  获取用户名，密码，封装成一个token（没有封装用户的权限信息），这一步并不进行认证/校验

3. 获取所有登陆方式/或者说是所有类型的token（如：用户名密码登录，短信验证码登陆），判断有没有支持当前登录的登录方式

4. 使用UserDetail接口进行登录校验

   1. 没拿到user抛异常

   2. 拿到用户，执行预检查和附加检查

      如：预检查用户是否可用，用户是否被所锁定，是否过期

      附加检查，校验密码

```java
//之前自定义的这个类的返回值，返回后其实就是springsecurity进行了数据库密码和用哪个户输入密码校验
public class UserService implements UserDetailsService
  
  
  return new User("admin",
			  passwrd,
			  AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
```

      最后检查完后把用户的权限封装到token中

5. 登陆成功调用自定义的登陆成功处理器

   登录失败调用自定义的登录失败处理器

#### 在整个的认证流程中如何实现的信息共享

检查过程中把session中的信息放到线程里（线程里可以设置线程的变量，在认证过程中，同一个线程里的不同方法可以像是拿去全局变量一样拿用户信息）检查完后，把线程里的用户信息再放到session里

简单的说就是把信息放进session里

#### 登陆后怎么获取到用户信息

登陆成功后，后端可以直接使用以下代码获取用户信息

```java
SecurityContextHolder.getContext().getAuthentication();
```

或直接在参数表里写

```java
@RequestMapping("/me")
public Object getCurrentUser(Authentication authentication){
    return authentication;
}
```

或这样写，只获取部分信息

```java
@RequestMapping("/me")
public Object getCurrentUser(@AuthenticationPrincipal UserDetails user){
    return user;
}
```



### 记住我功能



1. 注册token工厂(在securityconfig类中注册)

   ```java
       // 注册前注册数据源和userDetail
   
   	// 先前没有装配这个就能登陆，现在装配是为了配合记住我功能
   	@Autowired
       UserService userService;
   
       @Autowired
       DataSource dataSource;
   
   	//注册token工厂
       @Bean
       public PersistentTokenRepository getTokenRepository() {
           JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
           jdbcTokenRepository.setDataSource(dataSource); //配置数据源
           jdbcTokenRepository.setCreateTableOnStartup(true); //第一次启动时在数据库里创建一张token的表
           return jdbcTokenRepository;
       }
   ```
   
2. 配置

   ```java
   @Override
       protected void configure(HttpSecurity http) throws Exception {
   
   //        Logger logger = LoggerFactory.getLogger(getClass());
   
           http.formLogin() //使用表单登陆
                   .loginPage("/myLogin.html") //自定义的登录页面的url
                   .loginProcessingUrl("/myLoginController") //修改处理登录请求的url，默认是/login，修改后只是名字改了，其实还是跳转到那了，就像设置了视图跳转器一样
                   .successHandler(successHandler) //配置登录成功后的处理器
                   .failureHandler(failureHandler) //配置登录失败后的处理器
                   .and()
                   .rememberMe() //开启记住我功能
                   .tokenRepository(getTokenRepository()) //配置tolen工厂
                   .tokenValiditySeconds(Content.TOKETIME) //设置token过期时间
                   .userDetailsService(userService) // 配置...额，用这个指定的类登录。设置记住我后，自动登录时会调用这个类执行登陆
                   .and()
                   .authorizeRequests() //对请求做授权
                   .antMatchers("/myLogin.html").permitAll() //匹配到指定的页面并放行/或者说选择指定的页面放行
                   .anyRequest() // 所有请求
                   .authenticated() //都需要被认证
                   .and()
                   .csrf().disable(); //关闭跨域限制
       }
   ```

3. 在前端页面添加“记住我”单选框

   ```html
   <!--记住我功能，标签的名字是固定的 name="remember-me" -->
   <tr>
       <td colspan="2">
           <input type="checkbox" name="remember-me" value="true"/>记住我
       </td>
   </tr>
   ```

![[../../020 - 附件文件夹/Pasted image 20230402225053.png]]

**流程**

开启记住我后

登陆认证

- 认证成功
  - 从数据库里查找token
    - 找不到，生成新的token
    - 找到，使用指定的类（我这里是继承了UserDetails的类）登录
- 认证失败，另说



- 从数据库里查找token
  - 找不到，重新登陆认证，生成新token
  - 找到，使用指定的类（我这里是继承了UserDetails的类）登录



### 图片验证码 

- 开发图形验证码登陆接口
  - 生成随机数作为图形验证码
  - 验证码存入session
  - 校验session中的验证码和用户提交的验证码是否一致
- 校验图形验证码登陆
- 重构代码

#### 执行流程

1. 进入指定页面，但因为没有登陆会跳转到指定登录页面

2. 在指定登录页面，因为有后端url，所以会执行接口，生成验证码



3. 点击登陆，进入配置好的过滤器，先检查验证码是否匹配在考虑是放行到登陆过滤器还是return（到登陆页）

#### 编写流程

1. 写生成验证码的接口，保证进入登陆页面时可以看到验证码

   ```java
   @GetMapping("/code/image")
       public void creatCode(HttpServletRequest request, HttpServletResponse response) throws IOException {
   
           ImageCode imageCode = createImageCode(request);
           sessionStrategy.setAttribute(new ServletWebRequest(request), SESSION_KEY, imageCode);
           ImageIO.write(imageCode.getImage(),"JPEG", response.getOutputStream());
   
       }
   
       private ImageCode createImageCode(HttpServletRequest request) {
   
           int width = 76;
           int height = 32;
           BufferedImage image = new BufferedImage(width,height, BufferedImage.TYPE_INT_RGB);
   
           Graphics g = image.getGraphics();
   
           Random random = new Random();
   
           g.setColor(getrandColor(200, 250));
           g.fillRect(0,0,width,height);
           g.setFont(new Font("Times New Roman", Font.ITALIC, 20));
           g.setColor(getrandColor(160, 200));
   
           for(int i = 0; i < 155; i++ ){
               int x = random.nextInt(width);
               int y = random.nextInt(height);
               int xl = random.nextInt(12);
               int yl = random.nextInt(12);
               g.drawLine(x, y, x+xl, y+yl);
           }
   
           String sRand = "";
   
           for(int i = 0; i < 4; i++){
               String rand = String.valueOf(random.nextInt(10));
               sRand += rand;
               g.setColor(new Color(20 + random.nextInt(110), 20 + random.nextInt(110), 20 + random.nextInt(110)));
               g.drawString(rand, 13 * i + 6, 16);
           }
   
           System.out.println("创建验证码的方法创造的验证码是：" + sRand);
   
           g.dispose();
   
           return new ImageCode(image, sRand, 60);
   
       }
   
       //生成随机背景条纹
       private Color getrandColor(int fc, int bc){
           Random random = new Random();
           if(fc > 255){
               fc = 255;
           }
   
           if(bc > 255){
               bc = 255;
           }
           int r = fc + random.nextInt(bc - fc);
           int g = fc + random.nextInt(bc - fc);
           int b = fc + random.nextInt(bc - fc);
   
           return new Color(r,g,b);
       }
   ```

   

2. 写过滤器，过滤器首先判断是不是登陆请求，如果是，那么对比验证码是否匹配，不是，放行

   ```java
       private AuthenticationFailureHandler authenticationFailureHandler;
   
       private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();
   
       @Override
       protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
   
           if (StringUtils.equals("http://localhost:8080/myLoginController", String.valueOf(request.getRequestURL())) &&
               StringUtils.equalsIgnoreCase("post", request.getMethod())) {
               try {
                   validate(new ServletWebRequest(request));
   
               } catch (ValidCodeException e) {
                   authenticationFailureHandler.onAuthenticationFailure(request, response, e);
                   return;
               }
           }
           filterChain.doFilter(request, response);
       }
   
       private void validate(ServletWebRequest request) throws ServletRequestBindingException {
   
           ImageCode codeInSession = (ImageCode) sessionStrategy.getAttribute(request,
                   ValidCodeController.SESSION_KEY);
   
           String codeInRequest = ServletRequestUtils.getStringParameter(request.getRequest(), "imageCode");
   
           if (StringUtils.isEmpty(codeInRequest)) {
               throw new ValidCodeException("验证码的值不能为空");
           }
           if (codeInSession == null) {
               throw new ValidCodeException("验证码不存在");
           }
           if (false) {
               throw new ValidCodeException("验证码已过期");
           }
           if (!StringUtils.equals(codeInSession.getCode(), codeInRequest)) {
               System.out.println("验证码不匹配");
               throw new ValidCodeException("验证码不匹配");
           }
           sessionStrategy.removeAttribute(request, ValidCodeController.SESSION_KEY);
       }
   ```

   

3. 配置过滤器

   ```java
   ValidCodeFilter validCodeFilter = new ValidCodeFilter();
   
           validCodeFilter.setAuthenticationFailureHandler(failureHandler);
   
   //        Logger logger = LoggerFactory.getLogger(getClass());
   
           http.addFilterBefore(validCodeFilter, UsernamePasswordAuthenticationFilter.class)  // 添加验证码过滤器
               .formLogin() //使用表单登陆
               .loginPage("/myLogin.html") //自定义的登录页面的url
               .loginProcessingUrl("/myLoginController") //修改处理登录请求的url，默认是/login，修改后只是名字改了，其实还是跳转到那了，就像设置了视图跳转器一样
               .successHandler(successHandler) //配置登录成功后的处理器
               .failureHandler(failureHandler) //配置登录失败后的处理器
               .and()
               .rememberMe() //开启记住我功能
               .tokenRepository(getTokenRepository()) //配置tolen工厂
               .tokenValiditySeconds(Content.TOKETIME) //设置token过期时间
               .userDetailsService(userService) //配置...额，用这个指定的类登录
               .and()
               .authorizeRequests() //对请求做授权
               .antMatchers("/myLogin.html", "/code/image").permitAll() //匹配到指定的页面并放行/或者说选择指定的页面放行
               .anyRequest() // 所有请求
               .authenticated() //都需要被认证
               .and()
               .csrf().disable(); //关闭跨域限制
   ```




#### 代码重构

- 验证码基本参数可配置（如：验证码位数，配置多个接口可用过滤器）

  使用@ConfigurationProperties注解的类（在这里要学会是哟红配置属性类），可以实现在yum文件里修改类 里的值

![[../../020 - 附件文件夹/Pasted image 20230402225112.png|700]]  

  默认配置就是在配置类中设置默认值（注：这些值的属性不能是final，便于下面修改）

  应用级配置就是在yum中开发者自定义改变默认设置

  请求及设置就是在url后面直接跟参数?xx=xx后端会获取的

  ```java
  ServletRequestUtils.getIntParam(request.getRequest(),xx,xx);
  ```

  

- 验证码拦截的接口可配置

 ```java
  /** * 存放所有需要校验验证码的url */
  private Map<String, ValidateCodeType> urlMap = new HashMap<>();
 ```
 ```java
protected void addUrlToMap(String urlString, ValidateCodeType type) {
        if (StringUtils.isNotBlank(urlString)) {
            String[] urls = StringUtils.splitByWholeSeparatorPreserveAllTokens(urlString, ",");
            for (String url : urls) {
                urlMap.put(url, type);
            }
        }
    }
 ```
  不把调用拦截器的地方局限于登录请求

  同上，把url设置为yum里可配置的参数

  过滤器实现InitializingBean接口

  在security上配置set参数方法

  叭叭叭

- 验证码的生成逻辑可配置

  把生成验证码的方法封装到接口里，再创建一个该接口的实现类generatorCodeImp(这个实现类并不使用@component注解)

  在config类中使用@bean注册generatorCodeImp

  虽然这和在实现类上使用@component效果一样

  但是在@Bean下加上注解@ConditionalOnMissingBean(name="myBean")的话
  
  就可以实现像sproingboot做到的一样，当用户自定义了一个bean（ 如：@Component("myBean") ），并在某个地方进行了这个bean的自动装配，这时自动装配的就是自定义的bean，而带有@ConditionalOnMissingBean(name="myBean")的bean被自定义的bean覆盖掉了
  
  
  
  目的：可以达到，其他人可重写该生成验证码的逻辑
  
  其中里面还涉及了与springboot源码类似的逻辑，所以，了解即可,....



### 实现短信认证登陆

1. 开发短信验证码登陆

2. 校验短信验证码登陆

3. 重构代码



- 开发短信验证码登陆
  1. 创建短信验证码
  2. 把短信验证码放进session里
  3. 从前端获取传来的电话号码
  4. 向指定的电话号码发送短信
  
  

1. 开发短信验证码登陆

分析：

- 需要短信验证码类
- 需要创建短信验证码的类
- 需要发送短信验证码的类

- 重构第一步的代码

![[../../020 - 附件文件夹/Pasted image 20230402225151.png|700]]

2. 校验发送端短信验证码登陆

短信验证码登陆和用户名密码一样混在一块，应该模仿用户名密码登陆流程写一个独立的短信验证码登陆流程

![[../../020 - 附件文件夹/Pasted image 20230402225202.png|700]]

## Spring Social开发第三方登录

### OAuth简介

OAuth协议简介

- 协议要解决的问题
- 协议中的各种角色
- 协议运行流程



- 协议要解决的问题

  只给第三方部分用户的数据，不让第三方获取所有数据

- 协议中的各种角色

  服务提供商（provider）提供token

  ​	认证服务器（Authorization Server）认证身份（登录），产生令牌

  ​	资源服务器（Resource Server）保存用户资源，验证令牌

  资源所有者（owner）用户 --拥有用户的所有资源--

  第三方应用（Client）需要获取用户的资源/部分资源

![[../../020 - 附件文件夹/Pasted image 20230402225224.png|700]]  

- 协议运行流程

  1. 用户请求使用第三方应用提供的功能
  2. 第三方应用请求用户授权
  3. 用户同意授权（在第三方应用里处理），第三方申请令牌
  4. 认证服务器发放令牌
  5. 第三方向资源服务器申请资源
  6. 资源服务器返回开放资源
  
![[../../020 - 附件文件夹/Pasted image 20230402225253.png|700]]

**授权模式**

- 授权码模式

  在流程里第3步中，用户同意授权是在认证服务器里处理

  而且添加了一步认证服务器发授权码，第三方再申请token前发授权码的过程

- 密码模式

- 客户端模式

- 简化模式

后两个模式课里不讲

OAuth协议示意图

![[../../020 - 附件文件夹/Pasted image 20230402225309.png|700]]

服务提供商（实现ServiceProvider接口实现即可）

前5步：标准流程处理类（实现OAuth2perations）

第6步：处理获取到返回公共信息的类（自定义api，继承抽象类AbstractOAuth2ApiBinding）

第7步：实现类Connection ConnectionFactory

![[../../020 - 附件文件夹/Pasted image 20230402225325.png|700]]

> 有点像mvc，ApiAdapter把数据适配后拿过来，ServiceProvider处理从哪拿token，并拿token，数据库操作层存取用户数据

### QQ登陆

[QQ互联，获取公共信息的接口](https://wiki.connect.qq.com/get_user_info)

**实现步骤**

1. 实现自定义api用于检查token和返回公共信息

   文档上说，要获取qq信息需要使用指定接口传三个参数

   https://graph.qq.com/user/get_user_info?access_token=YOUR_ACCESS_TOKEN&oauth_consumer_key=YOUR_APP_ID&openid=YOUR_OPENID

   openid是通过另一个接口获取的

   https://graph.qq.com/oauth2.0/me?access_token=YOUR_ACCESS_TOKEN 

   

   ```java
   /**
    * 资源服务器
    * 这里是qq的自定义api
    */
   public class QQImpl extends AbstractOAuth2ApiBinding implements QQ {
   
       //获取openid的url
       private static final String URL_GET_OPENID = "https://graph.qq.com/oauth2.0/me?access_token=%s";
   
       //获取用户公共信息的url
       private static final String URL_GET_USERINFO = "https://graph.qq.com/user/get_user_info?oauth_consumer_key=%s&openid=%s";
   
       private String appid;
   
       private String openid;
   
       //把json格式的字符串转换为指定的类
       private ObjectMapper objectMapper = new ObjectMapper();
   
       public QQImpl(String accessToken, String appid){
           //把access_token作为参数，并使用第二个参数指定拼接到url后面，而不是放到请求体中
           super(accessToken, TokenStrategy.ACCESS_TOKEN_PARAMETER);
   
           this.appid = appid;
   
           //发送请求获取openid
           String url = String.format(URL_GET_OPENID, accessToken);
           String result = getRestTemplate().getForObject(url, String.class);
   
           System.out.println(result);
   
           this.openid = StringUtils.substringBetween(result, "\"openid\":", "}");
       }
   
       @Override
       public QQUserInfo getQqUserInfo() {
   
           String url = String.format(URL_GET_USERINFO, appid, openid);
           String result = getRestTemplate().getForObject(url, String.class);
   
           System.out.println(result);
   
           try {
               return objectMapper.readValue(result, QQUserInfo.class);
           } catch (Exception e) {
               throw new RuntimeException("获取用户信息失败", e);
           }
       }
   
   }
   ```

   

2. 实现OAuth2perations

   现在是使用spring默认的实现类

3. 实现ServiceProvider

   ```java
   /**
    * 服务提供商
    * 添加了资源服务和认证服务
    */
   public class QQServiceProvider extends AbstractOAuth2ServiceProvider<QQ> {
   
       private String appid;
   
       private static final String URL_AUTHORIZE = "https://graph.qq.com/oauth2.0/authorize";
   
       private static final String URL_ACCESS_TOKEN = "https://graph.qq.com/oauth2.0/token";
   
       public QQServiceProvider(String appid, String appSecret) {
           //第1个参数是第三方发送的将用户导向服务器的地址 / 第2个参数好像是授权码
           //第3个参数是第三方申请授权码的地址 / 第四个参数是用授权码获取token的接口
           //new是服务提供商的认证服务器，这里用的是spring自带的
           super(new OAuth2Template(appid, appSecret, URL_AUTHORIZE, URL_ACCESS_TOKEN));
       }
   
       @Override
       public QQ getApi(String accessToken) {
           return new QQImpl(accessToken, appid);
       }
       
   }
   ```

   这里就实现了服务提供商的实现

4. 实现APIAdapter

   ```java
   /**
    * 资源服务器
    * 这里是qq的自定义api
    */
   public class QQAdapter implements ApiAdapter<QQ> {
       //测试api是否可用
       @Override
       public boolean test(QQ qq) {
           return true;
       }
   
       //适配
       @Override
       public void setConnectionValues(QQ qq, ConnectionValues connectionValues) {
   
           QQUserInfo userInfo = qq.getQqUserInfo();
   
           //适配用户名
           connectionValues.setDisplayName(userInfo.getNickname());
           //适配头像信息
           connectionValues.setImageUrl(userInfo.getFigureurl_qq_1());
           //适配主页，因为qq没有主页，所以不配
           connectionValues.setProfileUrl(null);
           //适配openid
           connectionValues.setProviderUserId(userInfo.getOpenId());
   
       }
   
       //和上面的方法类似，以后再讲
       @Override
       public UserProfile fetchUserProfile(QQ qq) {
           return null;
       }
   
       //qq里不用，所以这里啥也不做
       @Override
       public void updateStatus(QQ qq, String s) {
       }
   }
   ```

   ConnectionFactory的两个组件到这里就写完了，下面开始下ConnectionFactory

5. 实现全部ConnectionFactory

   ```java
   //在构造方法里把两个组件添加进来即可
   //providerId是提供商的唯一标识，它的值在配置里配进来就行了
   public class QQConnectionFactory extends OAuth2ConnectionFactory<QQ> {
       public QQConnectionFactory(String providerId, OAuth2ServiceProvider<QQ> serviceProvider, ApiAdapter<QQ> apiAdapter, String appid, String appSecret) {
           super(providerId, new QQServiceProvider(appid, appSecret), new QQAdapter());
       }
   }
   ```

   

6. 实现Connection

   使用ConnectionFactory的方法创建的，不用自己写具体实现/不用自己处理

7. 实现UserConnectionRespository （JdbcUserConnectionRespository ）用来把获取到的用户公共信息保存到数据库里的工具类，spring已经实现好了（有现成的可以白嫖）具体配置在第八步中

8. 实现其他（如：配置第7步的UserConnectionRespository ）

   ```
   public class SocialConfig extends SocialConfigurerAdapter
   ```

   配置UserConnectionRespository 等

   ```java
   /**
    * 配置类
    * UsersConnectionRepository在这里配置
    * UsersConnectionRepository需要数据源，那就自动装配个数据源放进UsersConnectionRepository的构造参数里
    */
   public class SocialConfig extends SocialConfigurerAdapter {
   
       @Autowired
       DataSource dataSource;
   
       @Override
       public UsersConnectionRepository getUsersConnectionRepository(ConnectionFactoryLocator connectionFactoryLocator) {
           //第三个参数是加密方式，这里选择不加密
        JdbcUsersConnectionRepository repository = new JdbcUsersConnectionRepository(dataSource,connectionFactoryLocator, Encryptors.noOpText());
           return repository;
       }
   }
   ```

在JdbcUsersConnectionRepository目录中所在位置下有关于生成他所需要的标准表的sql语句

![[../../020 - 附件文件夹/Pasted image 20230402225439.png|800]]

![[../../020 - 附件文件夹/Pasted image 20230402225458.png]]

前三个字段是关键

紧接着后面四个字段是用户信息

最后四个是oauth协议里的东西



要通过字段里的id查找用户并封装为一个UserDetails

添加SocialUserDetailsService即可

```java
@Component
public class UserService implements UserDetailsService, SocialUserDetailsService {
    private Logger log = LoggerFactory.getLogger(UserService.class);
    @Autowired
    PasswordEncoder passwordEncoder;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        log.info("表单登陆");
        return builder(username);
    }
    @Override
    public SocialUserDetails loadUserByUserId(String userId) throws UsernameNotFoundException {
        log.info("第三方登录");
        return builder(userId);
    }
    private SocialUserDetails builder(String userId){
        System.out.println("登陆的用户名是：" + userId);
        String passwrd = passwordEncoder.encode("111");
        System.out.println("数据库的密码是：" + passwrd);
        return new SocialUser(userId,
                passwrd,
                true, true,true,true,
                AuthorityUtils.commaSeparatedStringToAuthorityList("admin,vip"));
    }
}
```



>  SocialConfigurerAdapter  SocialProperties
> 这个类在2.x的springboot中没有，要么换版本，要么把老版本中的类直接拷贝过来


设置qq的配置（设置）


```java
@Configuration
//如果配置文件里有注解里指定的配置，那么代码里的配置将会生效
@ConditionalOnProperty(prefix = "cx.security.social.qq", name = "app-id")
public class QQAutoConfig extends SocialAutoConfigurerAdapter {

    @Autowired
    private SecurityProperties securityProperties;

    @Override
    protected ConnectionFactory<?> createConnectionFactory() {
        QQProperties qqProperties = securityProperties.getSocial().getQq();

        return new QQConnectionFactory(qqProperties.getProviderId(), qqProperties.getAppId(), qqProperties.getAppSecret());
    }
    
    //这个函数，不加就报错说无法初始化userIdSource，不知道为啥
     @Override
    public UserIdSource getUserIdSource() {
        return new AuthenticationNameUserIdSource();
    }
}
```

```java
public class QQProperties extends SocialProperties {    
    private String providerId = "qq";    
    public String getProviderId() {        
        return providerId;    
    }    
    public void setProviderId(String providerId) {        
        this.providerId = providerId;    
    }
}
```

```java
public abstract class SocialProperties {
    private String appId;
    private String appSecret;

    public SocialProperties() {
    }

    public String getAppId() {
        return this.appId;
    }

    public void setAppId(String appId) {
        this.appId = appId;
    }

    public String getAppSecret() {
        return this.appSecret;
    }

    public void setAppSecret(String appSecret) {
        this.appSecret = appSecret;
    }
}

```

**重点，添加oauth过滤器**

注册SpringSocialConfigurer

```java
@Configuration
@EnableSocial
public class SocialConfig extends SocialConfigurerAdapter {

    @Autowired
    DataSource dataSource;

    @Override
    public UsersConnectionRepository getUsersConnectionRepository(ConnectionFactoryLocator connectionFactoryLocator) {
        //第三个参数是加密方式，这里选择不加密
        JdbcUsersConnectionRepository repository = new JdbcUsersConnectionRepository(dataSource,connectionFactoryLocator, Encryptors.noOpText());
        return repository;
    }

    @Bean
    public SpringSocialConfigurer springSocialConfigurer(){
        return new SpringSocialConfigurer();
    }

}
```

在Security配置类里配置SpringSocialConfigurer（会自动把过滤器添加进去）

过滤器会自动拦截以"/auth"开头的url

```java
@Autowired
SpringSocialConfigurer springSocialConfigurer;

http
    . ...
	.and()
	.apply(springSocialConfigurer);
```



在页面上添加

```html
<h3>社交登录</h3>
<a href="/auth/qq">QQ登录</a>
```

启动项目，跳转到qq登陆（虽然不能用）





为解决问题需要重新设置redirect url

卡在回调地址了

![[../../020 - 附件文件夹/Pasted image 20230402225519.png]]

### 退出登录

- 如何退出
- Spring Security默认的退出处理逻辑
- 与退出登录相关的配置



#### quickly start

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<h2>首页</h2>
<h2><a href="/logout">退出登录</a></h2>

</body>
</html>
```

登录后进入首页，点击退出登录即可

#### 退出处理

- 使当前session失效
- 清除与当前用户相关的remember-me
- 清空当前的SecurityContext
- 重定向到登录页

**上手快，直接使用即可，个性化也只需要一些简单的步骤即可**

```java
.and()
                //退出登录配置
                .logout()
                .logoutUrl("/signOut") //个性化设置退出的url和设置登录的url一样
//                .logoutSuccessUrl("/myLogout.html") //logoutSuccessUrl和logoutSuccessHandler只能用一个
                .logoutSuccessHandler(logoutSuccessHandler)
```



### 暂时跳过注册



## SpringSecurity OAuth 开发APP认证



> 开始token，哈哈哈哈

- 在资源服务器在SpringSecurity过滤器链中添加了一个过滤器OAuthenticationProcessingFilter

用于检查token，检查请求是否被授权（就是检查token是不是正确，不正确抛OAuth2Exception。如果就没传token是不会抛异常的，会正常结束不过返回信息是有error的信息）

- 认证服务器不光要发放token，还要存token

![[../../020 - 附件文件夹/Pasted image 20230402225534.png|800]]

> 绿色部分是spring现成的，红色部分是自己写的



内容简介

 - 实现标准OAuth协议中Privoder角色的主要功能
 - 重构之前三种认证方式的代码，使其支持token
 - 高级特性
    - jwt
    - sso单点登录

### 开始开发

创建服务器提供商

```java
@Configuration
@EnableAuthorizationServer //开启了这个注解，这个程序就会被当做认证服务器
public class AuthenticationServiceConfig {

}
```



开启后，启动服务器时会自动创建很多个接口的url

```
url: 
/oauth/authorize , methods=GET //用来获取授权码的，目前还没有成功使用过
/oauth/token , methods=POST/GET
```


### quick start

1. 添加资源服务器和认证服务器（在这之前，工作重心由browser转到了app，所以，demo不再依赖browser，而去依赖app，所以需要重新配置服务提供商（资源服务器和认证服务器））

![[../../020 - 附件文件夹/Pasted image 20230402225625.png]]

2. 配置username和password（这里是webapp（第三方应用）的用户名和密码）
3. postman传参（传真正的用户名和密码）获取token

![[../../020 - 附件文件夹/Pasted image 20230402225630.png]]

![[../../020 - 附件文件夹/Pasted image 20230402225638.png]]

4. 使用获取到的token进行操作

![[../../020 - 附件文件夹/Pasted image 20230402225650.png]]

(可用token)[D:\workspace\Typora-workspace\code\jwt_login]


### 重构登录

![[../../020 - 附件文件夹/Pasted image 20230402225705.png|750]]

![[../../020 - 附件文件夹/Pasted image 20230402225753.png|750]]

目的：登录成功后执行自定义的实现了AuthenticationSuccessHandler的类

在这个类里，自己去实现生成token的流程，这样就可以控制生成token的细节

在执行实现了AuthenticationSuccessHandler的类前，需要两个参数（OAuth2Request, Authentication），其中Authentication作为参数直接传到了要执行的业务逻辑中，所以只需要OAuth2Request即可

而OAuth2Request是由下图中的流程创建出来的

![[../../020 - 附件文件夹/Pasted image 20230402225806.png|300]]

```java
@Component
public class AuthenticSuccess extends SavedRequestAwareAuthenticationSuccessHandler {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    ClientDetailsService clientDetailsService;

    @Qualifier("defaultAuthorizationServerTokenServices")
    @Autowired
    AuthorizationServerTokenServices authorizationServerTokenServices;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {

        logger.info("登录成功");

//        UsernamePasswordAuthenticationToken authRequest = authenticationConverter.convert(request);
//        if (authRequest == null) {
//            chain.doFilter(request, response);
//            return;
//        }
//
//        String username = authRequest.getName();


        String header = request.getHeader("Authorization");

        if (header == null || !header.startsWith("Basic ")) {
            throw new UnexpectedException("抛异常");
        }
        String[] tokens = extractAndDecodeHeader(header, request);
        assert tokens.length == 2;

        String clientId = tokens[0];

        String clientSecret = tokens[1];

        ClientDetails clientDetails = clientDetailsService.loadClientByClientId(clientId);

        //校验ClientDetails
        if (clientDetails == null) {
            throw new UnexpectedException("clientId信息不存在");
        } else if (!StringUtils.equals(clientDetails.getClientSecret(), clientSecret)) {
            throw new UnexpectedException("密码不匹配");
        }

        //因为方法的参数表里的authentication有所有参数，所以第一个参数（所有参数组成的集合）就不需要了
        //第四个参数，因为这是不同于默认的4中认证方法，是自定义的认证方法，所以自定义名字
        TokenRequest tokenRequest = new TokenRequest(MapUtils.EMPTY_MAP, clientId, clientDetails.getScope(), "custom");

        //创建OAuth2Request
        OAuth2Request oAuth2Request = tokenRequest.createOAuth2Request(clientDetails);

        //创建OAuth2Authentication
        OAuth2Authentication oAuth2Authentication = new OAuth2Authentication(oAuth2Request, authentication);

        OAuth2AccessToken token = authorizationServerTokenServices.createAccessToken(oAuth2Authentication);

        //输出一下token
        System.out.println("token : " + token);

        //执行跳转
        super.onAuthenticationSuccess(request, response, authentication);

    }

    //源码里拷贝的（老版源码里手抄的）
    private String[] extractAndDecodeHeader(String header, HttpServletRequest request) throws IOException {

        byte[] base64Token = header.substring(6).getBytes("UTF-8");
        byte[] decoded = new byte[0];

        try {
            decoded = Base64.decodeBase64(base64Token);
        } catch (Exception e) {
            e.printStackTrace();
        }

        String token = new String(decoded, "UTF-8");

        int delim = token.indexOf(":");

        if (delim == -1) {
            throw new BadCredentialsException("Invalid basic authentication toekn");

        }

        return new String[]{
                token.substring(0, delim), token.substring(delim + 1)
        };
    }

}
```


### 暂时跳过第三方登录的重构


### Json Web Token


#### Token处理

- 基本的Token参数配置
- 使用JWT替换默认的Token
- 扩展和解析JWT的信息


**两段核心代码**

```java
/**
 * 资源服务器
 */
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .anyRequest()
                .authenticated();
    }
}
```


```java
/**
 * 认证服务器
 */
@Configuration
@EnableAuthorizationServer
public class AuthenticationServiceConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private AuthenticationManager authenticationManager;
    
    @Qualifier("userService")
    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private PasswordEncoder passwordEncoder;

    //如果用的是token生成的就是默认的乱序的字符串
    //如果使用的是jwt，那么生成的就是jwt，超级长的带有签名和信息的看起来是乱序的字符串
    //    @Qualifier("getRedisTokenStore")
    @Qualifier("getJwtTokenStore")
    @Autowired
    private TokenStore redisTokenStore;

    @Qualifier("getJwtAccessTokenConverter")
    @Autowired
    private JwtAccessTokenConverter jwtAccessTokenConverter;

    @Qualifier("getTokenEnhancer")
    @Autowired
    private TokenEnhancer jwtTokenEnhancer;

    //配置入口信息
    //使用自定义的userDetailsService
    //使用
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.userDetailsService(userDetailsService)
                .tokenStore(redisTokenStore)
                .authenticationManager(authenticationManager);


        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        List<TokenEnhancer> list = new ArrayList<>();

        list.add(jwtTokenEnhancer);
        list.add(jwtAccessTokenConverter);
        tokenEnhancerChain.setTokenEnhancers(list);

        endpoints
                .tokenEnhancer(tokenEnhancerChain)
                .accessTokenConverter(jwtAccessTokenConverter);
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {

        clients.inMemory() //token存到内存里，这样服务重启后token就没了
                .withClient("cx") //配置client-id
                .secret(passwordEncoder.encode("cx")) //配置client-secret
                .accessTokenValiditySeconds(7200) //配置token有效时间
                .authorizedGrantTypes("refresh_token", "password") //设置申请token的方式，此后只能使用设定的方式申请token
                .accessTokenValiditySeconds(3600) //设置令牌有效期3600秒
                .refreshTokenValiditySeconds(3600 * 24) //设置刷新令牌的有效期是一天
                .scopes("all", "read", "write"); //设置用户能获得的权限都有哪些

    }

}
```

使用token时需要使用redis来存token，这样的话，可以做到重启服务器，token还能继续使用

使用jwt后不需要redis来存数据，因为token是乱序的无意义的字符串，jwt里存有有用的数据，服务器可以直接根据jwt的信息读取jwt，而不需要redis来存储

- 获取jwt

![[../../020 - 附件文件夹/Pasted image 20230402225828.png|750]]

- 刷新jwt

![[../../020 - 附件文件夹/Pasted image 20230402225841.png|750]]

关于验证token是否正确，是否过期

程序会自动处理

如：token过期

![[../../020 - 附件文件夹/Pasted image 20230402225855.png|725]]

### 单点登录

> 猜测：不同服务间（不同的springboot项目）共享一个安全认证/授权中心
>
> 在淘宝登录后，打开天猫直接就是登录状态，无需再登陆天猫（淘宝，天猫显然不是一个项目，也不是一个域名/ip）

大致流程如图

![[../../020 - 附件文件夹/Pasted image 20230402225932.png|750]]

如果是在前后端分离的项目中（虽然说是前后端分离，但是下面说的客户端等都是后端java的东西，而非前端）

`应用A`和`应用B`是web客户端的模块

`认证服务器`是发放令牌的应用

`资源服务器`是service服务端的应用（疑问：资源服务器不是指校验令牌，设置认证和授权信息的服务器吗？咋变成服务端的应用了？）

==注：单点登录虽然做到了，不同服务一次登录。但是，不同服务/应用使用的令牌不是同一个，虽然他们令牌解析后的内容是一样的，但是令牌自身的那串字符串是不同的==



**开发流程**

> 下述的项目所使用的依赖，暂时不描述
>
> 下述项目使用的认证模式是`授权码模式`+`refresh_token`

1. 创建`service`模块作为认证服务器（认证模块）
2. 创建`client`模块作为应用



1. 配置认证服务器（创建启动类后，添加此配置类）

   ```java
   /**
    * 认证服务器
    * 只配置此类即可启动，当然还可配置更多其他东西
    */
   @Configuration
   @EnableAuthorizationServer
   public class SsoAuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
   	/**
   	 * 为应用配置授权模式（例子中用了两个应用，所以此处配置了两个Client的内容）
   	 */
   	@Override
   	public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
   		clients.inMemory()
   				.withClient("imooc1")
   				.secret("imoocsecrect1")
   				.authorizedGrantTypes("authorization_code", "refresh_token")
   				.scopes("all")
   				.and()
   				.withClient("imooc2")
   				.secret("imoocsecrect2")
   				.authorizedGrantTypes("authorization_code", "refresh_token")
   				.scopes("all");
   	}
   
   	/**
   	 * 配置令牌存储，此处配置了令牌签名
   	 */
   	@Override
   	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
   		endpoints.tokenStore(jwtTokenStore()).accessTokenConverter(jwtAccessTokenConverter());
   	}
   
   	/**
   	 * 配置拦截请求表达式
   	 * 没太懂这个表达式，表达式说明：只有带有指定签名令牌（签名为下述中一个类的"imooc"）的请求才能进行认证。
   	 */
   	@Override
   	public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
   		security.tokenKeyAccess("isAuthenticated()");
   	}
   	
   	@Bean
   	public TokenStore jwtTokenStore() {
   		return new JwtTokenStore(jwtAccessTokenConverter());
   	}
   	
   	@Bean
   	public JwtAccessTokenConverter jwtAccessTokenConverter(){
   		JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
           converter.setSigningKey("imooc");
           return converter;
   	}
   
   }
   ```

2. 写`client`模块，主配置类上添加注解

   ```java
   @EnableOAuth2Sso
   ```

3. 添加配置文件

```properties
# oauth2的配置
security.oauth2.client.clientId = imooc2
security.oauth2.client.clientSecret = imoocsecrect2
# 请求授权码的url
security.oauth2.client.user-authorization-uri = http://127.0.0.1:9999/server/oauth/authorize
# 请求令牌的url
security.oauth2.client.access-token-uri = http://127.0.0.1:9999/server/oauth/token
# 服务拿到令牌后，需要获取秘钥解析令牌时，向此url获取秘钥
security.oauth2.resource.jwt.key-uri = http://127.0.0.1:9999/server/oauth/token_key
# 本模块自己的相关配置
server.port = 8060
server.context-path = /client2
```

4. 添加`client`模块的启动类（分布式的话，就多写几个即可）

   ```java
   @SpringBootApplication
   @RestController
   @EnableOAuth2Sso
   public class SsoClient1Application {
   	
   	@GetMapping("/user")
   	public Authentication user(Authentication user) {
   		return user;
   	}
   
   	public static void main(String[] args) {
   		SpringApplication.run(SsoClient1Application.class, args);
   	}
   	
   }
   ```

**存在的问题**（让人不爽的地方）

1. 登陆业务不是自定义的（不是自己写的用户名密码校验流程，这个问题，实现并配置一个implents UserLoginService接口的类即可）
2. 没有配置不同url的拦截情况（比如：指定设置不拦截A应用的`xxx:8080/client/test1`，不拦截应用B的`xxx:8060/clien2/test2`）
3. 输入用户名密码成功登陆后，需要手动点击一个确定按钮（其解决方案，看教程的源代码的`SsoApprovalEndpoint``SsoSpelView`类，这两个类是`SpringSecurity`源代码中的类，需要稍加修改源代码即可，这里的修改源代码是指，自己创建类，然后将源代码完全复制过来再修改）



因此，主要问题在第二个问题上。当所需要配置的认证url不是认证服务器项目中的url时怎么配置

下面有两个解决方案

1. 每个模块都依赖一个完整的安全模块

![[../../020 - 附件文件夹/Pasted image 20230402230039.png|700]]

2. 创建一个值用于颁发令牌的`认证中心`，每个需要有认证拦截服务的应用都依赖一个`xxx-web-util`模块，此模块用于创建拦截器，配置拦截路径，解析令牌信息（即==`资源服务器`==，拥有接口的API和解析令牌的功能）



## 权限开发 

简介：

对于权限开发的一般应用场景和需求

![[../../020 - 附件文件夹/Pasted image 20230402230053.png|600]]

**业务系统的简单的权限**

### quick start（静态权限设置）

- 设置权限

```java
 .and()
     .authorizeRequests() //对请求做授权
     .antMatchers(
     "/myLogin.html",
     "/code/*",
     "/qqLogin/callback.do",
     "/oauth/default.aspx",
     "/oauth/*").permitAll() //匹配到指定的页面并放行/或者说选择指定的页面放行
     .antMatchers("/me2").hasRole("ADMIN")
     .anyRequest() // 所有请求
     .authenticated() //都需要被认证
```

- 给用户权限

```java
//在UserService（继承UserDetailsService）
        return new User(username,
                passwrd,
                true, true,true,true,
                AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_ADMIN,vip"));
```

> 关于设置权限时用"ADMIN"，而给用户权限时使用"ROLE_ADMIN"这个问题，下面再说
>
> 原因：在源码里程序会给ADMIN添加上ROLE_的前缀

存在的问题

1. 对于restful风格的url来说这样设置权限，只是指定了访问某个url需要的权限，而没有指定这个有权限限制的url是哪种请求方式？
2. 带有占位符的url怎么设置？

解决方案

```java
 .antMatchers(HttpMethod.GET, "/me2/*").hasRole("ADMIN")
//指定请求方式
//使用*表示占位符
```



### 源码执行流程

在quick start中，当程序执行时的流程是怎样的？

![[../../020 - 附件文件夹/Pasted image 20230402230115.png|750]]

权限表达式有哪些

![[../../020 - 附件文件夹/Pasted image 20230402230127.png|725]]

一般权限表达式每次使用一个

如果要对某个url使用多个权限限制，使用下面这种形式

```java
.access("hasRole('ADMIN') and hasIpAddress('127.0.0.1')")
```

优化SpringSecurity核心配置类的配置，暂时不练，直接看源代码练

![[../../020 - 附件文件夹/Pasted image 20230402230142.png|750]]

**内管系统的自定义权限**

权限和角色一般都放到数据库里

这就涉及到了数据库里的数据结构

### 动态权限设置

**通用RBAC（role-based access controller）数据模型**

![[../../020 - 附件文件夹/Pasted image 20230402230156.png|725]]

实现以上这些表其实都是与SpringSecurity没有关系的

有关系的是资源表里的url

要想把数据模型与security建立起关系，需要



## Security与分布式

> 下面将说明SpringSecurity使用jwt在分布式环境下认证服务和资源服务分离使用
>
> [参考博客](https://my.oschina.net/wotrd?q=security)

`认证服务器` 用于发放，校验令牌

`资源服务器` 拥有资源（暂时将接口视为资源）



### 认证服务器

创建项目`security-server`

1. 添加认证服务器注解`@EnableAuthorizationServer`
2. `UserDetailsService`接口实现的业务逻辑，用于校验`username`，`password`
3. 放行`/oauth/check_token`，用于对外暴露校验token的地址（默认关闭此地址，放行配置在`ResourceServerConfigurerAdapter`的实现类中配置）



### 资源服务器

创建项目`resource-client`

1. 添加资源服务器注解`@EnableResourceServer`
2. 配置接口权限，在`WebSecurityConfigurerAdapter`的实现中配置
3. 配置文件中指定  校验令牌的地址（对应上述创建认证服务器的第3步）



遇到的问题

上下文：设置了签名方法，没有设置增强器

问题：从认证服务器获取令牌后，重启认证服务器，再次用令牌发送请求后，返回值为令牌失效的信息

猜测：没有设置自定以增强器或签名方法并没有生效，导致认证服务器再次解析令牌时出错



# 问题

**遇到了在分布式环境下，SpringSecurity认证如何拦截对其他服务的请求的问题**

单点登录可能能解决这个问题，所以先学单点登录。如果不能解决问题，再找博客学SpringSecurity在分布式中的应用



解决方案

1. 创建一个可独立运行的模块作为认证服务器，可实现单点登录。但是会导致所有接口都需要认证，且无法自定义放行那些url
2. 创建一个供其他模块依赖的模块，模块有完整的认证逻辑，且对外提供自定义为指定接口设置权限的接口。但是这将无法做到单点登录，且导致所有需要有认证的服务都要依赖此模块。这和微服务，分布式的初衷相悖（服务专业化，逻辑服务只提供服务的业务逻辑，认证服务才做请求的拦截和认证。我是这么想的）



==认证服务器是颁发令牌，资源服务器是校验令牌的吗？？？？？资源服务器是不是就是消费者/客户端==



**认证服务器中配置app的密码加密还是不加密**

有时加密有时不加密

（单机应用下）原设置为加密，请求'/oauth/token'后返回401，去掉加密后可正常访问并获取返回值中的令牌（但是之前都是加密的，之前并没有出现过问题）

（分布式下）加密，能正常访问'/oauth/token'



# todo

实现服务重启后，只要jwt没有过期，认证中心依旧能识别并解析jwt

问题：如上述，但是，原程序能做到服务重启，原jwt依旧能解析。现分布式项目中却不能

方案：先尝试原单机项目是否还能重启服务还能解析原令牌，如果能，对比新旧项目。如果不能再说

结果：单机应用中不会出现此问题，分布式会出现。两者验证token时执行的方法不同，前者不知道，后者执行`'/oauth/check_token'`它将传递的token字符串传到一个tokenStore中，其里面有个map，记录String和对应的令牌该map名为InMemberXX（反正就是和内存有关系）也就是说认证服务器会将token存入内存，重启认证服务器后令牌自然会消失。无法通过这个map.get获得到令牌，结果就是抛出令牌错误的信息



问题：无法实现密码模式下的单点登录。

期望效果：

1. 在浏览器访问client1的接口
2. 返回没有权限的响应，并要求输入用户名和密码
3. 正确输入后，后端返回token，并拿token访问原目标接口
4. 在浏览器访问client2的接口，因为单点登录功能的实现，无需手动输用户名密码。自动再次请求获取token，并拿token成功访问接口

现状（使用postman，不是用浏览器的url地址栏访问的，因为浏览器不方便添加token）：

1. 访问client1的接口，直接报错，说没有权限，不会自动跳转要求输入用户名密码
2. 手动向认证中心发送请求获取token
3. 访问client1的接口时，手动添加token
4. 访问client2的接口时，不手动添加token，会报错（如上述1）。且使用访问client1接口时用的token也能访问成功



问题：使用同一个token可以访问不同的客户端的所有接口（并不是应该是client1发送请求，认证中心检查token中的id和secret和client1的id和secret是否匹配。但是认证中心并没有检查）



# 学习路线

2. 权限，这一块，视频里好像讲的比较简单，甚至没有使用codesheep代码里使用的权限注解，要结合codesheep的代码学权限
4. 第三方登录

## 没有掌握的类/对象

- String.format(str, ..) 把字符串格式化，有后面的可变参数填充字符串里的%d  %s  ......

- ServletWebRequest

- ServletRequestUtils.getIntParam(request.getRequest(),xx,xx);

- 接口InitializingBean

- StringUtils.splitByWholeSeparatorPreserveAllTokens() 

- AntPathMatcher

  因为有些路径带有占位符，使用equals检查时由于占位符是动态的不方便比对，所以使用这个类创建的对象
  
- @EnableSocial

- ObjectMapper 类  有把json数据转换成自定义对象的功能
  
  objectMapper.readValue(result, QQUserInfo.class);
  
- response.getwriter.write("XXX");

  ```java
  //在返回值是void的方法里返回自定义结果集
  response.setContentType("application/json;charset=UTF-8");
  response.getWriter().write(objectMapper.writeValueAsString(token));
  ```

