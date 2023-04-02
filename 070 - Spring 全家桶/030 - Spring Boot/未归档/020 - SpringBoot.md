#还没有复习 

# Hello World



## Quick Start

1. 拆功能键`maven`项目，导入`maven`依赖

   ```xml
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>2.2.2.RELEASE</version>
   </parent>
   
   <dependencys>
       <!--上述的parent使用的依赖，是Springboot各种依赖的版本仲裁中心（如果仲裁中心中指定依赖的版本，那么下面的依赖无需指定版本默认使用仲裁中心指定的版本。好处：不需要自己解决版本依赖不兼容的问题，因为版本仲裁中心的依赖版本都是兼容的）-->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
   </dependencys>
   ```

2. 创建入口程序

   ```java
   package com.cx;
   
   @SpringBootApplication
   public class BootApplication {
       public static void main(String[] args) {
               SpringApplication.run(BootApplication.class ,args);
       }
   }
   ```

3. 创建`HelloController.java`

   ```java
   package com.cx.controller;
   
   @RestController
   public class HelloController {
       @GetMapping("/hello")
       public String hello(){
           return "hello";
       }
   }
   ```

4. 创建`application.yml`配置文件

   ```yaml
   server:
     port: 8080
   ```

5. 测试。运行程序，在浏览器地址栏上输入`localhost:8080/hello`。返回"hello"即为成功

这样，一个 网关**接口** 就写完了



## SpringBoot都做了什么

`@SpringBootApplication`：使用此注解的类被视为SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class), @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication
```

重点在于`@SpringBootConfiguration`和``@EnableAutoConfiguration`



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Configuration
public @interface SpringBootConfiguration
```

`@SpringBootConfiguration`此注解中有`@Configuration`，那么SpringBoot的启动类也可视为一个配置类，且是SpringBoot的**主配置类**



```java
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration
    
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage
```

`@EnableAutoConfiguration`向容器中添加了两个类

- `AutoConfigurationPackages.Registrar` 将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器
- `AutoConfigurationImportSelector`将一些必要的后置处理器，前端控制器，视图解析器等自动导入容器。添加新的Spring的相关依赖时，此类也会自动扫描这些依赖中的组件并添加到容器中（例如，添加了Spring MVC的依赖后，不需要再配置文件中手动向容器中添加前端控制器，视图解析器等组件。此类会自动帮你完成这些事）



### AutoConfigurationPackages.Registrar

**先分析以下这个类能干什么**（能向容器中添加`BeanDefinition`对象）

![[../../../020 - 附件文件夹/Pasted image 20230402122501.png|500]]

如果忘了`ImportBeanDefinitionRegistrar`是干什么，可以查看Spring注解式开发中

使用`@Import`+`ImportBeanDefinitionRegistrar`的实现类 的方式导入组件



再分析这个类干了什么（将启动类所在的包和子包下的组件的`BeanDefinition`对象添加到容器中）

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        register(registry, new PackageImport(metadata).getPackageName());
    }

    ...

}
```

`registerBeanDefinitions`方法扫描启动类所在的包及其自包下的所有的类，并将扫描到的组件注册到容器中（省去了自己写`@ComponentScan`扫描组件的步骤）

`new PackageImport(metadata).getPackageName()`的返回值为启动类所在的包的包名



### AutoConfigurationImportSelector

**先分析这个类能干什么**（能像容器中添加bean）

![[../../../020 - 附件文件夹/Pasted image 20230402122515.png]]

如果忘了`ImportSelector`是干什么，可以查看Spring注解式开发中

使用`@Import`+`ImportSelector`的实现类 的方式导入组件


**再分析这个类干了什么**（向容器中添加bean）

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
        .loadMetadata(this.beanClassLoader);
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
                                                                              annotationMetadata);
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```

给容器中导入非常多的自动配置类（xxxAutoConfiguration）

例如：`JestAutoConfiguration`，`RestClientAutoConfiguration`这些配置类将再导入其他组件


==问题：给上述方法打断点后调试，程序并没有执行该方法==



# 使用配置文件给组件中的变量赋值

### @Value

使用此注解的使用已在“Spring注解式开发”中写过

### @ConfigurationProperties

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Component
@ConfigurationProperties(prefix = "person")
public class User {
    private String name;
}
```

`application.yml`

```yaml
person:
  name: 'cat'
```

测试

```java
@Autowired
User user;

@Test
public void test(){
    System.out.println(user); // name=cat
}
```



### @PropertySource

和@Value一样，使用此注解的使用已在“Spring注解式开发”中写过


# @EnableConfigurationProperties

向此注解中传入有`@ConfigurationProperties`注解的类对象可将此类作为配置类注入容器

或者直接在有`@ConfigurationProperties`注解的类上添加`@Component`注解


在springboot中  XXXAutoConfiguration类中使用`@EnableConfigurationProperties`的方式注入配置文件，而不是使用`@Component` 


比如 yml 中配置了`config.info`

读取其值仅需要一个 `@ConfigurationProperties(Config.class)`

```java
@ConfigurationProperties(prefix="config")
public class Config{
    private String info;
    // 省略 info 的 setter 方法
}
```

其作用是创建一个 Config 对象并把配置文件读进去，再放到容器中


# Web开发

## 嵌入式Servlet容器


### 向容器中注册三大组件

三大组件：`Servlet`，`Listener`，`Filter`

ServletRegistrationBean

```java
//注册三大组件
@Bean
public ServletRegistrationBean myServlet(){
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet(),"/myServlet");
    return registrationBean;
}

```

FilterRegistrationBean

```java
@Bean
public FilterRegistrationBean myFilter(){
    FilterRegistrationBean registrationBean = new FilterRegistrationBean();
    registrationBean.setFilter(new MyFilter());
    registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
    return registrationBean;
}
```

ServletListenerRegistrationBean

```java
@Bean
public ServletListenerRegistrationBean myListener(){
    ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
    return registrationBean;
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





# 启动配置原理

几个重要的事件回调机制

配置在META-INF/spring.factories



**ApplicationContextInitializer**

**SpringApplicationRunListener**



只需要放在ioc容器中

**ApplicationRunner**

**CommandLineRunner**



启动流程：



```java
@SpringBootApplication
public class BootApplication {
    public static void main(String[] args) {
		SpringApplication.run(BootApplication.class ,args);
    }
}

public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
	return run(new Class<?>[] { primarySource }, args);
}

public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    // 创建SpringApplication对象并调用run方法	
	return new SpringApplication(primarySources).run(args);
}
```



## **1、创建SpringApplication对象**



```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    
   // 参数resourceLoader为null，即赋空值
   this.resourceLoader = resourceLoader;
   Assert.notNull(primarySources, "PrimarySources must not be null");
    
   // 将启动类封装到set中赋给this.primarySources
   this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    
   // 这是个枚举类，一般这里值为 SERVLET  （servlet） 
   this.webApplicationType = WebApplicationType.deduceFromClasspath();
    
   // setInitializers方法，将参数封装到ArrayList中赋值给此 SpringApplication的this.initializers
   // getSpringFactoriesInstances方法，从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer的实现类，再将类对象封装到set中（这里的META-INF/spring.factories不是特指spring-boot-autoconfiguration而是所有META-INF/spring.factories）
   setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
   
   // 和上面相似，将ApplicationListener的实现类封装到set中，赋值给this.listeners
   setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    
   // 获取当前方法栈中的所有方法栈，将栈中方法名为main的方法所在的类赋值给this.mainApplicationClass（将其作为主配置类）
   this.mainApplicationClass = deduceMainApplicationClass();
}
```



## 2、运行run方法

运行`SpringApplication`的`run`方法

```java
public ConfigurableApplicationContext run(String... args) {
    
   	// StopWatch 译：秒表。用于记录程序启动和停止时间 
   	StopWatch stopWatch = new StopWatch();
   	stopWatch.start();
    
   	// 声明上下文对象。ConfigurableApplicationContext是个接口
   	ConfigurableApplicationContext context = null;
    // 不重要
   	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    // 与awt相关配置，不重要
	configureHeadlessProperty();
    
	// 从类路径下META-INF/spring.factories，获取所有SpringApplicationRunListener实现类
  	SpringApplicationRunListeners listeners = getRunListeners(args);
	// 执行所有SpringApplicationRunListener.starting()方法
  	listeners.starting();
    
   	try {
		// 封装命令行参数
      	ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
		
     	// 如果this.environment!=null就直接return。
     	// 如果为null就创建一个，并设置激活的profile和参数解析器，这里使用的是解析${}表达式的解析器
     	// 创建环境完成后调用参数listeners 遍历执行listener的environmentPrepared()；表示环境准备完成
		ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
      
        // 不重要
		configureIgnoreBeanInfo(environment);
     	// 执行此句，在控制台打印spring的图标和版本号
        Banner printedBanner = printBanner(environment);
       
     	// 创建ApplicationContext，根据this.webApplicationType的值创建不同类型的上下文
        // this.webApplicationType是SERVLET 创建AnnotationConfigServletWebServerApplicationContext
        // BeanDefiition将在这里创建，详情请看  启动配置原理 的 补充
   	   	context = createApplicationContext();
       
        // 从类路径下META-INF/spring.factories，获取所有SpringBootExceptionReporter实现类
      	exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[] { ConfigurableApplicationContext.class }, context);
       
       	// 将environment保存到上下文中
        // postProcessApplicationContext(context); 调用context的父类GenericApplicationContext的成员变量this.beanFactory为beanFactory set一些成员变量
       	// applyInitializers(); 回调之前保存的所有的ApplicationContextInitializer的initialize方法
      	// 回调所有的SpringApplicationRunListener的contextPrepared()；
        // load(context, sources.toArray(new Object[0])); sources里是主启动类，为主启动类创建BeanDefinition
      	prepareContext(context, environment, listeners, applicationArguments, printedBanner);
       	//prepareContext运行完成以后回调所有的SpringApplicationRunListener的contextLoaded（）；
       
      	// 执行容器的refresh方法
      	refreshContext(context);
        
      	// spring2.0后此方法为空方法
      	afterRefresh(context, applicationArguments);
        
        // 停止计时
      	stopWatch.stop();
        
      	if (this.logStartupInfo) {
        	 new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
      	}
        
        // 执行所有SpringApplicationRunListener的started方法
       	listeners.started(context);
        
        // 从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调。调试时容器中并没有可用的ApplicationRunner和CommandLineRunner
		callRunners(context, applicationArguments);
	}
    catch (Throwable ex) {
        ...
    }

    try {
        // 执行所有SpringApplicationRunListener的running方法
        listeners.running(context);
    }
    catch (Throwable ex) {
        ...
    }
    return context;
}
```



关于`SpringApplication`中的成员变量

`this.environment`包含了默认的和当前激活的Profiles。还有propertyResolver解析器，这里解析的是`${}`

![[../../../020 - 附件文件夹/Pasted image 20230402122600.png|375]]

## 3、事件监听机制

配置在META-INF/spring.factories

**ApplicationContextInitializer**

```java
public class HelloApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        System.out.println("ApplicationContextInitializer...initialize..."+applicationContext);
    }
}

```

**SpringApplicationRunListener**

```java
public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {

    //必须有的构造器
    public HelloSpringApplicationRunListener(SpringApplication application, String[] args){

    }

    @Override
    public void starting() {
        System.out.println("SpringApplicationRunListener...starting...");
    }

    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        Object o = environment.getSystemProperties().get("os.name");
        System.out.println("SpringApplicationRunListener...environmentPrepared.."+o);
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextPrepared...");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextLoaded...");
    }

    @Override
    public void finished(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("SpringApplicationRunListener...finished...");
    }
}

```

配置（resource目录下创建META-INF/spring.factories）

```properties
org.springframework.context.ApplicationContextInitializer=\
com.atguigu.springboot.listener.HelloApplicationContextInitializer

org.springframework.boot.SpringApplicationRunListener=\
com.atguigu.springboot.listener.HelloSpringApplicationRunListener
```





只需要放在ioc容器中

**ApplicationRunner**

```java
@Component
public class HelloApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("ApplicationRunner...run....");
    }
}
```



**CommandLineRunner**

```java
@Component
public class HelloCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner...run..."+ Arrays.asList(args));
    }
}
```





## 补充

运行`SpringApplication`的`run`方法时使用以下语句创建`AnnotationConfigServletWebServerApplicationContext`的实例

```java
context = createApplicationContext();
```

使用反射调用`AnnotationConfigServletWebServerApplicationContext`的构造方法创建实例

```java
public AnnotationConfigServletWebServerApplicationContext() {
    // 创建对象的同时会为一些后置处理器创建BeanDefinition，再put到DefaultListableBeanFactory的成员变量this.beanDefinitionMap中
	this.reader = new AnnotatedBeanDefinitionReader(this);
    // 在ClassPathBeanDefinitionScanner的父类中执行this.includeFilters.add(new AnnotationTypeFilter(Component.class));
    // 这个过滤器是用来检查类和类上注解的注解解析器。会在扫描组件时用于检查类是否有@Component（@Controller等注解自带有@Component，所以8也会被扫描进来）
	this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

