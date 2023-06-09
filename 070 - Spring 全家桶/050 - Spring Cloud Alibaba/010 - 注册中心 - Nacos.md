#还没有复习 

Nacos 在微服务中的功能：注册中心，配置中心

Nacos 是 C/S 架构

当它作为注册中心时，nacos server 需要提供以下服务

- 服务注册：接受服务提供者的服务注册
- 服务发现：向服务消费者提供其所需要的服务列表
- 心跳机制和服务健康检查：定时检查服务提供者是否正常在线并提供服务

nacos client 有能力向 nacos server 发送服务注册请求，服务器发现请求，心跳包



在看源码前必须得会用它，nacos 有两种模式：CP（一致性，集群容错） 和 AP（可用性，集群容错）

[参考资料](E:\编程资料\图灵学院\资料\课件代码\五：微服务专题\06-Alibaba Nacos注册中心源码剖析(上)-诸葛\04-VIP-Alibaba Nacos注册中心源码剖析.pdf)

# 构建源码&找启动类

用 1.4 版本，并在 shell 脚本里找 nacos server 的启动类或 jar 包，然后在 jar 包的 META-INFO 文件夹里找主启动类或在 pom 文件里找著启动类。一开始采用单机模式启动 nacos server

启动后，尝试启动两个实例向其注册服务

结合官网学习。nacos server 本身就是一个 SpringBoot 写的 web 项目，通过对外提供 http 接口提供服务

实现上述功能其实就是 nacos client 向 nacos server 的指定接口发送请求



发送 http 请求的常见 client 有：HttpClient，JDKHttpClient（jdk 自带的）...

> nacos 2.0 后把很多地方都改为了 gRPC 通信

# 单机 server 下的服务注册

目标：从源码角度研究 client 发起服务注册和 server 接收服务注册的两个行为

从何看起：

Q：从 client 里看，找哪个类/方法，或者说在哪打断点作为整个注册行为发起的入口呢？

A：从 nacos client 依赖的自动装配配置了哪些核心类入手。找其依赖下的 spring.factory 里的核心自动配置类，自动配置类会装配很多 bean，优先看类名里带 auto 的或优先看用到了其他 bean 作为参数的。然后看看这个类是否实现了哪些很核心很重要的类，比如实现了 Spring 架构的 Listener 或 Aware 或 ...



猜测服务注册的行为：client 通知 server 把某个服务注册一下，server 收到请求后把信息放到某个容器中（实际上这个容器是一个很复杂的嵌套 map，还嵌套了 set，同一个服务被保存到一个  set 里了？艹，这个大 map 嵌套了三四层才到真正保存服务实例的地方）



读源码技巧：源码喜欢用卫模式进行 if 判断，通常卫模式下带有 return 的 if 在正常情况下是不会执行的，第一次翻源码看代码整体执行流程时如果卫模式后还有源码，可以大胆跳过卫模式的 if 直接看后面的代码



服务注册里，client 干的事其实就是在开启服务注册后，启动 spring 时向 nacos server 的服务注册接口发送 http 请求请求注册服务。所以在解析源码时 client 的注册行为（client 除了注册行为还有其他行为，这些行为可能还是需要好好看看的）需要分析但不是服务注册的核心，核心在于 server 如何处理服务注册请求



通过分析 client 的行为&源码，知道了 client 向某个接口发送了请求。下面就分析 server 的这个接口，直接在源码里找这个接口即可

- 直接用 IDEA 的 RestController 插件，获取所有接口列表，从里面找
- 或用和行为相关的关键字 + Controller 作为类名进行搜索



在看源码的时候，有这么个小地方值得注意，服务注册需要一个 ephemeral 参数表示这个实例是否是临时节点

如果是临时节点，则不会把实例信息写入磁盘。这和 nacos 采用的是 CP 还是 AP 有关。（为保证一致性，当然需要将实例信息写入磁盘。为保证可用性，就不能进行写磁盘这种耗时行为）



在保证数据一致性的时候可能会采用各种数据一致性协议，比如 nacos 有 Distro 协议（阿里自己写的），raft 协议（nmd，不久注册个实例吗，至于用这么复杂的协议吗...）



在看完 nacos server 处理服务注册时并没有看见有任何向服务注册 map 中添加数据的行为，假设注册行为由 DistroXXX 实现的，这个实现类向一个阻塞队列中 offer 了服务信息，但没有向服务注册的嵌套 map 中添加服务信息。它肯定是会在某个地方某个时机从阻塞队列中获取数据再向 map 中添加了服务信息，比如 DistroXXX 里有一个 runable 的实现类，其会向阻塞队列中取实例信息并执行真正的注册逻辑



Q：上述的注册流程相当于异步注册，为什么要异步注册，而不是由主线程同步注册？

A： 1. DistroXXX 里有一个 runable 的实现类的 run 方法是用 spring 的初始化方法里被调用的。

		2. nacos 是阿里开发的，它们要求这些软件都得是能应对高并发，高可用，高xx。当 nacos server / client 启动时，大量的 client 发起注册，如果 server 的注册是同步处理的，就需要大量的 client 同步等待 server 处理请求。如果 client 不光有 nacos 还用了其他十几个中间件，那 client 启动时每初始化一个中间件的 client 就要同步等待一下，这能卡死人，所以中间件的初始化啊，注册 client 啊这些流程通常都被设计为异步的，尽可能不要影响使用了中间件的 client 的性能

通常新的实例注册进 nacos 所用时间很少，所以服务消费者基本能实时获取最新的服务列表



其实嵌套 map 就是一个树形结构。nacos 用很复杂的嵌套 map 目的是为了应对如下场景

- 嵌套 map 的实际层级格式：`nameaspace:group:serviceName:cluster:instance`
- 举例：`dev:trade:order:henan:实例对象`
- 说明：`测试环境:交易组服务:订单服务:河南机房:实例对象`。这个实例对象被详细说明了所属的开发环境，微服务组，具体提供的微服务，地理位置。这些信息就是个例子，实际开发时你想设置成啥就设成啥。其设置层级的目的是为了实现高的扩展性和通用性



服务注册 = 向容器中执行写操作

服务发现 = 对容器进行读操作

**为了提高并发量，nacos 采用写时复制思想（copyonwrite）**，用读写分离实现读写都能并发执行来提高服务列表的管理效率，实现低配服务器单机 TPS 达到 13000。但牺牲了一致性，即服务注册进去之后需要一小段时间（但这个时间很短）后 client 才能感知到（因为读写不在同一个容器内进行，复制行为需要消耗时间）

复制行为由 nacos server 的单个线程执行，所以复制时不存在并发修改异常

复制过程如下：在注册实例时，会先把 server 的旧服务列表用 for 循环复制一份，然后在新服务列表中注册实例，注册完后，让服务列表的引用直接指向新服务列表

Q：如果服务列表中的实例过多，复制的时间和空间开销会不会很大？

A：不会，因为执行写时复制服务列表不是整个嵌套 map，而是以一个 cluster 为单位的，每次复制只会复制 cluster 中的对象。而且复制行为是浅拷贝，只复制引用，不创建新对象，所以空间开销很小

Q：新服务列表里的服务实例是从旧服务列表里用浅拷贝拷过来的，那在新服务列表里对服务实例的修改就是对旧服务列表里实例的修改，这不存在并发修改异常吗？

A：不存在并发修改异常。向 newList 新增服务实例显然不会对 oldSet 有任何影响，对 newList 中的服务实例更新的行为 = 删除旧服务实例 + 新增新服务实例，而不是修改原来的服务实例对象的配置。最后用 newList 直接取代 oldSet 所以也不存在并发修改异常，因为就没有修改，而是创建一个新的对象取代旧对象



# 服务发现

本质上就是向 nacos server 发送一个有条件的批量查询请求

条件是`nameaspace:group:serviceName:cluster`，然后获取其下的所有路由信息



client 会定时执行拉取，并缓存到 client 的 map 里，定时更新本地路由信息



这些逻辑比较简单



# 心跳机制和服务健康检查



## 心跳机制

client 发起服务注册时还会起一个定时任务，定时向心跳接口发送心跳请求保活（默认 5s 发一次）

server 收到心跳请求后更新这个服务实例的最新心跳时间



## 服务健康检查

server 收到服务注册后还会开启一个定时任务，定时检查服务实例是否健康

如果服务实例的上次心跳时间至今已经超过阈值（默认 15s 还没收到就令其下线）就认为服务已经不再可用，设服务实例的健康标志位

如果超时时间过长，server 就会自定调用自己的 delete 接口（自己调自己可还行），删除这个服务实例



为了保证 client 能尽可能实时感知 server 对服务列表的修改，server 采用观察者模式，当服务列表被修改时，server 会主动推送修改信息给 client

用的是事件发布模型，用 UDP 向所有  client 发送



client 采用短链接和 server 通信，zk 采用长连接。zk 用 watch 机制保证数据一致性，让 client 能尽快感知 zk server 的数据修改并更新本地数据





# AP 集群架构下的 nacos

zk 是 CP 架构的

eureka 作为注册中心时，为提高并发性能，采用多级缓存 + 读写分离。因为多级缓存间存在数据不一致问题，所以 eureka 的实时性不好



nacos 的集群需要结合 nginx + mysql，MySQL 用于持久化 nacos 的信息。nacos 和 MySQL 结合时，nacos 提供了 sql 语句



先启动集群，然后向集群注册一个服务实例测试集群是否启动成功。如果启动成功，那么所有 nacos 应该都能持有一个服务实例

服务实例的配置文件只需要指定一台 nacos server 即可，注册成功后，集群中的其他 nacos server 都会感知到

AP 架构下的 nacos 中没有主从，nacos server 都是对等节点。CP 上才有主从概念



目标：整明白 AP 集群下的 nacos server 是如何实现的 server 之间的通信，如何实时感知并同步新增的服务实例和修改服务实例配置信息的



集群模式下，每个 server 启动时都会初始化两个定时任务，一个用于 server 间感知节点是否还存活的心跳通信，另一个用于同步 server 保存的服务实例



Q：server 如何实现的通信

A：采用定时任务实现。一个服务实例的心跳检测只需要在一个 nacos server 里做，其他 server 同步获取到服务实例后并不执行心跳检测。只有执行心跳检测的 server 负责对服务实例的健康管理，如果服务实例下线，这个 server 会通知其他 server 把这个服务实例下线

一个服务实例只会在一个 nacos server 执行心跳和健康检查，这是在定时任务中判断服务实例所在的 nacos server 是否是自己发起注册的 server （通过哈希函数 + 取模计算实现的判断）实现的。如果是就执行心跳&健康检查，如果不是，不执行心跳&健康检查



nacos server 之间也会用心跳进行通信检查 server 节点是否还健康存活，如果有 server 宕机，存活的 server 会及时更新存活的 server 数量，以保证上述取模计算的正确性（因为取模操作需要当前存活的 server 的数量信息）

所以不需要用一致性哈希也能实现



> 有两处地方都需要实现服务实例列表的同步
>
> - server 和 client 之间。为保证 client 能实时感知 服务实例列表的变化，client 会定时主动拉取最新的服务实例列表，server 在对服务实例列表进行修改后也会向 client 发送 UDP 通知最新的服务实例列表
> - server 之间。为保证数据一致性，client 只向一个 server 发送服务注册，收到服务注册的 server 主动向其他 server 发送同步信息

server 间同步服务实例时需要采用一致性算法，之前说过了默认用阿里的 Distro 算法，但还提供了 Raft 算法



此外 server 在启动时还会执行一个任务，同步集群中其他节点中的服务实例列表

场景：有两个 server 节点组成的集群，里面有很多服务实例。现在又起一台 server，这个 server 启动时会同步其他 server 里的服务实例列表





看完注册中心功能了，该看配置中心的功能了。该看 17-2 了
