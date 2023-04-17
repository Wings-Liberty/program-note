#还没有复习 

# 概述

声明式 Web 客户端。能识别并自行解析 SpringMVC 的 @GeMapping 等注解，并自动生成动态代理来向远程进行调用

所以 OpenFeign 仅用于消费端


# 基本使用方式

1. 导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2. 配置一个注册中心，比如 [[Nacos 注册中心 的基本使用方式|Nacos]]

```yml
spring:
  application:
    name: myapp
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址
```

3. 创建并启动客户端

```java
@EnableFeignClients // 标记在主启动类上
```

4. 用声明式方式指定远程服务

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE") // 指定远程的服务名
public interface PaymentFeignService {
    @GetMapping(value = "/payment/get/{id}")
    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
}
```

5. 调用远程服务

```java
@RestController
public class OrderFeignController {

	// 注入动态代理
    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id) {
	    // 调用代理对象
        return paymentFeignService.getPaymentById(id);
    }
}

```


> [!NOTE] OpenFeign 自带 Ribbon 负载均衡，所以可以在配置文件里改 Ribbon 的配置


# 日志打印功能

OpenFeign 自带有日志输出功能，修改配置可以输出远程调用的日志

1. 设置 OpenFeign 里定义的日志输出级别

```java
/**
 * NONE：默认的，不显示任何日志；
 * BASIC：仅记录请求方法、URL、响应状态码及执行时间；
 * HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；
 * FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据
 **/
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

2. 设置日志框架的输出

```yml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.atguigu.springcloud.service.PaymentFeignService: debug
```
