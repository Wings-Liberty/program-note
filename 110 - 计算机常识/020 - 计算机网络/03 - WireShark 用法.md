#还没有复习 

# WireShark 使用流程


WireShark 用于抓取网络报文，问题是它会抓取宿主机上的所有报文

所以需要知道如何抓取指定报文，如何查看数据包细节

如何抓取指定报文？要解决这个问题需要使用者明确自己要抓取哪个网卡，源主机，目的主机的地址，端口，协议等信息

明确需要抓取哪些报文后，通过 WireShark 的过滤器编写过滤规则即可获取所需要的报文


# Quick Start


选择 WireShark 的报文输入接口，并定义捕获过滤器

![Pasted image 20220911150756](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911150756.png)


【开始】抓取报文后就会进入 WireShark 面板

## 主面板

![Pasted image 20220911150908](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911150908.png)


## 工具栏介绍

![Pasted image 20220911150958](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911150958.png)

每种报文都有自己的默认着色规则，可以在【视图】-【着色规则】里查看详情或自定义

![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416110423.png)



## 数据包列表介绍

![Pasted image 20220911151238](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911151238.png)

- 【No】是数据包的相对序号，也是被捕获的相对时间
- 【Time】可以设置为绝对时间，便于阅读。在【视图】-【时间显示格式】里自定义

数据包列表面板左侧的线条标记符号及其含义如下

![Pasted image 20220911151704](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911151704.png)


但是 WireShark 不会只捕获一个会话的数据包，所以数据包列表混合着多个会话的数据包，如果想只看某一个会话的数据包，需要跟踪流

![Pasted image 20220911152208](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911152208.png)



> [!NOTE] 如何抓取移动设备的报文
> 在 WireShark 的宿主机上开 wifi 热点，让移动设备连接 wifi 热点。移动设备收发数据就会都通过开热点的设备，WireShark 选择捕获选项时就能选择 wifi 热点对应的接口设备抓包
> ![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416110447.png)


# Wireshark 过滤器


支持两种过滤器

- 捕获过滤器。在捕获选项面板设置，用于**减少抓取的报文数量**，使**用 BPF 语法**，功能相对有限，但语法比较通用，**多数抓包工具都能用 BPF 语法写过滤语句**

![Pasted image 20220911160302](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911160302.png)

- 显示过滤器。在抓包主面板设置，**对已经抓取到的报文过滤显示**，功能强大，**是 WireShark 独有的语法**

![Pasted image 20220911160242](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911160242.png)


## 捕获过滤器

Berkeley Packet Filter，在设备驱动级别提供抓包过滤接口，多数抓包工具都支持此语法（比如 `tcpdump` 就支持此语法）


捕获**过滤器用 expression 表达式表示**，表达式**由多个原语组成**


### Expression 表达式

Expression = 多个 **Primitives**（原语）+ **原语运算符**

### Primitives 原语

Primitives = <font color='red'>名称或数字</font> + 以及描述它的多个 **Qualifiers**（限定词 ）

### Qualifiers 限定词

![Pasted image 20220911161015](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911161015.png)


### 原语运算符
- 与：&& 或者 and
- 或：|| 或者 or
- 非：! 或者 not

![Pasted image 20220911161200](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911161200.png)



### 限定词列表

原语中的名称，数字，运算符都好说。但是限定词比较多，因此在此列举几个常用限定词



**Type：设置数字或者名称所指示类型** 

- host、port
- net 。设定子网，`net 192.168.0.0 mask 255.255.255.0` 等价于 `net 192.168.0.0/24`
- portrange。设置端口范围，例如 `portrange 6000-8000`

**Dir：设置网络出入方向** 

- src、dst、src or dst、src and dst
- ra、ta、addr1、addr2、addr3、addr4（仅对 IEEE 802.11 Wireless LAN 有效）这些是链路层的限定词，**了解即可**

**Proto：指定协议类型**

（捕获过滤器只支持部分基础协议，像是 websocket，http，dns 应用层协议统统不支持。显示过滤器支持）

- ether、fddi、tr、 wlan、 ip、 ip6、 arp、 rarp、 decnet、 tcp、udp、icmp、igmp、icmp、igrp、pim、ah、esp、vrrp

**其他限定词**

- gateway：指明网关 IP 地址，等价于 `ether host ehost and not host host`
- broadcast：广播报文，例如 `ether broadcast` 或者 `ip broadcast`
- multicast：多播报文，例如 `ip multicast` 或者 `ip6 multicast`
- less, greater：小于或者大于


比较复杂的表达式可以写到抓取 XX 协议报文中某个位是指定值的程度，比如：

- 捕获所有 TCP 中的 RST 报文。`tcp[13]&4==4`
- 抓取 HTTP GET 报文。`port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420`




## 显示过滤器


Q：显示过滤器的表达式可以过滤什么？

A：任何在报文细节面板中解析出的字段名，都可以作为过滤属性


显示过滤器使用**过滤属性作为过滤条件表达式**，任何在报文细节面板中解析出的**字段名都可以作为过滤属性** 

在【视图】-【内部】-【支持的协议】面板里，可以看到各字段名对应的属性名

![Pasted image 20220911162721](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911162721.png)


例如，在报文细节面板中 TCP 协议头中的 Source Port，对应着过滤属性为 tcp.srcport


![Pasted image 20220911162752](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911162752.png)

![Pasted image 20220911163048](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911163048.png)


#### 过滤值的比较符号

- 使用英文缩写或符号均可
- 可以用正则表达式

![Pasted image 20220911163147](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911163147.png)



#### 过滤值类型

![Pasted image 20220911163207](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911163207.png)


#### 表达式的逻辑运算符


![Pasted image 20220911172048](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911172048.png)


此外还有**集合**运算符和**切片**操作符

**集合操作符`{}`**

例如 `tcp.port in {443 4430..4434}` ，实际等价于 `tcp.port == 443 || (tcp.port >= 4430 && tcp.port ⇐ 4434) `

**切片操作符`[]`**  

（数字的单位为字节）

- [n:m]表示 n 是起始偏移量，m 是切片长度
  - eth.src[0:3] == 00:00:83
- [n-m]表示 n 是起始偏移量，m 是截止偏移量
  - eth.src[1-2] == 00:83
- [:m]表示从开始处至 m 截止偏移量
  - eth.src[:4] == 00:00:83:00
- [m:]表示 m 是起始偏移量，至字段结尾
  - eth.src[4:] == 20:20
- [m]表示取偏移量 m 处的字节
  - eth.src[2] == 83
- [,]使用逗号分隔时，允许以上方式同时出现
  - eth.src[0:3,1-2,:4,4:,2] \=\= 00:00:83:00:83:00:00:83:00:20:20:83



### 可用函数

| 函数名 | 英文含义                                            |
| ------ | --------------------------------------------------- |
| upper  | Converts a string field to uppercase.               |
| lower  | Converts a string field to lowercase.               |
| len    | Returns the byte length of a string or bytes field. |
| count  | Returns the number of field occurrences in a frame. |
| string | Converts a non-string field to a string.            |



### 显示过滤器表达式可视化面板

在这个面板可以查看都有哪些属性能作为显示过滤器的选项，也便于构建表达式

【分析】-【显示过滤器表达式】

![Pasted image 20220911173925](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911173925.png)


