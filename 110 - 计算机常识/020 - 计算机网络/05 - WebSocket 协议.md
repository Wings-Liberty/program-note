# Websocket 协议的功能

HTTP/1.1 request-response 模式下，支持单向通信。消息只能由客户端主动发送请求，服务器被动返回响应
    
Websocket 下，**支持双向通信**。客户端和服务器建立 WS 连接后，服务器能主动向客户端发送数据。效率要比客户端轮询 / 定时查询的效果


# WS 和 HTTP 协议的区别及其优缺点


-   只能通过 HTTP/1.1 升级协议的方式建立 Websocket 连接

-   端口复用，兼容 HTTP 协议。ws 协议默认使用 80 端口，wss （ws +SSL）协议默认使用 443。80 端口和 443 端口也是 HTTP 和 HTTPS 的默认端口

-   优点：双向通信；缺点：可扩展性差（稍后再说）

-   会话管理方式：ws 会话由客户端和服务器双向控制，关闭 ws 会话需要两边都发起关闭会话

-   长连接维护方式：客户端和服务器通过 ping/pong 心跳保持长连接

-   支持扩展：比如使用插件让 ws 支持压缩数据等


**缺点**


**HTTP 请求数量激增时，可通过搭建集群**，将请求分摊到集群中的节点上。**能这样做的原因是：HTTP 协议是无状态的协议**。在不用 Session 的前提下，一个请求可以让集群中任意一个节点处理

WS 协议是基于 TCP 连接的协议，**客户端需要和单台服务器建立 WS 连接后才能双向通信**。服务器使用集群后，客户端建立的 WS 连接意味着同一个 WS 连接下，客户端只能向一台服务器发送数据

为了实现 WS 连接下客户端也能和集群中任意节点进行双向通信，需要设计复杂的系统（牺牲了简单性），比如下图

![Pasted image 20220911235329](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911235329.png)


上图中处理 WS 的集群中，WS 请求发送到消息分发系统前的门面服务器时，门面服务器先将消息转换为简单的格式后发送给消息分发系统，消息分发系统将消息传递给源服务器进行负载均衡

消息分发系统可以是：Kafka，RabbitMQ，RocketMQ，Redis 等


# WS 的设计哲学

在 Web 约束下暴露 TCP 给上层。通信双方都尽可能直接面向 TCP 层，这样就能实现任意一方向另一方发数据

![Pasted image 20220911235622](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911235622.png)




# WS 的传输格式


## WS 的数据帧格式介绍

![Pasted image 20220911235745](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911235745.png)

PS：每行 32 位 = 4 字节，前两个字节的数据必须有
    
-   RSV1/RSV2/RSV3：默认为 0，仅当使用 extension 扩展时，由扩展决定其值
-   opcode 决定了帧的类型。比如，帧是 ping/pong 心跳帧、数据帧（文本帧或二进帧）、关闭帧（关闭 WS 连接时需要发送的帧）
    -   持续帧
        -   0：继续前一帧（表示此帧的类型和前一个 WS 协议的帧的类型一样）
    -   非控制帧
        -   1：文本帧（UTF8）
        -   2：二进制帧
        -   3-7：为非控制帧保留
    -   控制帧
        -   8：关闭帧
        -   9：心跳帧 ping
        -   A：心跳帧 pong
        -   B-F：为控制帧保留
-   FIN 位：用于控制一条 message 的结束，同时隐含下一个 frame 是新 message 的开始。如当前帧的类型不是控制帧，且 FIN 位为 1 ，表示此 message 的结束
    


## WS 的 URL 格式

ws-URI = `ws://host[:port]path[query_params]`

默认 port 端口 80


wss-URI = `wss://host[:port]path[query_params]`

默认 port 端口 443


此外客户端还要发送如下信息

必选

-   握手随机数：Sec-WebSocket-Key


可选

-   选择子协议： Sec-WebSocket-Protocol
-   扩展协议： Sec-WebSocket-Extensions
-   CORS 跨域：Origin
    

## HTTP 升级到的 Websocket 的过程


![Pasted image 20220912000237](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912000237.png)

-   红色头部：必选头部
    
-   绿色头部：Sec-WebSocket-Key 头部是携带随机数的头部，随机数被 SHA1 和 BASE 64 编码过
    
-   蓝色头部：跨域相关头部
    
-   黑色头部：可选头部，比如 Sec-WebSocket-Extensions
    

## 如何证明握手被服务器接受？预防意外


WS 握手升级请求会携带 Sec-WebSocket-Key 头部，如果响应中的 Sec-WebSocket-Accept 能返回合适的证明值就说明握手被服务器接受并建立了 WS 连接


请求中 Sec-WebSocket-Key 携带随机数

-   例如 `Sec-WebSocket-Key: A1EEou7Nnq6+BBZoAZqWlg==`
    

响应中 Sec-WebSocket-Accept 携带随机数的证明值

-   证明值的构造规则：BASE64(SHA1(<font color='red'>Sec-WebSocket-Key</font><font color='green'>GUID</font>)) （随机数+指定字符串后 先进行 SHA1 编码再进行 BASE 64 编码）
    
    -   GUID（是 RFC4122 规定的魔法值）：258EAFA5-E914-47DA-95CA-C5AB0DC85B11
        
    -   拼接值：A1EEou7Nnq6+BBZoAZqWlg\=\=258EAFA5-E914-47DA-95CA-C5AB0DC85B11
        
    -   SHA1 值：713f15ece2218612fcadb1598281a35380d1790f
        
    -   BASE 64 值：cT8V7OIhhhL8rbFZgoGjU4DReQ8=
        
-   最终头部应该是：`Sec-WebSocket-Accept: cT8V7OIhhhL8rbFZgoGjU4DReQ8=`


## WS 的消息和数据帧

WS 在逻辑角度来看，通信双方发送的是一条条的 Message 消息

在物理角度来看，一条消息会被拆分成若干个 Frame 数据帧帧进行发送和接收。一条消息的组成帧应该属于同一类型


- 数据帧在组成上分为：持续帧，非持续帧，控制帧

- 数据帧在数据类型上分为：文本帧，二进制帧


这里关注的是数据帧中 指定数据长度的 len 和帧中的数据 data


![Pasted image 20220912001430](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912001430.png)

![Pasted image 20220912001441](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912001441.png)


使用 WS 连接发送消息时

-   确保 WebSocket 会话处于 OPEN 状态
    
-   以帧来承载消息，一条消息可以拆分多个数据帧
    
-   客户端发送的帧必须基于掩码编码
    
-   一旦发送或者接收到关闭帧，连接处于 CLOSING 状态
    
-   一旦发送了关闭帧，且接收到关闭帧，连接处于 CLOSED 状态
    
-   TCP 连接关闭后，WebSocket 连接才完全被关闭
    

## 掩码：防御代理缓存污染攻击


针对代理服务器的缓存污染攻击

![Pasted image 20220912001648](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912001648.png)



> [!help] 没看懂


## Websocket 保持会话心跳和关闭会话

**心跳帧**（可以插在数据帧中传输 ）

-   ping 帧 （opcode=9，可以含有数据）
    
-   pong 帧 （opcode=A，必须与 ping 帧数据相同）
    

**关闭会话**流程

-   控制帧中的关闭帧：在 TCP 连接之上的双向关闭
    
    -   发送关闭帧后，不能再发送任何数据
        
    -   接收到关闭帧后，不再接收任何到达的数据
        
    -   opcode=8，可以含有数据，但仅用于解释关闭会话的原因（前 2 字节为无符号整型，遵循 mask 掩码规则）
        
-   TCP 连接意外中断（因为 WS 连 是建立在 TCP 连接上的，如果 TCP 连接断掉，WS连接也就断了）
    

**关闭帧的错误码**

![Pasted image 20220912001933](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912001933.png)




> [!NOTE] 参考博客
> [WebSocket 详解系列](http://www.52im.net/thread-331-1-1.html)
