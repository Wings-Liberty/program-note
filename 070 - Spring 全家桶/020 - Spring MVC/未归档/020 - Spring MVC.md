#还没有复习 

# 使用 Spring MVC

SpringMVC 是遵守 Servlet 标准的 web 框架，因此使用 SpringMVC 就需要将其放到 web 容器中

使用 SringMVC 的流程分为：把 SpringMVC 整合到 Web 容器中，用 SpringMVC 写业务逻辑（接口），配置 MVC



**写接口**

`@EnableWebMvc`

```java
@Controller
public class UserController {
    @GetMapping("/user")
    public User getUser() {
        return new User();
    }
}
```



**把 SpringMVC 提供的 Servlet 添加到 Web 容器中**

SpringMVC 用 Spring IOC 作为依赖注入的框架就要启动一个 IOC 容器，同时提供 Servlet

启动 IOC 容器的方式有很多种：Listener 启动，注解启动，Servlet 启动，xml 配置文件启动。SpringMVC 和 SpringBoot 均采用 Listener 启动

SpringMVC 提供的 DispatcherServlet，其父类持有 IOC 容器，所以就要在配置 DispatcherServlet 时就要配置一个 IOC 容器

在 web.xml 里添加 DispatcherServlet 的配置，并用 init-param 方式指定 IOC 容器类型及其所需要的 IOC 配置文件

```xml
<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.itranswarp.learnjava.AppConfig</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>
```

可用内嵌式的 Tomcat 启动 SpringMVC

```java
public static void main(String[] args) throws Exception {
    Tomcat tomcat = new Tomcat();
    tomcat.setPort(Integer.getInteger("port", 8080));
    tomcat.getConnector();
    Context ctx = tomcat.addWebapp("", new File("src/main/webapp").getAbsolutePath());
    WebResourceRoot resources = new StandardRoot(ctx);
    resources.addPreResources(
        new DirResourceSet(resources, "/WEB-INF/classes", 
                           new File("target/classes").getAbsolutePath(), 
                           "/"));
    ctx.setResources(resources);
    tomcat.start();
    tomcat.getServer().await();
}
```



**配置 MVC**

SpringMVC 提供了开箱即用的`@EnableWebMvc`注解，它提供了大量的默认配置（向容器中`DelegatingWebMvcConfiguration`类型的 bean）

如果要高度定制 MVC，则需自行添加一个实现`WebMvcConfigurer`接口的 bean

如果仅修改少数配置，则可自行添加一个继承`WebMvcConfigurationSupport`或`DelegatingWebMvcConfiguration`类的 bean

MVC 中有什么组件，这里不做介绍



# 支持 json

添加接口对 json 的支持，`com.fasterxml.jackson.core:jackson-databind`



- `@ResponseBody`作用在方法上或类上，表示对接口返回值不做处理
- `@RequestBody`作用在方法参数列表的参数上，表示此参数来自请求体。SpringMVC 自行把请求体中的 json 数据反序列化为响应的数据类型
- `RestController`相当于`@Controller`和`@ResponseBody`的组合



# SpringMVC 集成 Filter

- 如果仅在`web.xml`中声明`Filter`，则`Filter`中通过`@Autowired`注入的 bean 将不会生效，因为这个`Filter`是由 Web 容器创建的，不是 IOC 创建的
- 如果仅在`Filter`实现类上加`@Component`，则`Filter`将只是一个普通的 bean，Web 容器不会将其视为一个`Filter`



**解决方式**

在`web.xml`中添加`DelegatingFilterProxy`。其会在 init 时自行获取 IOC 容器，并从中获取指定`Filter`，采用代理模式代理`Filter`

```
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐ ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┐
| ┌─────────────────────┐ | |    ┌───────────┐   │
│ │DelegatingFilterProxy│─│─│─ ─>│AuthFilter │   |
| └─────────────────────┘ | |    └───────────┘   │
│ ┌─────────────────────┐ │ │    ┌───────────┐   |
| │  DispatcherServlet  │─ ─ ─ ─>│Controllers│   │
│ └─────────────────────┘ │ │    └───────────┘   │
│    Servlet Container    │ │  Spring Container  |
└─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

```xml
<filter>
    <filter-name>basicAuthFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <!-- 指定 Bean 的名字，如果没有指定默认代理 IOC 中和 filter-name 同名的 bean -->
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>authFilter</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>authFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



# Interceptor 拦截器

Interceptor 仅能拦截 Controller 的接口方法，即仅对 Controller 的接口方法做了 AOP

```xml
       │   ▲
       ▼   │
     ┌───────┐
     │Filter1│
     └───────┘
       │   ▲
       ▼   │
     ┌───────┐
     │Filter2│
     └───────┘
       │   ▲
       ▼   │
┌─────────────────┐
│DispatcherServlet│<───┐
└─────────────────┘    │
 │              ┌────────────┐
 │              │ModelAndView│
 │              └────────────┘
 │ ┌ ─ ─ ─ ─ ─ ─ ─ ─ ┐ ▲
 │    ┌───────────┐    │
 ├─┼─>│Controller1│──┼─┤
 │    └───────────┘    │
 │ │                 │ │
 │    ┌───────────┐    │
 └─┼─>│Controller2│──┼─┘
      └───────────┘
   └ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

向容器中添加 Interceptor 需要实现 HandlerInterceptor 通过 WebMvcConfigurer 并向 mvc 注册拦截器





# 跨域访问

- 有`@CrossOrigin`标记的接口方法会允许跨域访问
- 在 WebMvcConfigurer 中重写 addCorsMappings 方法配置全局跨域策略（推荐）
- 在 web.xml 中配置 CorsFilter



# 异步处理

暂跳，因为没理解 Java 中的异步处理的实质是什么，所以会用也没用



# WebSocket

首先需要引入新的依赖

- org.apache.tomcat.embed:tomcat-embed-websocket
- org.springframework:spring-websocket

然后添加 ws 配置类：WebSocketConfigurer

为 ws 配置路由和 wsHandler 的映射，可为 ws 添加拦截器（ws 的拦截器和 http 接口的拦截器不共用。http 接口的拦截器的拦截范围仅为接口方法）

```java
@Bean
WebSocketConfigurer createWebSocketConfigurer(
        @Autowired ChatHandler chatHandler,
        @Autowired ChatHandshakeInterceptor chatInterceptor)
{
    return new WebSocketConfigurer() {
        public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
            // 把URL与指定的WebSocketHandler关联，可关联多个:
            registry.addHandler(chatHandler, "/chat").addInterceptors(chatInterceptor);
        }
    };
}
```

wsHandler 的实现类必须实现 ChatHandler 接口



ws 中每个 WebSocketSession 实例对应一个客户端 / Server 和 Client 的会话。可通过 WebSocketSession 实例和 Client 通信

所以通常用 map 把 WebSocketSession 保存起来
