#还没有复习 

# SpringCloud



## SpringCloud和Dubbo的区别

1. 通信方式不同

   dubbo使用RPC（远程过程调用）

   SpringCloud使用基于HTTP的Restful API（额，好熟悉的词，可能和我说的是同一个东西）

   



关于Robbin的自定义调用消费者的轮询规则，暂时没有成功使用过

Hystrix

> 服务熔断和服务降级可同时存在，之前我给整岔了

服务熔断：写在服务端（生产者），当调用某个服务出现异常时，再调用提前制定好的方法（可以理解为一个补救方法）

应用场景：和统一异常处理差不多，不过这个比统一异常处理类要更灵活

但是过多的服务熔断会造成耦合（熔断服务和服务调用都是在一个类中），可以使用aop代替

==但是在学习的代码里我写到消费者里了，不过也能用，没有抛异常==



服务降级：写在api，如果某个服务关闭不能用，就调用在api模块里提前指定好的方法，用来返回一个友好的响应（比如，返回该服务已停用，而不是客户端因无法调用服务端而返回null或直接抛异常，或显示调用失败等垃圾，不友好的信息）

应用场景：因为某些原因要停止一些服务，但是不能因为停止服务无法调用服务就报404等原始的，恶心的页面和错误提示信息



两者，一个是在能调用服务但是调用过程中出现异常执行的，另一个是当服务连调用都不能调用时执行的



zuul可以隐藏真是路由

可通过一个定制的url访问到服务

这不就是客户端的功能吗？？？为毛还要zuul





# 前言





## 结构

> 由多个可独立运行的子模块构成

每个模块有各自的服务，子模块可大致分为

`api模块` 封装接口，公用对象（一般DTO）

`服务注册模块` 这里用Eureka实现举例，启动的服务将注册到这里，以便调度和管理

`服务端模块`或者叫做生产者，这里是接口的具体实现

`客户端模块`或者叫做消费者，前端调用的是这里的接口（接口的具体实现在服务端）

`配置模块`微服务的统一配置信息来自于此，用于解决，众多服务都有各自的配置，而导致服务的配置过于繁琐且不好管理 



## 搭建套路

maven依赖 + 必要的配置文件配置 + EnableXXX注解 + 必要的Java代码

 





# 搭建一个SpringCloud微服务



下述搭建过程使用了

- RestFul接口实现
- Eureka服务注册（服务发现不是重点，不做描述）
- Ribbon负载均衡
- Feign负载均衡
- Hystrix断路器
- zuul路由网关
- SpringCloud Config分布式配置中心



## 服务注册模块Eureka

> 使用Eureka实现

### 创建Eureka服务端



1. 导入maven依赖

```xml
<!--eureka-server服务端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

2. 添加配置信息

```yaml
eureka:
  instance:
    hostname: eureka-7001 # eureka服务端的实例名称
  client:
    register-with-eureka: false # false表示不向注册中心注册自己。
    fetch-registry: false # false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      # 此项待搭建eureka集群时再配置 
      defaultZone: http://localhost:7002/eureka/,http://localhost:7003/eureka/ # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
```

3. 启动类上添加注解

```java
@EnableEurekaServer // 表示此项目作为EurekaServer服务端，接受其它微服务注册进来
```



### 使用Eureka客户端向Eureka服务端注册服务

> 这里只说关于SpringCloud微服务的相关操作，rest接口实现不再说明

1. 导入服务注册的依赖

```xml
<!-- 将微服务provider侧注册进eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

2. 添加配置，将服务注册到`服务注册模块`

```yaml
eureka:
  client:
    service-url:
      # 指定注册中心地址
      defaultZone: http://localhost:7001/eureka/,http://localhost:7002/eureka,/http://localhost:7003/eureka/
  instance:
    instance-id: privoder-8001 # 微服务名，相同服务的生产者的此名字可以不同
    prefer-ip-address: true # 访问路径可以显示IP地址
```

3. 启动类上添加注解

```java
@EnableEurekaClient //本服务启动后会自动注册进eureka服务中
```



### 自我保护&服务发现



`Eureka`的自我保护内容详情看脑图



服务发现：服务之间互相可见。不重要，跳过



### Eureka的集群配置



`Eureka`的集群配置很简单

分两部分配置

1. 在所有`Eureka`服务端指定其他`Eureka`节点的地址

   ```yaml
   eureka:
     ...
     client:
       ...
       service-url:
         # 在Cloud-Eureka7001模块中指定其他Eureka服务端的地址
         defaultZone: http://localhost:7002/eureka/,http://localhost:7003/eureka/ 
   ```

2. 在所有`Eureka`客户端指定所有`Eureka`服务端节点的地址

   ```yaml
   eureka:
     ...
     client:
       service-url:
         # 指定所有Eureka服务端地址
         defaultZone: http://localhost:7001/eureka/,http://localhost:7002/eureka,/http://localhost:7003/eureka/
   ```

   

## Ribbon负载均衡

==这一步是在上一步（服务注册）的基础上进行的==

> Ribbon实现，用于实现合理化调用服务

使用`Ribbon`做负载均衡的实现均在客户端中做

1. 添加maven依赖

```xml
<!-- Ribbon相关依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
    <version>1.3.1.RELEASE</version>
</dependency>
```

2. 添加负载均衡的注解

```java
// 之前使用RestTemplate时，将@LoadBalanced加到@Bean注解的方法上即可
```

3. 轮询规则

- **如果使用内置的规则**

在一个配置类中注入一个内置的轮询规则即可

```java
@Bean
public IRule myRule(){
    //return new RoundRobinRule();
    //return new RandomRule();// 达到的目的，用我们重新选择的随机算法替代默认的轮询。
    return new RetryRule();
}
```

 - **如果自定义了轮训规则，在启动类上添加注解**

```java
// name指定要进行负载均衡的服务名，configuration指定自定义轮询规则
@RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration=MySelfRule.class)
```

关于name

```yaml
# 上述的name指的服务名是指，服务端的配置文件中的该配置
spring:
  application:
    name: CLOUDPROVIDERDEPT
```

关于自定义轮询规则

`向容器中注入一个继承了extends AbstractLoadBalancerRule并实现其方法的类`



自定义轮询规则还没有成功实现过



## Feign整合restful



> Feign负载均衡，它集成了Ribbon
>
> 1. 实现负载均衡（默认使用Ribbon的轮询方式做负载均衡。有待探究：因为教程并没有讲怎么使用Feign自定义轮询方式）
> 2. 实现使用服务的接口远程调用服务的实现（Ribbon使用的是RestTemplate调用接口）



### 执行流程



1. 在`API模块`的接口中添加`@RequestMapping`等注解，将接口的参数列表修改为和网关接口一样的参数列表，在接口上添加`@FeignClient`注解
2. 在`消费者模块`的启动类上添加`@EnableFeignClients`并指定包，将api扫描进去，在`消费者模块`的Controller中注入API接口对象，调用接口方法。
3. 这些接口对象会向`Eureka`发送请求，`Eureka`将请求转发给选择的服务，服务处理请求，返回数据给`Eureka`。`Eureka`将数据返回给`消费者模块`





### 修改api

1. 添加maven依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

2. 添加接口代码

```java
@FeignClient(value = "MICROSERVICECLOUD-DEPT") // value指定该接口所对应的服务名
public interface DeptClientService{
    
    // 这些url对应的是生产者中的接口
    
	@RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
	public Dept get(@PathVariable("id") long id);

	@RequestMapping(value = "/dept/list", method = RequestMethod.GET)
	public List<Dept> list();

	@RequestMapping(value = "/dept/add", method = RequestMethod.POST)
	public boolean add(Dept dept);
}
```



### 修改客户端

1. 添加maven依赖

```xml
<!--feign启动器-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

2. 写java代码

`原使用RestTemplate的实例对象调用接口，现使用接口对象，远程调用服务。如下例`

```java
@RestController
public class DeptController_Consumer{
	@Autowired // 装配api里的服务接口
	private DeptClientService service;

	@RequestMapping(value = "/consumer/dept/get/{id}")
	public Dept get(@PathVariable("id") Long id){
		return this.service.get(id);
	}

	@RequestMapping(value = "/consumer/dept/list")
	public List<Dept> list(){
		return this.service.list();
	}

	@RequestMapping(value = "/consumer/dept/add")
	public Object add(Dept dept){
		return this.service.add(dept);
	}
}
```

3. 启动类上添加注解

```java
@EnableFeignClients(basePackages= {"com.cloud"}) // 添加此注解后，好像就不用@RibbonClient这个注解了
// 指定所要扫描的包，它会扫描restful的rul和接口
```



==注：Feign和Eureka使用的不是RPC框架，所以可以使用泛型，不会因为 出现使用泛型而导致响应中的泛型失效==



## Hystrix

> 它主要做了两件事
>
> 1. 服务熔断
> 2. 服务降级

==这里或的Hystrix是他的功能，虽然Fegin集成有它的功能，但是并不完善==

### 服务熔断

> 调用**接口的实现**失败后（可以调用实现，但是服务的实现出错）会调用一个备选方法，用以返回一个友善的响应

服务熔断的代码写在客户端或服务端都行

服务熔断和SpringBoot的统一异常处理有点像

1. 添加maven依赖

```xml
<!--  hystrix -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

2. 写服务熔断的java代码

```java
// 这里的举例，服务熔断代码写在客户端
@RestController
public class DeptController {
    @Autowired
    private DeptService deptService;

    @HystrixCommand(fallbackMethod = "fallBack")
    @GetMapping("/dept/{id}")
    public Dept getDept(@PathVariable Integer id){
        Dept dept = this.deptService.getDept(id);
        if (dept == null) {
            throw new RuntimeException("找不到dept，抛异常");
        }
        return dept;
    }
	// fallbackMethod方法的参数表要和原接口参数表一致
    public Dept fallBack(@PathVariable Integer id){
        Dept dept = new Dept();
        dept.setDeptno(0L);
        dept.setDname("这是HystrixCommand的dept");
        dept.setDbSource("HystrixDb");
        return dept;
    }
}
```

3. 启动类上添加注解

```java
@EnableCircuitBreaker // 对hystrixR熔断机制的支持
```

==服务熔断要写在Controller类中，这导致了耦合，如果且一个接口对应一个fallbackMethod会导致代码膨胀。所以可以使用aop代替==

### 服务降级

> 因关闭服务而导致客户端无法调用服务时启用，这样即使调用不存在的服务也不会出现返回 找不到服务等，连统一异常处理都没有做的返回值

代码写在`api模块中`，不需要导入依赖（因为之前已经导入了Fegin的依赖）

1. 添加实现FallbackFactory接口的类

```java
@Component // 不要忘记添加，不要忘记添加。这里的泛型是服务接口
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService>{
	@Override
	public DeptClientService create(Throwable throwable){
		return new DeptClientService() {
			// 此匿名内部类要实现接口所有的方法，作为服务降级的代码
            // 如果调用接口时找不到接口的服务，那么接口会调用这里的代码
		};
	}
}
```

2. 添加注解，声明指定服务所调用的FallbackFactory

```java
@FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory=DeptClientServiceFallbackFactory.class)
public interface DeptClientService{
    ......
}
```



## 网关服务

> 这里使用zuul实现

==暂时不做笔记，因为完全不知道zuul和不使用zuul的客户端有什么区别，或者说不知道为什么要用zuul==



## SpringCloud Config

> 用来统一微服务的配置



config分`客户端`和`服务端`

服务端是用来指定配置文件所在地

客户端是所有要使用config服务的模块



### 服务端

1. 添加mave依赖

```xml
<!-- springCloud Config -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

2. 修改配置文件

```yaml
server: 
  port: 3344 
  
spring:
  application:
    name:  microservicecloud-config
  cloud:
    config:
      server:
        git:
          uri: git@github.com:zzyybs/microservicecloud-config.git #GitHub上面的git仓库名字
```

3. 主启动中添加注解

```java
@EnableConfigServer
```

启动程序后，使用`http://config-3344.com:3344/application-test.yml`可直接获取配置文件的信息



### 客户端

1. 添加maven依赖

```xml
<!-- SpringCloud Config客户端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

2. 添加一个bootstrap.yml

```yaml
spring:
  cloud:
    config:
      name: microservicecloud-config-client #需要从github上读取的资源名称，注意没有yml后缀名
      profile: test   #本次访问的配置项
      label: master   
      uri: http://config-3344.com:3344  #本微服务启动后先去找3344号服务，通过SpringCloudConfig获取GitHub的服务地址
```

