#还没有复习 

# HTTP 2.0 和 HTTP 1.0 的区别

简单点说，传输速度更快，冗余信息更少，性能更好，功能更多


## HTTP 1.1 的缺点

**需求**

-   请求携带的**数据量变大**
    
-   一个页面上的**请求数量变多**每个页面小于 10 个资源，到每页面 100 多个资源
    
-   请求的**数据类型多样化**。从文本为主的内容，到富媒体（如图片、声音、视频）为主的内容
    
-   用户对**低延迟**的要求越来越苛刻。对页面内容**实时性**高要求的应用越来越多
    

**问题**

-   随着网络**带宽增加**（光纤入户等），**延迟并没有显著下降**
    
    -   浏览器发起的**并发连接有限**
        
    -   HTTP/1.1 要求**同一连接同时只能在完成一个 HTTP 事务**（请求/响应）才能处理下一个事务
        
-   REST 架构下要求 HTTP/1.1 能做到无状态
    
    -   导致每个请求都必须尽可能携带完整的头部（各种可能用到的头部都必须有）。据统计，**多个请求间会携带大量重复的头部**
        
    -   每次请求都要将域名下的所有 Cookie 放到请求头中。有时 **Cookie 的大小会很大**
        
-   HTTP/1.1 **不支持双向传递消息**。只能将协议升级为 Websocket 才能实现双向传递消息
    

**HTTP/1.1 使用的解决方式**

使用各种拼接合并方案将多张小图片或多个 css 文件或多个小 js 文件合并到一次响应中减少请求次数

将资源分布到不同域名下。以便减缓浏览器对同一个域名只能建立有限个连接的限制


## HTTP/2 对 HTTP/1.1 的适配

HTTP2（RFC7540，2015.5）

-   在应用层上修改，**传输层仍用 TCP 协议**
    
-   仍**用 request - response 模型**
    
-   老的 **scheme 不变**，没有 http2://
    
-   使用 http/1.x 的客户端和服务器可以无缝的通过代理方式转接到 http/2 上
    
-   不识别 http/2 的代理服务器可以将请求降级到 http/1.x
    

## HTTP/2 的主要特性

-   传输数据量的大幅减少
    
    -   以二进制方式传输（HTTP/1.1 以 ASC 方式传输）
        
    -   标头压缩
        
-   多路复用及相关功能
    
    -   消息优先级
        
-   服务器消息推送
    
    -   并行推送（HTTP/2 充分利用带宽的扩大，使用长肥管道并行发送请求，并行接收响应。而不是 HTTP/1.1 一样，同一个连接只能串行发送请求并等待响应返回后再发送其他请求）


[在这里](https://http2.akamai.com/demo)可以体验 HTTP/1.1 和 HTTP/2 的响应差别。网站主要通过将大图片被分割成多个图片，使用 HTTP/1.1 和 HTTP/2 请求这些小图片的响应速度反映两者的区别



# TLS / SSL 在 HTTP 2.0 中的应用


> [!todo] TLS 内容待补充



# HTTP 2.0 中的帧，消息和流


## 帧，消息，流


在 HTTP/2 中为支持多路复用，除了 message 的概念外又引入了 stream，而实际承载数据的是 frame

### 帧，消息，流的关系

-   连接 Connection：1个 TCP 连接，包含一个或者多个 Stream
    
-   数据流 Stream：一个双向通讯数据流，包含 1 条或者多条 Message
    
-   消息 Message：对应 HTTP/1 中的一条请求或者响应，包含一条或者多条 Frame
    
-   数据帧 Frame：最小单位，以二进制压缩格式存放 HTTP/1 中的内容
    

![Pasted image 20220912090938](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912090938.png)



> [!warning] 目前 HTTP 1.1 不能很好地支持管线化 / 流水线
> 参考[这里](https://blog.csdn.net/qq_44918090/article/details/120757316)和 MDN。而 HTTP 2.0 采用 Stream 方式支持了管线化，让每个帧都记录自己属于哪个请求或响应。



## 帧和流的关系

-   每一个 Frame 都有一个 Stream ID 表示自己属于哪一个 Stream
    
-   但是 Frame 没有标识位表示自己属于哪一个 Message
    

![Pasted image 20220912091223](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912091223.png)


## 消息的组成

HTTP 2.0 的消息有以下特点

-   每条 Message （请求或响应）中有多个 Frame ，由 1 个 HEADERS（可能含有 0 个或者多个持续帧构成） 及 0 个或者多个 DATA 帧构成
    
-   Message 在帧格式中比较难找到 HEADERSS 帧和 DATA 帧的对应关系，但在应用层就能比较明确地联系起来
    
-   HEADERS 消息同时包含 HTTP/1.1 中的 start line 与 HEADERSs 部分
    
-   取消 HTTP/1.1 中的不定长 Chunk 消息
    

![Pasted image 20220912091352](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912091352.png)


下面一个抓包实例图（Wireshark 抓包抓的 HTTP2 报文，每一条都是 HTTP/2 中一个 Frame）

![Snipaste_2021-02-07_21-45-17](file://D:/image/blog/Snipaste_2021-02-07_21-45-17.png?lastModify=1662943048)

## Frame 传输中无序，接收时组装

-   同一条消息的多个 Frame 在同一个 Stream 中必须有序，但可以不连续
    
    -   比如：必须先传 HEADERSS 帧再传 DATA 帧
        
    -   比如：Stream1 中的多个 Frame 可以不连续传递。在 Stream1 传一部分数据后，传一部分 Stream3 的帧再继续传 Stream1 的帧
        
-   如果一条消息的 Frame 都是连续传递的，那就和 HTTP/1.1 的串行传递一样了
    

![Pasted image 20220912091903](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912091903.png)


## HTTP 2.0 帧格式

![Pasted image 20220912091950](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912091950.png)

## 标准帧头部

9 字节标准帧头部 + Fram Payload（帧负载的不定长的数据）

![Pasted image 20220912092017](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912092017.png)


-   Length：帧的总长度（2^24 位 = 16 MB）
    
-   Type：帧类型
    
-   Flags：根据 Type 的不同，Flags 的含义也不同
    
-   Stream Identifier：Stream ID 流的唯一标识符
    
-   Frame Payload：帧负载的不定长的数据
    

## Stream ID 的作用

-   实现多路复用的关键
    
    -   接收端的实现可据此并发组装消息
        
    -   同一 Stream 内的 frame 必须是有序的（无法并发）
        
    -   SETTINGS_MAX_CONCURRENT_STREAMS 控制着并发 Stream 数


![Pasted image 20220912092134](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912092134.png)


-   由客户端建立的流必须是奇数
	
-   由服务器建立的流必须是偶数
        

![Pasted image 20220912092209](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912092209.png)

        
-   流状态管理的约束性规定
    
    -   新建立的流 ID 必须大于曾经建立过的状态为 opened 或者 reserved 的流 ID
        
    -   在新建立的流上发送帧时，意味着将更小 ID 且为 idle 状态的流置为 closed 状态
        
    -   Stream ID 不能复用，长连接耗尽 ID 应创建新连接（Stream ID 是用 Frame 中 31 bit 表示的，如果 31 bit 全为 1 后还要新建 Stream 就要新建连接）
        
-   应用层流控仅影响数据帧
    
    -   Stream ID 为 0 的流仅用于传输控制帧
        
-   在HTTP/1 升级到 h2c 中，以 ID 为 1 流返回响应，之后流进入half-closed (local) 状态
    

## 不同的帧类型及设置帧的子类型


![Pasted image 20220912092410](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912092410.png)

-   Type：帧类型（每种帧除了标准帧头部外，其他的部分都不一样。HEADERSS，DATA，WINDOW_UPDATE帧最为常见）
    
![Pasted image 20220912092330](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912092330.png)



## 设置帧

**Setting 设置帧格式（type=0x4）**

-   设置帧并不是“协商”，而是发送方向接收方通知其特性、能力
    
-   一个设置帧可同时设置多个对象
    

设置帧通过以下格式设置 K 的 V

![Pasted image 20220912092525](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912092525.png)

-   Identifier：设置对象。比如设置：并发流的大小，最大帧的 size 等
    
-   Value：设置值
    

**设置类型**

-   SETTINGS_HEADERS_TABLE_SIZE (0x1): 通知对端索引表的最大尺寸（单位字节，初始 4096 字节）。比如设置头部压缩格式的设置帧
    
-   SETTINGS_ENABLE_PUSH (0x2): Value设置为 0 时可禁用服务器推送功能，1 表示启用推送功能
    
-   SETTINGS_MAX_CONCURRENT_STREAMS (0x3): 告诉接收端允许的最大并发流数量
    
-   SETTINGS_INITIAL_WINDOW_SIZE (0x4): 声明发送端的窗口大小，用于Stream级别流控，初始值2^16-1 (65,535) 字节
    
-   SETTINGS_MAX_FRAME_SIZE (0x5):设置帧的最大大小，初始值 2^14 (16,384)字节
    
-   SETTINGS_MAX_HEADERS_LIST_SIZE (0x6): 知会对端头部索引表的最大尺寸，单位字节，基于未压缩前的头部
    

设置帧抓包实例图

这是一个空设置帧，因为它没有

![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416110629.png)


这是一个含有多个设置项的设置帧

![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416110644.png)
