## 待完成

- 09中的`finally`块
- 为什么匿名内部类中使用的局部变量都需要被声明为`final`（和变量的声明周期有关）

![[../020 - 附件文件夹/Pasted image 20230401225404.png|475]]

## 问题

`import`

- 如果import指定的是具体的类或接口，导入指定包下的public类或接口，便可以在此.java文件中使用类或接口
- 如果import指定*，导入包所在目录下的类或接口，但是不导入子包中的类或接口
- .java文件默认导入java.lang包，无需主动声明

`import static`

- 导入某个类下的静态方法和静态属性
- finally和return，见[博客](https://blog.csdn.net/loongshawn/article/details/50489706) 和 **JVM虚拟机P238的字节码原理讲解**

`枚举类在JVM中的存在形式`

- [参考博客](https://www.cnblogs.com/alter888/p/9163612.html)

`时间点类和日历类`

- Date  主要是用于获取时间点（当前距离“纪元”的毫秒数。（ epoch）, 它 是 UTC 时间 1970 年 1 月 1 日 00:00:00。

  UTC 是 Coordinated Universal Time 的缩写）

- LocalDate 主要用于日历操作

- 创建这两个类的目的是将时间点操作和日历操作分开，所以Date中很多和日历操作相关的方法都被弃用（现在 Date 基本弃用）

- 日历操作指，获取某个时间点在日历中的信息，如月份，年份，某个时间点是本月第几周，本周第几等操作

- `System.currentTimeMillis()`也能获取时间点信息。

`更改器和访问器`

- 更改器和访问器指的是类的方法，用于修改，访问对象的属性。通常访问器只用来访问并返回指定数据，修改器用于修改对象属性，在修改时可能会访问对象的属性
- 属性的get和set方法就是更改器和访问器的一种实现
- 强调更改其和访问器是因为有不可变的类的存在，不可变的类的对象的方法想要修改对象的属性时通常时构造一个新对象并返回，而调用方法的对象的属性并没有被修改。修改器方法重点在，它修改了调用方法的对象的属性

`可变长度参数`

- 在虚拟机中，或者在方法栈中，可变参数是怎么从多个参数被封装到以一个数组中的？
- `Stream<String> lines = Files.lines(path);`获取文件的Stream流，来操作文件的所有行

`instanceof`

```java
// Double 的 equals判断参数是不是Double。不是，返回false。其他包装类类似
public boolean equals(Object obj) {
    return (obj instanceof Double)
        && (doubleToLongBits(((Double)obj).value) ==
            doubleToLongBits(value));
}
```



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



