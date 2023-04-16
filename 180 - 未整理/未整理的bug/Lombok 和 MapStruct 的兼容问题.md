## 问题描述


场景：项目引入了 lombok 和 mapstruct。项目用到了 lombok 的 @Data，@Getter，@Sl4j 等注解，又自定义了一个 MapStruct 的接口


问题：启动项目后，用到 @Sl4j 注解的类在编译时抛找不到变量符号 log 的异常，log 变量本应该是 @Sl4j 在编译期间自行创建的变量

## 解决方案

项目起初仅引入了 lombok 和 mapstruct 的依赖，mapstruct 的依赖包含 mapstruct 和 mapstruct-processor

```xml
<dependency>  
	<groupId>org.mapstruct</groupId>  
	<artifactId>mapstruct</artifactId>  
	<version>${org.mapstruct.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct-processor</artifactId>  
    <version>${org.mapstruct.version}</version>  
</dependency>

<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId>  
    <version>${org.projectlombok.version}</version>  
</dependency>
```

```xml
<org.projectlombok.version>1.18.20</org.projectlombok.version>  
<lombok-mapstruct-binding.version>0.2.0</lombok-mapstruct-binding.version>  
```


在 base-chat 项目中

尝试官方提供的解决 lombok 和 mapstruct 冲突方案（按照官方 github 里的 pom.mxl 进行了配置）后冲突依然存在，项目还是不能启动

PS：官方 github 上的 lombok 用了 `<scope>provided</scope>` 但她会让 lombok 失效，更启动不起来

在 auto-enrollment 项目中

官方提供的解决方案是可行的

可能是因为 base-chat 其实报错报的不是 lombok 和 mapstruct 的冲突
