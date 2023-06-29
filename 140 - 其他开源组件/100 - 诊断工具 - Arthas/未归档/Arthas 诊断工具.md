#还没有复习 

# OGNL 表达式使用方式


# 常见使用场景 & 常用命令详解

## 获取 Spring IOC 容器 & 调用简单的静态方法

1. 获取静态类的类加载器
2. 调用静态方法

```java
sc -d cn.hutool.extra.spring.SpringUtil
```

假设得到的哈希值是 `377dca04`

```java
ognl '@cn.hutool.extra.spring.SpringUtil@getApplicationContext()' -c 377dca04
```

## 获取 SpringBoot 配置文件和文件内容

**查询当前使用的配置文件**

实现方式：Spring 容器 ApplicationContext 实现了 Environment，这个接口提供了获取环境信息的方法

```java
String[] getDefaultProfiles();
String[] getActiveProfiles();
```

也可以直接用 SpringUtil 封装的静态方法

```java
ognl '@cn.hutool.extra.spring.SpringUtil@getActiveProfiles()' -c 377dca04

ognl '@cn.hutool.extra.spring.SpringUtil@getApplicationContext().getEnvironment().getDefaultProfiles()' -c 377dca04
```

**查询配置文件中的值**

```java
ognl '@cn.hutool.extra.spring.SpringUtil@getApplicationContext().getEnvironment()..getProperty("sql.debug")' -c 377dca04
```


## 查询静态变量和成员变量

**查询静态变量实现方式**：获取到类对象后，用 `.` 访问符访问即可

```java
ognl '@com.cx.service.Service@SERVICE_NAME' -c 377dca04
```

或者也可以用 `getstatic` 命令。不过这个命令也没有代码补全

```java
getstatic com.cx.service.Service SERVICE_NAME
```


> [!NOTE] arthas 没有提供直接修改静态变量的方式
> 直接用 `=` 修改静态变量会报错
> 
> 虽然可以调用反射的方法修改静态变量的值，不过这会有点麻烦


**查询成员变量实现方式**：获取到对象后，用 `.` 访问符访问即可

```java
ognl '@cn.hutool.extra.spring.SpringUtil@getApplicationContext().getBean("service").hello' -c 377dca04
```

限制：如果成员变量是一个复杂对象，直接访问成员变量 a，遍历 a 的深度仅为 1。比如直接访问 `service`

![Pasted image 20221207003658](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207003658.png)

log 是一个复杂对象，但现在看不到 log 对象的细节


**修改成员变量实现方式**：获取到对象后，用 `=` 赋值即可

在赋值前，需要先修改 `arthas` 的设置，因为默认 `arthas` 不允许直接修改成员变量的值

```java
options strict false

ognl '@cn.hutool.extra.spring.SpringUtil@getApplicationContext().getBean("service").hello="modify"' -c 377dca04
```

## 调用参数复杂的静态方法和成员方法


实现方式：在 ognl 表达式里创建好所有参数对象后再调用方法

比如调用静态方法

```java
ognl '#user=new com.cx.bean.entity.User(), #user.username="test",@com.cx.util.TestUtil@echoUser(#user)' -c 377dca04

// 也支持 null 关键字
ognl '#user=null,@com.cx.util.TestUtil@echoUser(#user)' -c 377dca04
```

调用成员方法也一样

```java
ognl '#user=new com.cx.bean.entity.User(), #user.username="test", #user.password="test", @cn.hutool.extra.spring.SpringUtil@getApplicationContext().getBean("service").getHashCode(#user)' -c 377dca04
```

## 监控方法调用的入参，返参和异常 - watch

arthas 浅使用阶段不考虑用 `tt`，因为 `tt` 使用方式比较繁琐

- [ ] 写 `watch` 和 `tt` 的详细使用方式

根据 `watch` 的特点，使用 `watch` 的方式有两种

- 监控方法调用前，方法的入参
- 监控方法调用后，方法的入参，返参和异常（方法调用后入参的值可能已经被方法修改了，所以获取到的入参值是执行过方法调用以后的值）

```java
watch 类名 函数名 [观察表达式] [条件表达式] [选项]
```


```java
// 在函数调用前观察。由于观察事件点是在函数调用前，此时返回值或异常均不存在
watch com.cx.controller.TestController * {params,target} -b -x 2
```

```java
// 默认在函数结束之后(正常返回和异常返回)观察
watch com.cx.service.Service * {params,returnObj,throwExp} -x 2
```


> [!NOTE] `watch` 等命令也能结合 `grep` 命令使用
> 比如我想看异常信息，但打印异常时经常会打印整个调用链。如果用 `grep` 就可以只看想看的部分，比如可以用 `watch ... | grep com.cx` 只查看业务包里的异常


> [!NOTE] `-b` 在调用方法前就会在 `arthas` 上输出日志
> 当方法传参不符合预期时，方法的执行时间可能会非常长。如果用 `-b` 就可以在方法执行前就获取到入参，不过 `-b` 会取不到方法返回值和异常


## 反编译代码并输出到文件里

`jad` 输出的代码里携带的代码行数和源代码里的代码函数是一致的

```java
// --source-only 是常用选项
jad 全类名 [选项] > /home/tmp/Test.java
```

```java
jad com.cx.controller.TestController --source-only > /opt/TestController.java
```

在 vim 里打开文件就能看源代码了


## 热部署

arthas 提供的热部署功能存在限制，而且进行一次热部署需要执行好几条命令

**所以直接用 arthas IDEA 插件**

**如果修改的内容过多，或者修改超出了热部署的限制，直接放弃热部署**，老实重新打包，上传，部署

参考[这里](https://www.yuque.com/arthas-idea-plugin/help/pwxhb4)。简单地说，在本地修改完代码后，用插件生成一条超长命令，放到服务器上执行就好了。不过被修改的代码有一些限制，不是所有的修改都能生效，并且 arthas 有些命令会导致被热部署的 .class 重置回最初的版本

比如修改 `/test` 接口的返回值，之前返回 `test` 现在改为返回 `hot swap`

重新编译这个类

![Pasted image 20221207152336](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207152336.png)

然后用插件获取热部署命令

![Pasted image 20221207152144](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207152144.png)

得到了一长串命令后，在服务器上直接执行。注意，不是在 arthas 的控制台执行，是在 ssh 的 shell 里直接执行

修改前得到的响应结果是 `test`

![Pasted image 20221207152517](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207152517.png)

修改后得到的响应结果是 `hot swap`

![Pasted image 20221207152631](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207152631.png)


如果热部署的 class 里添加了没有被加载的类，执行热部署脚本时会报错。但是貌似 util 包里的类除外（好像是因为 jdk 自己的类就可以不用指定）

![[../../../020 - 附件文件夹/Pasted image 20230628202839.png]]




## 修改日志输出级别

用 logger 指令可以修改日志输出级别

修改日志级别需要先找到 logger 的名字，然后用 logger 命令修改

所以先找指定的 logger 的名字

比如要修改 com.cx.controller.TestController 的日志输出级别

```java
// 查询有 appender 的 logger
logger
// 查询所有 logger
logger --incloud-no-appender
```

通常没有 appender 的 logger 比较多，所以用 grep 过滤一下

```java
logger --incloud-no-appender | grep com.cx.controller
```


> [!warning] 遇到过调用 `logger --incloud-no-appender` 还是只有带 appendper 的 logger
> 重启 arthas 才解决了问题
> 
> 经过分析，复现步骤为先调用
> `logger --incloud-no-appender | grep name | grep com.cx.controller`，发现没有输出后，再调用 `logger --incloud-no-appender` 就不要好用了。
> 但如果用 `logger --incloud-no-appender | grep com.cx.controller` 就没问题


![Pasted image 20221207154431](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207154431.png)

然后获取类的加载器，一般 logger 用的类加载器和这个类一样

修改前，logger 的输出级别和效果是这样

![Pasted image 20221207154839](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207154839.png)

访问 `/test`，只输出了 info 的

![Pasted image 20221207154632](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207154632.png)

```java
logger --name com.cx.controller.TestController --level debug -c 377dca04
```

修改以后

![Pasted image 20221207154935](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207154935.png)

再访问 `test`，就能看到效果

![Pasted image 20221207155002](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020221207155002.png)

但是注意，上述的日志均是控制台输出的，只不过启动命令把控制台的输出重定向到了文件里。项目配置的 logback.xml 还会把日志输出到 info-yyyy-MM-dd.log 和 error-yyyy-MM-dd.log 里，但是它们配置了 filter

```xml
<!-- 控制台输出 -->  
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">  
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
        <pattern>${CONSOLE_LOG_PATTERN}</pattern>  
        <charset>utf8</charset>  
    </encoder>  
</appender>  
  
<!-- 生成日志文件 -->  
<appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">  
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
        <!-- 日志文件输出的文件名 -->  
        <FileNamePattern>target/log/info-%d{yyyy-MM-dd}.log</FileNamePattern>  
        <!--日志文件保留天数-->  
        <MaxHistory>180</MaxHistory>  
    </rollingPolicy>  
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
        <pattern>%n%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{traceId}] [%logger{50}] %n%-5level: %msg%n</pattern>  
    </encoder>  
    <!-- 打印日志级别。如果指定了日志级别为 info 那么 appender 就不会把 debug 日志记录到日志文件里-->  
    <filter class="ch.qos.logback.classic.filter.LevelFilter">  
        <level>INFO</level>  
        <onMatch>ACCEPT</onMatch>  
        <onMismatch>DENY</onMismatch>  
    </filter>  
</appender>  
  
<!-- 生成日志文件 -->  
<appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">  
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
        <!-- 日志文件输出的文件名 -->  
        <FileNamePattern>target/log/error-%d{yyyy-MM-dd}.log</FileNamePattern>  
    </rollingPolicy>  
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
        <pattern>%n%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{traceId}] [%logger{50}] %n%-5level: %msg%n</pattern>  
    </encoder>  
    <!-- 打印日志级别 -->  
    <filter class="ch.qos.logback.classic.filter.LevelFilter">  
        <level>ERROR</level>  
        <onMatch>ACCEPT</onMatch>  
        <onMismatch>DENY</onMismatch>  
    </filter>  
</appender>
```

LevelFilter 只会让符合设定级别的日志通过，其他级别的均被过滤掉

> 六方云的项目启动脚本会把控制台的输出重定向到 `/dev/null`。所以六方云的项目可能不能很好地使用修改日志级别功能
> 修改 LevelFilter 的效果很差（Level 是一个复杂对象，并且 LevelFilter 不是一个 Spring bean），所以暂时放弃此功能


- [ ] 查看 sql 语句
- [ ] 获取被代理的目标对象。对象经常会因为 aop，事务等被代理，如果直接获取 bean 得到的就是代理对象，所以有其他手段能取到代理对象。参考[这里](https://www.yuque.com/arthas-idea-plugin/help/uldktc)

- [ ] 详细介绍 watch 和 tt 命令

# 参考网站

[arthas 使用技巧和 arthas IDEA 插件大纲](https://www.yuque.com/arthas-idea-plugin)

# arthas-boot 离线版本改造计划

目标：让 arthas-boot 的整个运行过程不需要联网

应该只需要修改 `arthas-atthas-all-3.6.7` 的 `boot` 项目，也就是 `arthas-boot`

它最终会找 `~/.arthas/lib/3.6.7/arthas-core.jar` 执行


所以只需要把 `boot` 的联网部分代码去掉即可

然后写一个脚本

把 lib 包放到 ~/.arthas 下，

压缩 tar -zcf lib.tar.gz lib

解压缩 tar -zxf lib.tar.gz

移动 mv lib ~/.arthas/lib

arthas-cmd.sh
- `--install` `-i` 安装命令，比如 `arthas-cmd.sh -i lib.tar.gz`
- `--uninstall` `-u` 卸载。删除 ~/.arthas

# arthas-tunnel

不好用。因为它要求被监控的程序要添加一个 arthas 的依赖

# 使用案例


## xxl-job

场景：xxl-job 执行器执行时间比较长，执行出问题或被阻塞

需求：用 arthas 定位是哪行出了问题

### 新需求


查看指定方法里的局部变量的值，最好能指定查看这个局部变量在代码执行到第 n 行时 

- [ ] 方案一：热部署，加上log输出（不建议）
- [ ] 方案二：看看有没有直接看局部变量值的方法

火焰图可以用来看


跟踪指定的方法用 `-c` ，官网好像写错了，写成 `-e` 了，而且也没有使用案例

```
profiler start -c com.cdos.soar.flow.config.service.ActionServiceTask.execute
```


# ognl 表达式怎么写条件语句

