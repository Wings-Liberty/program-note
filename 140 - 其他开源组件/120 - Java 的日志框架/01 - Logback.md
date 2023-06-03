
# Chapter 1: Introduction

简单介绍了 Logback：它是一个记录日志的工具，它是 `slf4j-api` 的实现，还提供了其他日志框架没有的功能

在用它的记录日志时候，通常只会在类里导入 slf4j 的类，单从代码里是感受不到实现类是 logback。比如下面的代码

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

上述代码会使用默认配置文件的配置

- 把 `ConsoleAppender` 添加到 root logger 并把日志打印到控制台上（Appender 是日志数据的目的地，不同的 Appender 实现会把日志输出到不同的地方，比如控制台，磁盘文件，syslog，TCP Socket，JMS 等）
- 用默认的日志格式打印日志，比如上面的代码会打印以下内容

```
20:49:07.962 [main] DEBUG chapters.introduction.​HelloWorld1 - Hello world.
```

logback 内置了一个状态系统，它用来监控 logback 自己，比如打印 logback 自己相关的日志，定位 logback 本身存在的问题。由于这部分不是重点，所以可以略过

如果没有设置配置文件，logkack 会用默认的配置文件。所以项目接入 logback 很简单

1. 在 classpath 里添加：`slf4j-api.jar`，`logback-core.jar`， `logback-classic.jar`
2. 不创建配置文件（用默认内置的配置文件）或创建文件
3. 调用上面的代码打印日志

上面的内容涉及到了一些专业名词，会在后面的章节一一介绍


# Chapter 2: Architecture

