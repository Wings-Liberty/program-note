#还没有复习 

# Tomcat 的目录结构

- bin 存放 Tomcat 服务器的可执行程序
- conf 存放 Tocmat 服务器的配置文件
- lib 存放 Tomcat 服务器的 jar 包
- logs 存放 Tomcat 服务器运行时输出的日记信息
- temp 存放 Tomcdat 运行时产生的临时数据
- webapps 存放部署的 Web 工程
- work 是 Tomcat 工作时的目录，存放 Tomcat 运行时 jsp 翻译为 Servlet 的源码，和 Session 钝化的目录

```
.
├── Catalina
│   └── localhost
├── catalina.policy
├── catalina.properties
├── context.xml
├── jaspic-providers.xml
├── jaspic-providers.xsd
├── logging.properties
├── server.xml 				// tomcat 服务端总配置文件
├── tomcat-users.xml
├── tomcat-users.xsd
└── web.xml					// 每个部署在 tomcat 中的项目都应有一个 web.xml，tomcat 内置的 web.xml 为项目的 web.xml 提供缺省值
```



# 访问 Tomcat

访问地址：`http://ip:port/虚拟路径/项目内的路由`

“虚拟路径” 也可以称为 “工程路径”



当我们在浏览器地址栏中输入访问地址如下： 

`http://ip:port/ `====>>>> 没有工程名的时候，默认访问的是 ROOT 工程。 

当我们在浏览器地址栏中输入的访问地址如下： 

`http://ip:port/工程名/` ====>>>> 没有资源名，默认访问 index.html 页面



# IDEA 下的动态 Web 工程和 Tomcat



## Web 工作目录

```
L 项目根目录
	L src				// 存放 Java 源代码
	L web				// 存放静态资源文件
		L WEB-INF		// 受保护的目录，浏览器无法直接通过路由访问到此目录中的内容
			L lib		// 第三方 Jar 包
			web.xml		// 整个动态 web 工程的部署描述文件，再次配置 web 组件：Servlet、Filter、Listener、Session...
```

没用 Maven 并使用 lib 目录保存 Jar 包表示这，项目所需的所有 Jar 包必须由项目自己持有，即依赖的 Jar 包会直接导致项目文件很大





## IDEA 动态部署 war 包

在 IDEA 中配置创建 Artifacts 并设置为 war 包格式，直接运行项目时，IDEA 会自动打 war 包并传至 IDEA 指定好的 Tomcat 的 webapps 目录中，并启动 Tomcat





# Servlet 技术

## 什么是 Servlet

1. Servlet 是 JavaEE 规范之一。规范就是接口
2. Servlet 就 JavaWeb 三大组件之一。三大组件分别是：Servlet 程序、Filter 过滤器、Listener 监听器
3. Servlet 是运行在服务器上的一个 Java 小程序，它可以接收客户端发送过来的请求，并响应数据给客户端。 



## 如何实现 Servlet 程序

1. 编写一个类并实现 Servlet 接口（但通常是继承 HttpServlet，而不是直接实现 Servlet 接口）

2. 实现 service 方法，处理请求，并响应数据

   ```java
   public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {}

3. 到 web.xml 中配置 servlet 程序和访问地址映射关系。即为一个 servlet 程序配置一个路由

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app ...>
       <!-- servlet 标签给 Tomcat 配置 Servlet 程序 -->
       <servlet> 
           <!--servlet-name 标签 Servlet 程序起一个别名（一般是类名） --> 
           <servlet-name>HelloServlet</servlet-name> 
           <!--servlet-class 是 Servlet 程序的全类名-->
       	<servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>
       </servlet>
       <!--servlet-mapping 标签给 servlet 程序配置访问地址-->
   	<servlet-mapping> 
   		<!--servlet-name 标签的作用是告诉服务器，我当前配置的地址给哪个 Servlet 程序使用-->
   		<servlet-name>HelloServlet</servlet-name>
   		<!--
   			url-pattern 标签配置访问地址
   			/ 斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径
   			/hello 表示地址为：http://ip:port/工程路径/hello 
   		-->
   		<url-pattern>/hello</url-pattern> 
       </servlet-mapping> 
   </web-app>
   ```

   

## Servlet 的生命周期

1. 执行 Servlet 构造器方法，执行 init 初始化方法。是在第一次访问，的时候创建 Servlet 程序会调用
2. 每次通过 url 访问 Servlet 时执行 service 方法
3. 在 web 工程停止的时候调用，执行 destroy 销毁方法



## 接口程序

Servlet 要求一个完成的接口程序应该包括：1 个 Servlet，1 个 ServletConfig，1 个所有 Servlet 共享的全局 ServletContext

- Servlet：仅用于处理请求。是无状态的
- ServletConfig：可以获取 Servlet 程序的别名 servlet-name 的值，获取初始化参数 init-param，获取 ServletContext 对象。即保存 Servlet 的一些**初始**状态
- ServletContext：上下文。1 个项目仅能有 1 个上下文对象，有以下功能
  - 获取 web.xml 中配置的上下文参数 context-param 
  - 获取当前的工程路径，格式: /工程路径 
  - 获取工程部署后在服务器硬盘上的绝对路径
  - 像 Map 一样在程序运行时存取数据

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app ...>
    <!--context-param 是 ServletContext 上下文参数(它属于整个 web 工程)--> 
    <context-param> 
        <param-name>username</param-name>
        <param-value>context</param-value>
    </context-param>
    <!--context-param 是 ServletContext 上下文参数(它属于整个 web 工程)-->
    <context-param> 
        <param-name>password</param-name>
        <param-value>root</param-value> 
    </context-param>
    
    <servlet> 
        <servlet-name>HelloServlet</servlet-name> 
    	<servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>
        
        <!--init-param 是初始化参数-->
        <init-param>
            <!--是参数名--> 
            <param-name>username</param-name> 
            <!--是参数值--> 
            <param-value>root</param-value>
        </init-param>
        <!--init-param 是初始化参数--> 
        <init-param> 
            <!--是参数名-->
            <param-name>url</param-name>
            <!--是参数值-->
            <param-value>jdbc:mysql://localhost:3306/test</param-value>
        </init-param>
    </servlet>
    
	<servlet-mapping> 
		<servlet-name>HelloServlet</servlet-name>
		<url-pattern>/hello</url-pattern> 
    </servlet-mapping> 
</web-app>
```



# 转发和重定向

在 HTTP / Web 应用中，客户端就是指浏览器

## 转发

请求转发是指，服务器收到请求后，从一次资源跳转到另一个资源的操作叫请求转发

- 客户端发送一次请求，服务器发送若干请求。客户端地址栏不发生变化
- 转发只能跳转到本站点的资源



## 重定向

指客户端给服务器发请求，然后服务器告诉客户端说。我给你一些地址。你去新地址访问。叫请求重定向（因为之前的地址可能已经被废弃）

- 客户端发送若干次请求，每次发送请求时客户端的地址栏都会发生变化



重定向是客户端行为，转发是服务器端行为



# Cookie 和 Session



## Cookie 定义

- Cookie 是服务器通知客户端保存键值对的一种技术
- 客户端有了 Cookie 后，每次请求都发送给服务器
- 每个 Cookie 的大小不能超过 4kb





## Cookie 的生命周期

**创建**

客户端会自行把收到的响应头的 Set-Cookie 中的 kv 保存下来

客户端每次发送请求时，把 “所有” Cookie 的 kv 都放在请求头的 Cookie 里

**有效期**

Cookie 的声明周期由服务端创建 Cookie 时指定。过期后删除，而非关闭浏览器时就立即删除

**有效路径**

Cookie 的 path 属性可以有效的过滤哪些 Cookie 可以发送给服务器。哪些不发。 

path 属性由服务端创建 Cookie 时指定

```
CookieA path=/工程路径
CookieB path=/工程路径/abc 

请求地址如下：
http://ip:port/工程路径/a.html 
CookieA 发送 
CookieB 不发送 

请求地址如下：
http://ip:port/工程路径/abc/a.html 
CookieA 发送
CookieB 发送
```



## Session 的定义

Session 就是会话。它是用来保存客户端和服务器一次会话连接中的状态信息

客户端和服务器的 1 个 会话包含 1 个 Session ID 和 1 个保存在服务端的 Session，在服务端 Session 池保存了一堆 Session，Session 用于保存 1 个会话中创建的数据的集合





## Session 的生命周期

Session 的有效期可由 web.xml 指定，也可在程序中用代码控制。下面是用 web.xml 方式指定有效期

```xml
<!--表示当前 web 工程。创建出来的所有 Session 默认是 20 分钟超时时长--> 
<session-config>
    <session-timeout>20</session-timeout> 
</session-config>
```

Session 的有效期是指 1 个会话内两次请求的最大时间间隔



**Session 的创建**

客户端无 Cookie 并首次发送请求时，服务端会创建 Session 和 Session ID，在响应头的 Set-Cooike 中用 JSESSIONID 指明本次会话的 id 号，以便于服务端查找本次会话的相关数据。客户端收到后解析并识别出 Session ID，再保存下来



# Filter 过滤器

常用于权限验证，日志记录，切面处理，捕捉异常

## Filter 定义

三大组件之一，用于拦截请求，过滤响应

```java
public interface Filter {
	void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;
}
```



````java
@Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
    // pre
	filterChain.doFilter(servletRequest,servletResponse);
    // post
}
````

在 web.xml 中配置 Filter 的拦截地址

```xml
<!--filter 标签用于配置一个 Filter 过滤器--> 
<filter> 
    <!--给 filter 起一个别名-->
    <filter-name>AdminFilter</filter-name>
    <!--配置 filter 的全类名-->
    <filter-class>com.atguigu.filter.AdminFilter</filter-class>
</filter>

<!--filter-mapping 配置 Filter 过滤器的拦截路径-->
<filter-mapping>
    <!--filter-name 表示当前的拦截路径给哪个 filter 使用-->
    <filter-name>AdminFilter</filter-name> 
    <!--
		url-pattern 配置拦截路径 / 表示请求地址为：http://ip:port/工程路径/ 
		映射到 IDEA 的 web 目录 /admin/* 
		表示请求地址为：http://ip:port/工程路径/admin/* 
	--> 
    <url-pattern>/admin/*</url-pattern>
    <!-- 同一个 filter-mapping 可配置多个拦截路径 -->
    <url-pattern>/admin/*</url-pattern>
</filter-mapping>
```

过滤器链中多个 Filte 的执行顺序和 web.xml 中配置的顺序相同



PS：Filter 过滤器它只关心请求的地址是否匹配，不关心请求的资源是否存在。所以过滤地址不存在，程序也不会报错



Filter 也是无状态的，每个 Filter 也都有一个 FilterConfig，但貌似已被弃用（因为 Tomcat 没在任何地方实现它并使用它）



## **FilterChain** 过滤器链

多个过滤器执行的特点

- 所有 filter 和目标资源默认都执行在同一个线程中
- 多个 filter 共同执行的时候，它们都使用同一个 request 对象



# Tomcat 的错误页面

在 web.xml 中设置指定异常状态码映射到指定的页面，当遇到要返回这些状态码时，返回这些错误页面

```xml
<!--error-page 标签配置，服务器出错之后，自动跳转的页面-->
<error-page>
    <!--error-code 是错误类型-->
    <error-code>500</error-code> 
    <!--location 标签表示。要跳转去的页面路径-->
    <location>/pages/error/error500.jsp</location>
</error-page> 

<!--error-page 标签配置，服务器出错之后，自动跳转的页面--> 
<error-page> 
    <!--error-code 是错误类型--> 
    <error-code>404</error-code> 
    <!--location 标签表示。要跳转去的页面路径-->
    <location>/pages/error/error404.jsp</location> 
</error-page>
```

