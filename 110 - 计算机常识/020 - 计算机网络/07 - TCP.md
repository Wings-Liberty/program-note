
> TCP 是传输层协议
> 
> 以 TCP 链接的生命周期为主线介绍 TCP
> 
> -   如何管理 TCP 连接
>     
>     -   如何建立 TCP 连接
>         
>     -   如何用 TCP 连接传递数据（传递数据）
>         
>     -   拥塞控制是怎样进行的（拥塞控制）
>         
>     -   如何关闭 TCP 连接（关闭连接）
>         
> -   TCP 的其他特性（keepalive，校验，外带数据，多路复用，四层负载均衡）
>     
> 
> 最后在使用层面对 TCP 进行一个总结

TCP 将消息拆分成多个 TCP segment 进行发送

# TCPv4 协议分层后的互联网世界

![Snipaste_2021-02-19_12-56-32](file://D:/image/blog/Snipaste_2021-02-19_12-56-32.png?lastModify=1662943048)

-   网络层的作用是：协助报文跨越不同的网络寻找目标主机
    
-   传输层的作用是：报文找到主机后如何向主机发送数据
    

### TCP/IP 的设计理念



TCP 的主要目的和设计原则是前 3 条

 -   能容错。例如滑动窗口，拥塞控制都是容错机制

-   要求能支持不同类型的设备。比如：华为，中兴，小米等不同类型的手机和电脑

-   能连接不同类型的网络。比如：wifi，海底光缆


## TCP 协议的分层

TCP：面向连接的、可靠的、基于字节流的传输层通信协议

-   面向连接：一台主机和另一台主机进行 1 对 1 的通信
    
-   基于字节流：以字节为单位，不限制数据长度/大小
    
-   可靠的：TCP 由队列的概念。属于同一个数据流的字节需要按队列排队接收，**只有当接收前一个字节后才会接收下一个字节**，如果先接收到下一个字节而没接收到前一个字节，就会一直等待前一个字节
    

![Pasted image 20220912093044](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912093044.png)


协议的分层结构使得 TCP 只要在报文中添加一个 TCP 头部即可实现其功能

PS：网络设备，主机，物理链路都是不可靠的网络传输，TCP 承担了保证网络传输安全的责任

## TCP 协议的特点

-   **用 IP 协议**，解决网络通讯可依赖问题
    
-   **点对点**（不能广播、多播），**面向连接**
    
-   双向传递（全双工） HTTP/1.1 是半双工的，而 Websocket 是全双工的因为TCP将全双工功能暴露给了WS
    
-   **字节流**：打包成报文段、保证有序接收、重复报文自动丢弃
    
-   缺点：**不维护应用报文的边界**（对比 HTTP、GRPC）
    
-   优点：不强制要求应用必须离散的创建数据块，**不限制数据块大小**
    
-   流量缓冲：**解决速度不匹配问题**（通信双方所在的设备的性能是不同的 ，TCP 需要解决双方设备性能不同的问题）
    
-   **可靠传输**服务（保证可达，丢包时通过重发进而增加时延实现可靠性）
    
-   **拥塞控制**


# TCP 报文格式

TCP Segment 报文段格式

![Pasted image 20220912093426](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912093426.png)

-   Sequence Number 和 Acknowledgment Number 都是 TCP Segment 报文段的唯一标识符
    
-   Acknowledgment Number 还有标识报文可达性的作用。通常 SYN 报文中此值置 0 ，ACK 报文中此值才有意义
    
-   关于 Offset ，TCP Flags ，Window 在后面再说
    
-   TCP Segment 报文段除固定的 20 字节外还有不定长的 Options，下图是 Option 的类型和一个 TCP 报文中 Options 的示意图
    
    -   每个 Option 都有一个 Length ，其值为整个 Option 的长度（以字节为单位），长度将 Length 值的长度也包括在内

![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416110731.png)



# tcpdump 命令分析网络报文

> Windows 下使用 Wireshark 进行抓包。在学习使用 Wireshark 进行远程抓包前先学习用 Linux 的 tcpdump 直接在应用服务器上进行本地抓包
> 
> tcpdump 使用的捕获过滤器语法和 Wireshark 使用的捕获过滤器语法相同

tcpdump 的使用主要包括

-   捕获过滤器语法
    
-   列举或选择网卡设备
    
-   捕获停止条件（限定只抓取多少条报文）
    
-   文件操作（将报文保存到文件中或读取文件中的报文；拆分报文文件，防止文件过大；设置报文中的时间戳格式）
    
-   分析报文详情（显示报文中的详细信息；以 16 进制/ Ascii 方式 显示报文；显示/不显示数据链路层/网络层的报文）
    

PS：报文文件可以被 Wireshark 读取

tcpdump 的选项和参数详情可见课件或 Utools 的 Linux 命令手册

**获取网卡信息**

```bash
$ tcpdump -D
1.eth0 [Up, Running]  
2.any (Pseudo-device that captures on all interfaces) [Up, Running]  
3.lo [Up, Running, Loopback]  
4.docker0 [Up]  
5.nflog (Linux netfilter log (NFLOG) interface)  
6.nfqueue (Linux netfilter queue (NFQUEUE) interface)  
7.usbmon1 (USB bus number 1)
```

eth0 是对外服务的网卡，也是默认的网卡；lo 是环回地址使用的网卡

```bash
$ tcpdump host www.baidu.com
```

```bash
$ tcpdump -V c # 文件c的内容为：第一行是a，第二行是b。a和b是两个文件的文件名，文件内容为某次tcpdump的抓包日志
```

```
捕获条件和停止条件  
    -D 列举所有网卡设备  
    -i 选择网卡设备，后跟网卡设备名  
    -c 抓取多少条报文  
    --time-stamp-precision 指定捕获时的时间精度，默认毫秒 micro，可选纳秒 nano  
    -s 指定每条报文的最大字节数，默认 262144 字节  

文件操作  
    -w 输出结果至文件（可被Wireshark读取分析）  
    -C 限制输入文件的大小，超出后以后缀加 1 等数字的形式递增。注意单位是 1,000,000 字节  
    -W 指定输出文件的最大数量，到达后会重新覆写第 1 个文件  
    -G 指定每隔N秒就重新输出至新文件，注意-w 参数应基于 strftime 参数指定文件名  
    -r 读取一个抓包文件  
    -V 将待读取的多个文件名写入一个文件中，通过读取该文件同时 读取多个文件  
输出时间戳格式  
    -t 不显示时间戳  
    -tt 自 1970年 1 月 1 日 0 点至今的秒数  
    -ttt 显示邻近两行报文间经过的秒数  
    -tttt 带日期的完整时间  
    -ttttt 自第一个抓取的报文起经历的秒数  
分析信息详情  
    -e 显示数据链路层头部  
    -q 不显示传输层信息  
    -v 显示网络层头部更多的信息，如 TTL、id 等  
    -n 显示 IP 地址、数字端口代替 hostname 等  
    -S TCP 信息以绝对序列号替代相对序列号  
    -A 以 ASCII 方式显示报文内容，适用 HTTP 分析  
    -x 以 16 进制方式显示报文内容，不显示数据链路层  
    -xx 以 16 进制方式显示报文内容，显示数据链路层  
    -X 同时以 16 进制及 ACII 方式显示报文内容，不显示数据链路层  
    -XX 同时以 16 进制及 ACII 方式显示报文内容，显示数据链路层
```

# TCP 的三次握手

## 三次握手建立链接

> 这里讲 3 次握手建立连接的
> 
> -   目的
>     
> -   SYN 和 ACK 报文的格式，SYN 和 ACK 如何同步序列号
>     

**握手的目的**

-   同步 Sequence 序列号
    
    -   初始序列号 ISN（Initial Sequence Number）
        
-   交换 TCP 通讯参数
    
    -   如 MSS、窗口比例因子、选择性确认、指定校验和算法
        

**SYN 报文和 ACK 报文的格式**

![Pasted image 20220912094141](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912094141.png)

三次握手动图

![114bd9b53c554319ae0db15b57b472b8](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/114bd9b53c554319ae0db15b57b472b8.gif)

三次握手（1）：SYN 的报文

-   Flags 中的 SYN 位置 1
    
-   Seq 中填上 Client 的 ISN
    

三次握手（2）：SYN/ACK 报文

-   将 Flags 中的 SYN 位和 ACK 位置 1
    
-   Seq 中填上 Server 的 ISN
    
-   Ack 中填上 Client 发送的 SYN 报文中的 ISN+1 的值
    

ACK 的报文

-   将 Flags 中的 ACK 位置 1
    
-   Ack 中填上 Server 发送的 SYN/ACK 报文中的 ISN+1 的值
    

下图是 tcpdump 在抓包 3 次握手的 TCP 报文示例图

![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416110832.png)


问题

-   为什么 Client 和 Server 的 ISN 是不一样的？
    
-   为什么 Client 和 Server 的 INS 不是从 0 开始递增的？
    

答：网络通信中存在复制重发，延迟，丢失。为防止造成通信双方互相影响，初始的 ISN 采用随机值

## 三次握手过程中的状态变迁

TCP 连接有 5 种状态

-   CLOSED（起始状态都是关闭状态）
    
-   LISTEN（监听状态。服务器创建 socket 后监听端口便会进入此状态等待收发报文）
    
-   SYN-SENT（发送 SYN 时进入此状态但时间极短，很难观察到）
    
-   SYN-RECEIVED（服务器接收到 SYN 后进入此状态，直到接收到 ACK 后 ESTABLISHED）
    
-   ESTABLISHED（成功建立连接后进入的状态）
    

![Pasted image 20220912094254](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912094254.png)


可以使用 netstat 查看本机 tcp 连接的状态

```bash
$ netstat -anp | grep tcp  # -a 查看所有套接字 -n 将域名或 hostname 替换为ip地址 -p 显示连接使用的 Socket 的程序识别码或程序名
```

**TCB**： Transmission Control Block，保存连接使用的源端口、目的端口、目的 ip、序号、 应答序号、对方窗口大小、己方窗口大小、tcp 状态、tcp 输入/输出队列、应用层输出队列、tcp 的重传有关变量等。是服务器通信时保存对端信息的数据结构

下面讲：在建立好 TCP 连接上如何可靠地传输数据

## 三次握手中的性能优化和安全问题

> -   调整超时时间和缓冲队列
>     
> -   Fast Open 降低时延
>     
> -   如何应对 SYN 攻击
>     
>     -   调整 Linux 内核参数，设置新的拒接策略
>         
>     -   tcp_syncookies
>         
> -   TCP_DEFER_ACCEPT
>     

## 操作系统内核中三次握手的流程

![Pasted image 20220912094413](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912094413.png)

SYN 队列 和 ACCEPT 队列的长度体现了服务器能同时处理的连接数和请求数

### 超时时间和缓冲队列参数设置

-   应用层 connect 超时时间调整
    
-   操作系统内核限制调整（ Linux 的内核参数可通过 /etc/sysctl.conf 文件进行修改）
    
    -   服务器端 SYN_RCV 状态
        
        -   net.ipv4.tcp_max_syn_backlog：SYN_RCVD 状态连接的最大个数
            
        -   net.ipv4.tcp_synack_retries：被动建立连接时，发SYN/ACK的重试次数（服务器收不到 ACK 后会尝试重发 SYN/ACK ）
            
    -   客户端 SYN_SENT 状态
        
        -   net.ipv4.tcp_syn_retries = 6 主动建立连接时，发 SYN 的重试次数
            
        -   net.ipv4.ip_local_port_range = 32768 60999 建立连接时的本地端口可用范围
            
    -   ACCEPT队列设置（可通过 backlog 进行设置）
        

### Fast Open 降低时延

Fast Open ：使用缓存方式减少同一个客户端和服务器进行非首次建立 TCP 连接的 TCP 握手次数

Client 第 1 次和 Server 建立 TCP 连接时执行 3 次握手，Client 第 2 次建立连接时还要进行 3 次握手

开启 Fast Open 后。Client 首次和 Server 建立 TCP 连接，Server 返回 Cookie ，其中保存了建立连接所协商的内容。当下次建立连接时，客户端直接复用 Cookie 中的设置（即默认上次协商的内容这次还能使用）。Client 发送应用层报文时直接把 Cookie 中的内容一并携带过去。即省略 3 次握手，直接发送应用层协议报文

net.ipv4.tcp_fastopen：系统开启 TFO 功能

-   0：关闭
    
-   1：作为客户端时可以使用 TFO
    
-   2：作为服务器时可以使用 TFO
    
-   3：无论作为客户端还是服务器，都可以使用 TFO
    

### 应对 SYN 攻击的手段

SYN 攻击：攻击者短时间伪造不同 IP 地址的 SYN 报文，快速占满 backlog 队列，使服务器不能为正常用户服务


参考[如何应对泛洪/拒绝服务攻击/DDos攻击](https://www.cnblogs.com/fly-wlz/p/13913689.html)


# TCP 协议上的数据传输

虽然 **TCP 不限制数据流的长度**，但是 TCP 层之下的**网络层和数据链路层发送报文时所使用的长度是有限的**，所以 TCP 会将从应用层传来的任意长度数据流切分成若干报文段

这节讲拆分数据流的依据是什么，以及如何拆分报文段

**数据流分段的依据**

-   MSS：防止 IP 层分段（如果 TCP 不对数据进行分段，那么分段的工作会交给 IP 层，而 IP 协议的分段效率低下）
    
-   流控：接收端的能力
    

MSS：Max Segment Size

-   定义：仅指 TCP 承载数据，不包含 TCP 头部的大小
    
-   MSS 选择目的
    
    -   尽量每个 Segment 报文段携带更多的数据，以减少头部空间占用比率
        
    -   防止 Segment 被某个设备的 IP 层基于 MTU 拆分（ MTU 会在 IP 协议中说）
        
-   默认 MSS：536 字节（默认 MTU576 字节，20 字节 IP 头部，20 字节 TCP 头部）
    
-   握手阶段协商 MSS
    
-   MSS 分类
    
    -   发送方最大报文段 SMSS：SENDER MAXIMUM SEGMENT SIZE
        
    -   接收方最大报文段 RMSS：RECEIVER MAXIMUM SEGMENT SIZE
        

MMS 是在 TCP 3 次握手的报文中 Options 中设置的。Options 是 [TCP 报文格式](#TCP%20%E6%8A%A5%E6%96%87%E6%A0%BC%E5%BC%8F)中的内容

Client 发送 SYN 报文中携带的 Options 会指定推荐的 MSS 值，Server 发送 SYN/ACK 的 Options 中返回实际使用的 MSS 值，比如

Client 建议 MSS 的值为 1460

![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416110940.png)

Server 决定实际使用的 MSS 的值为 1400

![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416110923.png)


## 重传与确认

发送方发送的报文可能会在发送途中丢失，因此发送方需要有重发机制和确认接收端接收到报文的机制

> -   发送端发送数据后，如何确认接收端是否接收到？
>     
> -   如果发送端觉得接收端没有接收到，发送端如何重发数据？
>     
> -   重传与确认是怎么演化为滑动窗口？
>     
> -   演化为滑动窗口后，TCP 报文中的序列号和确认序列号是什么？
>     
> -   序列号的设计理念是什么？
>     

### PAR 重发和确认机制

PAR：Positive Acknowledgment with Retransmission

设计思路：**发送报文后，启动定时器**。接收端接收到报文后返回 ACK 通知发送端报文已经收到了。如果超出定时器指定的时间范围（超时）后，**发送端还未接收到 ACK ，发送端会重发报文**。**只有发送端接收到 ACK 后才会发送下一个报文**

缺点：串行发送报文导致效率低下

![Pasted image 20220912095306](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912095306.png)

### 提升并发能力的 PAR 改进版

相对于 PAR 的改进之处：

-   发送方无需收到 ACK 就能发送下一个报文
    
-   为确认发送端收到的 ACK 是对哪条报文的确认响应，Message 和 Ack 报文都会通过序列号确认关系。使用的序列号是 Seq Number 和 Ack Number
    
-   并发传送导致接收方接收数据的压力增大，为确保接收方是否有能力再接收报文，ACK 报文会指出接收端还能接收多少字节（TCP 报文格式中，用 Window 表示接收端还能接收多少字节）
    

![Pasted image 20220912095322](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912095322.png)


这个解决方案并不完美，逐渐优化方案，直到演化出滑动窗口，更好地解决了接收端接收大量报文的问题

### Sequence 序列号/Ack 序列号

-   设计目的：解决应用层字节流的可靠发送
    
    -   跟踪应用层的发送端数据是否送达
        
    -   确定接收端有序的接收到字节流
        
-   序列号的值针对的是字节而不是报文（序列号的值不是每发一个报文就 +1）
    
    下一个序列号 = 当前序列号 + TCP Segment Len 如下图：2030 = 644 + 1386
    
    ![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416111009.png)
    
-   序列号占用的位数有限。当发送的字节数超过序列号能表示的最大值后，序列号从 ISN 开始复用
    
    -   缺点：在长肥网络下，存在 PAWS（ Protect Against Wrapped Sequence numbers ）问题
        
    -   PAWS 问题：某次发送报文时使用序列号 a ，但是发送失败。由于序列号复用，之后某次发送报文再次用到序列号 a ，接收端可能会误以为这次的报文是上次报文的重发
        
    -   PAWS 问题的解决方案：为每个报文添加时间戳。TCP timestamp 是 TCP 报文中的可选 Option
        

### RTO 重传定时器的计算

改进版的 PAR 使用定时器设置超时时间，超时时间的设定比较重要

使用 TCB 计算 RTT 的时间

如果 RTT 时间在 RTO 范围内，则认定报文发送成功。


## 滑动窗口：发送窗口和接收窗口

滑动窗口源于改进版的 PAR，但是要比它复杂很多，功能也很强大


### 发送窗口

举例：滑动窗口：发送窗口快照

1.  已发送并收到 Ack 确认的数据：1-31 字节
    
2.  已发送未收到 Ack 确认的数据：32-45 字节
    
3.  未发送但总大小在接收方处理范围内：46-51 字节
    
4.  未发送但总大小超出接收方处理范围：52-字节
    

![Pasted image 20220912095530](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912095530.png)


![Pasted image 20220912095804](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912095804.png)



PS：上图中，黑框代表窗口，框内的字节代表窗口内的数据

发送窗口中有 3 个重要的变量

-   SND.WND 窗口能容纳的字节数
    
-   SND.UNA 指向发向对端但还未收到 ACK 的数据的第一个字节的指针
    
-   SND.NXT 指向将要发送但还未发送的数据的第一个字节的指针
    

### 接收窗口

![Pasted image 20220912095907](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912095907.png)

接收窗口中有 2 个重要的变量

-   RCV.WND 接收窗口能容纳的字节数
    
-   RCV.NXT 指向下一个将要接收数据的地址的指针
    

### 滑动窗口与流量控制

实际演示例子展示发送窗口和接收窗口

为简化过程，设定如下限制：MSS 不产生影响，窗口不变

场景：客户端发送一个请求，服务器分别返回响应头部和包体

1.  客户端发送 140 字节
    
2.  服务器接收 140 字节数据，发送 80 字节响应头部和 ACK （稍后发送 280 字节的包体）
    
3.  客户端接收 80 字节响应头部和 ACK，并返回 ACK
    
4.  服务器发送包体的前 140 字节
    
5.  客户端接收前 140 字节的包体并返回 ACK
    
6.  服务器接收到第 2 步发送的头部的 ACK
    
7.  服务器接收到第 4 步发送的一部分包体的 ACK
    
8.  服务器发送包体的后 160 字节数据
    
9.  客户端接收包体后 160 字节的数据，并返回 ACK
    
10.  服务器接收到到第 8 步的 ACK
    

![Pasted image 20220912100047](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912100047.png)


左图为客户端消息的发送，右图为服务器消息的发送

![Pasted image 20220912100114](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912100114.png)


![Pasted image 20220912100135](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912100135.png)


### 操作系统缓冲区与滑动窗口的关系

-   滑动窗口存放在操作系统的缓冲区中
    
-   操作系统的缓冲区会被操作系统调整
    
-   应用进程无法读取数据时也会对缓冲区造成影响，比如：窗口收缩
    

操作系统的缓冲区时如何影响发送窗口和接收窗口的

Server 的窗口接收到数据后，应用层程序如果没有及时读取缓存将会导致窗口收缩

窗口收缩后，ACK 报文中使用 Window 字段指明接收端当前的窗口的大小

问题：为什么应用程序不及时读取缓存会导致窗口收缩，而不是窗口的可用大小减小？

**收缩窗口导致的丢包**

窗口收缩后还未来得及通知对端，对端就发来新的数据

由于对端发送数据时还不知道接收端窗口收缩，新的数据长度可能大于当前窗口。窗口无法容纳这些数据，操作系统将直接丢弃包中数据

操作系统为解决这种问题，采用如下手段解决（这些手段保证了上述方式的丢包情况将不会发生）

-   先收缩窗口，再减少缓存
    
-   接收端窗口关闭后，发送端定时发送探测包探测接收端窗口是否开启
    

**窗口设置为多少比较合适**

设置为 bps * RTT

比如带宽（bps）为100M，时延的平均值在 1s 左右，窗口设置为 100M

缓存不仅要为窗口分配空间，还要为应用缓存分配空间

 net.ipv4.tcp_adv_win_scale = 1  # 这是应用缓存和窗口空间的比例

应用缓存 = buffer / （2^tcp_adv_win_scale）

**Linux中对TCP缓冲区的调整方式**

-   net.ipv4.tcp_rmem = 4096 87380 6291456
    
    -   读缓存最小值、默认值、最大值，单位字节，覆盖 net.core.rmem_max
        
-   net.ipv4.tcp_wmem = 4096 16384 4194304
    
    -   写缓存最小值、默认值、最大值，单位字节，覆盖net.core.wmem_max
        
-   net.ipv4.tcp_mem = 1541646 2055528 3083292
    
    -   系统无内存压力、启动压力模式阀值、最大值，单位为页的数量
        
-   net.ipv4.tcp_moderate_rcvbuf = 1
    
    -   开启自动调整缓存模式
        

## 减少小报文提高网络效率

这里说的小报文既有发送端主动发送多个小报文，也有因对端滑动窗口空间不足导致发送端不得不将数据拆分成多个小报文，还有为每个非 ACK 报文回复一个 ACK（单纯的 ACK 报文作用单一，最少 20 字节）



### TCP delayed acknowledgment 延迟确认

-   当有响应数据要发送时，ack 会随着响应数据立即发送给对方
    
-   如果没有响应数据，ack 的发 送将会有一个延迟，以等待看是否有响应数据可以一起发送（等待时间和服务器的时钟周期有关）
    
     cat /boot/config-`uname -r` | grep '^CONFIG_HZ=' # 可用此命令查看
    
-   如果在等待发送 ack 期间，对方的第二个数据段又到达了，这时要立即发送 ack
    


## 拥塞控制

> -   什么是拥塞，拥塞控制是什么
>     
> -   慢启动
>     
> -   拥塞避免
>     
> -   快速重传
>     
> -   快速恢复
>     

由于 TCP 不限制传输的字节流的长度。所以当大量 TCP 连接发送大量字节流时会造成网络拥塞

![Pasted image 20220912101130](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912101130.png)


-   拥塞窗口cwnd（congestion window）
    
    -   通告窗口 rwnd（receiver‘s advertised window）
        
    -   发送窗口 swnd = min(cwnd，rwnd)
        
-   每收到一个 ACK，cwnd 扩充一倍


缺点：cwnd 扩充一倍后，传输的数据量激增，可能出现大量丢包

慢启动如果出现问题会导致丢包数量很大，拥塞避免能解决这个问题

![Pasted image 20220912101156](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912101156.png)


    
## SACK 与选择性重传算法

TCP 的序列号采用累计确认

快速重传中，接收端返回的 ACK 会携带它所期待的缺口 ACK 序列号。比如接收端期望 a 报文段

在重复接收 a 报文段缺失的消息后，发送端知道了 a 报文段传送失败。到那时发送端并不知道 a 后的 b，c 报文段是否发送成功

### 选择性重传算法

当发送端重发报文段时有两种选择

-   只重发 a（乐观）
    
-   重发 a，b，c（悲观。浪费带宽，大量丢包时效率低下）
    

### SACK

SACK：TCP Selective Acknowledgment

接收端返回 ACK 时，除携带希望发送端发送的下一个报文段序号外还携带接收端已经接收到了哪些报文段。这样发送端就不会重复发送成功接收到的报文段


下面是一次丢包和 SACK 的示意图

-   ACK Number 表示希望对端下次发送的报文序列号
    
-   SACK 的 left edge 和 right edge 是左闭右开的区间。表示已经收到了哪些报文段，让发送端不要再重复发这些了
    
-   在 Wireshark 中第 3 个报文是 TCP Previous segment not captured。是发送端发送报文时，Wireshark 发现发送端上次发送的报文没被抓到
    
-   在 Wireshark 中第 5,6 个报文是 TCP Dup ACK 。是接收端发送的 SACK 报文
    

![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416111052.png)



SACK 也可以用于拥塞控制的快速重传





## TCP 的 4 次握手关闭连接和状态变迁

关闭连接时使用 4 次握手的目的：防止数据丢失；通知应用层优雅关闭连接（优雅关闭 Socket）

-   FIN 包：结束包
    
-   ACK 包：确认包
    

### 正常关闭连接

![0c17d72769aa4bd2b95bbaee1749b39d](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/0c17d72769aa4bd2b95bbaee1749b39d.gif)



![Pasted image 20220912102059](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912102059.png)

1.  Client 发送 FIN 报文，进入 FIN-WAIT-1 状态
    
2.  Server 接收到 FIN 包后告诉应用层准备关闭连接并进入 CLOSE-WAIT 状态，返回 ACK 报文。如果应用程序一直不对 FIN 包做处理，TCP 连接将一直处于 CLOSE-WAIT 状态
    
3.  Client 接收到 ACK 报文后进入 FIN-WAIT-2 状态
    
4.  Srever 的应用层程序调用`socket.close()`后发送 FIN 报文，进入 LAST-ACK 状态
    
5.  Client 接收到 FIN 报文后返回 ACK 报文并进入 TIME-WAIT ，并开始计时，等待 2 个 MSL 后关闭连接并进入 CLOSED 状态
    
6.  Server 收到 ACK 报文后关闭连接
    

PS：MSL 是一个报文在网络中最大的存活时间

上述握手都有支持重发和确认流程

### 通信双方同时主动关闭连接

双方会进入一个 CLOSING 中间状态

![Pasted image 20220912102113](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912102113.png)


### TIME-WAIT 优化

TIME-WAIT 最长可达两个 MSL，对于处理并发的服务器来说需要减少处于 TIME-WAIT 状态的端口。所以 TIME-WAIT 不能设置太长/大

问：为什么 TIME-WAIT 要设置为两个 MSL，而不能在短些

答：如下解释。如果 TIME-WAIT 很短或不存在会发生以下严重错误

1.  第一个 TCP 连接中有个报文发送出现延迟
    
2.  连接关闭后应用程序又**立刻复用**端口建立了 TCP 连接
    
3.  之前的报文由于延迟，接收端现在才收到报文
    

但是这个迟到的报文是第一个连接的报文，不是第二个连接的。这导致了两次连接上的报文混合，出现错乱。所以 TIME-WAIT 是有保护作用的


-   MSL(Maximum Segment Lifetime)
    
    -   报文最大生存时间
        
-   维持 2MSL 时长的 TIME-WAIT 状态
    
    -   原因：保证至少一次报文的往返时间内端口是不可复用
        

**Linux 下能对 TIME-WAIT 的优化操作**

Linux 对 TCP 层的优化都是通过配置文件实现的

-   net.ipv4.tcp_tw_reuse = 1
    
    -   开启后，作为客户端时新连接可以使用仍然处于 TIME-WAIT 状态的端口
        
    -   由于 timestamp 的存在，操作系统可以拒绝迟到的报文。需配置 net.ipv4.tcp_timestamps = 1
        
    -   此选项危险性小
        
-   net.ipv4.tcp_tw_recycle = 0
    
    -   开启后，同时作为客户端和服务器都可以使用 TIME-WAIT 状态的端口
        
    -   不安全，无法避免报文延迟、重复等给新连接造成混乱。所以不建议开启此功能
        
-   net.ipv4.tcp_max_tw_buckets = 262144
    
    -   time_wait 状态连接的最大数量
        
    -   超出后直接关闭连接
        
-   使用 RST 复位报文关闭连接，而不使用 4 次握手关闭连接
    
    -   使用场景：进程突然关掉或其他严重问题
        

# Keepalive

这里只讲 Keep-Alive 功能在 Linux 内核中的参数说明和 keepalive 断开连接的依据

-   发送心跳周期
    
    -   net.ipv4.tcp_keepalive_time = 7200 # 单位为秒
        
-   探测包发送间隔
    
    -   net.ipv4.tcp_keepalive_intvl = 75
        
-   探测包重试次数
    
    -   net.ipv4.tcp_keepalive_probes = 9
        

如果 TCP 连接上 keepalive_time 内没有发送任何数据就启动探测，根据探测结果决定是否关闭连接

发送端将发送多个探测包，如果对端能返回应答，这个连接将再等待一个 keepalive_time

如果没有应答，发送端将重试。超过重发次数后还没收到响应就断开连接

# PUSH 报文

比如发送端发送 10MB 数据，数据被分成多个Segment，最后一个报文会将 PSH 位设为 1

含义是数据已发送完毕，通知操作系统尽快读取缓冲区的数据，因为已经接收到了一个完整的数据



# 校验和

TCP 报文中提供校验和，它会校验 TCP Segment 中的 Data，Header，甚至会校验 IP 层的报文。这违反了分层原则



# 带外数据

也叫带外数据，但通常不叫它带外数据。**功能**：用于紧急处理数据

使用场景：

-   比如在 Linux 中执行一些远程通信的指令，执行时用户输入 Ctrl +c，就会发送USR，对端优先执行某些操作（停止远程通信并终止命令的执行）
    
-   在下载文件时点击取消下载，就会发送 USR ， 发送端会优先执行一些操作（比如终止传输文件数据）
    

# epoll 模型


[epoll 模型参考博客](https://my.oschina.net/u/4299156/blog/3233158)


# 四层负载均衡

下面介绍工作在传输层的伪 TCP，UDP 服务的四层负载均衡

## OSI 模型下的七层 LB 与四层 LB

![Pasted image 20220912102529](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220912102529.png)


### 四层负载均衡与连接五元组

TCP 使用四元组确定通信两端的主机和端口，四层负载均衡/传输层 通过五元确定通信两端

-   Source IP
    
-   Destination IP
    
-   Source Port
    
-   Destination Port
    
-   Protocol （传输层使用的协议，比如：TCP，UDP）

