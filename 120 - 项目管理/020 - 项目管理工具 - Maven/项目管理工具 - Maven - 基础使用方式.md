#还没有复习 

# maven


## 1.项目打包存在包名过长问题

使用自定义打包名即可

```xml
<build>        
	<!--设置打war后的war包名-->
    <finalName>myProjectNmae</finalName>
</build>
```



## 2.依赖版本管理

使用

```xml
<properties>
	<model.version>0.0.1-SNAPSHOT</model.version>
</properties>
```

就像是定义了一个名字是model.version的变量一样

需要使用时就在使用的地方写

```xml
${model.version}
```

即可



## 3.父子工程常用知识

### 创建父子工程

1. 创建一个工程，type选择maven pom或packaging选择pom

   目的是让父工程的pom文件中有

   ```xml
   <packaging>pom</packaging>
   ```

2. 创建模块（子工程）new model

3. - 在父工程中添加子模块

   ```xml
   <modules>
      	<module>controller</module>
   	<module>bean</module>
   </modules>
   ```

   - 在子模块中声明父工程

   ```xml
   <!--修改默认生成的父工程声明为自定义的-->
     <parent>
     	<groupId>com.example</groupId>
     	<artifactId>parent</artifactId>
     	<version>0.0.1-SNAPSHOT</version>
   </parent>
   ```

     **目的**：子模块可以直接使用父工程中的依赖
   
4. controller模块调用bean模块的类

   - 在controller模块中添加bean模块的依赖

     ```xml
     <dependency>
     	<groupId>com.example</groupId>
     	<artifactId>bean</artifactId>
     	<version>0.0.1-SNAPSHOT</version>
     </dependency>
     ```

     

     如果添加依赖后没效果，选中所依赖的子模块，右键点击rebuild module ‘xxx’



### 填充代码

完成多模块的继承和依赖后，开始写代码

web的启动类需要继承springbootservletinitial并重写config方法（目的是在启动tomcat后可以启动webapp的spring，因此项目越多启动时间就越长）

````java
return build.sources(WebApplication.class);
````







### 打包

- 打包时需要跳过jar包的test

  1. 直接删除test文件夹

  2. 执行打包指令时带上参数

     ```
     mvn install -Dmaven.test.skip=true
     ```

- web模块是controller，mapper.xml，resources文件夹所在的模块

- web模块打包方式是war

- 父工程打包方式是pom

- 打包后只需要web模块的war包即可

- 为什么所有模块都需要有一个启动类

[打包时出现程序包xx找不到](https://www.cnblogs.com/ITPower/p/10015471.html)

关闭跨域不能访问的配置

```java
@Configuration
public class MyConfig implements WebMvcConfigurer {    
	private CorsConfiguration buildConfig() {        
		CorsConfiguration corsConfiguration = new CorsConfiguration();       							corsConfiguration.addAllowedOrigin("*");
   	    corsConfiguration.addAllowedHeader("*");        
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addExposedHeader("Authorization");        
        return corsConfiguration;    
    }   
    
    @Bean    
    public CorsFilter corsFilter() {     
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();        				source.registerCorsConfiguration("/**", buildConfig());     
        return new CorsFilter(source);   
        }   
    
    @Override   
    public void addCorsMappings(CorsRegistry registry) {    
        registry.addMapping("/**")        
                .allowedOrigins("*")             
                .allowCredentials(true)             
                .allowedMethods("GET", "POST", "DELETE", "PUT")          
                .maxAge(3600);    }
}
```



# mybatis逆向工程

## 1. 配置文件

   **注：targetPackage就是所在的com.xx.xx，targetProject填绝对路径，而且最好不要提前创建所填写的这些文件夹**

   ```xml
   <!--自定义文件名是-->
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE generatorConfiguration
           PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
           "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
   
   <generatorConfiguration>
   
       <context id="Generator" targetRuntime="MyBatis3">
   
           <!--        这个标签可以去注释-->
           <commentGenerator>
               <!--            去除注释-->
               <property name="suppressAllComments" value="true"/>
               <!--            去除时间戳-->
               <property name="suppressDate" value="true"/>
           </commentGenerator>
   
           <!--配置数据库信息-->
           <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                           connectionURL="jdbc:mysql://127.0.0.1:3306/share?serverTimezone=UTC&amp;useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false&amp;nullCatalogMeansCurrent = true"
                           userId="root"
                           password="root">
           </jdbcConnection>
   
           <!--        java JDBC数据类型转换-->
           <javaTypeResolver>
               <property name="forceBigDecimals" value="false"/>
           </javaTypeResolver>
   
           <!--javaModekGenerator javaBean配置
               targetPackage输入包名 输出路径
               targetProject输出项目位置
           -->
           <javaModelGenerator targetPackage="com.test.entity"
                               targetProject="D:\IDEA-workspace\project\entity\src\main\java\com\test\entity">
               <!--            是否开启子包名称，是否在报名后追加scheme名称-->
               <property name="enableSubPackages" value="false"/>
               <!--            在set中加入trim-->
               <property name="trimStrings" value="true"/>
           </javaModelGenerator>
   
           <!--        创建mapper.xml的位置-->
           <sqlMapGenerator targetPackage="mapper"
                            targetProject="D:\IDEA-workspace\project\web\src\main\resources\mapper">
               <property name="enableSubPackages" value="false"/>
           </sqlMapGenerator>
   
           <!--        java接口的位置-->
           <javaClientGenerator type="XMLMAPPER"
                                targetPackage="com.test.mapper"
                                targetProject="D:\IDEA-workspace\project\dao\src\main\java\com\test\mapper">
               <property name="enableSubPackages" value="true"/>
           </javaClientGenerator>
   
           <!--        数据库，本剧数据库中的表生成-->
           <table schema="share" tableName="user"/>
           <!--        <table schema="share" tableName="blog"/>-->
           <!--        <table schema="share" tableName="comment"/>-->
           <!--        <table schema="share" tableName="resources"/>-->
           <!--        <table schema="share" tableName="blog_user"/>-->
   
   
       </context>
   </generatorConfiguration>
   ```

## 2. 逆向工程代码

   **配置文件最好写绝对路径**

   ```java
   @Test
       public void generator() throws Exception{
           List<String> warnings = new ArrayList<String>();
           boolean overwrite = true;
           File configFile = new File("D:\\IDEA-workspace\\project\\web\\src\\generatorConfig.xml");
           ConfigurationParser cp = new ConfigurationParser(warnings);
           Configuration config = cp.parseConfiguration(configFile);
           DefaultShellCallback callback = new DefaultShellCallback(overwrite);
           MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
           myBatisGenerator.generate(null);
       }
   ```

## 3. 运行

走你

## 4. 注意事项

### 1. 扫描不在同一模块下的bean

```java
@ComponentScan(basePackages = {"com.test"})
```

### 2. 扫描mapper接口

```java
@MapperScan("com.test.mapper")
```

虽然已经把mapper接口当做bean注册了，但是还是要用mapperscan再扫描一次

### 3. 配置xml路径

> 就是让程序知道mapper和xml在哪匹配，防止找不到xml里的sql语句

- 使用yum配置

  ```yum
  mybatis:
    mapper-locations: classpath:mapper/*.xml
    type-aliases-package: com.test.entity
  ```

- 使用配置类配置，略

### 4. 配置mapper.xml的namespace

默认生成的mapper.xml的namespace是错误的，需要手动修改，暂未解决方案