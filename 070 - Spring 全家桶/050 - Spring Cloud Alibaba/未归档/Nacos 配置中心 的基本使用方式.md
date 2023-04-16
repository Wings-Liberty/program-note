# 服务端的安装和启动

Nacos Server 是一个独立运行软件

开发时可单例模式运行

```shell
startup.cmd -m stantdalone
```


# 客户端的安装和启动

1. 导入依赖

```xml
<!--nacos-config-->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2. 写配置文件

在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动

SpringBoot 中配置文件的加载是存在优先级顺序的，bootstrap 优先级高于 application

```yaml
# bootstrap.yaml nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
 
```


```yaml
spring:
  profiles:
    active: dev # 表示开发环境
```

3. 主启动类上加注解

```java
@EnableDiscoveryClient
```


4. 在应用类上获取动态配置

```java
@RestController
@RefreshScope // 在控制器类加 @RefreshScope，使当前类下的配置支持 Nacos 的动态刷新功能
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}

```


# 配置文件匹配规则

后端服务 nacos-config-client 指定了 nacos server 的地址作为配置中心，nacos server 上有很多配置文件。nacos-config-client 用哪个文件作为配置文件呢？

由于微服务的复杂性，不同的服务有各自的服务名，运行时环境（开发，测试，生产），配置文件的文件类型（.properties，.yaml 等）

所以后端服务 nacos-config-client 不是直接指定 nacos server 上的配置文件

nacos 是这样指定配置文件的：

当前项目的配置文件名是

```
${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
```

![Pasted image 20220822235252](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220822235252.png)

所以当前项目启动后，会去 nacos 找名为 "nacos-config-client-dev.yaml" 的配置文件

在 nacos server 上，每个配置文件的都有一个属性 —— Data ID，这个属性应该和配置文件名相同


# 配置文件分类规则


nacos 作为配置中心时，要定位到一个配置文件需要 namespace + group + data id

对于一般的微服务来说，一个微服务运行时会用哪个配置文件使用的加载方案只需要 data id 就够了。如上节 “配置文件匹配规则”，nacos-config 客户端会根据 服务名+当前激活的配置文件+配置文件格式拼出一个完整的配置文件名

如果仅用 data id  方案加载配置文件不能满足当前微服务架构，nacos 还提供了 namespace 和 group

在 nacos 客户端指定 namespace 和 group，服务启动时就会在指定的 namespace + group 下寻找拼接出来的和 data id 同名的配置文件

```yaml
# nacos注册中心
server:
  port: 3377

spring:
  application:
    name: nacos-order
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 # Nacos 服务注册中心地址
      config:
        server-addr: localhost:8848 # Nacos 作为配置中心地址
        file-extension: yaml # yaml 格式的配置文件
        namespace: 5da1dccc-ee26-49e0-b8e5-7d9559b95ab0 # 缺省值为 public 的值
        # 缺省值为 DEFAULT_GROUP
        group: TEST_GROUP
```

