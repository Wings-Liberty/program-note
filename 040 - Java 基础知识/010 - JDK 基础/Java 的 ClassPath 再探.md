#还没有复习 

# 问题描述

**场景**

学习 Antlr 的使用时（Antlr 是一个 jar 包），需要运行此 jar 中的某些类。教程提示需要把此 jar 文件的地址（包括 jar 文件的文件名）添加的环境变量中（最后才知道，教程原意是把 jar 文件的地址添加到名为 CLASSPATH 的环境变量中）

教程信息如下

```
Windows
1. Download https://www.antlr.org/download/antlr-4.9.2-complete.jar.
2. Add antlr4-complete.jar to CLASSPATH, either:
	1. Permanently: Using System Properties dialog > Environment variables > Create or append to CLASSPATH variable
	2. Temporarily, at command line: SET CLASSPATH=.;C:\Javalib\antlr4-complete.jar;%CLASSPATH%
3. Create batch commands for ANTLR Tool, TestRig in dir in PATH
 antlr4.bat: java org.antlr.v4.Tool %*
 grun.bat:   java org.antlr.v4.gui.TestRig %*
```



**目录内容**

```
L antlr-4.9.2-complete.jar
L antlr4.bat
L grun.bat
```



问题：执行`antlr4.bat`时报`错误: 找不到或无法加载主类 org.antlr.v4.Tool`



# 问题分析

说明 antlr-4.9.2-complete.jar 没被放在 classpath 里。下面有几个问题

- classpath 是什么，有什么用，在程序运行 / 启动时承担什么责任
- 为什么把 jar 文件的位置添加到 classpath 中，就能直接使用 java 命令找到指定的类



# 参考博客

参考以下博客，重新学习，认识 jar 包和 classpath

[jar包是什么？](jar包是什么？)

[java启动jar包中的指定类](https://blog.csdn.net/weixin_30252709/article/details/95808239)

**[classpath和jar](https://www.liaoxuefeng.com/wiki/1252599548343744/1260466914339296)**

**[-jar参数运行应用时classpath的设置方法](https://www.cnblogs.com/aggavara/archive/2012/11/16/2773246.html)**

[jdk9后引入的模块化对classpath带来的影响](https://www.liaoxuefeng.com/wiki/1252599548343744/1281795926523938)

# 分析参考博客

**jar 包**

jar 包是 Java 代码文件的归档，jar 和 rar 类似，它的目的是把多个文件打包，归档，让多个文件看起来像一个文件。在网络上传输时，jar 也被视为一个文件传输。同时 jar 包也是一种压缩格式。所以把 Java 代码文件打为 **jar 包意味着归档和压缩**

jar 包中用 META-INF/MANIFEST.MF 文件描述 Java 程序信息，比如在文件中指定哪个类是主类。这让 jar 包变成一个可执行文件，通过`java -jar yourJarName`命令运行主类的 mian 方法



**classpath**

在很早之前就知道 JDK 中很多类都在 rt.jar 中，rt.jar 在 jdk/jre/lib 中。而 classpath 通常都会有 rt.jar 的文件地址。所以在使用 rt.jar 中的类时，以 MyClass 为例，类加载器会从 classpath 的多个目录下寻找 MyClass.class。结果前面几个目录或 jar 文件下都找不到，最后在某个目录 / jar 中找到了

所以 classpath 是一组目录 / jar 包的集合。当类加载器需要加载某个类时就会从 classpath 提供的地址中找类。如果找不到就抛 ClassNotFoundException

执行 java your.class.file.full.package.name 时，类加载器从 classpath 提供的目录 / jar 中找。此时并没有显式指定 classpath 的值

其默认值为系统环境变量中`CLASSPATH`的值（不推荐）或命令行中`-classpath`或`-cp`指定的值。如果系统环境变量没提供 CLASSPATH，也没有用命令行选项指定的话，默认值为`.`，即执行`java`命令时所在的目录



> 以下内容引用自廖雪峰的 Java 教程
>
> 
>
> 在 IDE 中运行Java程序，IDE 自动传入的`-cp`参数是当前工程的`bin`目录和引入的 jar 包。
>
> 通常，我们在自己编写的`class`中，会引用 Java 核心库的`class`，例如，`String`、`ArrayList`等。这些`class`应该上哪去找？
>
> 有很多 “如何设置 classpath ” 的文章会告诉你把 JVM 自带的`rt.jar`放入`classpath`，但事实上，根本不需要告诉 JVM 如何去Java核心库查找`class`，JVM 怎么可能笨到连自己的核心库在哪都不知道？
>
>  不要把任何 Java 核心库添加到 classpath 中！JVM 根本不依赖 classpath 加载核心库！
>
> 更好的做法是，不要设置`classpath`！默认的当前目录`.`对于绝大多数情况都够用了。



> 以下内容引用自廖雪峰的 java 教程
>
> jar 包就是 zip 包，所以，直接在资源管理器中，找到正确的目录，点击右键，在弹出的快捷菜单中选择 “发送到”，“压缩(zipped)文件夹”，就制作了一个zip文件。然后，把后缀从`.zip`改为`.jar`，一个jar包就创建成功
>
> **jar 包的说明文件**
>
> 在大型项目中，不可能手动编写`MANIFEST.MF`文件，再手动创建 zip 包。Java 社区提供了大量的开源构建工具，例如 [Maven](https://www.liaoxuefeng.com/wiki/1252599548343744/1255945359327200)，可以非常方便地创建 jar 包



# 解决方案

从以下几个方案中选一个

- ~~在系统环境变量中设置`CLASSPATH`的值，为其追加 jar 包的位置~~

- 执行 java 命令时用`-classpath`或`-cp`显式指定 classpath

  

# 总结

👴 终于终于知道 classpath 是啥了。知道了 classpath 后有助于更深入理解类加载器和类加载机制。了解程序启动时 JVM 加载类时都干了什么事

JVM？垃圾！Java？垃圾！