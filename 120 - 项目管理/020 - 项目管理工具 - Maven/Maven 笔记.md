#还没有复习 

# 构建maven项目的几个主要环节

①**清理**：删除以前的编译结果，为重新编译做好准备。

②**编译**：将 Java 源程序编译为字节码文件。

③**测试**：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性。

④**报告**：在每一次测试后以标准的格式记录和展示测试结果。

⑤**打包**：将一个包含诸多文件的工程封装为一个压缩文件用于安装或部署。Java 工程对应 jar 包，Web

工程对应 war 包。

⑥**安装**：在 Maven 环境下特指将打包的结果——jar 包或 war 包安装到本地仓库中。

⑦**部署**：将打包的结果部署到远程仓库或将 war 包部署到服务器上运行。


# Maven的核心概念

接下来从maven的九个核心概念入手

- pom
- 结构目录
- 坐标
- 依赖管理
- 仓库管理
- 生命周期
- 插件和目标
- 继承
- 聚合

maven的核心程序只定义了抽象的声明周期，而maven的具体操作都是由maven的插件完成的


## POM

`Project Object Model`项目对象模型


.pom文件



## 结构目录

![[../../020 - 附件文件夹/Pasted image 20230404221410.png|700]]

## 坐标


坐标 可以定位到一个maven工程

```xml
<groupId>com.atguigu.maven</groupId>
<artifactId>Hello</artifactId>
<version>0.0.1-SNAPSHOT</version>
```


## 依赖管理



### 依赖的目的

在Maven工程A中使用工程B的类，这就需要让A依赖B

```xml
<dependency>
    <groupId>com.atguigu.maven</groupId> 
    <artifactId>Hello</artifactId>
    <version>0.0.1-SNAPSHOT</version> 
    <scope>compile</scope>
</dependency>
```


### 依赖的范围

指上述scope标签的值。有以下三个值可用

compile、test、provided


maven构建的项目src下有main和test

main目录存放主程序

test存放测试程序

![[../../020 - 附件文件夹/Pasted image 20230404221429.png|700]]

compile的依赖可在main和test中使用

test的依赖只能在test中使用

![[../../020 - 附件文件夹/Pasted image 20230404221602.png|700]]

参与部署是啥先跳过


### 依赖的传递性

场景：A依赖B，B依赖C

问题：A能用C吗？

答：只有B依赖C时使用的时compile时A才能用C



### 依赖的排除

```xml
<dependency>
     <groupId>com.atguigu.maven</groupId>
     <artifactId>HelloFriend</artifactId>
     <version>0.0.1-SNAPSHOT</version>
     <type>jar</type>
     <scope>compile</scope>
    <!--排除掉指定的依赖-->
     <exclusions> 
         <exclusion> 
             <groupId>commons-logging</groupId> 
             <artifactId>commons-logging</artifactId> 
         </exclusion> 
     </exclusions> 
</dependency>
```



### 声明变量指定版本号

在pom文件中使用properties标签自定义变量，便于用变量名指定版本号

```xml
<properties>
 	<atguigu.spring.version>4.1.1.RELEASE</atguigu.spring.version>
</properties>
```


```xml
<dependencies>
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-core</artifactId>
         <version>${atguigu.spring.version}</version>
     </dependency>
</dependencies>
```


### 版本依赖控制中心


### 依赖的原则，jar包冲突

![[../../020 - 附件文件夹/Pasted image 20230404221612.png|700]]

## 仓库管理

### 仓库分类

本地库

远程库

​	私服

​	maven中央仓库

​	中央仓库的镜像站



### 仓库中有什么

- maven插件
- 自己开发的项目模块
- 第三方框架或jar包


## maven的生命周期

①Clean Lifecycle 在进行真正的构建之前进行一些清理工作。

②Default Lifecycle 构建的核心部分，编译，测试，打包，安装，部署等等。

③Site Lifecycle 生成项目报告，站点，发布站点。


可以直接运行 mvn clean install site 运行所有这三套生命周期



核心构建流程

①**清理**：删除以前的编译结果，为重新编译做好准备。

②**编译**：将 Java 源程序编译为字节码文件。

③**测试**：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性。

④**报告**：在每一次测试后以标准的格式记录和展示测试结果。

⑤**打包**：将一个包含诸多文件的工程封装为一个压缩文件用于安装或部署。Java 工程对应 jar 包，Web

工程对应 war 包。

⑥**安装**：在 Maven 环境下特指将打包的结果——jar 包或 war 包安装到本地仓库中。

⑦**部署**：将打包的结果部署到远程仓库或将 war 包部署到服务器上运行。



几个核心命令mvn 

clean 清理当亲工程

compile 编译项目的代码

test 使用合适的单元测试框架进行测试，测试代码不会被打包

package 打包，如将项目打包称.jar文件

install 将打好的包安装至本地仓库，供其他项目依赖


自动化构建：运行任何一个阶段的时候，它前面的所有阶段都哦会被运行

例如，执行mvn install 时将会执行编译，测试，打包，将包安装到本地库


## 插件和目标

- Maven 的核心仅仅定义了抽象的生命周期，具体的任务都是交由插件完成的。
- 每个插件都能实现多个功能，每个功能就是一个插件目标。
- Maven 的生命周期与插件目标相互绑定，以完成某个具体的构建任务。

例如：compile 就是插件 maven-compiler-plugin 的一个目标；pre-clean 是插件 maven-clean-plugin 的一个目标。

![[../../020 - 附件文件夹/Pasted image 20230404221629.png|550]]

## 继承


父子工程

每个maven工程的pom文件中均用parent标签指定另一个maven工程为其父工程

父工程的打包方式应该为pom

子工程的打包方式为jar或war

问：父工程能干什么？

答：

- 子工程能使用父工程的所有依赖（不是重点）
- 父工程中在dependencyManagement标签中显式声明依赖的版本，子工程使用相同的依赖时直接使用父工程中指定的版本，不需要再显式声明版本号


## 聚合

聚合是什么？


创建多模块项目时，要发布项目就需要对所有模块打包

修改代码后需要重新打包。因为有多个模块，所以不方便



聚合就是所有子工程都使用 同一个父工程

在父工程的pom文件中使用modules标签指定所有模块的位置



打包时，根据父工程指定的子项目一次性打包所有项目



