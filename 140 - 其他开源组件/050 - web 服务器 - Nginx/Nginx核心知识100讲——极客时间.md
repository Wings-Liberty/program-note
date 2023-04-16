# 课程模块

> 学习模块
>
> 1. 初识`Nginx`（基本用法）
> 2. `Nginx`架构基础（进程模型和数据结构）
> 3. 详解`HTTP`模块（`Nginx`处理`Http`请求的流程，及其`Nginx`指明使用`https`）
> 4. 反向代理与负载均衡（7层负载均衡和4层负载均衡，搭建不同上游协议的反向代理，控制上下游流量的交互）
> 5. `Nginx`的系统层性能优化（在`Linux`下如何优化`Nginx`）
> 6. 从源码视角深入使用`Nginx`与`OpenResty`（打通前五部分，以及`Nginx`如何搭配`OpenResty`）



# 01 初识Nginx



## Nginx 的三大使用场景

- 动静分离
- 反向代理：缓存 + 负载均衡
- API 服务：OpenResty



应用服务如：`Tomcat`

当应用服务搭建集群时，需要`Nginx`提供的负载均衡，负载均衡需要做到

- 动态扩容。应用服务集群添加新节点后，`Nginx`能自动识别并将其添加到代理中，以便请求能访问新的节点服务
- 容灾。当集群中有节点宕掉后，`Nginx`应该能自行调整能被访问的节点的列表。例如集群服务中A节点宕掉后，`Nginx`在10分钟内将不再向此节点转发请求



## Nginx 的优点



- 高并发，高性能（并发性好，轻松处理多请求）
- 可扩展性好（模块化，方便添加插件）
- 高可靠性（`Nginx`能长时间运行，而其他同类产能品需要定期重启来清除垃圾）
- 热部署（不重启的前提下，升级`Nginx`）
- BSD 许可证（开源，且允许修改其源代码）



## Nginx 的组成

由四部分组成

- `Nginx`二进制可执行文件。由各模块源码编译出的一个文件（每次添加新的模块时需要重新生成此文件）
- `nginx.conf`配置文件。控制`Nginx`的行为
- `access.log`访问日志。记录每一条`http`请求信息
- `error.log`错误日志。定位问题



## Nginx 版本选择

版本号格式形如：`aa.bb.cc`

`bb`是单数——不稳定版本；双数——稳定版本



`Tengine`是阿里修改`Nginx`主干代码后的同类型软件。能使用`Nginx`官方版本的的第三方模块。但是其不能与`Nginx`的官方版本同步更新（`MP`就与`MB`同步更新，且完全兼容），所以不推荐使用

PS：`ProcessOn`使用了`Tengine`和`OpenResty`



`OpenResty`，`Lua`语言实现的非阻塞事件框架，用于开发`Nginx`的第三方模块（`Nginx`提供的开发第三方模块实现方式难度大）

`OpenResty`的目标是让你的`Web`服务直接跑在`Nginx`服务内部，充分利用 `Nginx`的非阻塞`I/O`模型，不仅仅对`HTTP`客户端请求，甚至于对远程后端诸如 `MySQL`、以及`Redis`等都进行一致的高性能响应。



## Nginx 的安装方式

两种安装方式

- `Linux`下使用`yum`或`apt`安装
- 手动安装压缩包，并自行解压和编译

但是，如果想要加入第三方模块或开启自带的模块，需要重新编译`Nginx`的二进制可执行文件。所以对于`Nginx`，手动安装更能将对`Nginx`的控制掌握在自己手中

下面讲解手动安装和编译`Nginx`的流程



1. 下载 nginx
2. 介绍各目录
3. Configure
4. 中间文件介绍
5. 编译
6. 安装



编译时指定需要将哪些模块编译进来。编译后生成的`Nginx`的二进制可执行文件即可使用这些模块。如需要添加新的模块，需要重新编译，在编译时指定新的模块将其编译进来



**介绍各目录**

```
L auto		# 目录。存放Nginx支持的特性，对操作系统的判断等
L CHANGES	# 文件。Nginx的更新日志
L CHANGES.ru # 文件。俄文版的更新日志
L conf		# 目录。存放配置文件
L configure	# 可执行文件。用于生成中间文件，执行编译动作
L contrib	# 目录。vim对nginx.conf中的语法没有匹配的高亮设置。将contrib/vim下的所有文件添加到~/.vim目录下，再重新打开nginx.conf即可实现高亮
L html # 目录。存放两个标准的htl文件。50x.html，index.html。访问Nginx服务时默认首页是index.html，响应出现50x时访问50x.html
L LICENSE # 证书文件
L man # 目录。存放对nginx命令的说明文件
L README
L src	# Nginx的源代码
```

此后将此路径都称为`.`便于接下来的描述



**Configure**

编译`Nginx`需要执行`configure`下的可执行文件



有关在编译时，如何开启默认关闭的模块，关闭默认开启的模块，添加新的模块等操作，可输入此命令打印帮助信息

```shell
./configure --help
```

其中`--with-xxx`选项是开启默认关闭的模块，`--without-xxx`选线是关闭默认开始的模块

`--prefix=PATH`选项是指定执行`./configure`后生成的`Nginx`的所有文件的位置（这些新的文件才是运行`Nginx`时使用的文件）

我们将`--prefix`执行的地址称为`Nginx`的安装目录



**中间文件**

在执行完`./configure`后，会在`.`生成`objs/`目录存放中间文件

```
L autoconf,err
L Makefile
L ngx_auto_config.h
L ngx_auto_headers.h
L ngx_modules.c	# 此文件中指定了，接下来编译阶段有哪些模块将被编译进来
L src	# 编译后的C语言文件都放在这里
```



**编译**

在`.`下执行`make`

完成后，`/objs`下会生成`nginx`可执行文件

通常我们将这个文件拷贝到`Nginx`的安装目录下的`sbin/`目录下



PS：如果当前操作是在做`Nginx`的版本升级，那么应该将新生成的可执行文件拷贝到上述目录下再执行`make install`



**安装**

在`.`下（不是安装目录，还是 nginx 的解压缩宝所在目录）执行`make install`（首次安装时可以执行此命令）

安装完成后，安装目录下有以下内容

```
L client_body_temp/
L conf/	# 目录。存放Nginx的配置文件，内容和源码下conf/中的内容相同
L fastcgi_temp/
L html
L logs/	# access.logs和error.log文件在此目录下
L proxy_temp/
L sbin/
L scgi_temp/
L uwsg1 temp/
```



## Nginx 配置文件语法

- 配置文件由指令与指令块构成
- 每条指令以`;`分号结尾，指令与参数间以空格符号分隔
- 指令块以`{}`大括号将多条指令组织在一起
- `include`语句允许组合多个配置文件以提升可维护性
- 使用`#`符号添加注释，提高可读性
- 使用`$`符号使用变量（是`Nginx`提供的变量）
- 部分指令的参数支持正则表达式
- 指令块和指令的具体格式由模块规定（配置文件就是一堆字符串，配置文件的作用就是让程序用读写文件的方式按代码逻辑中规定好的格式去读取，截取指定格式的字符串）



配置文件中的时间单位

| 配置文件中的写法 | 含义            |
| ---------------- | --------------- |
| ms               | milliseconds    |
| s                | seconds         |
| m                | minutes         |
| h                | hours           |
| d                | days            |
| w                | weeks           |
| M                | months, 30 days |
| y                | years, 365 days |



配置文件中的空间单位

| 配置文件中的写法 | 含义      |
| ---------------- | --------- |
| -                | bytes     |
| k/K              | kilobytes |
| m/M              | megabytes |
| g/G              | gigabytes |



以`http`模块为例的指令块，包含有`http`，`server`，`upstream`，`location`



## Nginx 命令行操作

格式：`nginx -s reload`  nginx + 选项 + 参数

帮助：`-?` `-h`

使用执行的配置文件：`-c`（默认使用的配置文件执行`./configure`是指定的配置文件）

指定配置指令：`-g`（覆盖`nginx.conf`中的指令）

指定运行目录：`-p`（如：重新指定日志文件位置）

发送信号：`-s`

- 立即停止服务：`stop`
- 优雅地停止服务：`quit`
- 重载配置文件：`reload`（在`nginx`不停止服务器的前提下重载配置文件）
- 重新开始记录日志文件：`reopen`（清除当前日志文件中的数据并重新记录。通常用定时任务-corntab定期执行`reopen`，防止单个日志文件大小膨胀）

测试配置文件是否有语法错误：`-t` `-T`

打印`nginx`的版本信息、编译信息等：`-v` `-V`



使用场景：重载配置文件，热部署，切割日志文件

**重载配置文件**

前提：`nginx`服务正在运行

1. 修改`nginx.conf`文件
2. 执行`nginx -s reload`



**热部署**

场景：更换`nginx`的版本（以下知识和`nginx`的数据结构给有关）

1. 生成新的`nginx`二进制可执行文件，并替换掉老的二进制文件
2. 使用`kill -USR2 processID`向老的`nginx`的`master`进行发送信号（`nginx`会自动执行新的`nginx`文件，开启新的`master`和`worker`进程）
3. 使用`kill -WINCH processID`向老的`nginx`的`master`进行发送信号（关掉老`worker`进程，并使用新的`master`进程接收请求，但老`master`进程仍存在）

老`master`进程仍然存在，便于新版本发生不稳定时进行版本回退



**切割日志文件**

1. 将当前使用的日志文件`access.log`备份一份
2. 执行`nginx -s reopen`

效果`access.log`文件将被清空，`nginx`将在这个空文件中继续追加日志文件



## Nginx 搭建静态资源 Web 服务器



场景：本地`dlib`目录下有`.html`文件和包含有`.html`文件的子目录

直接打开`.html`文件，浏览器的地址栏上显示的是`file:///+文件地址`

用`nginx`搭建静态资源 Web 服务器流程如下

1. 将`dlib`移指`nginx`的安装目录下

2. 在`nginx.conf`的`server`块中配置如下

   ```
   location / {
   	alias	dlib/; # 所有对/aa/bb的访问都会访问到dlib/aa/bb下
   }
   ```

3. 重载配置文件



**压缩静态文件**

`nginx`可以压缩响应中携带的静态资源。可在`http`块中进行如下配置

```
gzip on; # on—表示开启gzip压缩 off—表示关闭
gzip_min_length 1; # 单位字节，小于此大小的文件不进行压缩
gzip_comp_level 2; # 压缩等级
gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-phpimage/jpegimage/gifimage/png; # 对这些格式的静态资源进行压缩
```

开启文件压缩后，可在响应头中看到`Content-Encoding: gzip`



**开启文件目录索引功能**

参考[这里](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html)

使用场景：做下载站时需要展示所有目录和文件



**开启限速**

参考[这里](http://nginx.org/en/docs/http/ngx_http_core_module.html#variables)

在`location`块中配置限速变量

```
location / {
	set $limit_rate 1K; # 表示响应中的文件传输速度被限制在每秒1K字节，即1KB/s
}
```



**修改 access 日志的日志格式**

需求：希望日志文件的格式为

````
客户端的ip地址 + 请求发送的时间 + 请求的接口地址 + 客户端使用的浏览器及其版本号 + ...
````



修改日志格式的流程：

1. 在`http`块中定义多种日志格式，并为每种格式赋一个名字
2. 在`server`块中指定日志文件的地址和日志文件使用的格式



举例。参考[这里](http://nginx.org/en/docs/http/ngx_http_log_module.html)

```
http {
	# 定义一个日志格式并命名为 main。这里使用的变量可以是所有被编译的模块中的所有合适的变量
	log_format main '$remote_addr - $remote user [$time_local]';
	
	server {
		listen 8080;
		# 指定日志文件的位置和日志格式
		access_log logs/access.log main
	}
}
```



ps：`location `块中的`root`等命令自行查文档即可



## Nginx 搭建具有缓存功能的反向代理服务

反向代理服务器：作为门面的服务器，用于接收客户端发来的请求。再将请求转发到它身后的应用服务器集群中的解节点服务器

其负载均衡算法将做到合理地分配请求，只将请求发给正常的节点，动态添加节点服务器并使其能接收请求等功能



Tips：上游服务器（真正提供接口服务的应用服务器）通常不对公网提供访问，只对反向代理服务器提供访问



**反向代理**

1. 在`http`块中用`upstream`配置上游服务器节点，并为节点块命名
2. 在`location`块中用`proxy_pass`指定上游服务器所在的`upstream`节点块

参考[这里](https://blog.csdn.net/qq_40737025/article/details/85053164)



**缓存**

前提：代理服务器指明会缓存 bbb 资源

缓存流程

1. A 用户访问 bbb 资源
2. 代理服务器向应用服务器请求 bbb 资源，并为 A 用户以 kv 形式缓存起来。k 是用户的标识符，v 是资源
3. A 用户再次访问 bbb 资源。代理服务器用 k 命中 v 后直接返回资源，而不再向应用服务器请求资源



配置流程（参考[这里](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path)）

1. 在`http`块中配置`proxy_cache_path`
2. 在`location`块中配置`proxy_cache`，`proxy_cache_key`，`proxy_cache_valid`

![Snipaste_2021-01-13_15-06-23](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-13_15-06-23.png)

![Snipaste_2021-01-13_15-06-26](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-13_15-06-26.png)



## GoAccess 实现可视化监视 access 日志

> GoAccess 用 WebSocket 协议实现对 access 日志的实时监控



1. 安装下载`goaccess`（apt，yum或手动下载解压并编译源码）
2. 用`goaccess`指令生成一个实时更新的`.html`文件
3. 用`nginx`配置静态资源服务器的形式访问此页面



# 02 Nginx 结构基础



## Nginx 处理请求的流程

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-13_16-42-07.png" alt="Snipaste_2021-01-13_16-42-07" style="zoom:67%;" />

- 尝试用本机的静态资源和磁盘缓存作为响应结果
- 记录 access 访问日志和 error 错误日志
- 继续各种代理



处理网络中的三种流量：Web，电子邮件，TCP流量

传输层状态机：处理TCP，UDP传输层及其下面3层的状态机

HTTP状态机：处理HTTP请求

Mall状态机：处理电子邮件



## Nginx 进程结构

`nginx`有两种进程结构：单进程结构（用于开发，调试环境），多进程结构（用于线上环境。默认是多进程）



多进程中有以下几种进程

- `Master`进程（管理`Worker`进程，比如：`Worker`进程是否需要重新载入配置文件，热部署等）
- `child`进程
  - `Worker`进程（处理请求的进程）
  - Cache相关进程（`Cache Manager`管理缓存，`Cache Loader`载入缓存）



`HTTP`模块中每一个`Server`对应一组如下进程：

一个`Master`进程

一个`Cache Manager`进程，一个`Cache Loader`进程

多个`Worker`进程，其数量和 CPU 数量一致。一个`Worker`绑定到一个 CPU 上还可以充分利用 CPU 的缓存

PS：`Master`进程是其他进程的父进程



**为什么 Nginx 采用的是多进程结构，而不是多线程结构？**

因为`Nginx`要保证高可用和高可靠

多线程下，线程们共享变量或一些资源。如果某个模块中使用的数据的段地址出现越界或其他错误将导致其他线程也会出现问题

多进程下，进程之间不共享数据，不会出现上述错误

`Nginx`中进程间的通信通过共享内存



## Nginx 进程结构实例演示和使用信号管理父子进程

**演示**

场景1：重载配置文件

```shell
$ nginx -s reload
```

`Master`会关闭`Worker`进程，读取新的配置文件后再创建新的`Worker`进程

补充：向`Master`进程发送`SIGHUP`信号的效果和上述重载配置文件效果相同

```shell
$ kill -SIGHUP masterProcessID
```



场景2：关闭`Worker`进程

```shell
$ kill -SIGERM workerProcessID
```

子进程关闭并向`Master`进程发送关闭信号。`Master`检查到当前`Worker`进程因为减少一个导致其数量少于CPU的数量，因此`Master`进程会控制再创建一个`Worker`进程



**小结**：管理员通过向`Master`发送信号来控制`Nginx`。这些信号其实就是操作系统对进程的控制信号（因为`-s`发出的信号都有`kill`发的信号向对应）





## reload 重载配置文件

reload 命令进行了如下行为

1. 向 master 发送 HUP 信号
2. 校验新的配置文件语法是否正确
3. 打开性的监听端口
4. 用新配置文件启动新的 worker 子进程
5. master 向老 worker 子进程发送 UQIT 信号，老 worker 关闭监听，结束进程



这是不停机载入的执行方式：先创建新进程再关闭老进程



可能存在这样的情况：

老进程处理的请求时出现问题导致老worker进程迟迟没有关闭，但是新的请求只会由新的worker进程处理，所以问题不大。只是老的worker进程可能会长时间存在

新版本的 nginx 添加了一个 worker_shutdown_timeout 选项，执行 reload 后，如果老 worker 进程超过指定事件还没有关闭，就会被强制关闭



## 热升级的完整流程



1. 将旧 Nginx 文件换成新 Nginx 文件（注意备份）
2. 向 master 进程发送 USR2 信号
3. master进程修改 pid 文件名，加后缀.oldbin
4. master 进程用新 Nginx 文件启动新 master 进程
5. 向老 master 进程发送 QUIT 信号，关闭老 master 进程
6. 回滚：向老 master 发送 HUP，向新 master 发送 QUIT



第5步中，老的 master 进程并没有被杀死，它依然存在（但是它被关闭了），便于回滚



## 优雅地关闭worker进程

优雅地关闭指：master接收到quit信号后，不再接收新的请求，处理完已经存在的 http 请求后再关闭

（由于协议的定义，nginx无法识别websocket协议和TCP，UDP通信是否已经处理完，所以这里说的优雅地关闭针对的时http请求）



## 网络收发与 Nginx 事件的对应关系

> 所谓“事件”，就是网络事件。`Nginx`中每个网络连接都会有两个网络事件（读事件，写事件）



怎么区分读写事件？

如果服务器只需要读取请求中的数据而不需要在返回的响应中写入数据，那就是读事件（即使服务器读取请求后，服务器最请求中的数据进行计算，但只要不将数据写入响应，不把这些数据发送到网络上那就不是写事件）



这也是 Netty 收到建立链接的请求后注册读事件而不是写事件的原因（读时间指需要读取 socket 中的数据，而不要求向客户端发送数据）



## Nginx 事件驱动模型 和 epoll 模型

nginx 的事件循环采用 epoll

[epoll的本质](https://www.sohu.com/a/317147914_827544)





## 同步&异步、阻塞&非阻塞之间的区别



异步和同步：关注的是消息通信机制。调用者调用完 api 后需要等待方法执行结束并返回结果，就叫同步。如果调用方法后不需要等待也不获取结果就继续向下执行，就叫异步。异步调用的结果通常用通知的方式回调，类似观察者模式

阻塞和非阻塞：关注的是调用者的状态。被调用者需要在执行完结果后再返回结果就叫阻塞。如果不需要执行完结果就直接返回或返回给调用者一个 Feature 就叫非阻塞



Java 中的多路复用 IO 的 NIO是同步非阻塞

即需要获取方法执行的返回值，但方法调用可以直接返回一个 Future（同步），而耗时行为由多线程处理，主线程不需要进入阻塞等待（非阻塞）



## Nginx 的模块

- Nginx 的模块是什么？
- Nginx 模块的分类



> 1. 将模块编译进`Nginx`
> 2. 提供哪些配置项
> 3. 模块何时被使用
> 4. 提供哪些变量



**提供哪些配置项和变量**

可在官网查找

也可打开`src/modules/xxx.c`查找`ngx_command_t`结构体



**模块的分类**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-16_16-37-35.png" alt="Snipaste_2021-01-16_16-37-35" style="zoom:60%;" />

`ngx_core_module`核心模块

`ngx_conf_module`用于解析`nginx.conf`文件的模块（解析配置文件等同于处理字符串，将字符串按某种规则划分成多个有意义的字符串并保存或使用起来）



- 上图可知，模块中也存在模块套模块（父子模块）。就像 Java 中的内部类一样
- 一个模块中有多个子模块，子模块有索引排序，即每个子模块都有自己的编号
- 每个模块中通常都会有一个`xxx_core_module`模块，保存其他模块的通用部分，同时也是父模块中索引为1的子模块



**代码结构中模块的存在形式**

在`Nginx`的下载目录下有这些目录，它们对应各个模块

![Snipaste_2021-01-16_16-47-00](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-16_16-47-00.png)

PS：`core`目录下是`nginx`的核心架构的代码，不是`ngx_core_module`



在这些模块目录下，一些官方提供的可有可无的模块被放在`modules`目录下 



## Nginx 如何通过连接池处理网络请求

`Nginx`的内存池有连接内存池和请求内存池，这里说的是连接内存池

每个 worker 进程都有这三个数组

- 链接数组
- 读事件数组
- 写事件数组



这三个数组的默认大小为512，通常需要调大，因为 nginx 需要有同时处理多个连接的需要



## worker 进程如何共享内存

基础同步工具：信号，共享内存

高级通讯方式：自旋锁，Slab 内存管理器



用于解决读写冲突和内存分配问题





**Slab管理器**

共享内存是一片内存，当给创建的对象分配内存时使用什么内存分配策略，还等保证线程安全。这是Slab管理器做的事

Slab 适合小内存对象的内存管理

因为Slab内存每次分配的大小为2<sup>n</sup>字节。如果对象大小不到2<sup>n</sup>字节也会分配那么多，如果对象不是小对象就会造成大量的大的内存碎片（没有被使用的内存）





可视化监视Slab的内存分配情况

`slab_stat`工具是`Tengine`官方提供的工具，此模块不是独立的模块

需要安装解压`Tengine`，`slab_stat`模块在其`modules`目录下

需要手动`Configure`将其添加进来

```shell
$ curl http://localhost:80/slab_stat
```



## 哈希表的max_size和bucket_siza

`Nginx`常用的6大容器

- 数组
- 链表
- 队列
- **哈希表**
- **红黑树**
- 基数树



## 使用动态模型提高运维效率

- 静态编译生成的可执行文件包含了 core + 其他模块的代码实现静态库
- 动态编译生成的可执行文件包含了 core + 其他模块的声明。模块的实现在 OS 的动态库中

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-18_15-44-21.png" alt="Snipaste_2021-01-18_15-44-21" style="zoom:67%;" />



静态库的代码会直接被编译进可执行文件中

动态库的代码不会被编译进可执行文件。可执行文件中通过通信调用动态库中的模块

动态编译可以方便地向运行时的 nginx 服务添加或更新模块



> PS：不是所有的模块都能作为动态模块被添加进动态库中



一些模块支持动态编译选项，形如`--with-xx-xx=dynamic`，在Configure时添加上这种选项

使用动态编译后，Configure，make后nginx的安装目录下会多出modules目录（不要执行make install，否则会只使用此次Configure中的选项，而不是将其作为追加选项）

目录下有以 .so 结尾的模块文件



在`nginx.conf`中加入

`load_module .so文件的相对路径`命令

即可在下面的命令块中使用新添模块的命令块和命令



再执行 reload 即可



# 03 详解 HTTP 模块

> HTTP模块中包含有繁多的指令和指令块。难记
>
> 此章用于从处理 HTTP 请求的完成流程的角度，以此为主线讲解 HTTP 模块中众多指令和指令块。以方便记忆和理解
>
> 
>
> 包含
>
> - HTTP 模块处理请求的 11 个阶段
> - HTTP 过滤模块-加工响应
> - Nginx 的变量



http 模块处理请求无非是对：收到请求阶段，处理请求阶段，返回响应阶段

- 收到请求阶段，可以执行鉴权，过滤，尝试查缓存等行为
- 处理请求阶段，可以尝试动静分离直接返回本机资源，转发请求，重定向，转发请求前修改请求头&体等
- 返回响应阶段，可以尝试修改响应头&体后再向下游返回响应



nginx 处理 http 请求，首先要找到一个与之匹配的 server 块，再根据 server 块中的规则处理请求



## 冲突的配置指令以谁为准

一些模块使用的指令的值需要在`nginx.conf`中配置

可能出现以下情况

- 配置文件中没有显式声明指令的值
- 一个指令在同级块中被多次声明值
- 一个指令在不同级的块中被多次声明指

这些情况下，以哪些指令的值为准？



以下对指令和指令块统称为指令

**指令的基本存在形式**

````
main # 并没有main块，这里的main指的是配置文件中最外围的部分
http {
	upstream {...}
	split_clients {...}
    map {...}
	geo {...}
	server{
		if() {...}
		location {
			limit_except {...}
		}
		location {
            location{
        	}
        }
	}
	server {
    }  
}
````



指令的格式如下

```
Syntax: log_ format name [escape=default|json|none] string...;
Default: log format combined "...";
Context: http
```

```
Syntax: access_log_path [format [buffersize] [gzip[levell] [flush=time] [if-condition]];
		access_log off;
Default: access_log logs/access log combined;
Context: http,server,location,if in location, limit_except
```

`Syntax`表示指令

`Default`表示指令的默认值

`Context`表示指令能存在于哪些块中



**指令的合并**

值指令：存储配置项的值，一个值指令可以有多个值

​	示例：`root`，`access_log`，`gzip`

动作指令：指定行为，只有声明，没有值，不可以合并

​	示例：`rewrite`，`proxy_pass`



值指令的继承规则：向上覆盖

子配置块中不存在时，直接使用父配置块

子配置块中存在时，不使用父配置块



## Listen 指令的用法

```
# 指定监听的地址和端口（因为一台服务器可能有多个网卡——多个ip地址，所以要指明监听的是哪个ip下的端口）
listen address[:port]
# 指定监听的端口
listen port
# 指定监听的socket地址，只能用于本机通讯
listen unix:path
```



`listen`的使用方式见[官网](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen)



## 处理 HTTP 请求头部的流程

**接收请求事件模块**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-18_18-36-40.png" alt="Snipaste_2021-01-18_18-36-40" style="zoom:67%;" />

**接收请求HTTP模块**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-18_18-46-00.png" alt="Snipaste_2021-01-18_18-46-00" style="zoom:67%;" />

标识`header`会做一些事，比如选择哪个`server`块来处理这个请求

进入11个阶段的`http`请求处理前的工作都是`Nginx`框架核心代码执行的

进入11个阶段的`http`请求处理后就是进入了各个模块中。而这些11个阶段是`Nginx`框架规定的。（这是模板方法。`Nginx`框架规定好核心流程后，动态调用所有模块的回调方法，回调方法由模块自行实现）



## 如何找到处理请求的 server 指令块



**Nginx 中的正则表达式**通常被用来匹配 url 或域名等

例如，location 指令中可以使用正则表达式匹配指定格式 url



**server_name指令**

在`server_name`指令下指定域名或 ip，可以同时指定多个域名或 ip。指明此`server`块监听的域名或 ip 地址

> Syntax:	server_name name ...;
> Default:	server_name "";
> Context:	server



1. 指令后可以跟多个域名，第一个是主域名

   如果没有开启`server_name_redirect off`指令（默认关闭），发送请求时，返回的响应中服务器地址就是请求中的服务器地址

   如果开启`server_name_redirect on;`指令。发送请求时，返回的响应报文中的服务器地址是主域名的地址

   > Syntax:	server_name_in_redirect on | off;
   > Default:	server_name_in_redirect off;
   > Context:	http, server, location

2. *泛域名：仅支持在前或最后

   例如：`server_naem *.geek.com;`

3. 正则表达式：加~前缀

   例如：`server_name ~^www.geek.com;`

4. 使用正则表达式创建变量：略

5. 其他

   - `.geek.com`可以匹配`geek.com`和`*.geek.com`
   - `_`匹配所有
   - "" （双引号表示的空字符串）匹配没有传递Host头部



**server 块的匹配顺序**

解析请求行里的 Host，并拿来和所有`server`块中的`server_name`进行匹配，匹配上哪个块就用哪个块中的指令

假设请求行里的 Host 是`geek.com`

匹配顺序如下：

1. 精确匹配（`geek.com`与所有直接使用普通字符串的`server_name`匹配）

2. *在前的泛域名

3. *在后的泛域名

4. 按文件中的从上到下顺序匹配使用正则表达式的`server_name`

5. `default server`
   - 默认是第一个`server`块
   
   - 也可使用`listen`指令 + `default_server`指定此`server`块是`default`
   
     ```
     listen      80 default_server;
     ```
   
     



使用上述规则找到`server`块后，使用块中指令处理请求



## 详解处理HTTP请求的11个阶段

> `Nginx`中除HTTP的过滤模块和只提供变量的模块之外，所以其他HTTP模块都必须在11个阶段中使用



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-18_21-07-07.png" alt="Snipaste_2021-01-18_21-07-07" style="zoom:67%;" />

左侧是11阶段示意图

右侧是具体的11阶段和常用的`HTTP`模块



**11阶段的处理顺序**

通常一个阶段可以有多个模块处理

但是一个模块如果不将请求传给下一个模块，下一个模块将无法处理请求（过滤器&责任链模式）

可能存在这种情况：

	- 一个模块将请求传递给下一个阶段中的模块，而同阶段中的其他模块将没有机会处理请求
	- 一个模块直接返回响应，而同阶段的其他模块和余下阶段的所有模块都不处理请求

遵循以上规则后，以一些模块为例，介绍11阶段的处理流程

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-18_21-12-52.png" alt="Snipaste_2021-01-18_21-12-52" style="zoom:67%;" />



### postread 阶段

获取真实客户端地址的`realip`模块



场景：在使用多层反向代理后，反向代理接收到的请求报文中请求的源地址是另一个反向代理服务器的地址，而不是用户的 ip 地址

问题：如何拿到真实的用户IP地址？因为`nginx`可能会需要用用户的 ip 做限流等操作



前提知识

- `TCP`连接四元组（src ip, src port, dst ip, dst port）
- `HTTP`头部`X-Forwarded-For`用于传递`IP`
- `HTTP`头部`X-Real-IP`用于传递用户`IP`
- 前提：网络中存在许多反向代理



**Nginx 获取真实用户IP后如何使用**

`nginx`用变量保存 ip 地址，如`binary_remote_addr`、`remote_addr`

它们的值默认是之间创建`TCP`连接的`IP`。但是我们想获取用户真正的`IP`地址

于是`realip`模块在这个阶段将这两个变量的值改为`X-Real-IP`或`X-Forwarded-For`的值。这样就可以将变量的值用于`limit_conn`模块做限流



**realip模块**

- 模块默认不会编译进`Nginx`，需要通过`--with-http_realip_module`启用功能

- 功能：修改客户端地址

- 提供变量：`realip_remote_addr`、`realip_remote_port`保存直接创建`TCP`连接的`IP`地址（因为`realip`将保存源地址的变量修改为 了`X-Real-IP`或`X-Forwarded-For`的值）

- 提供的指令

  - `set_real_ip_from`如果 scr dst 的值和此指令的值相等，`realip`才会为`binary_remote_addr`、`remote_addr`

    的值做替换工作

  - `real_ip_header` 决定`binary_remote_addr`、`remote_addr`的值是`X-Real-IP`还是`X-Forwarded-For`

  - `real_ip_recursive` 如果`X-Real-IP`和`X-Forwarded-For`的最后一个地址相同，那么取`X-Forwarded-For`最后一个地址的上一个地址作为真正的用户`IP`



### rewrite 阶段



**return指令**

执行 return 指令后，直接返回响应，其他模块将得不到处理请求的机会



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_15-11-24.png" alt="Snipaste_2021-01-19_15-11-24" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_15-13-43.png" alt="Snipaste_2021-01-19_15-13-43" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_15-14-19.png" alt="Snipaste_2021-01-19_15-14-19" style="zoom:67%;" />



问题1：有 return404 后，error_page 404 /403.html还会执行吗？

答：没有机会

问题2：return指令同时出现在 server 块和 location 块下还会有指令合并的行为吗？

答：动作类指令没有合并关系，一旦执行到一个，另一个就不会被执行。通常动作类指令先执行最外围块中的指令。如上述图示中，404 时返回 403.html 而不是"find nothing!"



**rewrite 指令重写 URL**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_15-21-58.png" alt="Snipaste_2021-01-19_15-21-58" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_15-35-34.png" alt="Snipaste_2021-01-19_15-35-34" style="zoom: 80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_15-35-52.png" alt="Snipaste_2021-01-19_15-35-52" style="zoom: 80%;" />

`rewrite_log`指令开启后：`rewrite_log on;`

每次执行`rewrite`指令后都会在`error.log`中记录一条日志



**条件判断**

用 if 指令和变量的值组成条件判断，如果条件成立就执行条件块中的指令

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_15-38-14.png" alt="Snipaste_2021-01-19_15-38-14" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-08-36.png" alt="Snipaste_2021-01-19_16-08-36" style="zoom: 80%;" />

![Snipaste_2021-01-19_16-08-41](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-08-41.png)

### find_config 阶段

> 找处理请求的`location`指令块
>
> 此阶段后，其他阶段提供的指令大多数都可以在`location`指令块中使用

![Snipaste_2021-01-19_16-21-16](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-21-16.png)

![Snipaste_2021-01-19_16-21-20](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-21-20.png)

![Snipaste_2021-01-19_16-21-42](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-21-42.png)

![Snipaste_2021-01-19_16-21-38](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-21-38.png)



### preaccess阶段

> 这个阶段可以作限制并发连接数的工作

**对连接数做限制的limit_conn模块**

![Snipaste_2021-01-19_16-39-29](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-39-29.png)



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-39-50.png" alt="Snipaste_2021-01-19_16-39-50" style="zoom:67%;" />

![Snipaste_2021-01-19_16-40-10](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-40-10.png)

![Snipaste_2021-01-19_16-40-20](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_16-40-20.png)

**对请求数作限制的limit_req模块**

限制每个连接能同时处理多少个请求

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-00-15.png" alt="Snipaste_2021-01-19_17-00-15" style="zoom:67%;" />

限流算法：`leaky bucket`算法（漏桶算法）

![Snipaste_2021-01-19_17-17-02](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-17-02.png)

漏桶算法需要设置：桶有多大，桶的漏水速度是多少

![Snipaste_2021-01-19_17-17-45](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-17-45.png)

![Snipaste_2021-01-19_17-17-51](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-17-51.png)

![Snipaste_2021-01-19_17-18-09](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-18-09.png)

问题：当`limit_req`和`limit_conn`配置同时生效时，哪个有效？

答：`limit_req`有效。因为`limit_req`模块先于`limit_conn`模块处理请求，当请求达到漏桶极限时，直接返回503（`limit_req`），

`limit_conn`将没有机会处理请求



### access阶段

> 控制权限
>
> 比如设置ip黑名单。当黑名单中的客户端访问时，返回拒绝访问的响应
>
> 对于用户名密码的权限校验，虽然nginx提供了实现，但是这件事还是让上游的认证服务来做更为专复杂，安全，全面



**对IP作限制的access模块**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-24-49.png" alt="Snipaste_2021-01-19_17-24-49" style="zoom:67%;" />

![Snipaste_2021-01-19_17-25-01](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-25-01.png)

可用于内网中服务器之间简单的权限控制



**对用户名密码做限制的auth_basic模块**

![Snipaste_2021-01-19_17-30-51](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-30-51.png)

![Snipaste_2021-01-19_17-31-00](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-31-00.png)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-31-11.png" alt="Snipaste_2021-01-19_17-31-11" style="zoom:80%;" />



**使用第三方做权限控制的auth_request模块**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-40-59.png" alt="Snipaste_2021-01-19_17-40-59" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_17-41-17.png" alt="Snipaste_2021-01-19_17-41-17" style="zoom:67%;" />

但使用`Shiro`或`Spring Security`时，接口中有过滤器，如果请求没有权限，请求会直接被过滤器过滤掉



而`nginx`的这个模块的做法是，访问资源的请求访问到`nginx`时会被发送一个子请求，根据子请求返回的响应决定源请求是否有权限访问接口资源



**satify指令**

![Snipaste_2021-01-19_22-07-33](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-19_22-07-33.png)

- access阶段中三个模块的执行顺序就是`access`——>`auth_basic`——>`auth_request`

- `satisfy`指令的作用和`Spring Security`中的投票器投票策略类似

  `satisfy all;`类似一票否决。只要有一个模块不放行请求，`access`阶段就返回4xx/5xx的响应

  `satisfy any;` 类似一票同意。只要有一个模块同意放行请求，就执行`access`阶段的下一个模块





### precontent阶段



**按序访问资源的try_file模块**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-01-10.png" alt="Snipaste_2021-01-20_13-01-10" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-01-33.png" alt="Snipaste_2021-01-20_13-01-33" style="zoom:67%;" />

反向代理中经常用到（感觉没啥用）



**实施拷贝流量的mirror模块**

生产环境下处理请求时，可能需要将请求同步或复制一份发到测试环境中。mirror能创建一份镜像流量，复制一份请求发送到测试环境中

![Snipaste_2021-01-20_13-08-23](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-08-23.png)

也感觉没啥用

### content阶段



> `static`模块提供`root`指令和`alias`指令。这个模块默认是`nginx`框架中的，所以无法移除

**详解root和alias指令**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-31-16.png" alt="Snipaste_2021-01-20_13-31-16" style="zoom:67%;" />

![Snipaste_2021-01-20_13-31-35](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-31-35.png)



**sattic模块提供的3个变量**

![Snipaste_2021-01-20_13-37-43](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-37-43.png)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-38-06.png" alt="Snipaste_2021-01-20_13-38-06" style="zoom:67%;" />

![Snipaste_2021-01-20_13-38-20](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-38-20.png)



**static模块对url不易斜杠结尾却访问目录的做法**

![Snipaste_2021-01-20_13-43-32](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-43-32.png)

![Snipaste_2021-01-20_13-43-42](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_13-43-42.png)



**index模块和autoindex模块**

> 开启autoindex后，当我们访问一个目录时，我们希望返回的是目录下的文件列表。但是返回的是某个文件的文件内容
>
> 这是因为index模块先于autoindex生效，访问一个目录时，如果目录下有index.html文件。返回结果就是该目录下的index.html的内容
>
> 如果希望访问目录，且返回结果不是index.html文件。那就需要删除index.html文件或用index指令修改index的默认值

![Snipaste_2021-01-20_14-36-18](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_14-36-18.png)



![Snipaste_2021-01-20_14-36-41](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_14-36-41.png)



![Snipaste_2021-01-20_14-36-52](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_14-36-52.png)



![Snipaste_2021-01-20_14-37-07](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_14-37-07.png)



**提升多个小文件性能的concat模块**

> 阿里巴巴提供的模块。以此响应中返回多个小文件的模块



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_15-53-54.png" alt="Snipaste_2021-01-20_15-53-54" style="zoom:67%;" />

淘宝网就用到了，比如在控制台的`NetWork`选项中就有`https://g.alicdn.com/??secdev/entry/index.js,alilog/oneplus/entry.js`



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_15-54-30.png" alt="Snipaste_2021-01-20_15-54-30" style="zoom:80%;" />

![Snipaste_2021-01-20_15-54-52](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_15-54-52.png)



### log阶段

> 用来记录`access.log`日志的模块

功能：将`HTTP`请求相关信息记录到日志中

模块：`ngx_http_log_module`，无法禁用

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_16-02-34.png" alt="Snipaste_2021-01-20_16-02-34" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_16-02-51.png" alt="Snipaste_2021-01-20_16-02-51" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_16-03-04.png" alt="Snipaste_2021-01-20_16-03-04" style="zoom:80%;" />



## HTTP 过滤模块

> 处理 HTTP 请求的11阶段执行完后，生成响应。HTTP过滤模块是用于加工响应内容的
>
> 它是在 log 阶段前，content 阶段后生效的



![Snipaste_2021-01-20_16-13-15](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_16-13-15.png)



### HTTP过滤模块的调用流程

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_16-14-02.png" alt="Snipaste_2021-01-20_16-14-02" style="zoom:80%;" />



### 过滤模块更改响应中的字符串：sub模块

功能：将响应中指定的字符串，替换成新的字符串

模块：`ngx_http_sub_filter_module`模块，默认未编译进`Nginx`,。通过`--with-http_sub_module`启用

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_16-22-47.png" alt="Snipaste_2021-01-20_16-22-47" style="zoom:80%;" />

![Snipaste_2021-01-20_16-23-03](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_16-23-03.png)



### 过滤模块在http想用的前后添加内容：addition模块

功能：和`sub`模块不同，`addition`模块是向响应结果前后添加内容，添加的内容由指定的子请求的响应结果而定

模块：`ngx_http_addition_filter_module`，默认未编译进`Nginx`，通过`--with-http_addition_module`启用

![Snipaste_2021-01-20_16-34-06](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_16-34-06.png)

![Snipaste_2021-01-20_16-34-16](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_16-34-16.png)

## Nginx变量的运行原理

![Snipaste_2021-01-20_18-46-33](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_18-46-33.png)

变量的特性

- 惰性求值：在HTTP11阶段中，需要使用变量时再调用解析变量的方法获取变量的值
- 变量的值可以时刻变化，其值为使用的那一时刻的值

![Snipaste_2021-01-20_18-48-24](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_18-48-24.png)





### HTTP框架提供的变量

示例（变量和某次请求中获取到的变量值）

 ![Snipaste_2021-01-20_19-42-39](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_19-42-39.png)

![Snipaste_2021-01-20_19-43-29](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-20_19-43-29.png)





HTTP框架提供的变量

- HTTP请求相关的变量
- TCP连接相关的变量
- Nginx处理请求过程中产生的变量
- 发送HTTP响应时相关的变量
- Nginx系统变量

先举一些和HTTP请求相关的变量，再举一些其他变量

**HTTP请求相关的变量**

| 变量名            | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| arg_参数名        | URL中某个具体参数的值                                        |
| query_string      | 与args变量完全相同                                           |
| args              | 全部URL参数                                                  |
| is_args           | 如果请求URL中有参数则返回?否则返回空                         |
| content_length    | HTTP请求中标识包体长度的Content-Length头部的值               |
| content_type      | 标识请求包体类型的Content-Type头部的值                       |
| uri               | 请求的URI（不同于URL，不包括?后的参数）                      |
| document_uri      | 与uri相同                                                    |
| request_uri       | 请求的URL（包括URI以及完整的参数）                           |
| scheme            | 协议名，例如HTTP或HTTPS                                      |
| request_method    | 请求方法，例如GET或POST                                      |
| request_length    | 所有请求内容的大小，包括请求行，头部，包体                   |
| remote_user       | 由HTTP Basic Authentication协议传入的用户名                  |
| request_body_file | 临时存放请求的文件（此变量在第四部分再做详解）<br>- 如果包体非常小则不会存文件<br>- client_body_in_file_only 强制所有包体存入文件，且可决定是否删除 |
| request_body      | 请求中包体，这个变量当且仅当使用反向代理，且设定用内存暂存包体时才有效 |
| request           | 原始的url请求，含有方法与协议版本，例如GET /?a=1&b=2 HTTP/1.1 |
| host              | 请求资源所在服务器的域名或IP<br>1. 先从请求行里获取<br>2. 如果含有Host头部，则使用其值替换掉请求行中的主机名<br>3. 如果前两者都取不到，则使用匹配上的server_name（不一定是主域名，而是匹配上的server_name） |
| http_头部名字     | 返回一个具体请求的头部的值                                   |

- 上述的URL参数是url后的?aa=xx&aabb=xx

- http_头部名字

  特殊处理的头部`http_host`, `http_user_agent`, `http_referer`, `http_via`, `http_x_forwarede_for`, `http_cookie`

  只是nginx在获取这些字段时做了一些处理，对我们来说这些处理时透明的



**TCP连接相关的变量**

| 变量名              | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| binary_remote_add   | 客户端地址的整形格式，对于IPv4是4字节，对于IPv6是16字节      |
| connection          | 递增的连接序号                                               |
| connection_requests | 当前连接上执行过的请求数，对于keepalive连接有意义            |
| remote_addr         | 客户端地址                                                   |
| remote_port         | 客户端端口                                                   |
| proxy_protocol_addr | 若使用了proxy_protocol协议则返回协议中的地址，否则返回空<br>此地址和realip类似，区别在于realip用于获取真实IP问题，此变量用于获取多层反向代理中真实的地址 |
| proxy_protocol_port | 若使用了proxy_protocol协议则返回协议中的端口，否则返回空（和上一个变量一样，获取真实的客户端的端口号） |
| server_addr         | 服务器端地址                                                 |
| server_port         | 服务器端端口                                                 |
| TCP_INFO            | tcp内核层参数，包括$tcpinfo_rtt，$tcpinfo_rttvar，$tcpinfo_snd_cwnd，$tcpinfo_rcv_space |
| server_protocol     | 服务器端协议，例如HTTP/1.1                                   |



`Nginx`处理请求过程中产生的变量

| 变量名             | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| request_time       | 请求处理到现在的耗时，单位为秒，精确到毫秒                   |
| server_name        | 匹配上请求的server_name值                                    |
| https              | 如果开启了TLS/SSL，则返回on，否则返回空                      |
| request_completion | 若请求处理完则返回OK，否则返回空                             |
| request_id         | 以16进制输出的请求标识id，该id共含有16个字节，是经过4次random后随机生成的 |
| request_filename   | 待访问文件的完整路径                                         |
| document_root      | 由URI和root/alias规则生成的文件夹路径                        |
| realpath_root      | 将document_root中的软连接等替换成真实路径                    |
| limit_rate         | 返回客户端响应时的速度上限，单位为每秒字节数。可以通过set指令修改对请求产生效果 |



**发送HTTP响应时相关的变量**

| 变量名             | 描述                   |
| ------------------ | ---------------------- |
| body_bytes_sent    | 响应中body包体的长度   |
| bytes_sent         | 全部http响应的长度     |
| status             | http响应中的返回码     |
| sent_trailer_名字  | 把响应结尾内容里值返回 |
| sent_http_头部名字 | 响应中某个具体头部的值 |

- `sent_http_头部名字`中也有被特殊处理的头部，对我们来说这些处理时透明的



**Nginx系统变量**

| 变量名        | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| time_local    | 以本地时间标准输出的当前事件，例如14/Nov/2018:15:55:37 +0800 |
| time_iso8601  | 使用ISO 8601标准的当前时间，例如2018-11-14T15:55:37+08:00    |
| nginx_version | Nginx版本号                                                  |
| pid           | 所属worker进程的进程id                                       |
| pipe          | 使用了管道则返回p，否则返回.                                 |
| hostname      | 所在服务器的主机名，与hostname命令输出一致                   |
| msec          | 1970年1月1日到现在的时间，单位为秒，小数点后精确到毫秒       |



### 使用防盗链模块体会变量的使用（referer模块和secure_link模块）

**referer模块**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_13-56-44.png" alt="Snipaste_2021-01-21_13-56-44" style="zoom:60%;" />

![Snipaste_2021-01-21_13-57-06](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_13-57-06.png)

![Snipaste_2021-01-21_13-57-19](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_13-57-19.png)

![Snipaste_2021-01-21_13-57-30](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_13-57-30.png)

客户端可以很容易修改请求中的`referer`头部的值，所以只使用`referer`模块只能实现简单的防盗链处理

尽管客户端能轻松修改`referer`头部，但是多数网站都不会去修改（可能是懒或者是笨）



**secure_link**

> 比`referer`模块更安全的防盗链模块

![Snipaste_2021-01-21_14-39-53](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_14-39-53.png)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_14-40-11.png" alt="Snipaste_2021-01-21_14-40-11" style="zoom:60%;" />



使用`secure_link`有两种防盗链的实现

- 对某个用户和时间戳进行防盗
- 对某个资源进行防盗（实现更简单）



**对某个用户和时间戳进行防盗**

![Snipaste_2021-01-21_14-43-15](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_14-43-15.png)

![Snipaste_2021-01-21_14-43-35](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_14-43-35.png)

**对某个资源进行防盗**

![Snipaste_2021-01-21_14-44-09](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_14-44-09.png)

![Snipaste_2021-01-21_14-44-20](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_14-44-20.png)



### 为复杂的业务生成新的变量：map模块

功能：基于已有变量，使用类似switch-case语法创建新变量，为其他基于变量值实现功能的模块提供更多可能性

模块：`ngx_http_map_module`，默认编译进`Nginx`，通多`--without-http_map_module`禁用

![Snipaste_2021-01-21_15-11-14](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-11-14.png)

![Snipaste_2021-01-21_15-11-24](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-11-24.png)

![Snipaste_2021-01-21_15-11-32](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-11-32.png)



### 通多变量指定少量用户实现AB测试：split_client模块

> AB测试是：开发软件时开发A和B两个版本，用户从两个版本中选一个用

![Snipaste_2021-01-21_15-19-25](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-19-25.png)

![Snipaste_2021-01-21_15-19-35](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-19-35.png)

问题：百分比相加大于100%了

例子

![Snipaste_2021-01-21_15-20-07](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-20-07.png)

请求发来后，获取到`$http_testcli`的值，并对其进行哈希算法，得到哈希值后除以max得到百分比，根据百分比结果进行类似switch-case选择，将结果赋给`$variant`





### 根据IP地址范围的匹配生成新变量：geo模块

> 还是根据其他变量生成新变量，但是只能根据IP地址，子网掩码等生成变量
>
> 子网掩码：结合IP地址后，可以表示一个IP地址的范围

![Snipaste_2021-01-21_15-29-59](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-29-59.png)

![Snipaste_2021-01-21_15-30-13](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-30-13.png)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-30-25.png" alt="Snipaste_2021-01-21_15-30-25" style="zoom:60%;" />

### 使用变量获取用户的地理位置：geoip模块

![Snipaste_2021-01-21_15-46-09](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-46-09.png)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-46-21.png" alt="Snipaste_2021-01-21_15-46-21" style="zoom:60%;" />

`gepip_city`指令提供的变量

> Syntax：geoip_city file;
>
> Default：—
>
> Context：http

![Snipaste_2021-01-21_15-47-44](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-47-44.png)



## 对客户端使用keepalive提升连接效率

> 这里说的keepalive，说的是http协议的，不是tcp协议的

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-53-18.png" alt="Snipaste_2021-01-21_15-53-18" style="zoom:60%;" />

![Snipaste_2021-01-21_15-53-33](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_15-53-33.png)





# 04反向代理与负载均衡

> 前三部分讲了官方提供的除负载均衡和反向代理以外的模块。此部分讲负载均衡和反向代理模块的相关知识
>
> 主线：一个请求发送到`nginx`，反向代理+负载均衡又将请求发到上游服务器，上有服务器再将响应返回到客户端

> 反向代理与负载均衡（`http`模块的7层反向代理和`stream`模块的4层反向代理，搭建不同上游协议的反向代理，控制上下游流量的交互）
>
> 反向代理的知识中会涉及负载均衡和缓存这两个知识（负载均衡是反向代理中的知识）



## 反向代理和负载均衡原理



场景：选课管理系统的所有后端接口服务均在一台服务器上的一个项目上运行。当请求量大时，需要对上有服务器进行扩容

水平扩容：增加服务器。将同一个应用部署到多台服务器上，利用反向代理+负载均衡分摊请求

​					成本低。因为只需要将应用原封不动地多部署一些，再在反向代理服务器上动态扩容即可

垂直扩容：将一个服务拆分成多个服务。根据不同用户对服务的请求比例，在更多服务器上部署用户频繁访问的服务

​					成本高。因为将一个服务拆分成多个服务通常需要改代码

基于z轴扩容（上述两种是基于x，y轴扩容）：用CDN，或将指定用户的请求固定分发到指定的上有服务器（哈希算法）



PS:Round-Robin和least-connected算法是基于水平扩容的负载均衡算法

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_16-20-04.png" alt="Snipaste_2021-01-21_16-20-04" style="zoom:60%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-21_16-20-27.png" alt="Snipaste_2021-01-21_16-20-27" style="zoom:67%;" />



反向代理与缓存

缓存方式：

- 时间缓存（A用户请求z资源后资源被缓存到反向代理服务器，B用户请求资源时直接命中缓存）
- 空间缓存（反向代理服务器预先请求资源并缓存到本地）



## 反向代理策略：round-robin

> `nginx`与上游服务的交互是由`upstream`模块完成的（包括`upstream`，`stream`，`httpstream`）
>
> `nginx`的上游服务：应用服务器，资源服务器
>
> `nginx`的下游服务：指所有访问服务的浏览器客户端

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-28-47.png" alt="Snipaste_2021-01-22_13-28-47" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-29-11.png" alt="Snipaste_2021-01-22_13-29-11" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-29-29.png" alt="Snipaste_2021-01-22_13-29-29" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-29-41.png" alt="Snipaste_2021-01-22_13-29-41" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-29-54.png" alt="Snipaste_2021-01-22_13-29-54" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-30-05.png" alt="Snipaste_2021-01-22_13-30-05" style="zoom:80%;" />



`Round-Robin`算法是其他负载均衡算法的基础。此外，当其他算法失效时，通常也会使用`Round-Robin`算法进行负载均衡



## 负载均衡哈希算法：ip_hash与hash模块

![Snipaste_2021-01-22_13-46-23](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-46-23.png)

![Snipaste_2021-01-22_13-46-33](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-46-33.png)

![Snipaste_2021-01-22_13-46-49](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-46-49.png)



只是用这些配置的话。当某台上游服务器挂掉后，负载均衡的哈希算法不会重载哈希值，而是继续让请求发送到这台挂掉的上游服务

因为默认强制让指定用户或key访问指定上游服务，防止出现问题，哪怕某台上游服务器已经挂掉

下节的一致性哈希算法可以解决这个问题



## 一致性哈希算法：hash模块

![Snipaste_2021-01-22_13-57-49](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-57-49.png)

![Snipaste_2021-01-22_13-58-01](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_13-58-01.png)

由上图可知，如果使用哈希算法的负载均衡同时实现了动态扩容和容灾后，每次增减上游服务器节点时会导致用户的原目标上有服务器发生改变。这会导致用户使用的大量缓存失效从而引起恶性循环

一致性哈希算法能解决这个问题

![Snipaste_2021-01-22_14-00-21](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_14-00-21.png)

一致性哈希算法的原理见：左神算法 

问题：如何在使用哈希算法时开启一致性哈希算法？

答：使用基于任意关键字实现哈希算法的负载均衡中，提供的`hash key [consistent]`指令中，加上`consistent`这个可选参数即可



## 最少连接算法选取连接最少的上游服务节点：least-connection

> 此外还要介绍upstream_zone模块。所有负载均衡算法默认只在一个worker进程中生效。此模块将所有的上游服务信息存放在共享内存中，而不是一个进程独享



**upstream_least_conn模块**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_14-36-51.png" alt="Snipaste_2021-01-22_14-36-51" style="zoom:80%;" />





**upstream_zone模块**

 ![Snipaste_2021-01-22_14-37-11](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_14-37-11.png)

![Snipaste_2021-01-22_14-37-39](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_14-37-39.png)



## httpstream模块提供的变量（不含Cache）



| 变量名                  | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| upstream_addr           | 上游服务器的IP地址，格式为可读的字符串，例如 127.0.0.1:8012  |
| upstream_connect_time   | 与上游服务建立连接消耗的时间，单位为秒，精确到毫秒           |
| upstream_header_time    | 接收上游服务发回响应中http头部所消耗的时间，单位为秒，精确到亳秒 |
| upstream_response_time  | 接收完整的上游服务响应所消耗的时间，单位为秒，精确到毫秒     |
| upstream_http_名称      | 从上游服务返回的响应头部的值                                 |
| upstream_bytes_received | 从上游服务接收到的响应长度，单位为字节                       |
| upstream_cookie_名称    | 从上游服务发回的响应头 Set-Cookie 中取出的cookie值           |
| upstream_trailer_名称   | 从上游服务的响应尾部取到的值                                 |





## http反向代理proxy模块

> 从此开始介绍反向代理模块。先介绍第一个反向代理模块`proxy`
>
> 以proxy反向代理模块处理请求的流程为主线，讲解proxy模块

**http反向代理proxy处理请求的流程**

![Snipaste_2021-01-22_16-09-02](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-09-02.png)

问题1：`nginx`为什么先生成请求再和上游服务器建立连接，而不是先建连接再生成请求？

答：先生成请求后，建完连接就能发请求。这样比较快

问题2：为什么`proxy_request_buffering`和`proxy_buffering`默认值是`on`

答：on表示将读取完请求体后再发请求，off表示边读请求体边发请求体。后者会导致发送请求时间边长。而上游服务，例如tomcat并发能力差，同一时间能接收的请求数有限



**proxy模块中的proxy_pass指令**



功能：对上游服务使用http/https协议进行反向代理

指令：`ngx_http_proxy_module`，默认编译进`nginx`，同故宫`--wothout-http_proxy_module`禁用

开启指令：`proxy_pass`

> Syntax：proxy_pass URL;
>
> Default：—
>
> Context：location，if in location，limit_expect

![Snipaste_2021-01-22_16-26-39](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-26-39.png)



**根据指令修改发往上游的请求**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-38-12.png" alt="Snipaste_2021-01-22_16-38-12" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-38-23.png" alt="Snipaste_2021-01-22_16-38-23" style="zoom:80%;" />

![Snipaste_2021-01-22_16-38-37](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-38-37.png)



**接收用户请求包体的方式**

> 接收用户请求体的操作由`nginx`的核心框架规定，但是`proxy`模块经常使用这一操作，所以需要了解

![Snipaste_2021-01-22_16-51-59](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-51-59.png)

![Snipaste_2021-01-22_16-52-08](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-52-08.png)

![Snipaste_2021-01-22_16-52-20](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-52-20.png)

![Snipaste_2021-01-22_16-55-04](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-55-04.png)

![Snipaste_2021-01-22_16-55-20](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_16-55-20.png)

**与上游服务建立连接**

PS：`proxy_pass`是动作类指令，优先于`root`等指令前生效

![Snipaste_2021-01-22_17-08-49](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-08-49.png)

![Snipaste_2021-01-22_17-09-00](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-09-00.png)

![Snipaste_2021-01-22_17-09-10](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-09-10.png)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-10-06.png" alt="Snipaste_2021-01-22_17-10-06" style="zoom:80%;" />

![Snipaste_2021-01-22_17-10-18](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-10-18.png)

![Snipaste_2021-01-22_17-10-32](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-10-32.png)



**接收上游的响应**

![Snipaste_2021-01-22_17-32-34](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-32-34.png)

![Snipaste_2021-01-22_17-32-44](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-32-44.png)

![Snipaste_2021-01-22_17-32-55](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-32-55.png)

![Snipaste_2021-01-22_17-33-03](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-33-03.png)

![Snipaste_2021-01-22_17-33-13](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-33-13.png)

![Snipaste_2021-01-22_17-33-27](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-22_17-33-27.png)



**处理上游的响应头部**

上游服务器返回的响应头部中有的头部字段能直接控制`nginx`的行为

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_13-06-41.png" alt="Snipaste_2021-01-23_13-06-41" style="zoom:80%;" />

![Snipaste_2021-01-23_13-06-55](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_13-06-55.png)

![Snipaste_2021-01-23_13-07-08](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_13-07-08.png)

![Snipaste_2021-01-23_13-07-20](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_13-07-20.png)

![Snipaste_2021-01-23_13-07-32](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_13-07-32.png)

上述就是`nginx`接收下游客户端请求，转发请求，接收上游服务器响应，转发响应的全流程



## 上游出现失败时的容错方案

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_13-43-04.png" alt="Snipaste_2021-01-23_13-43-04" style="zoom:67%;" />

![Snipaste_2021-01-23_13-43-16](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_13-43-16.png)

![Snipaste_2021-01-23_13-43-25](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_13-43-25.png)



## 对上游使用SSL连接

> 之前说的SSL连接是客户端检查`nginx`的证书
>
> 这次说的是`nginx`检查客户端的证书；`nginx`和上游服务互相检查证书者3中场景

问题：上游服务通常都在内网部署，用内网和`nginx`进行通信/交互。默认就是受信任的服务，那为什么对上游服务还要使用SSL连接？

问题：检查证书不是指客户端单向检查服务端的证书吗？为什么这里出现了服务端要求检查客户端证书的场景？ 

![Snipaste_2021-01-23_18-41-56](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_18-41-56.png)

![Snipaste_2021-01-23_18-42-19](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_18-42-19.png)

![Snipaste_2021-01-23_18-42-31](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_18-42-31.png)

![Snipaste_2021-01-23_18-42-41](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_18-42-41.png)



## 缓存

缓存是最有效的提升访问速度的手段



### 用好浏览器的缓存

`nginx`可以通过控制响应来控制浏览器自身使用的缓存是否生效

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-01-15.png" alt="Snipaste_2021-01-23_21-01-15" style="zoom:80%;" />

![Snipaste_2021-01-23_21-01-37](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-01-37.png)

![Snipaste_2021-01-23_21-01-52](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-01-52.png)



`etag`指令

> Synatx：etag on | off;
>
> Default：etag on;
>
> Context：http，server，location

生成规则（以下生成规则是`nginx`代码提供的C语言程序）

```c
ngx_springf(etag->value.data, "\"%xT-%xO\"",
           r->headers_out.last_modified_time,
           r->headers_out.content_length_n)
```

![Snipaste_2021-01-23_21-10-35](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-10-35.png)

![Snipaste_2021-01-23_21-10-49](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-10-49.png)





### Nginx决策浏览器过期缓存是否有效

> 内容：`nginx`控制浏览器如何使用缓存

- `headers`模块提供的`expires`指令，告诉浏览器，缓存怎么过期
- `not_modified`模块。决策返回的状态码，决策依据是header响应头和`if_modified_since`指令



![Snipaste_2021-01-23_21-34-31](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-34-31.png)

![Snipaste_2021-01-23_21-34-44](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-34-44.png)

![Snipaste_2021-01-23_21-35-08](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-35-08.png)

![Snipaste_2021-01-23_21-35-18](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-35-18.png)

![Snipaste_2021-01-23_21-35-30](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-35-30.png)

![Snipaste_2021-01-23_21-35-42](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_21-35-42.png)



### 缓存的基本用法

> 内容：
>
> 1. 如何配置`nginx`之上——上游服务器 返回的响应的缓存
> 2. 稍微深入第二部分中`Nginx`进程结构中的`Cache Loader`和`Cache Manager`进程

![Snipaste_2021-01-23_22-15-02](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_22-15-02.png)

![Snipaste_2021-01-23_22-15-12](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_22-15-12.png)

![Snipaste_2021-01-23_22-15-23](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_22-15-23.png)

![Snipaste_2021-01-23_22-15-34](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_22-15-34.png)

![Snipaste_2021-01-23_22-15-49](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_22-15-49.png)

![Snipaste_2021-01-23_22-16-00](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_22-16-00.png)

![Snipaste_2021-01-23_22-16-10](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_22-16-10.png)

![Snipaste_2021-01-23_22-16-19](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-23_22-16-19.png)

### 对客户端请求的缓存处理流程&接收上游响应的缓存处理流程

> 缓存处理流程分为两部分
>
> - `nginx`接收到客户端的请求。判断此次请求是否可以被缓存
> - `nginx`返回响应

**`nginx`接收到客户端的请求**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_12-34-15.png" alt="Snipaste_2021-01-24_12-34-15" style="zoom:80%;" />

![Snipaste_2021-01-24_12-34-40](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_12-34-40.png)



**`nginx`接收到上游服务的响应后如何缓存并向客户端返回响应**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_12-42-56.png" alt="Snipaste_2021-01-24_12-42-56" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_12-43-13.png" alt="Snipaste_2021-01-24_12-43-13" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_12-43-22.png" alt="Snipaste_2021-01-24_12-43-22" style="zoom:80%;" />

![Snipaste_2021-01-24_12-43-47](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_12-43-47.png)



### 如何建请缓存失效时上游服务的压力

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_13-03-21.png" alt="Snipaste_2021-01-24_13-03-21" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_13-03-35.png" alt="Snipaste_2021-01-24_13-03-35" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_13-03-47.png" alt="Snipaste_2021-01-24_13-03-47" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_13-04-10.png" alt="Snipaste_2021-01-24_13-04-10" style="zoom:80%;" />

`proxy_cache_revalidate`指令的例子如下图：

![Snipaste_2021-01-24_13-04-23](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_13-04-23.png)

### 及时清除缓存

> 之前缓存失效都是发生在
>
> - 缓存资源时设定一个有效时间，如果过期，缓存失效
> - 修改源资源时，缓存和源资源不一致，缓存失效
>
> 及时清除缓存是，当请求访问到`nginx`的`location`块时，执行指令，主动立刻清除某个缓存
>
> `nginx`商业版中官方提供了此功能，开源版中需要引入第三方模块

![Snipaste_2021-01-24_13-51-34](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_13-51-34.png)



## uwsgi，fastcgi，scgi指令的对照表

> `http`的反向代理时`nginx`中最为复杂的反向代理
>
> `uwsgi`，`fastcgi`，`scgi`的反向代理比较简单
>
> 问题：`uwsgi`，`fastcgi`，`scgi`是应用层协议吗？如果是，那么上游服务能有`Spring`写吗？

客户端发送来的请求都是`http`协议的请求，`nginx`向上游服务器发送的请求可能不是`http`协议的请求，例如上述几种，或`WebSocket`等

当然，这些协议通常都是建立在`TCP`协议连接上建立的。这些反向代理的使用和http的差不多，它们的指令名也都十分类似，所以很好上手

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_14-24-34.png" alt="Snipaste_2021-01-24_14-24-34" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_14-24-47.png" alt="Snipaste_2021-01-24_14-24-47" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_14-24-59.png" alt="Snipaste_2021-01-24_14-24-59" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_14-25-13.png" alt="Snipaste_2021-01-24_14-25-13" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_14-25-25.png" alt="Snipaste_2021-01-24_14-25-25" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_14-25-39.png" alt="Snipaste_2021-01-24_14-25-39" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_14-26-00.png" alt="Snipaste_2021-01-24_14-26-00" style="zoom:67%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-24_14-26-15.png" alt="Snipaste_2021-01-24_14-26-15" style="zoom:80%;" />



## memacached反向代理的用法

> `memacached`是应用层协议

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-25_12-30-05.png" alt="Snipaste_2021-01-25_12-30-05" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-25_12-30-20.png" alt="Snipaste_2021-01-25_12-30-20" style="zoom:67%;" />



## 搭建websocket反向代理

![Snipaste_2021-01-25_13-05-15](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-25_13-05-15.png)

![Snipaste_2021-01-25_13-06-21](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-25_13-06-21.png)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-25_13-06-31.png" alt="Snipaste_2021-01-25_13-06-31" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-25_13-06-44.png" alt="Snipaste_2021-01-25_13-06-44" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-25_13-07-00.png" alt="Snipaste_2021-01-25_13-07-00" style="zoom:80%;" />

`WebSocket`环境测试

```
浏览器http://www.websocket.org/echo.html
				|
				|
				V
			  nginx
			  	|
			  	|
			  	V
	  	echo.websocket.org
```

![Snipaste_2021-01-25_13-09-06](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-25_13-09-06.png)

![Snipaste_2021-01-25_13-09-16](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-25_13-09-16.png)



> 暂时到此为止。等复习完HTTP（http协议，https协议（TLS套件啥的）），其他TCP/IP协议，操作系统（需要学习操作系统的epoll模型），一致性哈希算法（负载均衡中需要这个算法）



## 用分片提升缓存效率





## openfilecache提升系统性能





## http2协议介绍&搭建http2服务并推送资源





## grpc反向代理





## stream四层反向代理的7个阶段及常用变量





## proxyprotocol协议与realip模块





## 限并发连接、限IP、记日志







# 问题

- Nginx 需要搭建集群吗？
  - 如果需要搭建，那是不是不能用一个域名访问多态 nginx 服务