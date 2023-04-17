#还没有复习 

## SpringBoot整合套路

1. 添加maven依赖
2. 添加配置（application.yml）
3. 启动类上添加@EnableXX
4. 添加注入组件（配置类中添加bean）
5. 使用注解或代码写业务逻辑即可

# Cache缓存

## 一、搭建基本环境
创建一个可用的三层结构

使用了SpringBoot整合MB

## 二、快速体验缓存

---

1. 添加maven依赖

   ```xml
   <!--redis启动器-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

   

2. 主配置类上添加注解

   ```java
   @SpringBootApplication
   @EnableCaching
   public class CacheApplication {
       public static void main(String[] args) {
           SpringApplication.run(CacheApplication.class, args);
       }
   }
   ```

3. 方法上添加缓存注解

   ```java
   // 如果缓存中有数据，跳过执行，直接返回数据；如果没数据，查数据并存入缓存，下次就不用查数据了
   @GetMapping("/employee")
   @Cacheable(value = "emp", key = "#a0")
   public Employee getEmployeeById(Integer id){
       Employee employee = employeeService.getById(id);
       return employee;
   }
   ```



## 三、缓存的注解

**相关注解**

`@Cacheable`，`@CacheEvict`，`@CachePut`

---

###  @Cacheable

> 创建缓存或读取缓存

- cacheNames/value：指定缓存组件的名字;将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；

- key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值
             编写SpEL； `#i` d;参数id的值   `#a0`  `#p0`  `#root` .args[0]
             getEmp[2]

- keyGenerator：key的生成器；可以自己指定key的生成器的组件id（bean的id默认是类名，首字母小写或@Bean下的方法名）
             key/keyGenerator：二选一使用;

写一个KeyGenerator接口的实现类即可

- cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器

- condition：指定符合条件的情况下才缓存
             condition = "#id>0"
         	condition = "#a0>1"：第一个参数的值》1的时候才进行缓存

         多个条件可在字符串中用and隔开
         
- unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
             unless = "#result == null"
             unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
         
- sync：是否使用异步模式（开启异步后unless将不再可用）



### @CachePut

>  既调用方法，又更新缓存数据；同步更新缓存，如果没有缓存会创建缓存

==注：这里的方法需要有返回值才能指定key = "#result.id"==

修改了数据库的某个数据，同时更新缓存；
运行时机：

1. 先调用目标方法（@Cacheable是调用目标方法前使用）
2. 将目标方法的结果缓存起来
测试步骤：
1. 查询1号员工；查到的结果会放在缓存中；
	key：1  value：lastName：张三
2. 以后查询还是之前的结果
3. 
3. 更新1号员工；【lastName:zhangsan；gender:0】
	将方法的返回值也放进缓存了；
	key：传入的employee对象  值：返回的employee对象；
4. 查询1号员工？
	应该是更新后的员工；
	key = "#employee.id":使用传入的参数的员工id；
	key = "#result.id"：使用返回后的id
	@Cacheable的key是不能用#result
	为什么是没更新前的？【1号员工没有在缓存中更新】

### @CacheEvict

>  缓存清除

- key：指定要清除的数据

- allEntries = true：指定清除这个缓存中所有的数据

- beforeInvocation = false：缓存的清除是否在方法之前执行
  默认代表缓存清除操作是在方法执行之后执行;如果出现异常缓存就不会清除

- beforeInvocation = true：
  代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除



### @Caching

> 上面三个注解的组合注解

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Caching {

	Cacheable[] cacheable() default {};

	CachePut[] put() default {};

	CacheEvict[] evict() default {};

}
```



### @CacheConfig

> 作用于类上的抽取缓存的公共配置

作用于类上，可配置缓存的名字，键生成器等。可统一这个类下所有可缓存的方法的属性





默认使用的是ConcurrentMapCacheManager == ConcurrentMapCache；将数据保存在	ConcurrentMap<Object, Object>中
   开发中使用缓存中间件；redis、memcached、ehcache；
3. 整合redis作为缓存
     Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。
	
	1. 安装redis：使用docker；
	
	2. 引入redis的starter
     
     3. 配置redis
     
        配置一个redis序列化器，因为默认使用的是jdk的序列化器，一般使用json
     
     4. 测试缓存
        原理：CacheManager 创建 Cache 缓存组件来实际给缓存中存取数据
     
        1. 引入redis的starter，容器中保存的是 RedisCacheManager；
        2. RedisCacheManager 帮我们创建 RedisCache 来作为缓存组件；RedisCache通过操作redis缓存数据的
        3. 默认保存数据 k-v 都是Object；利用序列化保存；如何保存为json
            1. 引入了redis的starter，cacheManager变为 RedisCacheManager；
            2. 默认创建的 RedisCacheManager 操作redis的时候使用的是 RedisTemplate<Object, Object>
            3. RedisTemplate<Object, Object> 是 默认使用jdk的序列化机制
        4. 自定义CacheManager；

## 四、浅析原理







将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
  CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；

  原理：
    1、自动配置类；CacheAutoConfiguration
    2、缓存的配置类

```java
org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
```



3、哪个配置类默认生效：SimpleCacheConfiguration；

4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager
5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；

运行流程：
@Cacheable：
1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
   （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
   key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
       SimpleKeyGenerator生成key的默认策略；
               如果没有参数；key=new SimpleKey()；
               如果有一个参数：key=参数的值
               如果有多个参数：key=new SimpleKey(params)；
3、没有查到缓存就调用目标方法；
4、将目标方法返回的结果，放进缓存中

@Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；

核心：
   1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
   2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator

 



## 五、整合Redis

>  导入redis整合SpringBoot的启动器，在配置文件中添加redis的连接信息即可使用（无需手动添加配置类或在配置类中注入组件）

没有导入redis的时候默认将缓存存入内存中，使用redis后，将缓存存入redis中



序列化的问题

直接使用RedisTemplate组件或StringRedisTemplate组件的时候，默认存储的对象的类需要实现序列化接口。

且使用的序列化方式是jdk的，一般来说使用的序列化方式是json，因此需要修改

```java
@Bean
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
    throws UnknownHostException {
    RedisTemplate<Object, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(redisConnectionFactory);
    // 设置序列化器
    template.setDefaultSerializer(new GenericJackson2JsonRedisSerializer());
    return template;
}
```

如果使用缓存，只是自定义RedisTemplate组件，设置自定义的序列化器是没有用的，因为SpringBoot2.X后，Redis不再使用RedisTemplate中设置的序列化器

具体解决[看这里](https://blog.csdn.net/jy1690229913/article/details/84854471)



## 六、自定义缓存的属性



# 检索

> 这里学习的是ElasticSearch

---

## 一、快速开始

> 使用docker运行es

### 启动es

1. pull镜像

```shell
# 就算用阿里云加速了，search操作也可能会超时
$ docker search elasticsearch
$ docker pull elasticsearch
```

2. run镜像

```shell
# - e 设置初始分配内存大小和最大占用内存（因为运行这个很耗内存） d 后台运行 p 这里需要映射两个端口
$ docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name ES elasticsearch
$ docker run -e ES_JAVA_OPTS="-Xms100m -Xmx100m" -d -p 9200:9200 -p 9300:9300 -v /opt/config/elasticsearch.yml--name ES elasticsearch:7.4.2
```

 如果成功完成上述操作，即成功启动es



### es的数据结构和请求方式

es的数据结构由四部分组成`索引`，`类型`，`文档`，`属性`

文档指具体存储的数据，属性指数据的字段

这有点像数据库，表，数据，字段的关系

es使用http请求对数据进行增删改查，可理解为url为数据的key，文档为数据的value

这样看来，数据的key分三层`/索引/类型/id`



### 操作

> 使用http请求操作es，它也有增删改查。查最复杂，增改是一个方法
>
> 增改，删，查。分别对应`PUT`，`DELETE`，`GET`





1. 向elasticsearch中存数据

   `这里用postman，PUT方式，url为 索引+类型+id（不过索引，类型，id可无需提前声明，直接自定义），最后的占位符表示id`

   <img src="http://39.107.101.13:8083/file/908ecbb8-e624-4383-a1b1-e84d1cda762e比特截图2020-03-07-09-54-07.png"  />

   ```json
   // 这是返回的响应结果
   {
       // 索引
       "_index": "imakerlab",
       // 类型
       "_type": "mumber",
       // id
       "_id": "1",
       // 每次修改版本都会变
       "_version": 1,
       // 如果是修改，这里会是update
       "result": "created",
       // 这个是分区信息，暂时不解释
       "_shards": {
           "total": 2,
           "successful": 1,
           "failed": 0
       },
       "created": true
   }
   ```

   

   随后再多发几组数据

2. 向elasticsearch检索指定数据

   `这里还是用postman，url不变只是把PUT换为GET`

   ```json
   {
   	"_index": "imakerlab",
   	"_type": "mumber",
   	"_id": "3",
   	"_version": 1,
   	"found": true,
   	"_source": {
   		"name": "zy",
   		"age": 20
   	}
   }
   ```

   

   如果是DELETE，就表示删除数据

   如果是HEAD方式，表示查找是否存在指定数据，如果存在返回码为200（但是没有其他响应数据），找不到报404

   更新的还是用过PUT

7. 检索指定类型数据（获取此类型下的所有数据）

   url由原来的`索引+类型+id`，改为`索引+类型+"_search"`

   `/imakerlab/mumber/1` ——> `/imakerlab/mumber/_search`

8. 条件查询

   GET：`/imakerlab/mumber/_search?q=xx:xxx`

   q后加字段名:值

   或用json携带查询条件

   ```json
   {
       "query" : {
           "match" : {
               "xx" : "xxx",
               "xxx" : "xxx"
           }
       }
   }
   ```

   什么模糊查询（其实默认就是模糊查询，因为使用者一般都不用精准查询），正则表达式查询，还有一堆排序和其他想都想不到的查询方式
   
   关于json的条件查询有很多的条件可使用
   
   见博客或官网







### es的索引信息

> 使用`/_cat`的url请求会得到一堆可以使用的url

`/_cat/indices`的返回结果，展示了es所有索引的数据

`health` 集群健康状况 				`status`是否可用  

`index`索引名 							`uuid`索引统一编号 

`pri`主节点个数  						`rep`从节点个数 

`docs.count` ` 文档数 ` 				`docs.deleted` 被删文档数 

`store.size` 数据所占空间 		`pri.store.size` 主节点所占空间



### Kibanaes

>  es的可视化工具，也用docker实现

1. 下载kibana镜像

```shell
$ docker pull kibana
```

2. 运行

```shell
$ docker run -d -p 5601:5601 --name KB kibana
```

==docker的kibana容器的es地址需要修改，因为在docker中还需要下载vim才能修改配置文件。所以可使用docker的cp将容器的配置文件cp到本机，在本机改完后cp回去。但是我不会重启kibana，所以还要利用这个容器创建一个镜像。这样一来新的镜像中有所有正确的配置，run即可==

3. 修改配置文件，创建新镜像 / 重启kibana服务
4. run 新镜像或重启kibana服务
5. 使用5601端口在浏览器访问

**使用可视化界面操作es**

在kibana的UI中，点击`Dev Tools`，在里面写命令即可，例如

`GET _cat/indices`  （请求方式  + 空格 + url）

```
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://101.43.143.178:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true

docker run -d --name=KB -p 5601:5601  -v /opt/config/kibana.yml:/usr/share/kibana/config/kibana.yml kibana:7.4.2
```





### 存在的问题

1. es的状态并不是`green`，而是`yellow`，好像是因为docker的es缺少什么东西，但是es能正常使用

### 版本问题

当前docker使用的es是5.0版本的，在6.0版本中一个索引只能有一个类型，而5.0中一个索引可以有多个类型



## 二、SpringBoot整合

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
       <version>2.2.2.RELEASE</version>
   </dependency>
   ```

   

两种检索方式，jest和elasticsearch

jest好像过时了

前者需要在bean中的id属性上加@JestId注解

后者有两种实现

1. 写接口

   写一个集成ElasticSearchRepository接口的接口（和MB一样），传两个泛型，一个是实体类的类型，一个是主键的类型

   实体类上加@Document注解指定索引和类型

2. 用XXXTemplate操作



# 任务

## 一、异步任务

两个注解：
@EnableAysnc（写在启动类上）、@Aysnc（写在需要开启异步的方法上）

不知道是干啥的

## 二、定时任务



两个注解：@EnableScheduling（写在启动类上）、@Scheduled（写在方法上，写cron表达式，指定执行方法/任务的时间，可用cron在线生成器生成）



## 三、邮件任务

前两个任务的相关注解在springboot里就有无需导入额外的依赖

邮件任务需要导入`spring-boot-starter-mail`

1. 配置服务器的邮箱（邮件发送者的邮箱）账号和密码（不是邮箱的面，是邮箱开通第三方发送邮件功能后的授权码）

   `spring.mail.username  和 spring.mail.password`

2. 配置邮件发送地址，配置真正发送邮件第三方服务器（SMTP）

   `spring.mail.host`

   ```
   这是网易的
   
   服务器地址:
   POP3服务器: pop.163.com
   SMTP服务器: smtp.163.com
   IMAP服务器: imap.163.com
   ```

3. 添加一条配置（看情况配置，有些spring版本不需要这条配置，有些不加这条配置会报错）

   ```properties
   spring.mail.properties.mail.smtp.ssl.enable=true
   ```

   

4. 使用自动装配的`JavaMailSenderImpl`发送



# 热部署

SpringBoot提供的热部署工具

```xml
<dependency>  
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

使用这个依赖后，如果项目启动且要修改java代码，无需重启程序，使用build project即可（Ctrl + F9）