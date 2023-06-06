
# Chapter 1: Introduction

> 参考自 Logback 的 [Introduction 章节](https://logback.qos.ch/manual/introduction.html)

本章简单介绍了 Logback：它是一个记录日志的工具，它实现了 `slf4j-api` ，还提供了其他日志框架没有的功能

在用它的记录日志时候，通常只会在类里导入 `slf4j` 的类，单从代码里是感受不到实现类是 `logback`。比如下面的代码

```java
package chapters.introduction;
import org.slf4j.Logger; 
import org.slf4j.LoggerFactory; 
public class HelloWorld1 { 
	public static void main(String[] args) {
		Logger logger = LoggerFactory.getLogger(​"chapters.introduction.​HelloWorld1"); 
		logger.debug("Hello world."); 
	}
}
```

上述代码会用默认配置文件的配置

- 把 `ConsoleAppender` 添加到 `root logger` 并把日志打印到控制台上（`Appender` 是日志数据的目的地，不同的 Appender 实现会把日志输出到不同的地方，比如控制台，磁盘文件，syslog，TCP Socket，JMS 等）
- 用默认的日志格式打印日志，比如上面的代码会打印以下内容

```
20:49:07.962 [main] DEBUG chapters.introduction.​HelloWorld1 - Hello world.
```

logback 内置了一个状态系统，它用来监控 logback 自己，并把每个 logger 的信息实时放到 log 的上下文对象里，比如打印 logback 自己相关的日志，定位 logback 本身存在的问题。由于这部分不是重点，所以可以略过

如果没有设置配置文件，logkack 会用默认的配置文件。所以项目接入 logback 很简单

1. 在 `classpath` 里添加：`slf4j-api.jar`，`logback-core.jar`， `logback-classic.jar`
2. 不创建配置文件（用默认内置的配置文件）或创建文件
3. 调用上面的代码打印日志

上面的内容涉及到了一些专业名词，会在第二章章节介绍


# Chapter 2: Architecture

> 参考自 Logback 的 [Architecture 章节](https://logback.qos.ch/manual/architecture.html)

详细介绍了：对于 Logback 的使用者来说，Logback 的重要组件有什么，有什么用。还讲解了打印一条日志时的时序图调用


## Logback 的基本架构

它被分成了三个模块
|模块|作用 |
|:--|:--|
|logback-core |基本依赖包，被其他两个模块依赖|
|logback-classic |实现了 slf4j-api|
|logback-access |专门提供 HTTP 日志的功能|

`logback-access` 暂时不是我关注的重点，所以以下内容均围绕 `logback-classic`

`logback-core` 和 `logback-classic` 提供 3 大 class 供调用者用（当然还有其他重要组件，只不过这 3 个最常用）

它们分别是：`Logger`, `Appender` 和 `Layout`

> [!tip] 只有 Logger 是 classic 包的，其他两个类都是 core 包的
> 


## Logger - 日志记录器

日志框架和 `System.out.println` 最大的区别就是前者能让特定的打印语句不执行，选择性地输出日志

Logback 用 `Logger` 作为 API 供调用者调用，每个 Logger 都有一个名字作为唯一标识符。Logger 用以下方式创建

```java
Logger rootLogger = LoggerFactory.​getLogger(org.slf4j.Logger.​ROOT_LOGGER_NAME);
```

Logger 的名称是唯一的，第一次调用会创建对象，再次调用就是复用之前创建的对象了。也就是[[../../021 - 离线网页备份文件夹/设计模式 - 享元模式.html|设计模式 - 享元模式]]


### Logger 的继承关系

不同的 `Logger` 对象之间存在父子关系，规则如下：如果 `Logger` 的名字加上一个 `.` 后就是另一个 `Logger` 名字去掉最后一个单词后的名字，那前者就是后者的 `父`。举个例子

- 名字叫 `com.foo` 的 `Logger` 是名字叫 `com.foo.Cat` 的 `父`
- 名字叫 `com.foo` 的 `Logger` 是名字叫 `com.foo.sub.Dog` 的 `祖先`

所有 `Logger` 有一个共同的祖先 --  `Root Logger`，通常它的名字是 `org.slf4j.Logger.​ROOT_LOGGER_NAME` 的值


### Logger 的有效输出等级

日志输出等级有：`TRACE`，`DEBUG`，`INFO`，`WARN`，`ERROR`

它是 `Logger` 对象必须有的属性，如果创建 `Logger` 时没有指定等级，它会继承父 `Logger` 的等级，如果父 `Logger` 也没有等级，就接着向上递归找，直到找到有等级的祖先

`Root Logger` 的默认等级是 `DEBUG`

`Logger` 提供了和输出等级同名的方法用以输出日志，每次调用都会发送一个 `logging request`

对于 `Logger` 来说，当收到一条 `logging request` 时，如果请求等级 >= `Logger` 当前设置的有效输出等级时 `Logger` 才会处理这个请求

等级排序为：`TRACE < DEBUG < INFO <  WARN < ERROR`

> [!tip] `Logger` 的等级也可以设置为 `OFF` 表示不处理任何等级的 `logging request`
> 虽然也可以设置等级为 `OFF`，但通常讨论等级时不会列举 `OFF`，因为它不是真正的等级，而是一个开关


## Appenders 日志追加器

调用 `logger.info()` 后日志内容到底是打印在控制台，文件还是其他地方是由 `Logger`  对象里的 `Appender` 控制的

`Logger` 可以同时持有多个 `Appender` 对象，所以一条日志可以在打印到控制台的同时又写入到日志文件里

logback 的默认配置文件里给 `root logger` 设置了 `ConsoleAppender`，它会把日志打印到控制台


### Appender 的继承获取

一个 `Logger` 对象里持有的 `Appender` 有两个来源

1. 定义 `Logger` 时为其指定的 `Appender`，比如

```xml
<!-- 控制台输出 -->  
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">  
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
        <pattern>${CONSOLE_LOG_PATTERN}</pattern>  
        <charset>utf8</charset>  
    </encoder>  
</appender>

<logger name="com.foo.sub" level="INFO"/>
	<appender-ref ref="STDOUT"/>
</logger>
```

2. 从父 `Logger` 那继承 `Appender`

如果 `com.foo`（一个 `Logger` 对象的名字） 有一个叫 "FILE_APPENDER" 的 `Appender`，`com.foo.sub` （另一个 `Logger` 对象的名字） 也会有它

如果不想从父 `Logger` 那获取 `Appender`，就需要在定义 `Logger` 时设置一个标志位表示不从父 `Logger` 继承 `Appender`


### 参数化的 logging request 方法

下面这种调用效率低，因为调用前需要拼接字符串。调用后如果 `logger` 的等级小于 `debug` 就不处理这个 loggin request，那刚才的字符串拼接就白做了

```java
logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
```

最好的做法是像下面这样

```java
if (logger.isDebugEnabled()) {
	logger.debug("Entry number: {} is {}", i, String.valueOf(entry[i]));
}
```

这样做有以下几个优点

- 用 `{}` 参数化可以有效避免字符串拼接的开销，因为只有当 `logger` 决定处理这条 `logging request` 时才会格式化这个字符串
- 提前判断等级可以减少构造参数的开销。虽然 `{}` 避免了拼接字符串，但传入的参数经常会有 `.getXxx` 之类的方法，像 `String.valueOf(entry[i])` 这种方法的开销更大（实际上不大，但对于日志框架来说却是不容忽略的开销）

## 一条 logging request 的完整调用链是这样的

1. 被 `TurboFilterList` 过滤器链过滤

获取一个 `TurboFilterList` 过滤器链（这是一个全局过滤器），经过过滤器链计算后会得到一个枚举对象 `FilterReply`

|枚举值|含义|
|:--|:--|
|FilterReply.DENY|不处理这条 logging request|
|FilterReply.NEUTRAL|执行第 2 步，判断 logging request 等级是否大于 Logger 的有效等级|
|FilterReply.ACCEPT|跳过第 2 步，直接执行第 3 步|

2. 判断 `logging request `等级是否大于 `Logger` 的有效等级

3. 创建一个 `LoggingEvent` 对象，里面包含了这条 `logging request` 和上下文信息

4. 遍历并调用这个 `Logger` 的所有 `Appender.doAppend()`

5. 调用 `Layout` 对象格式化日志（这一步是 `Appender.doAppend()` 的一部分）得到一个 `String` 后放到 `LoggingEvent` 里

6. 发送  `LoggingEvent`（这一步也是 `Appender.doAppend()` 的一部分）

时序图如下

![[../../020 - 附件文件夹/Pasted image 20230604132218.png]]


# Chapter 3: Logback configuration

> 参考自 Logback 的 [Configuration 章节](https://logback.qos.ch/manual/configuration.html)

加载 logback 配置分两步

1. 用 `SPI` 机制获取所有 `Configurator`，它是配置的加载器。如果用户没有自定义 `Configurator`，`logback` 将用 `DefaultJoranConfigurator` 加载配置（一般没人自定义加载器）
2. 用 Configurator 加载配置，如果用 DefaultJoranConfigurator 加载配置
	1. 它会在 `classpath` 下先找 `logback-test.xml`
	2. 找不到 `logback-test.xml` 就找 `logback.xml`
	3. 找不到 `logback.xml` 就加载默认配置 `BasicConfigurator`（给 `root logger` 设置一个 `ConsoleAppender`，`root logger` 的默认等级是 `DEBUG`）

配置文件的详细配置和书写方式参考 [Configuration 章节的语法部分](https://logback.qos.ch/manual/configuration.html#syntax)，这里不做赘述

# 其他章节

目前只需要掌握 Logback 的 3 大重要组件是什么，有什么用，它们之间是什么关系就够了

其他章节中基本都是高级使用方式或某个组件的详细介绍和怎么在配置文件里高度自定义它们

比如 [Chapter 4: Appenders](https://logback.qos.ch/manual/appenders.html) 详细说明了 `Appender` 的接口继承体系和几个重要的实现类怎么在 xml 配置文件中使用，比如 `ConsoleAppender`，`FileAppender`，`RollingFileAppender` 以及其他不常用的 `Appender`，比如：`SocketAppender` 和 `SSLSocketAppender`

这些内容用时现查即可