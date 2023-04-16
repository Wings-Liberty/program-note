# 概述

旧版 Nacos Server 就是一个 Web 应用

接受 Nacos Client 的服务注册和服务发现


# 服务端的安装和启动

Nacos Server 是一个独立运行软件

开发时可单例模式运行

```shell
startup.cmd -m standalone
```

# 客户端的服务注册和服务发现


导入客户端依赖

```xml
<!--SpringCloud ailibaba 版本依赖管理 -->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-alibaba-dependencies</artifactId>
	<version>2.1.0.RELEASE</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
```

```xml
<!--SpringCloud ailibaba nacos -->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

## 服务注册

1. 配置文件配置 nacos server 地址和要注册的接口

```yml
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

2. 创建并启动 nacos 客户端

```java
@EnableDiscoveryClient // 标记在主启动类上
```

顺利启动项目后即可向 nacos 注册服务

## 服务发现

服务消费者用 nacos client 的服务发现获取到 nacos server 的服务列表后就能调用服务

nacos client 支持负载均衡，因为 nacos client 包含了 ribbon

1. 首先修改配置文件，指定 nacos server 地址和要发现的服务

```yml
spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848


#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider
```

2. 创建并启动 nacos 客户端

```java
@EnableDiscoveryClient // 标记在主启动类上
```

3. 调用服务

```java
restTemplate.getForObject(serverURL+"/payment/nacos/"+id,String.class);
```

serverURL = "http://nacos-payment-provider"

nacos client 会自行根据服务名调用服务

