#还没有复习 

[本地磁盘 Tomcat 专题.pdf](D:\编程学习资料\Tomcat 专题.pdf)

[移动硬盘 Tomcat 专题.pdf](E:\编程资料\尚硅谷\资料-Tomcat核心原理解析\Tomcat-02\文档\Tomcat 专题.pdf)



# 课程内容

| 序号 | 第一天            | 第二天         |
| ---- | ----------------- | -------------- |
| 1    | Tomcat 基础       | Web 应用配置   |
| 2    | Tomcat 架构       | Tomcat管理配置 |
| 3    | Jasper            | JVM配置        |
| 4    | Tomcat 服务器配置 | Tomcat集群     |
| 5    |                   | Tomcat安全     |
| 6    |                   | Tomcat性能调优 |
| 7    |                   | Tomcat附加功能 |

 

# Tomcat 基础

## Web 概念



**软件架构**

- C/S： 客户端/服务器端
- B/S： 浏览器/服务器端 

 

**资源分类**

- 静态资源： 所有用户访问后，得到的结果都是一样的，称为静态资源。静态资源可以直接被浏览器解析。如： html, css, JavaScript, jpg
- 动态资源: 每个用户访问相同资源后，得到的结果可能不一样 , 称为动态资源。动态资源被访问后，需要先转换为静态资源，再返回给浏览器，通过浏览器进行解析。如：servlet/jsp, php, asp....

 

 

## 常见 web 服务器软件

- webLogic：oracle 公司，大型 JavaEE 服务器，支持所有 JavaEE 规范，收费的
- webSphere：IBM 公司，大型 JavaEE 服务器，支持所有 JavaEE 规范，收费的
- JBOSS：JBOSS 公司的，大型 JavaEE 服务器，支持所有的 JavaEE 规范，收费的
- Tomcat：Apache 的，中小型 JavaEE 服务器，仅仅支持少量的 JavaEE 规范 servlet/jsp。开源的，免费的

 

## Tomcat 目录结构

Tomcat 的主要目录文件如下 ：

| 目录        | 目录下文件                 | 说明                                                         |
| ----------- | -------------------------- | ------------------------------------------------------------ |
| **bin**     | /                          | 存放Tomcat的启动、停止等批处理脚本文件                       |
|             | startup.bat , startup.sh   | 用于在windows和linux下的启动脚本                             |
|             | shutdown.bat , shutdown.sh | 用于在windows和linux下的停止脚本                             |
| **conf**    | /                          | 用于存放Tomcat的相关配置文件                                 |
|             | Catalina                   | 用于存储针对每个虚拟机的Context配置                          |
|             | context.xml                | 用于定义所有web应用均需加载的Context配置，如果web应用指定了自己的context.xml，该文件将被覆盖 |
|             | catalina.properties        | Tomcat 的环境变量配置                                        |
|             | catalina.policy            | Tomcat 运行的安全策略配置                                    |
|             | logging.properties         | Tomcat 的日志配置文件， 可以通过该文件修改Tomcat 的日志级别及日志路径等 |
|             | server.xml                 | Tomcat 服务器的核心配置文件                                  |
|             | tomcat-users.xml           | 定义Tomcat默认的用户及角色映射信息配置                       |
|             | web.xml                    | Tomcat 中所有应用默认的部署描述文件， 主要定义了基础Servlet和MIME映射。 |
| **lib**     | /                          | Tomcat 服务器的依赖包                                        |
| **logs**    | /                          | Tomcat 默认的日志存放目录                                    |
| **webapps** | /                          | Tomcat 默认的Web应用部署目录                                 |
| **work**    | /                          | Web 应用JSP代码生成和编译的临时目录                          |

 

## 搭建 Tomcat 源码环境

**下载**：[地址](https://tomcat.apache.org/download-80.cgi)

- 解压zip压缩包
- 进入解压目录，创建名为 home 的目录，并将conf、webapps 目录移入home 目录
- 在当前目录下创建一个 pom.xml 文件，引入 tomcat 的依赖包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project ...>
    <modelVersion>4.0.0</modelVersion> 
    <groupId>org.apache.tomcat</groupId>
    <artifactId>apache-tomcat-8.5.42-src</artifactId>
    <name>Tomcat8.5</name>
    <version>8.5</version>
    <build> 
        <finalName>Tomcat8.5</finalName>
        ... 
    </build>
    
    <dependencies> 
        <dependency> 
            <groupId>junit</groupId> 
            <artifactId>junit</artifactId> 
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.easymock</groupId> 
            <artifactId>easymock</artifactId>
            <version>3.4</version> 
        </dependency>
        <dependency> 
            <groupId>ant</groupId> 
            <artifactId>ant</artifactId>
            <version>1.7.0</version> 
        </dependency> 
        <dependency> 
            <groupId>wsdl4j</groupId> 
            <artifactId>wsdl4j</artifactId>
            <version>1.6.2</version> 
        </dependency> 
        <dependency>
            <groupId>javax.xml</groupId>
            <artifactId>jaxrpc</artifactId>
            <version>1.1</version> 
        </dependency>
        <dependency> 
            <groupId>org.eclipse.jdt.core.compiler</groupId>
            <artifactId>ecj</artifactId> 
            <version>4.5.1</version>
        </dependency>
    </dependencies> 
```

- 配置 idea 的启动类， 配置 MainClass ， 并配置 VM 参数。

 ```
 ‐Dcatalina.home=D:/idea‐workspace/itcast_project_tomcat/apache‐tomcat‐ 8.5.42‐src/home
 ‐Dcatalina.base=D:/idea‐workspace/itcast_project_tomcat/apache‐tomcat‐ 8.5.42‐src/home
 ‐Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
 ‐Djava.util.logging.config.file=D:/idea‐ workspace/itcast_project_tomcat/apache‐tomcat‐8.5.42‐ src/home/conf/logging.properties
 ```

- 手动初始化 JSP 解析器（不执行这步项目就启动不了）。在 ContextConfig 中 的 configureStart 函数中的`webConfig();`后追加

```java
context.addServletContainerInitializer(new JasperInitializer(), null);
```



重启tomcat就可以正常访问了

 

# Tomcat 架构

**2.1** **Http**工作原理

HTTP协议是浏览器与服务器之间的数据传送协议。作为应用层协议，HTTP是基于TCP/IP

协议来传递数据的（HTML文件、图片、查询结果等），HTTP协议不涉及数据包

（Packet）传输，主要规定了客户端和服务器之间的通信格式。
 

 用户通过浏览器进行了一个操作，比如输入网址并回车，或者是点击链接，接着浏览器获取了这个事件。

2） 浏览器向服务端发出TCP连接请求。

3） 服务程序接受浏览器的连接请求，并经过TCP三次握手建立连接。

4） 浏览器将请求数据打包成一个HTTP协议格式的数据包。

5） 浏览器将该数据包推入网络，数据包经过网络传输，最终达到端服务程序。

6） 服务端程序拿到这个数据包后，同样以HTTP协议格式解包，获取到客户端的意图。

7） 得知客户端意图后进行处理，比如提供静态文件或者调用服务端程序获得动态结果。

8） 服务器将响应结果（可能是HTML或者图片等）按照HTTP协议格式打包。

9） 服务器将响应数据包推入网络，数据包经过网络传输最终达到到浏览器。

10） 浏览器拿到数据包后，以HTTP协议的格式解包，然后解析数据，假设这里的数据是

HTML。

11） 浏览器将HTML文件展示在页面上。

那我们想要探究的Tomcat作为一个HTTP服务器，在这个过程中都做了些什么事情呢？主要是接受连接、解析请求数据、处理请求和发送响应这几个步骤。
 

## Tomcat 整体架构

HTTP 服务器不直接调用业务类，而是把请求交给容器来处理，容器通过 Servlet 接口调用业务类。因此Servlet接口和Servlet容器的出现，达到了HTTP 服务器与业务类解耦的目的

Servlet 接口和 Servlet 容器这一整套规范叫作 Servlet 规范

Tomcat 按照 Servlet 规范的要求实现了 Servlet 容器，同时它们也具有HTTP服务器的功能

程序员只需要实现一个Servlet，并把它注册到 Tomcat（Servlet容器）中

![[../020 - 附件文件夹/Pasted image 20230401234825.png|500]]

## Servlet 容器工作流程

为了解耦，HTTP 服务器不直接调用 Servlet，而是把请求交给Servlet容器来处理

1. 客户请求某个资源时，HTTP服务器会用一个 ServletRequest 对象把客户的请求信息封装起来
2. 调用 Servlet 容器的 service 方法，Servlet 容器拿到请求后，根据请求的URL 和Servlet的映射关系，找到相应的Servlet
3. 如果目标 Servlet 还没有被加载，就用反射机制创建这个Servlet，并调用Servlet的init方法来完成初始化
4. 调用 Servlet 的 service 方法来处理请求
5. 把 ServletResponse 对象返回给 HTTP 服务器，HTTP 服务器把响应发送给客户端

![[../020 - 附件文件夹/Pasted image 20230401234843.png|475]] 

## Tomcat 详细架构

Tomcat 要实现两个 核心功能：

- 处理 Socket 连接，负责网络字节流与 Request 和 Response 对象的转化（接收请求，返回响应）
- 加载和管理 Servlet，以及具体处理 Request 请求（处理请求）

因此 Tomcat 设计了两个核心组件

- 连接器（Connector）。负责对外交流（接收请求，返回响应）
- 容器（Container）负责内部处理（处理请求）

![[../020 - 附件文件夹/Pasted image 20230401234855.png|500]]
 

# 连接器 **-** Coyote



## 架构介绍

**定义**

Coyote 是 Tomcat 的连接器框架的名称 , 是Tomcat服务器提供的供客户端访问的外部接口

客户端通过 Coyote 与服务器建立连接、发送请求并接受响应

Coyote 封装了底层的网络通信（Socket 请求及响应处理），为 Catalina 容器提供了统一的接口，使 Catalina 容器与具体的请求协议及 IO 操作方式完全解耦

PS：Tomcat 并没有 Coyote 接口，实际上是 ProtocolHandler 接口和 CoyoteAdaptor 共同组成了 Coyote

**功能**

Coyote 将Socket 输入转换封装为 Request 对象，交由Catalina 容器进行处理，处理请求完成后, Catalina 通过Coyote 提供的Response 对象将结果写入输出流 。

Coyote 作为独立的模块，只负责具体协议和IO的相关操作， 与 Servlet 规范实现没有直接关系，因此即便是 Request 和 Response 对象也并未实现Servlet 规范对应的接口， 而是在 Catalina 中将他们进一步封装为 ServletRequest 和 ServletResponse（这两个遵守了 Servlet 规范）

 

## IO 模型与协议

在 Coyote 中 ， Tomcat 支持的多种 I/O 模型和应用层协议



**Tomcat 支持的IO模型**（自8.5/9.0 版本起，Tomcat 移除了 对 BIO 的支持）

| **IO**模型 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| NIO        | 非阻塞 I/O，采用 Java NIO 类库实现                           |
| NIO2       | 异步 I/O，采用 JDK 7最新的 NIO2 类库实现                     |
| APR        | 采用 Apache 可移植运行库实现，是 C/C++ 编写的本地库。如果选择该方案，需要单独安装 APR 库 |

 

**Tomcat 支持的应用层协议**

| 应用层协议 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| HTTP/1.1   | 这是大部分Web应用采用的访问协议                              |
| AJP        | 用于和Web服务器集成（如Apache），以实现对静态资源的优化以及集群部署，当前支持AJP/1.3 |
| HTTP/2     | HTTP 2.0大幅度的提升了Web性能。下一代HTTP协议 ， 自8.5以及9.0 版本之后支持 |

![[../020 - 附件文件夹/Pasted image 20230401234913.png|375]]

在 8.0 之前 ， Tomcat 默认采用的 I/O 方式为 BIO ， 之后改为 NIO。 无论 NIO、NIO2 还是 APR， 在性能方面均优于以往的 BIO。 如果采用APR， 甚至可以达到 Apache HTTP Server 的影响性能

![[../020 - 附件文件夹/Pasted image 20230401234929.png|500]]

Tomcat 为了实现支持多种 I/O 模型和应用层协议，一个容器可能对接多个连接器，组装后这个整体叫作 Service 组件

注意，Service 本身没有做什 么重要的事情，只是在连接器和容器外面多包了一层，把它们组装在一起

Tomcat 内可能有多个Service，这样的设计也是出于灵活性的考虑。通过在 Tomcat 中配置多个Service，可以实现通过不同的端口号来访问同一台机器上部署的不同应用



## 连接器组件

Coyote 中的组件包括

- EndPoint 

 

### EndPoint 通信端点

EndPoint ： Coyote 通信端点，即通信监听的接口，是具体 Socket 接收和发送处理器，是对传输层的抽象，用来实现 TCP/IP 协议

Tomcat 没有 EndPoint 接口，而是提供了一个抽象类 AbstractEndpoint ， 里面定义了两个内部类：Acceptor 和 SocketProcessor

Acceptor 用于监听 Socket 连接请求。SocketProcessor 用于处理接收到的 Socket 请求，它实现 Runnable 接口，在 run 方法里调用协议处理组件Processor 进行处理

为了提高处理能力，SocketProcessor 被提交到 线程池来执行。而这个线程池叫作执行器（Executor)，它是 Tomcat 扩展原生的Java线程池



### Processor 协议处理接口

Processor ： Coyote 协议处理接口 ，如果说 EndPoint 是用来实现TCP/IP协议的，那么 Processor 用来实现HTTP协议

Processor 接收来自 EndPoint 的 Socket，读取字节流解析成 Tomcat Request 和 Response 对象，并通过 Adapter 将其提交到容器处理， Processor是对应用层协议的抽象



### ProtocolHandler 协议接口

ProtocolHandler： Coyote 协议接口， 通过 Endpoint 和 Processor ， 实现针对具体协议的处理能力

Tomcat 按照协议和 I/O 为 ProtocolHandler 提供了 6 个实现类 ：

 AjpNioProtocol ， AjpAprProtocol， AjpNio2Protocol ， Http11NioProtocol ，Http11Nio2Protocol ， Http11AprProtocol

在配置 tomcat/conf/server.xml 时 ， 至少要指定具体的 ProtocolHandler , 当然也可以指定协议名称 ， 如 ： HTTP/1.1 ，如果安装了APR，那么将使用Http11AprProtocol ， 否则使用 Http11NioProtocol



### Adapter

ProtocolHandler  接口负责解析请求并生成 Tomcat  Request 类。但是这个 Request 对象不是标准的 ServletRequest

Tomcat 设计者的解决方案是引入 CoyoteAdapter，这是适配器模式的经典运用，

CoyoteAdapter 负责将 Tomcat Request 转成 ServletRequest，再调用容器的 service 方法



# 容器 **-** **Catalina**

Catalina 是 Tomcat 的 servlet 容器。

Catalina 是 Servlet 容器实现，包含了之前讲到的所有的容器组件，以及后续章节涉及到的安全、会话、集群、管理等Servlet 容器架构的各个方面

它通过松**耦合**的方式集成 Coyote，以完成按照请求协议进行数据读写。同时，它还包括我们的启动入口、Shell 程序等。





## Catalina 地位

Tomcat 的模块分层结构图， 如下

![[../020 - 附件文件夹/Pasted image 20230401234948.png|500]]

Catalina 是 Tomcat 的核心 ， 其他模块都是为Catalina 提供支撑的。 比如 ： 通过 Coyote 模块提供链接通信，Jasper 模块提供 JSP引擎，Naming 提供 JNDI 服务，Juli 提供日志服务



## Catalina 结构

Catalina 的主要组件结构如下：

![[../020 - 附件文件夹/Pasted image 20230401235006.png|500]] 

如上图所示，Catalina 负责管理 Server，而 Server 表示着整个服务器

Server 下面有多个服务 Service，每个服务都包含着多个连接器组件 Connector（Coyote 实现）和一个容器组件 Container。在 Tomcat 启动的时候， 会初始化一个 Catalina 的实例



Catalina 各个组件的职责：

| 组件      | 职责                                                         |
| --------- | ------------------------------------------------------------ |
| Catalina  | 负责解析 Tomcat 的配置文件 , 以此来创建服务器Server组件，并根据命令来对其进行管理 |
| Server    | 服务器表示整个Catalina Servlet 容器以及其它组件，负责组装并启动 Servlet 引擎，Tomcat 连接器。Server 通过实现 Lifecycle接口，提供了一种优雅的启动和关闭整个系统的方式 |
| Service   | 服务是 Server 内部的组件，一个 Server 包含多个Service。它将若干个 Connector 组件绑定到一个 Container（Engine）上 |
| Connector | 连接器，处理与客户端的通信，它负责接收客户请求，然后转给相关的容器处理，最后向客户返回响应结果 |
| Container | 容器，负责处理用户的 servlet 请求，并返回对象给 web 用户的模块 |



`Service`通过`addConnector`和`setConnector`管理连接器

 

## Container 结构

Tomcat 设计了 4 层容器，分别是

Engine、Host、Context 和 Wrapper

这 4 种容器不是平行关系，而是父子关系。， Tomcat 通过一种分层的架构，使得 Servlet 容器具有很好的灵活性。

![[../020 - 附件文件夹/Pasted image 20230401235306.png|650]]

| 容器    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| Engine  | 表示整个 Catalina 的 Servlet 引擎，用来管理多个虚拟站点，一个 Service 最多只能有一个 Engine，但是一个引擎可包含多个 Host |
| Host    | 代表一个虚拟主机，或者说一个站点，可以给 Tomcat 配置多个虚拟主机地址，而一个虚拟主机下可包含多个 Context |
| Context | 表示一个 Web 应用程序， 一个 Web 应用可包含多个 Wrapper      |
| Wrapper | 表示一个 Servlet，Wrapper 作为容器中的最底层，不能包含子容器 |

PS：1 个 Context 对应 1 个 Web 项目或者说对应 1 个能被 Tomcat 解析并运行的 war 包

> `Wrapper`包含`FilterChain`和`Servlet`

> 我们也可以再通过 Tomcat 的 server.xml 配置文件来加深对 Tomcat 容器的理解。Tomcat 采用了组件化的设计，它的构成组件都是可配置的，其中最外层的是 Server，其他组件按照一定的格式要求配置在这个顶层容器中。

```xml
<Server>
    <Service>
        <Connector/>
        <Connector/>
        <Engine>
            <Host>
                <Context></Context>
            </Host>
        </Engine>
    </Service>
</Server>
```

理论上： 只有 1 个`Server`，若干个`Service`

每个`Service`有若干个`Connector`和 1 个 `Engine`

1 个`Engine`有若干个`Host`

1 个`Host`有若干个`Context`

 

## 四层容器管理方式

Q：Tomcat是怎么管理这些容器的呢？

A：这些容器具有父子关系，形成一个树形结构。Tomcat 通过设计模式中的组合模式管理四层容器

具体实现方法是，所有容器组件都实现了 Container 接口，因此组合模式可以使得用户对单容器对象和组合容器对象的使用具有一致性

这里单容器对象指的是最底层的 Wrapper，组合容器对象指 Context、Host 或 Engine

![[../020 - 附件文件夹/Pasted image 20230401235326.png|725]]


Container 接口中提供了以下方法（截图中知识一部分方法） ：

![[../020 - 附件文件夹/Pasted image 20230401235415.png|500]]

Container接口扩展了LifeCycle接口，LifeCycle接口用来统一管理各组件的生命周期，后面会专门的篇幅去详细介绍


# Tomcat 工作流程

> 从源码上讲 Tomcat 的启动流程。但笔记只做介绍，不对每一步源码调试进行分析，具体调试自行实践
>
> PS：启动流程相对处理请求流程而言，没那么重要



## Tomcat 启动流程

Tomcat 的启动过程非常标准化， 统一按照生命周期管理接口 Lifecycle 的定义进行启动。首先调用 init() 方法进行组件的**逐级初始化**操作，然后再调用 start() 方法进行启动

每一级的组件除了完成自身的处理外，还要负责调用子组件响应的生命周期管理方法， 组件与组件之间是**松耦合**的，因此我们可以很容易的通过配置文件进行修改和替换

![[../020 - 附件文件夹/Pasted image 20230401235427.png]]

1. 启动 tomcat ， 调用 bin/startup.bat (在linux 目录下 , 需要调用 bin/startup.sh)， 在startup.bat 脚本中, 调用了catalina.bat
2. 在 catalina.bat 脚本文件中，调用了BootStrap 中的 main 方法
3. 在 BootStrap 的 main 方法中调用了 init 方法 ， 来创建 Catalina 及 初始化类加载器
4. 在 BootStrap 的 main 方法中调用了 load 方法 ， 在其中又调用了 Catalina 的 load 方法
5. 在 Catalina 的 load 方法中 , 需要进行一些初始化的工作, 并需要构造 Digester 对象，用于解析 XML
6. 然后在调用后续组件的初始化操作... 加载 Tomcat 的配置文件，初始化容器组件 ，监听对应的端口号， 准备接受客户端请求

PS：load 阶段会加载解析 server.xml 配置文件



## 源码解析

### Lifecycle

由于所有的组件均存在初始化、启动、停止等生命周期方法，拥有生命周期管理的特性， 所以 Tomcat 在设计的时候， 基于生命周期管理抽象成了一个接口 Lifecycle

组件 Server、Service、Container、Executor、Connector 组件 ， 都实现了一个生命周期的接口，从而具有了以下生命周期中的核心方法：

![[../020 - 附件文件夹/Pasted image 20230401235438.png]] 

### 各组件的默认实现

Server、Service、Engine、Host、Context 都是接口， 下图中罗列了这些接口的默认实现类

![[../020 - 附件文件夹/Pasted image 20230401235447.png]]

当前对于 Endpoint 组件来说，在 Tomcat 中没有对应的 Endpoint 接口， 但是有一个抽象类 AbstractEndpoint ，其下有三个实现类： NioEndpoint、Nio2Endpoint、AprEndpoint

这三个实现类，分别对应于前面讲解连接器 Coyote 时， 提到的链接器支持的三种IO模型：NIO，NIO2，APR

Tomcat8.5 版本中，**默认采用的是 NioEndpoint**

![[../020 - 附件文件夹/Pasted image 20230401235501.png]]

ProtocolHandler ： Coyote 协议接口，通过封装 Endpoint 和 Processor ， 实现针对具体协议的处理功能。Tomcat按照协议和IO提供了6个实现类。

 

### Tomcat 启动入口类

`org.apache.catalina.startup MainClass：BootStrap ‐‐‐‐> main(String[] args)`


# Tomcat 请求处理流程

## 请求流程

> Q：设计了这么多层次的容器，Tomcat 怎么确定每一个请求应该由哪个 Wrapper 容器里？（Tomcat 怎么完成 url 和 Wrapper 的映射）
>
> A：用 Mapper 组件来实现



Mapper 组件的功能就是将用户请求的 URL 定位到一个 Servlet

它的工作原理是： Mapper 组件里保存了 Web 应用的配置信息，其实就是容器组件与访问路径的映射关系

比如 Host 容器里配置的域名、Context 容器里的 Web 应用路径，以及 Wrapper 容器里 Servlet 映射的路径，这些配置信息就是一个多层次的 Map

PS：Context 容器里的 Web 应用路径就是一个 Web 项目 / war 包为 Web 配置的虚拟路径

![[../020 - 附件文件夹/Pasted image 20230401235523.png]]

当 1 个请求到来时，Mapper 组件通过解析请求 URL 里的域名和路径，再到自己保存的 Map 里去查找，就能定位到 1 个 Servlet

1 个请求 URL 最后只会定位到 1 个Wrapper容器，也就是 1 个 Servlet。

上面这幅图只是描述了根据请求的 URL 如何查找到需要执行的 Servlet 

下面的示意图中 ， 从 Tomcat 架构层面描述了当用户请求链接`http://www.itcast.cn/bbs/findAll`之后, 是如何找到最终处理业务逻辑的 servlet 

![[../020 - 附件文件夹/Tomcat处理请求.svg]]

1. Connector 组件 Endpoint 中的 Acceptor 监听客户端套接字连接并接收 Socket
2. 将连接交给线程池 Executor 处理，开始执行请求响应任务
3. Processor 组件读取消息报文，解析请求行、请求体、请求头，封装成 Request 对象
4. Mapper 组件根据请求行的 URL 值和请求头的 Host 值匹配由哪个 Host 容器、Context 容器
5. CoyoteAdaptor 组件负责将 Connector 组件和 Engine 容器关联起来，把生成的 Request 对象和响应对象 Response 传递到 Engine 容器中，调用 Pipeline
6. Engine 容器的管道开始处理，管道中包含若干个 Valve、每个 Valve 负责部分处理逻辑。执行完 Valve 后会执行默认的 StandardEngineValve（负责调用下一层容器的 Pipeline）
7. Host 容器的管道开始处理，流程类似，最后执行 Context 容器的 Pipeline
8. Context 容器的管道开始处理，流程类似，最后执行 Wrapper 容器的 Pipeline
9. Wrapper 容器的管道开始处理，流程类似，最后执行 Wrapper 容器对应的 Servlet 对象的处理方法

> 每个容器都有 1 个 Pipeline，Pipeline 中包含若干个 Valve

![[../020 - 附件文件夹/Pasted image 20230401235705.png|500]]

逻辑上 Coyote 和 Catalina 是分离的，实际上两者是**松耦合**。Catalina 持有 Coyote

> EndPoint ： Coyote 通信端点，即通信监听的接口，是具体 Socket 接收和发送处理器，是对传输层的抽象，用来实现 TCP/IP 协议
>
> Tomcat 没有 EndPoint 接口，而是提供了一个抽象类 AbstractEndpoint ， 里面定义了两个内部类：Acceptor 和 SocketProcessor
>
> Acceptor 用于监听 Socket 连接请求。SocketProcessor 用于处理接收到的 Socket 请求，它实现 Runnable 接口，在 run 方法里调用协议处理组件Processor 进行处理
>
> 为了提高处理能力，SocketProcessor 被提交到 线程池来执行。而这个线程池叫作执行器（**Executor**)，它是 Tomcat 扩展原生的 Java 线程池
>



## 请求流程源码解析

Tomcat 整体架构中的各个组件各司其职，组件之间松耦合，确保了整体架构的可伸缩性和可拓展性

组件内部，每个 Container 组件采用责任链模式来完成具体的请求处理，增强组件的灵活性和拓展性

在 Tomcat 中定义了 Pipeline 和 Valve 两个接口，Pipeline 用于构建责任链， 后者代表责任链上的每个处理器。Pipeline 中维护了一个基础的Valve，它始终位于 Pipeline 的末端（负责调用下一层容器的 Pipeline）

（最后执行），封装了具体的请求处理和输出响应的过程。当然，我们也可以调用 addValve() 方法， 为 Pipeline 添加其他的 Valve， 后添加的Valve 位于基础的 Valve之前，并按照添加顺序执行。Pipiline 通过获得首个 Valve 来启动整合链条的执行





# Tomcat 服务器配置

Tomcat 服务器的配置主要集中于 tomcat/conf 下的

catalina.policy、catalina.properties、context.xml、server.xml、tomcat-users.xml、web.xml 文件。



## server.xml

server.xml 是 tomcat 服务器的核心配置文件，包含了 Tomcat 的 Servlet 容器（Catalina）的所有配置

由于配置的属性特别多，在这里主要讲解其中的一部分重要配置

 ````xml
 <Server>
     <Service>
         <Connector/>
         <Connector/>
         <Engine>
             <Host>
                 <Context></Context>
             </Host>
         </Engine>
     </Service>
 </Server>
 ````



### Server

Server 是 server.xml 的根元素，用于创建一个 Server 实例，默认使用的实现类是 org.apache.catalina.core.StandardServer

```xml
<Server port="8005" shutdown="SHUTDOWN">
    ...
</Server>
```

- port : Tomcat 监听的关闭服务器的端口
- shutdown： 关闭服务器的指令字符串
- Server 内嵌的子元素为 Listener、GlobalNamingResources、Service

 ```xml
 <!--APR library loader. -->
 <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on"/>
 <!-- Prevent memory leaks due to use of particular java/javax APIs-->
 <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"/>
 <!-- 加载（服务器启动）和销毁（服务器停止）全局命名服务 -->
 <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener"/>
 <!-- Context 停止时，重建线程池中的线程避免线程池泄漏 -->
 <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener"/>
 ```



GlobalNamingResources 中定义了全局命名服务：

```xml
<GlobalNamingResources>
    <!-- Editable user database that can also be used by UserDatabaseRealm to authenticate users -->
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml"/>
</GlobalNamingResources>
```

 

 

### Service

该元素用于创建 Service 实例，默认使用 org.apache.catalina.core.StandardService。默认情况下，Tomcat 仅指定了Service 的名称， 值为 "Catalina"

```xml
<Service name="Catalina">
    ...
</Service>
```



Service 可以内嵌的元素为 ： Listener、Executor、Connector、Engine

- Listener 用于为 Service 添加生命周期监听器
- Executor 用于配置 Service 共享线程池
- Connector 用于配置 Service 包含的链接器
- Engine 用于配置 Service 中链接器对应的 Servlet 容器引擎



一个 Server 服务器，可以包含多个 Service 服务

 

### Executor

默认情况下，Service 并未添加共享线程池配置。 如果想添加一个线程池， 可以在下添加如下配置

```xml
<Executor name="tomcatThreadPool" 
          namePrefix="catalina‐exec‐" 
          maxThreads="200" 
          minSpareThreads="100" 
          maxIdleTime="60000" 
          maxQueueSize="Integer.MAX_VALUE" 
          prestartminSpareThreads="false" 
          threadPriority="5"
          className="org.apache.catalina.core.StandardThreadExecutor"/>
```

| 属性                    | 含义                                                         |
| ----------------------- | ------------------------------------------------------------ |
| name                    | 线程池名称，便于在 Connector 中指定所用的线程池              |
| namePrefix              | 所创建的每个线程的名称前缀，一个单独的线程名称为 namePrefix + threadNumber |
| maxThreads              | 池中最大线程数                                               |
| minSpareThreads         | 活跃线程数，也就是核心池线程数，这些线程不会被销毁，会一直存在 |
| maxIdleTime             | 线程空闲时间，超过该时间后，空闲线程会被销毁，默认值为6000（1分钟），单位毫秒 |
| maxQueueSize            | 待被处理的请求的最大数目，默认为 Int 的最大值，也就是广义的无限。除非特殊情况，这个值不需要更改， 否则会有请求不会被处理的情况发生 |
| prestartminSpareThreads | 启动线程池时是否启动 minSpareThreads部分线程。默认值为false，即不启动 |
| threadPriority          | 线程池中线程优先级，默认值为5，值从1到10                     |
| className               | 线程池实现类，未指定情况下，默认实现类为 org.apache.catalina.core.StandardThreadExecutor。如果想使用自定义线程池首先需要实现org.apache.catalina.Executor接口 |



如果不配置共享线程池，那么 Catalina  各组件在用到线程池时会独立创建。

 

### Connector

Connector 用于创建链接器实例。默认情况下，server.xml 配置了两个链接器，一个支持 HTTP 协议，一个支持 AJP 协议。因此大多数情况下，不需要新增链接器配置， 只是根据需要对已有链接器进行优化



 ```xml
 <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443"/>
 
 <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
 ```

- port：端口号。如果该属性设置为 0，Tomcat 将会随机选择一个可用的端口号给当前 Connector 使用
- protocol： 当前Connector 支持的访问协议。 默认为 HTTP/1.1 ， 并采用自动切换机制选择一个基于 JAVA NIO 的链接器或者基于本地 APR 的链接器（根据本地是否含有Tomcat的本地库判定）
- connectionTimeOut：Connector 接收连接后的等待超时时间， 单位为 毫秒。 -1 表示不超时
- redirectPort：当前 Connector 不支持 SSL 请求， 接收到了一个请求， 并且也符合 security-constraint 约束， 需要SSL传输，Catalina 自动将请求重定向到指定的端口
- executor：指定共享线程池的名称， 也可以通过 maxThreads、minSpareThreads 等属性配置内部线程池
- URIEncoding：用于指定编码 URI 的字符编码， Tomcat8.x 版本默认的编码为 UTF-8



如果不希望采用上述自动切换的机制， 而是明确指定协议， 可以使用以下值

HTTP 协议：

```
org.apache.coyote.http11.Http11NioProtocol 非阻塞式 Java NIO 链接器
org.apache.coyote.http11.Http11Nio2Protocol 非阻塞式 JAVA NIO2 链接器
org.apache.coyote.http11.Http11AprProtocol APR 链接器
```

AJP协议 ：

```
org.apache.coyote.ajp.AjpNioProtocol 非阻塞式 Java NIO 链接器
org.apache.coyote.ajp.AjpNio2Protocol 非阻塞式 JAVA NIO2 链接器
org.apache.coyote.ajp.AjpAprProtocol APR 链接器
```

 

```xml
<Connector port="8080"
    protocol="HTTP/1.1"
    executor="tomcatThreadPool"
    maxThreads="1000"
    minSpareThreads="100"
    acceptCount="1000"
    maxConnections="1000"
    connectionTimeout="20000"
    compression="on"
    compressionMinSize="2048"
    disableUploadTimeout="true"
    redirectPort="8443"
    URIEncoding="UTF‐8" />
```



###  Engine



```xml
<Engine name="Catalina" defaultHost="localhost">
	...
</Engine>
```

- name：用于指定 Engine 的名称， 默认为 Catalina 。该名称会影响一部分Tomcat的存储路径（如临时文件）
- defaultHost ： 默认使用的虚拟主机名称， 当客户端请求指向的主机无效时， 将交由默认的虚拟主机处理， 默认为 localhost



### Host

Host 元素用于配置一个虚拟主机， 它支持以下嵌入元素

Alias、Cluster、Listener、Valve、Realm、Context

如果在 Engine 下配置 Realm， 那么此配置将在当前 Engine 下的所有 Host 中共享。 同样，如果在 Host 中配置 Realm ， 则在当前 Host 下的所有Context 中共享

Context 中的 Realm 优先级 > Host 的 Realm 优先级 > Engine 中的 Realm 优先级

```xml
<Host name="localhost" appBase="webapps"
      unpackWARs="true" autoDeploy="true">
</Host>
```

- name：当前 Host 通用的网络名称， 必须与 DNS 服务器上的注册信息一致。 Engine 中包含的 Host 必须存在一个名称与 Engine 的 defaultHost 设置一致
- appBase： 当前 Host 的应用基础目录， 当前 Host 上部署的 Web 应用均在该目录下（可以是绝对目录，相对路径）。默认为webapps
- unpackWARs： 设置为true， Host 在启动时会将 appBase 目录下 war 包解压为目录。设置为 false， Host 将直接从 war 文件启动
- autoDeploy： 控制 tomcat 是否在运行时定期检测并自动部署新增或变更的 web 应用



这个时候就可以通过两个域名访问当前 Host 下的应用（需要确保 DNS 或 hosts 中添加了域名的映射配置）

 

### Context

一个 Context 对应一个 Web 应用

Context 用于配置一个 Web 应用，默认的配置如下：

```xml
<Host name="localhost" appBase="webapps"
      unpackWARs="true" autoDeploy="true">
</Host>
```

- docBase：Web 应用目录或者 War 包的部署路径。可以是绝对路径，也可以是相对于 Host appBase 的相对路径
- path：Web 应用的 Context 路径 - 虚拟路径。如果我们 Host 名为 localhost， 则该 web 应用访问的根路径为：`http://localhost:8080/myApp`

它支持的内嵌元素为：CookieProcessor， Loader， Manager，Realm，Resources， WatchedResource，JarScanner，Valve。

```xml
<Host name="www.tomcat.com" appBase="webapps" unpackWARs="true" autoDeploy="true">
	<Context docBase="D:\servlet_project03" path="/myApp"></Context>
	<Valve className="org.apache.catalina.valves.AccessLogValve"
           directory="logs"
           prefix="localhost_access_log"
           suffix=".txt"
           pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
```



## tomcat-users.xml

该配置文件中，主要配置的是 Tomcat 的用户，角色等信息，用来控制 Tomcat 中 manager， host-manager 的访问权限

生产环境下，应当关掉所有和权限相关的页面访问权限



# Web 应用配置 - web.xml

web.xml 是 web 应用的描述文件， 它支持的元素及属性来自于 Servlet 规范定义

在Tomcat 中， Web 应用的描述信息包括 tomcat/conf/web.xml 中**默认配置**以及 Web 应用 WEB-INF/web.xml 下的**定制配置**

![[../020 - 附件文件夹/Pasted image 20230401235905.png]]

## ServletContext 初始化参数

通过添加 ServletContext 初始化参数，可以在程序运行时使用`javax.servlet.ServletContext.getInitParameter()`方法获取参数

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
</web-app>
```

 

## 会话配置

用于配置 Web 应用会话，包括超时时间、Cookie 配置以及会话追踪模式。它将覆盖 server.xml 和 context.xml 中的配置

```xml
<session‐config>
    <session‐timeout>30</session‐timeout>
    <cookie‐config>
        <name>JESSIONID</name>
        <domain>www.example.cn</domain>
        <path>/</path>
        <comment>Session Cookie</comment>
        <http‐only>true</http‐only>
        <secure>false</secure>
        <max‐age>3600</max‐age>
        </cookie‐config>
    <tracking‐mode>COOKIE</tracking‐mode>
</session‐config>
```

- session‐timeout：会话超时时间，单位分钟
- cookie‐config：用于配置会话追踪 Cookie name：Cookie的名称
- domain：Cookie 的域名
- path：Cookie的路径
- comment：注释
- http‐only：cookie 只能通过 HTTP 方式进行访问，JS 无法读取或修改，此项可以增加网站访问的安全性
- secure：此 cookie 只能通过 HTTPS 连接传递到服务器，而 HTTP 连接则不会传递该信息。注意是从浏览器传递到服务器，服务器端的 Cookie对象不受此项影响
- max‐age：以秒为单位表示 cookie 的生存期，默认为 ‐1 表示是会话 Cookie 在浏览器关闭时就会消失
- tracking‐mode ：用于配置会话追踪模式，Servlet3.0版本中支持的追踪模式：COOKIE、URL、SSL
  - COOKIE : 通过HTTP Cookie 追踪会话是最常用的会话追踪机制， 而且 Servlet 规范也要求所有的 Servlet 规范都需要支持 Cookie 追踪
  - URL : URL 重写是最基本的会话追踪机制。当客户端不支持 Cookie 时，可以采用 URL 重写的方式。当采用 URL 追踪模式时，请求路径需要包含会话标识信息，Servlet容器会根据路径中的会话标识设置请求的会话信息。如： `http://www.myserver.com/user/index.html?jessionid=1234567890`
  - SSL：对于 SSL 请求， 通过 SSL 会话标识确定请求会话标识

 

## Servlet 配置

Servlet 的配置主要是两部分， servlet 和 servlet-mapping ：



```xml
<servlet> 
        <servlet-name>HelloServlet</servlet-name> 
    	<servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>
</servlet>
    
<servlet-mapping> 
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/hello</url-pattern> 
</servlet-mapping> 
```

servlet 文件上传设置。默认上传文件大小最大很小（不超过 5MB）

```xml
<servlet>
    <servlet‐name>uploadServlet</servlet‐name>
    <servlet‐class>cn.itcast.web.UploadServlet</servlet‐class>
    <multipart‐config>
        <location>C://path</location>
        <max‐file‐size>10485760</max‐file‐size>
        <max‐request‐size>10485760</max‐request‐size>
        <file‐size‐threshold>0</file‐size‐threshold>
	</multipart‐config>
</servlet>
```





## Listener 配置

Listener 用于监听 servlet 中的事件，例如 context、request、session 对象的创建、修 改、删除，并触发响应事件

> Spring MVC 中 就是由监听器创建 Spring IOC 容器的

Listener是观察者模式的实现，在 servlet 中主要用于对 context、request、session 对象的生命周期进行监控

在servlet2.5规范中共定义了 8 种Listener。在启动时，ServletContextListener 的执行顺序与 web.xml 中的配置顺序一致， 停止时执行顺序相反

````xml
<listener>
    <listener‐class>org.springframework.web.context.ContextLoaderListener</listener‐class>
</listener>
````





## Filter 配置

filter 用于配置web应用过滤器， 用来过滤资源请求及响应。 经常用于认证、日志、加密、数据转换等操作， 配置如下：

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



## 欢迎页面配置

当虚拟路径后没有路由时，自动引导至欢迎页

```xml
<welcome‐file‐list>
    <welcome‐file>index.html</welcome‐file>
    <welcome‐file>index.htm</welcome‐file>
    <welcome‐file>index.jsp</welcome‐file>
</welcome‐file‐list>
```

尝试请求的顺序，从上到下

 

## **5.7** 错误页面配置

error-page 用于配置 Web 应用访问异常时定向到的页面，支持HTTP响应码和异常类两种形式

```xml
<error‐page>
    <error‐code>404</error‐code>
    <location>/404.html</location>
    </error‐page>
<error‐page>
    <error‐code>500</error‐code>
    <location>/500.html</location>
    </error‐page>
<error‐page>
    <exception‐type>java.lang.Exception</exception‐type>
    <location>/error.jsp</location>
</error‐page>
```





## Tomcat 管理配置

Tomcat 提供了Web版的管理控制台，他们是两个独立的 Web 应用，位于 webapps 目录下

管理员的用户名和密码在`conf/tomcat- users.xml`中配置

- 用于管理的 Host 的host-manager`http://localhost:8080/host-manager/html`
- 用于管理Web应用的manager

 

# 集群



## Nginx + Tomcat 集群

独立运行两个 Tomcat，然后用 Nginx 进行负载均衡

为解决 Session 共享问题，可采取以下策略

- 令 Nginx 采用 ip_hash 策略，让同一个客户端只能访问同一台服务器
- 在 Tomcat 配置中开启 Session 复制（效率低下， 不用学）



## 共享Session

场景：一台电脑启动两个`Tomcat`服务。一个页面中，尝试获取并输出`SESSIONID`，并用k查询`SESSION`中的某个v。如果查不到v就输出没有并给 v 赋一个值

- 在`Tomcat`开启集群模式（`server.xml`和`web.xml`中各加一个标签即可开启）前，向两个`Tomcat`各发送一个请求，结果两个`SESSIONID`不同，且都获取不到k的v
- 在开启进群模式后，两个请求中，第二个请求获取到的`SESSIONID`和第一个请求获取到的相同，且第二次请求能获取到 k 的 v

PS：小型项目中可以用`Session`复制，大型项目中不用这种技术。官方说明：如果节点超过 4 个就不要用`Session`复制



推荐用单点登录（SSO）解决`Session`共享问题



# Tomcat 安全

## 配置安全

- 删除 webapps 目录下的所有文件，禁用 tomcat 管理界面
- 注释或删除 tomcat-users.xml 文件内的所有用户权限
- 更改关闭 tomcat 指令或禁用

> tomcat 的 server.xml 中定义了可以直接关闭 Tomcat 实例的管理端口（默认8005）。可以通过 telnet 连接上该端口之后，输入 SHUTDOWN （此为默认关闭指令）即可关闭 Tomcat 实例（注意，此时虽然实例关闭了，但是进程还是存在的）





## 传输安全 - Tomcat 开启 HTTPS

1. 生成密钥库文件

```shell
$ keytool ‐genkey ‐alias tomcat ‐keyalg RSA ‐keystore tomcatkey.keystore
```

输入对应的密钥库密码， 秘钥密码等信息之后，会在当前文件夹中出现一个秘钥库文件：tomcatkey.keystore

2. 将秘钥库文件 tomcatkey.keystore 复制到tomcat/conf 目录下
3. 配置 tomcat/conf/server.xml

```xml
<Connector port="8443" 
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="150" 
           schema="https" 
           secure="true"
           SSLEnabled="true">
    <SSLHostConfig certificateVerification="false">
        <Certificate  certificateKeystoreFile="D:/DevelopProgramFile/apache‐tomcat‐8.5.42‐windows‐x64/apache‐tomcat‐8.5.42/conf/tomcatkey.keystore"
                     certificateKeystorePassword="mypasswd" type="RSA" />
    </SSLHostConfig>
</Connector>
```



用 HTTPS 访问Tomcat

 

# WebSocket



## WebSsocket 介绍

`WebSocket`是`HTML5`新增的协议，它的目的是在浏览器和服务器之间建立一个不受限的双向通信的通道，比如说，服务器可以在任意时刻发送消息给浏览器。

Q：为什么传统的`HTTP`协议不能做到`WebSocket`实现的功能？

A：这是因为`HTTP`协议是一个请求 - 响应协议。只有浏览器能主动发送数据，服务器被动返回响应。服务器不能主动发数据给浏览器



直接轮询，或 Comet 机制（本质上也是轮询）效率低下

`WebSocket`标准实现让浏览器和服务器之间可以建立无限制的全双工通信，任何一方都可以主动发消息给对方

 `Websocket`并不是全新的协议，而是利用了`HTTP`协议来建立连接

![[../020 - 附件文件夹/Pasted image 20230401235935.png|625]]

首先， `WebSocket`连接必须由浏览器发起，因为请求协议是一个标准的`HTTP`请求，格式如下：

![[../020 - 附件文件夹/Pasted image 20230401235947.png]]

该请求和普通的HP请求有几点不同:

1. `GET`请求的地址不是类似`http://`而是以`ws://`开头的地址
2. 请求头`Connection: Upgrade`和请求头`Upgrade: websocket`表示这个连接将要被转换为`WebSocket`连接。（这个转换过程由浏览器控制，而不是我们控制）
3. `Sec-WebSocket-Key`是用于标识这个连接，是一个`BASE64`编码的密文，要求服务端响应一个对应加密的`Sec-WebSocket-Accept`头信息作为应答
4. `Sec-WebSocket-Version`指定了`WebSocket`的协议版本
5. `HTTP 101`状态码表明服务端已经识别并切换为`WebSocket`协议，`Sec-WebSocket-Accept`是服务端与客户端一致的秘钥计算出来的信息



## Tomcat 的 WebSocket

`Tomcat 7.0.5`版本开始支持`WebSocket`，并且实现了`Java Websocket`规范`JSR356`

`Java WebSocket`应用由一系列的`WebSocketEndpoint`组成

 `Endpoint`是一个`Java`对象，代表`WebSocket`链接的一端，对于服务端，我们可以视为处理具体`WebSocket`消息的接口，就像`Servlet`之与`http`请求一样

我们可以通过两种方式定义`Endpoint`

- 编程式，即继承类`javax.websocket.Endpoint`并实现其方法
- 注解式，即定义一个`POJO`，并添加`ServerEndpoint`相关注解



`Endpoint`实例在 `WebSocket`握手时创建，并在客户端与服务端链接过程中有效，最后在链接关闭时结束。在`Endpoint`接口中明确定义了与其生命周期相关的方法，规范实现者确保生命周期的各个阶段调用实例的相关方法。生命周期方法如下

| 方法    | 注解     | 含义描述                                                     |
| ------- | -------- | ------------------------------------------------------------ |
| onOpen  | @OnOpen  | 当开启一个新的会话时调用，该方法是客户端与服务端握手成功后调用的方法 |
| onClose | @OnClose | 当会话关闭时调用。                                           |
| onError | @OnError | 当连接过程中异常时调用。                                     |



通过为`Session`（`WebSocket`的`Session`，而不是`HTTP`的`Session`。用`websocket.getSession()`获取到对象）添加`MessageHandler`消息处理器来接收消息

当采用注解方式定义`Endpoint`时，可以通过`@OnMessage`注解指定接收消息的方法

发送消息则由`RemoteEndpoint`完成，其实例由`Session`维护，根据使用情况，可以通过`session.getBasicRemote()`获取同步消息发送的实例，然后调用其`sendXxx()`方法就可以发送消息

可以通过`session.getAsyncRemote()`获取异步消息发送实例



### 11.1.3 WebSocket DEMO案例

![[../020 - 附件文件夹/Pasted image 20230402000002.png]]

# Tomcat 性能调优

## Tomcat 性能测试

对于系统性能，用户最直观的感受就是系统的加载和操作时间，即用户执行某项操作的耗时

- 响应时间：请求的平均响应时间
- 吞吐量：即在给定的时间内，系统支持的事务数量，计算单位为 TPS

测试工具如： ApacheBench、ApacheJMeter、WCAT、WebPolygraph、LoadRunner

前两个是 Apache 的，免费的。用这俩


命令行下载 `httpd‐tools`

| 参数                                           | 指标说明                                                     |
| ---------------------------------------------- | ------------------------------------------------------------ |
| Requests per second                            | 吞吐率:服务器并发处理能力的量化描述，单位是reqs/s，指的是在某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。这个数值表示当前机器的整体性能，值越大越好。 |
| Time per request                               | 用户平均请求等待时间：从用户角度看，完成一个请求所需要的时间 |
| Time per equest:across all concurrent requests |服务器平均请求等待时间：服务器完成一个请求的时间|
| Concurrency Level                              | 并发用户数                                                   |


## Tomcat 性能优化



### JVM 参数调优

在禁用调`Tomcat`的管理页面后，可使用`jmap`或其他工具查看当前启动的`Tomcat`的堆内存使用情况



在`Sun`公司推出的 Hotspot 中，包含以下几种不同类型的垃圾收集器

| 垃圾收集器                                      | 含义说明                                                     |
| ----------------------------------------------- | ------------------------------------------------------------ |
| 串行收集器<br/>(Serial Collector)               | 采用单线程执行所有的垃圾回收工作，适用于单核cpU服务器，无法利用多核硬件的优势 |
| 并行收集器<br/>(Parallel Collector)             | 又称为吞吐量收集器，以并行的方式执行年轻代的垃圾回收<br/>该方式可以显著降低垃圾回收的开销(指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态)。<br/>适用于多处理器或多线程硬件上运行的数据量较大的应用 |
| 并发收集器<br/>(Concurrent Collector)           | 以并发的方式执行大部分垃圾回收工作，以缩短垃圾回收的暂停时间。<br/>适用于那些响应时间优先于吞吐量的应用，<br/>因为该收集器虽然最小化了暂停时间(指用户线程与垃圾收集线程同时执行，但不一定是并行的，可能会交替进行)，<br/>但是会降低应用程序的性能 |
| CMS收集器<br/>(Concurrent Mark Sweep Collector) | 并发标记清除收集器，适用于那些更愿意缩短垃圾回收暂停时间并且负担的起与垃圾回收共享处理器资源的应用 |
| G1收集器<br/>(Garbage-First Garbage Collector)  | 适用于大容量内存的多核服务器，可以在满足垃圾回收暂停时间目标的同时，以最大可能性实现高吞吐量(JDκ1.7之后) |



不同的应用程序，对于垃圾回收会有不同的需求。`JVM`会根据运行的平台、服务器资源配置情况选择合适的垃圾收集器、堆内存大小及运行时
编译器。

如无法满足需求，参考以下准则

- 程序数据量较小，选择串行收集器。
- 应用运行在单核处理器上且没有暂停时间要求，可交由J自行选择或选择串行收集器。
- 如果考虑应用程序的峰值性能，没有暂停时间要求，可以选择并行收集器。
- 如果应用程序的响应时间比整体吞吐量更重要，可以选择并发收集器

![[../020 - 附件文件夹/Pasted image 20230402000052.png]]

查看`Tomcat`中的默认的垃圾收集器

1. 在`tomcat/bin/catalina.sh`的配置中，加入如下配置

```shell
JAVA_OPTS="-Djava rmi server hostname192.168.192. 138-Dcom sun management. jmxzemote port=8999 Dcom. sun management. jmxremote rmi port=8999 -Dcom. sun management. jmxremote 3s1=false Dcom. sun. management. jmxremote authenticate=false"
```



2. 打开`jconsle`，查看远程的`tomcat`的概要信息

   连接远程`tomcat`



GC参数

| 参数                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| -XX:+UseSerialGC         | 启用串行收集器                                               |
| -XX:+UseParallelGC       | 启用并行垃圾收集器，配置了该选项，那么-XX:+UseparallelOldGC默认启用 |
| -XX:+UseParallelOldGC    | FullGC采用并行收集，默认禁用。如果设置了-XX:+UseparallelGC则自动启用 |
| -XX:+UseParNewGC         | 年轻代采用并行收集器，如果设置了-XX:+UseConcMarkSweepGC选项，自动启用 |
| -XX:ParallelGCThreads    | 年轻代及老年代垃圾回收使用的线程数。默认值依赖于JVM使用的CPU个数 |
| XX: +UseConcMar kSweepGC | 对于老年代，启用CMS垃圾收集器。当并行收集器无法满足应用的延迟需求是，推荐使用CMS或G1收集器<br/>启用该选项后，-XX:+UseParNewGC自动启用。 |
| -XX:+UseG1GC             | 启用G1收集器。G1是服务器类型的收集器，用于多核、大内存的机器。它在保持高吞吐量的情况下，高概率满足GC暂停时间的目标 |



我们也可以在测试的时候，将参数调整之后，将Gc的信息打印出来，便于为我们进行参数调整提供依据，具体参数如下：

| 选项                                   | 描述                                                   |
| -------------------------------------- | ------------------------------------------------------ |
| -XX:+PrintGC                           | 打印每次GC的信息                                       |
| -XX: +PrintGCApplicationConcurrentTime | 打印最后一次暂停之后所经过的时间，即响应并发执行的时间 |
| -XX: +PrintGCApplicationStoppedTime    | 打印GC时应用暂停时间                                   |
| -XX:+PrintGCDateStamps                 | 打印每次GC的日期戳                                     |
| -XX:+PrintGCDetails                    | 打印每次GC的详细信息                                   |
| -XX: +PrintGcTaskTime Stamps           | 打印每个GC工作线程任务的时间戳                         |
| -XX: +PrintGCTimeStamps                | 打印每次GC的时间戳                                     |


在bin/ catalina.sh的脚本中，追加如下配置

```shell
JAVA OPTS="-XX: +Use ConcMarkSweepGC -XX:+PrintGCDetails
```



### 配置优化

> `Tomcat`配置调优通常是通过修改连接器提高性能的

调整`tomcat/conf/server.xml`中关于链接器的配置可以提升应用务器的性能


| 参数           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| maxConnections | 最大连接数，当到达该值后，服务器接收但不会处理更多的请求，额外的请求将会阻塞直到连接数低于<br/>Maxconnections。可通过ulimit-a查看服务器限制。对于CPU要求更高（计算型）时，建议不要配置过大；<br/>对于CPU要求不是特别高时，建议配置在200左右。当然这个需要服务器硬件的支持 |
| maxThreads     | 最大线程数，需要根据服务器的硬件情况，进行一个合理的设置     |
| acceptCount    | 最大排队等待数，当服务器接收的请求数量到达 maxconnections，此时 Tomcat会将后面的请求，存放在任务队列中<br/>进行排序， acceptcount指的就是任务队列中排队等待的请求数。一台 Tomcat的最大的请求处理数量，是<br/>maxConnections+accept. |



