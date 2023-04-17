#还没有复习 

# 课程模块



## 课程模块概述

- 应用层
  - 第1部分：`HTTP/1.1`
  - 第2部分：`Websocket`
  - 第3部分：`HTTP/2.0`
- 应用层的安全基础设施
  - 第4部分：`TLS/SSL`
- 传输层
  - 第5部分：`TCP`
- 网络层及数据链路层
  - 第6部分：`IP`层和以太网



- 涉及主要工具
  - `Chrome`浏览器的`Network`面板
  - `Wireshark`
  - `tcpdump`——`Linux`上的抓包命令行工具
- 涉及次要工具
  - `curl`——`Linux`上的利用URL规则在命令行下工作的文件传输工具
  - `netstat`——`Linux`上的查看 Linux 中网络系统状态信息



课程是根据应用场景中的需求讲解知识的，比如不会详细讲解`HTTP/1.1`中的多个头部，而是根据实际场景中遇到的问题讲解



## 课程模块主线

> 自顶向下、由业务到逻辑
>
> - `HTTP/1.1`协议为什么要如此设计？它还有哪些约束
>   - 网络分层原理、`REST`架构
> - 协议的通信规则
>   - 协议格式、`URI`、方法与响应码概览
> - 连接与消息的路由
> - 内容协商与传输
> - `Cookie`的设计与问题
> - 缓存的控制

`HTTP/1.1`协议的缺点是不支持服务端主动向客户端发送数据，为此需要将`HTTP/1.1`协议升级为`WebSocket`协议

>- 支持服务器推送消息的`WebSocket`协议的生命周期
> - 建立会话
> - 消息传输
> - 心跳
> - 关闭会话
>- 全面优化后的`HTTP/2.0`协议（支持服务端主动向客户端发送数据，且比`WebSocket`协议效率高）
>- `HTTP/2.0`必须开启的`TSL/SSL`协议
>  - `TLS`协议包含哪些部分
>  - 对称加密算法（以`HTTP/3`协议中经常使用的`AES`为例）
>  - 非对称加密算法
>  - `DH`或椭圆曲线传递密钥的流程
>  - `DV`，`OV`，`EV`证书，以及这些证书依赖的`CA`，或`PKI`基础设施
>  - `TLS`在实际场景中的用法

当出现丢包率高，并发压力大导致网络拥堵等问题后，从应用层不能解决问题时，可能就需要从`TCP`与`IP`协议上分析问题

> - 传输层的`TCP`协议
>   - 建立连接 
>   - 传输数据 
>   - 拥塞控制 
>   - 关闭连接
> - 网络层的`IP`协议 
>   - `IP`报文与路由
>   - 网络层其他常用协议：`ICMP`、`ARP`、`RARP`
>   - `IPv6`的区别



PS：区分几个名词。浏览器，客户端（指前端代码），服务端/服务器（指提供后端服务的浏览器，方便起见下文也指后端服务本身）

文中常见的 RFC 文档可以在[这里](https://httpwg.org/specs/rfc7541.html)找

# 第 1 部分：HTTP/1.1



## 浏览器发送HTTP请求的典型场景

![Snipaste_2021-01-26_21-20-51](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-26_21-20-51.png)

用户在地址栏中输入`url`时，浏览器会联想出用户曾访问过的`url`。因为浏览器中有一个轻量级数据库保存了用户的一些操作



输入地址后，按下回车键。会发生以下事件

![Snipaste_2021-01-26_21-25-54](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-26_21-25-54.png)

上图表明，不管是从URL中解析域名，向DNS发送查询域名的请求，还是构造HTTP请求，这都是浏览器的工作，而不是HTML标记语言和JS语言做的事



> `HTTP/1.1`协议
>
> a stateless application-level request/response protocol that uses extensible semantics and self-descriptive message payloads for flexible  interaction with network-based hypertext information systems （RFC7230 2014.6） 
>
> 一种无状态的、应用层的、以请求/应答方式运行的协议，它使用可扩展的 语义和自描述消息格式，与基于网络的超文本信息系统灵活的互动



## 基于ABNF语义定义的HTTP消息格式



### 口语话描述

口语上不规范地表述HTTP消息格式通常是这样的

- `start-line`起始行（必须有）
  - `request-line`请求的起始行——请求行
  - `status-line`响应的起始行——状态行
- `HEADERS-field`头部/首部（0或多个首部）
- `message-body`包体（可选）



### ABNF语义规范

当需要详细表述每个部分中，消息的书写格式时。例如，哪里必须加空格，哪里必须加换行回车等。就需要ABNF语义

```
ABNF （扩充巴科斯-瑙尔范式）操作符 
• 空白字符：用来分隔定义中的各个元素 
	• method SP request-target SP HTTP-version CRLF 
• 选择 /：表示多个规则都是可供选择的规则 
	• start-line = request-line / status-line 
• 值范围 %c##-## ： 
	• OCTAL = “0” / “1” / “2” / “3” / “4” / “5” / “6” / “7” 与 OCTAL = %x30-37 等价 
• 序列组合 ()：将规则组合起来，视为单个元素 • 不定量重复 m*n： 
	• * 元素表示零个或更多元素： *( HEADERS-field CRLF ) 
	• 1* 元素表示一个或更多元素，2*4 元素表示两个至四个元素 
• 可选序列 []： • [ message-body ]
```

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-26_22-17-34.png" alt="Snipaste_2021-01-26_22-17-34" style="zoom:60%;" />



### ABNF语义描述的HTTP协议格式

> RFC规定的基于ABNF语义描述的HTTP协议格式
>
> **HTTP-message =** **start-line** ***(** **HEADERS-field** **CRLF ) CRLF [** **message-body** **]**
>
> - **start-line** = request-line / status-line 
>   - request-line = method SP request-target SP HTTP-version CRLF
>   - status-line = HTTP-version SP status-code SP reason-phrase CRLF 
> - **HEADERS-field** = field-name ":" OWS field-value OWS 
>   - OWS = *( SP / HTAB ) 
>   - field-name = token 
>   - field-value = *( field-content / obs-fold ) 
> - **message-body** = *OCTET

### 实践举例

```sh
$ telnet www.taohui.pub 80
```

发送请求报文

```http
GET /wp-content/plugins/Pure-Highlightjs_1.0/highlight/styles/default.css?ver=0.9.2 HTTP/1.1
Host: www.taohui.pub
```

接收响应报文

```http
HTTP/1.1 301 Moved Permanently
Server: openresty/1.19.3.1
Date: Tue, 26 Jan 2021 14:03:02 GMT
Content-Type: text/html
Content-Length: 175
Connection: keep-alive
Location: https://www.taohui.pub/wp-content/plugins/Pure-Highlightjs_1.0/highlight/styles/default.css?ver=0.9.2

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>openresty/1.19.3.1</center>
</body>
</html>
```

对于不可见的空格，换行回车符，可以在`Wireshark`中查看报文的二进制格式（实际使用的是16进制文件）找到。例`\r\n`被表示为`0d0a`



## 网络分层

> 在将
>
> - OSI概念模型
> - TC/IP模型
> - 报文头部示意图
> - 一个`HTTP`报文的示例分析

### OSI（Open System Interconnection Reference Model）概念模型

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-27_12-51-13.png" alt="Snipaste_2021-01-27_12-51-13" style="zoom:80%;" />

- 工作在数据链路层的物理设备被称为交换机
- 工作在网络层的物理设备被称为路由器

### OSI 模型与 TCP/IP 模型对照

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-27_12-52-19.png" alt="Snipaste_2021-01-27_12-52-19" style="zoom:80%;" />



### 报文头部示意图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-27_13-19-04.png" alt="Snipaste_2021-01-27_13-19-04" style="zoom:67%;" />



### 分析Wireshark中一个HTTP协议的报文

![Snipaste_2021-01-27_12-53-23](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-27_12-53-23.png)



## 不是人话系列—REST架构

好家伙



### 评估Web结构的七大关键属性

 

### 从五种架构风格推导出HTTP的REST架构

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-27_15-16-04.png" alt="Snipaste_2021-01-27_15-16-04" style="zoom:33%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-27_15-16-26.png" alt="Snipaste_2021-01-27_15-16-26" style="zoom:33%;" />



## 协议的通信规则



### 如何使用Chrome的Network面板分析HTTP报文



#### Network面板

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-27_20-40-51.png" alt="Snipaste_2021-01-27_20-40-51" style="zoom:80%;" />

- 控制器
  - `Preserve log` 同一个标签页中发生重定向时，清除请求列表中的旧抓包信息
  - `Disable cache` 禁用浏览器缓存
  - 离线模拟：Offline
  - 模拟限速网络连接：`Online`。可自定义上传下载速度
- 过滤器（过滤器的表达式中，多属性间通过空格实现AND操作）
  - `Hide data URLs`选项。隐藏嵌入在HTML代码中引发的请求
  - `domain`：仅显示来自指定域的资源。 可以使用通配符字符 (*) 纳入多个域 
  - `has-response-HEADERS`：显示包含指定 HTTP 响应标头的资源 
  - `is`：使用 is:running 可以查找 WebSocket 资源，is:from-cache 可查找缓存读出的资源（禁用浏览器cache后，浏览器将不会从磁盘读取缓存—`disk cache`，但还会从主存读取缓存`memory cache`）
  - `larger-than`： 显示大于指定大小的资源（以字节为单位）。 将值设为 1000 等同于设置为 1k 
  - `method`：显示通过指定 HTTP 方法类型检索的资源 
  - `mime-type`：显示指定 MIME 类型的资源（这里指的类型是响应中`Content-Type`头部值）
  - `mixed-content`：显示所有混合内容资源 (mixed-content:all)，或者仅显示当前显示的资源 (mixed-content:displayed)
  - `scheme`：显示通过未保护 HTTP (scheme:http) 或受保护 HTTPS (scheme:https) 检索的资源
  - `set-cookie-domain`：显示具有 Set-Cookie 标头并且 Domain 属性与指定值匹配的资源
  - `set-cookie-name`：显示具有 Set-Cookie 标头并且名称与指定值匹配的资源
  - `set-cookie-value`：显示具有 Set-Cookie 标头并且值与指定值匹配的资源
  - `status-code`：仅显示 HTTP 状态代码与指定代码匹配的资源
- 概览：显示`HTTP`请求、响应的时间轴
- 请求列表：默认时间排序，可选择显示列
  - 查看请求上下游:按住 shift键悬停请求上,绿色是上游,红色是下游（请求主页，主页是绿色，是上游。相对的请求主页后得到html，html中的src引发的和js发起的其他请求是下游请求）
  - `Name `: 资源的名称
  - `Status `: HTTP 状态代码
  - `Type `: 请求的资源的 MIME 类型
  - `Initiator `: 发起请求的对象或进程。它可能有以下几种值：
    - `Parser `（解析器） : Chrome的 HTML 解析器发起了请求
    - 鼠标悬停显示 JS 脚本 
    - `Redirect `（重定向） : HTTP 重定向启动了请求 
    - `Script `（脚本） : 脚本启动了请求 
    - `Other `（其他） : 一些其他进程或动作发起请求，例如用户点击链接跳转到面或在地址栏中输入网址
  - `ETag` 和缓存服务器相关的标记
  - `Size `: 服务器返回的响应大小（包括头部和包体），可显示解压后大小 
  - `Time` : 总持续时间，从请求的开始到接收响应中的最后一个字节 
- 概要：请求总数、总数据量、总花费时间等



#### 浏览器加载时间包括

解析HTML结构，加载外部脚本和样式表文件，解析并执行脚本代码（部分脚本会阻塞页面的加载 ）

DOM树构建完成（DOMContentLoaded事件），加载图片等外部文件，页面加载完毕（load事件）

[DOMContentLoaded与load的区别](https://www.cnblogs.com/caizhenbo/p/6679478.html)



#### 请求时间详细分布

- Queueing: 浏览器在以下情况下对请求排队
  - 存在更高优先级的请求 
  - 此源已打开六个 TCP 连接，达到限值，仅适用于 HTTP/1.0 和 HTTP/1.1 
  - 浏览器正在短暂分配磁盘缓存中的空间 
- Stalled: 请求可能会因 **Queueing** 中描述的任何原因而停止 
- DNS Lookup: 浏览器正在解析请求的 IP 地址 
- Proxy Negotiation: 浏览器正在与代理服务器协商请求
- Request sent: 正在发送请求 
- ServiceWorker Preparation: 浏览器正在启动 Service Worker 
- Request to ServiceWorker: 正在将请求发送到 Service Worker 
- Waiting (TTFB): 浏览器正在等待响应的第一个字节。 TTFB 表示 Time To First Byte （至第一字节的时间）。 此时间包括 1 次往返延迟时间及服务器准备响应所用的时间
- Content Download: 浏览器正在接收响应 
- Receiving Push: 浏览器正在通过 HTTP/2 服务器推送接收此响应的数据 
- Reading Push: 浏览器正在读取之前收到的本地数据



### URL的基本格式以及与URI的区别

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-28_11-23-31.png" alt="Snipaste_2021-01-28_11-23-31" style="zoom:80%;" />

对于url，urn，uri的区别暂不做区分



`URI`的ABFN语义描述

```
URI = scheme ":" hier-part [ "?" query ] [ "#" fragment ] 

- scheme = ALPHA *( ALPHA / DIGIT / "+" / "-" / "." ) 
  - 例如：http, https, ftp,mailto,rtsp,file,telnet 
- hier-part = "//" authority path-abempty / path-absolute / path-rootless / path-empty
  - authority = [ userinfo "@" ] host [ ":" port ] 
  - userinfo = *( unreserved / pct-encoded / sub-delims / ":" ) 
  - host = IP-literal / IPv4address / reg-name 
  - port = *DIGIT
  - path = path-abempty/ path-absolute/ path-noscheme / path-rootless / path-empty 
    - path-abempty = *( “/” segment ) 
      - 以/开头的路径或者空路径 
    - path-absolute = “/” [ segment-nz *( “/” segment ) ] 
      - 以/开头的路径，但不能以//开头 
    - path-noscheme = segment-nz-nc *( “/” segment ) 
      - 以非:号开头的路径 
    - path-rootless = segment-nz *( “/” segment ) 
      - 相对path-noscheme，增加允许以:号开头的路径 
    - path-empty = 0<pchar> 
      - 空路径
- query = *( pchar / "/" / "?" ) 
- fragment = *( pchar / "/" / "?" ) 
```



相对`URI`

URI-reference = URI/relative-ref 

- relative-ref = relative-part [ "?" query ] [ "#" fragment ] 
  - relative-part = "//" authority path-abempty / path-absolute / path-noscheme / path-empty 

```
https://tools.ietf.org/html/rfc7231?test=1#page-7
/html/rfc7231?test=1#page-7
```





### 为什么要对URI进行编码？

在ABNF语义描述的URL/URI中，使用`: @ / ? # @ `等作为分割符。这些字符称为保留字符

问题：问什么要对URI进行编码？

答：防止URI中其他地方出现以下几种产生歧义的数据

- 上述的保留字/分割符
- 不在ASCⅡ编码范围内的字符
- ASCⅡ中不可显示的字符
- 不安全的字符（传输环节可能会被不正确处理），如空格，引号，尖括号等



**百分号编码方式**



- pct-encoded = "%" HEXDIG HEXDIG （两个16进制数）
  - US-ASCII：128 个字符（95 个可显示字符，33 个不可显示字符） 
  - 参见：https://zh.wikipedia.org/wiki/ASCII 
- 对于 HEXDIG 十六进制中的字母，大小写等价 
- 非`ASCⅡ`码字符（例如中文）：建议先`UTF8`编码，再 US-ASCII 编码
- 对`URI`合法字符，编码与不编码是等价的





### 详解HTTP的请求行



> request-line = method SP request-target SP HTTP-version CRLF
>
> - `method`方法：指明操作目的，动词
> - `request-target` = origin-form / absolute-form / authority-form / asterisk-form
>   - `origin-form` = absolute-path [ "?" query ] 
>     - 向`origin server`发起的请求，path 为空时必须传递 / 
>   - `absolute-form` = absolute-URI 
>     - 仅用于向正向代理 proxy 发起请求时，详见正向代理与隧道 
>   - `authority-form` = authority 
>     - 仅用于 CONNECT 方法，例如 CONNECT www.example.com:80 HTTP/1.1。通常在建立VPN时使用
>   - `asterisk-form` = "*"
>     - 仅用于 OPTIONS 方法



> - HTTP/0.9：只支持 GET 方法，过时
> - HTTP/1.0：RFC1945，1996，常见使用于代理服务器（例如 Nginx 默认配置） 
> - HTTP/1.1：RFC2616，1999 
> - HTTP/2.0：2015.5正式发布



**常用方法**

- GET：主要的获取信息方法，大量的性能优化都针对该方法，幂等方法 
- HEAD：类似GET方法，但服务器不发送BODY，用以获取HEAD元数据，幂等方法 
- POST：常用于提交 HTML FORM 表单、新增资源等 
- PUT：更新资源，带条件时是幂等方法 
- DELETE：删除资源，幂等方法 
- CONNECT：建立`tunnel`隧道 
- OPTIONS：显示服务器对访问资源支持的方法，幂等方法。（接口支持的方法将会放在响应中的Allow头部中，并以逗号作为分隔符）
- TRACE：回显服务器收到的请求，用于定位问题。有安全风险（在nginx0.5.17后，所有`TRACE`方法的请求直接返回405）



**用于文档管理的WEBDAV方法**

- PROPFIND：从 Web 资源中检索以 XML 格式存储的属性。它也被重载，以允许一个检索远程系统的集合结构（也叫目录层次结构）
- PROPPATCH：在单个原子性动作中更改和删除资源的多个属性
- MKCOL：创建集合或者目录
- COPY：将资源从一个 URI 复制到另一个 URI 
- MOVE：将资源从一个 URI 移动到另一个 URI 
- LOCK：锁定一个资源。WebDAV支持共享锁和互斥锁。 
- UNLOCK：解除资源的锁定



`WinSCP`使用`WEBDAV`协议管理文件时就会使用上述方式

`nginx`的`http_dav_module`，`nginx-dav-ext-module`模块可以支持`WEBDAV`中的方法



### HTTP的正确响应码和错误响应码

- 1xx：请求已接收到，需要进一步处理才能完成，`HTTP/1.0`不支持
- 2xx：成功处理请求
- 3xx：重定向使用 Location 指向的资源或者缓存中的资源。在`RFC2068`中规定客户端重定向次数不应超过 5 次，以防止死循环



[HTTP状态码](https://baike.baidu.com/item/HTTP状态码/5053660?fr=aladdin)



PS:

205，通常用于画图软件。在画布上添加新节点后，服务器返回205提示节点添加成功提醒客户端刷新DOM树

429，通常在限流，限速等服务中，收到的请求过于频繁时通常不返回409，而是返回503`Service Unavailable`

431，当请求头部过大，过长时，通常返回414。因为uri和头部通常在同一段缓冲区被处理

507，将服务器内部信息暴露给客户端，这是不安全的，所以通常不会返回507

如果客户端识别不了返回的响应码（例如返回了555但是没有此响应码），客户端默认将其作为x00处理（比如将555当作500处理）



## 连接与消息的路由



### 如何管理跨代理服务器的长短连接？



**短链接发送HTTP请求和处理流程**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-28_13-57-43.png" alt="Snipaste_2021-01-28_13-57-43" style="zoom: 67%;" />



**从TCP网络编程看HTTP请求处理**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-28_13-58-14.png" alt="Snipaste_2021-01-28_13-58-14" style="zoom: 67%;" />



Connection头部

- Keep-Alive：长连接 
  - 客户端请求长连接 
    - Connection: Keep-Alive 
  - 服务器表示支持长连接 
    - Connection: Keep-Alive 
  - 客户端复用连接 
    - HTTP/1.1 默认支持长连接。Connection: Keep-Alive 无意义 
  - Close：短连接
  - 对代理服务器的要求 
    - 不转发 Connection 列出头部，该头部仅与当前连接相关



Connection 仅针对当前连接有效：当客户端携带`Connection: Keep-Alive`时会尝试和请求的服务器建立长连接，如果目标地址时代理服务器，那么代理服务器和上游服务器建立的连接不一定是长连接



如果代理服务器陈旧，不能正确的处理请求的Connection头部，会将客户端请求中的`Connection: Keep-Alive`原样转发给上游服务器

客户端和上游服务器会误以为和代理服务器建立了长连接，但实际上是短链接

解决方式：使用`Proxy-Connection: Keep-Alive`



### HTTTP消息在服务端的路由

> 此节将的是Host头部的作用



`Host` = uri-host [ ":" port ]

- HTTP/1.1 规范要求，不传递 Host 头部则返回 400 错误响应码 
- 为防止陈旧的代理服务器，发向正向代理的请求`request-target`必须以`absolute-form`形式出现 
  - `request-line` = method SP request-target SP HTTP-version CRLF
  - `absolute-form` = absolute-URI 
    - `absolute-URI` = scheme ":" hier-part [ "?" query ]



`Host` 是 HTTP 1.1 协议中新增的一个请求头，主要用来实现虚拟主机技术。

虚拟主机（virtual hosting）即共享主机（shared web hosting），可以利用虚拟技术把一台完整的服务器分成若干个主机，因此可以在单一主机上运行多个网站或服务。

举个栗子，有一台 ip 地址为 `61.135.169.125` 的服务器，在这台服务器上部署着谷歌、百度、淘宝的网站。为什么我们访问`https://www.google.com` 时，看到的是 Google 的首页而不是百度或者淘宝的首页？原因就是 `Host` 请求头决定着访问哪个虚拟主机。



以`nginx`为例。客户端和服务端建立 TCP 连接后，`nginx`会匹配 Host头部与域名寻找虚拟主机（server），再匹配url定位location



### 代理服务器转发消息时的相关头部



> - 代理服务器转发消息时的相关头部
> - 请求与响应的上下文
> - 内容协商与资源表述
>
> 以上这 3 节讲解 HTTP 的头部



#### 代理服务器向源服务器传递客户端IP

如果客户端和源服务器之间有代理服务器，代理服务器和源服务器建立的TCP连接中，发送的 HTTP 协议包中的源 IP 地址是代理服务器的地址的而不是客户端的地址

问题：源服务器怎么获取客户端的IP地址

- RFC 协议规定 代理服务器可以在向源服务器转发请求时在请求中添加`X-Forward-For`头部传递客户端的 IP
- `nginx`的`realip`模块通常会使用自定义的头部——`X-Real-IP`传递客户端的 IP



#### 消息的转发

`Max-Forwards`头部 

- 限制 Proxy 代理服务器的最大转发次数，仅对 TRACE/OPTIONS 方法有效 
- Max-Forwards = 1*DIGIT 

`Via`头部 

- 指明经过的代理服务器名称及版本
- ABNF 语义描述 Via 头部格式 Via = 1#( received-protocol RWS received-by [ RWS comment ] ) 
  - received-protocol = [ protocol-name "/" ] protocol-version 
  - received-by = ( uri-host [ ":" port ] ) / pseudonym 
  - pseudonym = token 

`Cache-Control:no-transform`头部

- 禁止代理服务器修改响应包体



### 请求与响应的上下文



#### User-Agent

指明客户端的类型信息，服务器可以据此对资源的表述做抉择。其值包含了使用的浏览器名&版本号，操作系统名&版本号等

> User-Agent = product *( RWS ( product / comment ) ) 
>
> - product = token ["/" product-version] 
> - RWS = 1*( SP / HTAB ) 
>
> 例如：
>
> - User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:66.0) Gecko/20100101 Firefox/66.0
> - User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36



#### Referer



浏览器对来自某一页面的请求自动添加的头部。非浏览器的客户端发送的请求未必会添加这个头部

> Referer = absolute-URI / partial-URI
>
> 例如：Referer: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/HEADERSs/User-Agent 
>
> Referer 不会被添加的场景 
>
> - 来源页面采用的协议为表示本地文件的 "file" 或者 "data" URI 
> - 当前请求页面采用的是 http 协议，而来源页面采用的是 https 协议 
>
> 服务器端常用 Referer 统计分析、缓存优化、防盗链等功能



#### Server



指明服务器上所用软件的信息，用于帮助客户端定位问题或者统计数据 

> Server = product *( RWS ( product / comment ) ) 
>
> - product = token ["/" product-version] 
>
> 例如： 
>
> - Server: nginx
> - Server: openresty/1.13.6.2



#### Allow与Accept-Ranges



Allow：告诉客户端，服务器上该 URI 对应的资源允许哪些方法的执行 

> `Allow = #method `
>
> 例如：Allow: GET, HEAD, PUT 



Accept-Ranges：告诉客户端服务器上该资源是否允许 range 请求。这和多线程下载和断点续传有关

> Accept-Ranges = acceptable-ranges 
>
> 例如：
>
> - Accept-Ranges: bytes （开启）
> - Accept-Ranges: none （关闭）



## 内容协商与传输

内容协商：每个 URI 指向的资源可以是任何事物，可以有多种不同的表述，例如一份文档可以有**不同语言的翻译**（国际化）、**不同的媒体格式**（文件格式）、可以针对不同的浏览器提供**不同的压缩编码**（传递数据时使用的压缩方式）等



由于内容协商的存在，不同客户端访问同一个 URL 得到的响应结果可能时不同的。这是浏览器与服务器内容协商的结果

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_12-14-47.png" alt="Snipaste_2021-01-29_12-14-47" style="zoom:80%;" />

### 主动式&被动式内容协商

Proactive 主动式内容协商： 

- 指由客户端先在请求头部中提出需要的表述形式，而服务器根据这些请求头部提供特定的 representation 表述 

Reactive 响应式内容协商： 

- 指服务器返回 300 Multiple Choices 或者 406 Not Acceptable，由客户端选择一种表述 URI 使用



主动式内容协商

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_12-21-37.png" alt="Snipaste_2021-01-29_12-21-37" style="zoom:50%;" />

被动式内容协商（由于 RFC 没有明确规定 Client 如何从多个选择中选一个内容的规则，所以不同浏览器的选择策略不同。没有明确的规范导致实际情况下很少使用被动式内容协商）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_12-25-38.png" alt="Snipaste_2021-01-29_12-25-38" style="zoom: 67%;" />



- 质量因子q：内容的质量、可接受类型的优先级





### 常见的协商要素

媒体资源的 MIME 类型

- Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
- Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3

字符编码：由于 UTF-8 格式广为使用， **Accept-Charset 已被废弃**

- Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7 

内容编码：主要指压缩算法 

- Accept-Encoding: gzip, deflate, br 

表述语言 

- Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
- Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2



### 国际化与本地化

> internationalization（i18n，i 和 n 间有 18 个字符） 
>
> - 指设计软件时，在不同的国家、地区可以不做逻辑实现层面的修改便能够以不同的语言显示 
>
> localization（l10n，l 和 n 间有 10 个字符） 
>
> - 指内容协商时，根据请求中的语言及区域信息，选择特定的语言作为资源表述



### 协商要素和资源表述

请求中携带`Accept`/`Accept-*`表示希望服务器能返回客户端要求形式的内容

响应中携带`Content`/`Content-*`表示服务器实际返回的协商内容的响应

- 媒体类型、编码 例：ontent-type: text/html; charset=utf-8
- 内容编码 例：content-encoding: gzip
- 语言 例：Content-Language: de-DE, en-CA



### HTTP包体的传输方式

> 以下几节讲解 HTTP 包体
>
> - 定长包体传输
> - 不定长包体传输
> - form 表单提交传输



请求或者响应都可以携带包体

> HTTP-message = start-line *( HEADERS-field CRLF ) CRLF **[ message-body ]** 
>
> - message-body = *OCTET：二进制字节流
>
> 以下消息不能含有包体 
>
> - HEAD 方法请求对应的响应 
> - 1xx、204、304 对应的响应 
> - CONNECT 方法对应的 2xx 响应





#### 定长包体传输



发送 HTTP 消息时已能够确定包体的全部长度 

- 使用 Content-Length 头部明确指明包体长度 
  - Content-Length = 1*DIGIT 
  - 用 10 进制（不是 16 进制）表示包体中的字节个数，且必须与实际传输的包体长度一致 

优点：接收端处理更简单



如果 Content-Length 小于实际包体，抓包后可知，TCP 连接中能找到完整的数据。但是 HTTP 响应中只读取 Content-Length 指定大小的数据，多余的数据抛弃掉

如果 Content-Length 大于实际包体，浏览器因无法正确读取数据而直接断开连接并报错



#### 不定长包体传输



发送 HTTP 消息时不能确定包体的全部长度 

使用 Transfer-Encoding 头部指明使用 Chunk 传输方式

- 含 Transfer-Encoding 头部后 Content-Length 头部应被忽略
- 包体将以 chunk 为单位发送多个 chunk ，直到包体全部发送完毕



优点

- 基于长连接持续推送动态内容
- 压缩体积较大的包体时，不必完全压缩完（计算出头部）再发送，可以边发送边压缩
- 传递必须在包体传输完才能计算出的 Trailer 头部



> Transfer-Encoding头部 
>
> transfer-coding = "**chunked**" / "compress" / "deflate" / "gzip" / transfer-extension 
>
> Chunked transfer encoding 分块传输编码： Transfer-Encoding：chunked
>
> chunked-body = *chunk
>
> chunk = chunk-size [ chunk-ext ] CRLF chunk-data CRLF 
>
> - chunk-size = 1*HEXDIG：注意这里是 16 进制而不是 10 进制
> - chunk-data = 1*OCTET
> - last-chunk = 1*("0") [ chunk-ext ] CRLF    # 如果传输的是最后一个 chunk 就加上此标识，通常是若干个0。提醒客户端包体发送完毕
> - trailer-part = *( HEADERS-field CRLF )     # 通常在传送最后的 chunk 时还可以传送头部。例如：常用头部，统计所有 chunk 大小总和的头部等
>
> PS：暂时忽略 chunk-ext



不少客户端都不支持 Trailer 头部的传输，所以如果客户端支持，会在请求中添加 TE 头部

TE 头部：客户端在请求在声明是否接收 Trailer 头部 

- TE: trailers 

Trailer 头部：服务器告知接下来 chunk 包体后会传输哪些 Trailer 头部 

- 例如：Trailer: Date 

以下头部不允许出现在 Trailer 的值中： 

- 用于信息分帧的首部 (例如 Transfer-Encoding 和 Content-Length) 
- 用于路由用途的首部 (例如 Host) 
- 请求修饰首部 (例如控制类和条件类的，如 Cache-Control，Max-Forwards，或者 TE) 
- 身份验证首部 (例如 Authorization 或者 Set-Cookie) 
- Content-Encoding, Content-Type, Content-Range，以及 Trailer 自身





#### MIME媒体数据类型

> MIME（ Multipurpose Internet Mail Extensions ） 

"Content-Type" ":" type "/" subtype *(";" parameter) 

- type := discrete-type / composite-type   （主类型：独立媒体类型和复合媒体类型）
  - discrete-type := "text" / "image" / "audio" / "video" / "application" / extension-token（文本，图片，音频，视频，应用程序 如js，扩展类型）
  - composite-type := "message" / "multipart" / extension-token 
  - extension-token := ietf-token / x-token 
- subtype := extension-token / iana-token （子类型，每种主类型下都有众多子类型，例如 text/plain   application/json）
- parameter := attribute "=" value 

大小写不敏感，但通常是小写 

例如：Content-type: text/plain; charset=utf-8



[更多MIME媒体数据类型](https://www.iana.org/assignments/media-types/media-types.xhtml)



#### HTMLform表单提交时的协议格式

> 分以下几个部分讲解
>
> - HTML 中的 form 表单详解
> - Multipart 包体格式
> - 实例演示



**HTML 的 form 表单**

- HTML：HyperText Markup Language，结构化的标记语言（非编程语言） 。浏览器可以将 HTML 文件渲染为可视化网页 

- HTML 是一个结构化的文本文档，没有交互能力和发送请求的能力但是 HTML 的表单却可以通过提交按钮直接发送 HTTP 请求

- FORM 表单：HTML 中的元素，提供了交互控制元件用来向服务器通过 HTTP 协议提交信息，常见控件有

  - Text Input Controls：文本输入控件 
  - Checkboxes Controls：复选框控件
  - Radio Box Controls ：单选按钮控件
  - Select Box Controls：下拉列表控件
  - File Select boxes：选取文件控件
  - Clickable Buttons：可点击的按钮控件 
  - **Submit** and Reset Button：提交或者重置按钮控件

- form 表单提交请求时的关键属性

  - action：提交时发起 HTTP 请求的 URI 

  - method：提交时发起 HTTP 请求的 http 方法 

    - GET：通过 URI，将表单数据以 URI 参数的方式提交
    - POST：将表单数据放在请求包体中提交 

  - enctype：在 POST 方法下，对表单内容在请求包体中的编码方式 

    - application/x-www-form-urlencoded 数据被编码成以 ‘&’ 分隔的键-值对, 同时以 ‘=’ 分隔键和值，字符以 URL 编码方式编码

    - multipart/form-data 包括 1个或多个  分隔符+资源表示  最后表示包体结束的结束分隔符    PS：资源表示=资源描述+数据

      

**Multipart包体格式（RFC822）**

>  一个 multipart/form-data 编码类型的包体可以包含 1 个或多个独立的资源表示，也就是有 1个或多个 encapsulation
>
> multipart/form-data 包括 1个或多个  分隔符+资源表示  最后表示包体结束的结束分隔符    PS：资源表示=资源描述+数据

multipart-body = preamble 1*encapsulation close-delimiter epilogue

- preamble := discard-text
- epilogue := discard-text
  - discard-text := *(text CRLF) 

 通常客户端会直接丢弃上述两个标识，所以不用管它

每一个 encapsulation 都对应 1 个资源表示。例如：1 个文件、图片、单选框、输入框等

每部分包体格式：encapsulation = delimiter body-part CRLF     # 每部分包括 分隔符 +（资源描述+数据）

- delimiter = "--" **boundary** CRLF
  - boundary 是真正分割符的组成部分，通常由浏览器指定一个分隔符。后文再介绍其ABNF语义描述
- body-part = fields *( CRLF *text ) 
  - field = field-name ":" [ field-value ] CRLF 
    - content-disposition: form-data; name="xxx"    # 这里的name对应form标签的子标签的name，比如单选框、输入框，文件标签的name
    - content-type 头部指明该部分包体的类型   # 这里的content-type不是 HTTP 报文中的头部，而是 body-part 的组成部分
  - text是真正的数据，编码格式可以是ASC或二进制等多种格式中的一种

close-delimiter = "--" **boundary** "--" CRLF      # 表示包体结束的结束分隔符



分隔符的语义描述

boundary := 0*69<bchars> bcharsnospace

- bchars := bcharsnospace / " " 
- bcharsnospace := DIGIT / ALPHA / "'" / "(" / ")" / "+" / "_" / "," / "-" / "." / "/" / ":" / "=" / "?"

例如：某次响应中指定的分隔符是----WebKitFormBoundaryRRJKeWfHPGrS4LKe

multipart/form-data 的头部举例：Content-type: multipart/form-data; boundary=----WebKitFormBoundaryRRJKeWfHPGrS4LKe

[xhr multipart boundary分隔符](https://www.cnblogs.com/ksyy/p/11506361.html)



**实例演示**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_17-38-53.png" alt="Snipaste_2021-01-29_17-38-53" style="zoom:80%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_17-39-07.png" alt="Snipaste_2021-01-29_17-39-07" style="zoom: 80%;" />

上图是 4 中表单示例

演示抓包时，浏览器的 Network 面板不能很好地展示数据，用 Wireshark 获取数据更详细

- get 请求的表单，只能使用 application/x-www-form-urlencoed 方式编码，且表单数据将拼接在url后，而不是包体中

- get 请求的表单能将参数拼接到 url 后的原因

  - applicarion/x-www-form-urlcoded 编码方式和 url 后的参数的编码方式刚好相同
  - get请求不能携带包体，而 applicarion/x-www-form-urlcoded 编码方式的参数刚好又能拼接到url后

- get 请求的表单不能使用 application/x-www-form-ubcoded 是因为这种编码方式必须将数据放在包体中

- applicarion/x-www-form-urlcoded 编码方式只能将数据以kv方式拼接，所以这种编码方式只能传递简单的kv数据类型，对于复杂的数据类型只能使用 post 方式提交 multipart/form-data 的表单，将数据放在包体中

  

post 请求的 applicarion/x-www-form-urlcoded 表单抓包示意图

![Snipaste_2021-01-29_17-47-05](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_17-47-05.png)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_17-47-19.png" alt="Snipaste_2021-01-29_17-47-19" style="zoom:80%;" />



PS：form标签中的action属性通常都是 /xxx/xxx，这是相对url。请求的完整url地址需要拼接前缀

问：如果tomcat中某个server配置了虚拟路径，前端使用表单时action是否需要主动添加上虚拟路径？猜测应该需要





### 断点续传和多线程下载是如何做到的？

> - 断点续传
> - 多线程下载
> - 随即点播（视频拖动滚动条播放。在线视频通常不会将视频文件全部加载，而是加载当前滚动条位置及其后面一部分内容）
>
> 上述 3 种场景是怎么做到的？围绕这 3 种场景展开此节内容



以上场景中，请求的路程如下

1. 客户端明确任务：从哪开始下载
   - 本地是否已有部分文件
     - 文件已下载部分在服务器端发生改变？
   - 使用几个线程并发下载
2. 下载文件的指定部分内容
3. 下载完毕后拼装成统一的文件



#### HTTP Range规范(RFC7233)

- 允许服务器基于客户端的请求只发送响应包体的一部分给到客户端，而客户端自动将多个片断的包体组合成完整的体积更大的包体 
- 服务器通过 Accept-Range 头部表示是否支持 Range 请求
  - Accept-Ranges = acceptable-ranges 
  - 例如：
    - Accept-Ranges: bytes：支持
    - Accept-Ranges: none：不支持



#### Range 请求范围的单位



示例：基于字节，设包体总长度为 10000。使用 Range 头部指定请求的数据的字节范围

- 第 1 个 500 字节：bytes=0-499
- 第 2 个 500 字节：
  - bytes=500-999
  - bytes=500-600,601-999
  - bytes=500-700,601-999
- 最后 1 个 500 字节：
  - bytes=-500
  - bytes=9500- 
- 仅要第 1 个和最后 1 个字节：bytes=0-0,-1



通过 Range 头部传递请求范围，如：Range: bytes=0-499



#### Range 条件请求



如果客户端已经得到了 Range 响应的一部分，并想在这部分响应未过期的情况下，获取其他部分的响应 

- 常与 If-Unmodified-Since 或者 If-Match 头部共同使用
- If-Range = entity-tag / HTTP-date 
  - 可以使用 Etag 或者 Last-Modified

每次请求数据后，响应中会携带一个 ETag 头部作为请求的指纹，请求下一部分资源时，将指纹的值放到 If-Range 头部中。示例如下图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_18-31-40.png" alt="Snipaste_2021-01-29_18-31-40" style="zoom:80%;" />

如果指纹不匹配，将返回 412 响应码



#### 服务端的响应



- 206 已处理部分请求
- 416 请求返回不合法
- 200 OK



- 206 Partial Content
  - Content-Range 头部：显示当前片断包体在完整包体中的位置 
  - Content-Range = byte-content-range / other-content-range     # 表示范围以字节为传输单位或以其他为单位为传输单位
    - byte-content-range = bytes-unit SP ( byte-range-resp / unsatisfied-range )     # bytes-unit 表示 content-range 使用的单位
      - byte-range-resp = byte-range "/" ( complete-length / "*" )
        - complete-length = 1*DIGIT    # 表示完整资源的大小，如果未知则用 * 号替代
        - byte-range = first-byte-pos "-" last-byte-pos 
  - 例如：
    - Content-Range: bytes 42-1233/1234 
    - Content-Range: bytes 42-1233/*
- 416 Range Not Satisfiable
  - 请求范围不满足实际资源的大小，其中 Content-Range 中的 complete-length 显示完整响应的长度，例如：Content-Range: bytes */1234 
- 200 OK
  - 服务器不支持 Range 请求时，则以 200 返回完整的响应包体



**206响应示例**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_18-46-11.png" alt="Snipaste_2021-01-29_18-46-11" style="zoom:80%;" />

**416响应示例**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-29_18-46-29.png" alt="Snipaste_2021-01-29_18-46-29" style="zoom:80%;" />



**简单的多范围请求与multipart响应示例**

- 请求
  - Range: bytes=0-50, 100-150
- 响应
  - Content-Type：multipart/byteranges; boundary=…
  - 其包体保存数据的方式和上节表单提交请求使用的包体格式基本相同，因为它们的 content-type 的主类型都是 multipart





## Cookie的格式和约束及第三方Cookie的工作原理



### Cookie 的工作原理

RFC6265, HTTP State Management Mechanism 

Cookie 被保存在客户端、由浏览器维护、由服务端创建

- 存放在内存或者磁盘中
- 服务器端生成 Cookie 在响应中通过 Set-Cookie 头部告知客户端（允许多个 Set-Cookie 头部传递多个值）
- 客户端得到 Cookie 后，后续请求都会自动将 Cookie 头部携带至请求中



### Cookie 头部和 Set-Cookie头部的定义

Cookie 头部中可以存放多个 name/value 名值对 

cookie-HEADERS = "Cookie:" OWS cookie-string OWS 

- cookie-string = cookie-pair *( ";" SP cookie-pair ) 
- cookie-pair = **cookie-name "=" cookie-value** 



Set-Cookie 头部一次只能传递 1 个 name/value 名值对，响应中可以含多个头部 

- set-cookie-HEADERS = "Set-Cookie:" SP set-cookie-string
  - set-cookie-string = cookie-pair *( ";" SP cookie-av ) 
    - cookie-pair = cookie-name "=" cookie-value
    - cookie-av：描述 cookie-pair 的可选属性



cookie-av = expires-av / max-age-av / domain-av / path-av / secure-av / httponly-av / extension-av

- expires-av = "Expires=" sane-cookie-date 
  - cookie 到日期 sane-cookie-date 后失效
- max-age-av = "Max-Age=" non-zero-digit \*DIGIT
  - cookie 经过 *DIGIT 秒后失效。max-age 优先级高于 expires
- domain-av = "Domain=" domain-value 
  - 指定 cookie 可用于哪些域名，默认可以访问当前域名 
- path-av = "Path=" path-value
  - 指定 Path 路径下才能使用 cookie
- secure-av = "Secure"
  - 只有使用 TLS/SSL 协议（https）时才能使用 cookie
- httponly-av = "HttpOnly"
  - 不能使用 JavaScript（Document.cookie 、XMLHttpRequest 、Request APIs）访问到 cookie



### Cookie 使用的限制

RFC 规范对浏览器使用 Cookie 的要求 

- 每条 Cookie 的长度（包括 name、value 以及描述的属性等总长度）的最大值由浏览器规定，但也不能小于 4KB
- 每个域名下至少支持 50 个 Cookie
- 至少要支持 3000 个 Cookie

代理服务器传递 Cookie 时会有限制



### 第三方Cookie

> 第三方 Cookie 常用于收集用户的轨迹信息或其他信息



跨域访问资源时，请求的地址是第三方服务器。响应中可能会携带有 Set-Cookie 头部。浏览器会将这些数据保存到本地 Cookie 中

在用户后续访问第三方服务器时，请求将自行携带上这些 Cookie



比如：

1. 用户访问A网站并购买商品
2. 用户在B网站浏览时，能在B网站查询到他在A网站的购买记录

说明用户的资源在访问A网站时被保存到了B网站





## 浏览器的同源策略

> 同源策略的目的是：防止不同站点间互相读取信息
>
> 比如：不同站点间互相读取 Cookie，甚至用盗取的数据互相修改DOM树等不安全的操作



### 同源策略的定义

同源策略只允许 协议，主机，端口相同的站点互相访问 url。比如在 A 站点的脚本中访问 B 站点的接口就是不行的

同源策略是浏览器规定的。浏览器规定不能进行跨域访问。而服务端不会做任何限制，所以在使用 postman 等工具进行接口测试时不会遇到跨域问题



### 如果没有同源策略

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_11-53-09.png" alt="Snipaste_2021-01-30_11-53-09" style="zoom:67%;" />





### <a id="浏览器如何处理跨域请求">浏览器如何处理跨域请求</a>

- 在浏览器中，某个域名下的脚本向服务器 A 发送跨域请求
- 服务器返回响应。浏览器知道此次请求是跨域请求
  - 浏览器会检查响应中是否有 Access-Control-Allow-Origin 头部且允许此域名进行跨域访问。
  - 如果响应中没有允许此站点进行跨域访问的信息，浏览器将不会对响应进行有效处理（浏览器接收了响应，但不让js脚本接收响应中的数据，也不会做保存响应中 Set-Coolkie 中的数据等有效操作），而是在控制台报错



如果同源策略是完全执行的，那站点请求每个其他站点的数据时都需要向其他站点提出允许跨域的申请。安全性大幅提高但可用性降低

为平衡安全性和可用性，同源策略做出以下妥协



- 可用性：HTML 的创作者决定跨域请求是否对本站点安全 
  - `<script>,<img>,<iframe>,<link>,<video>,<audio>`等标签带有的 src 属性可以跨域访问
  - 允许跨域写操作：例如表单提交或者重定向请求（如果连在A站点跳转到B站点都做不到那就太废了）
    - 这会存在CSRF（跨站站请求伪造攻击）安全性问题
- 安全性：浏览器需要防止站点 A 的脚本向站点 B 发起危险动作
  - Cookie、LocalStorage 和 IndexDB 无法读取 
  - DOM 无法获得（防止跨域脚本篡改 DOM 结构） 
  - AJAX 请求不能发送（前端的事，后端不用管）



### 如何防范 CSRF 攻击



CSRF 攻击是指其他站点盗取用户数据后（例如：SessionID，某些Cookie等），由攻击者主动或用户被动使用这些数据发送非法的请求

请求中携带的数据可能就有保存用户信息和权限的数据，这会导致非法的请求被伪造成合法用户发送的请求



有两种方案

- 用 Referer 头部判断发起请求的域名是否是被目标服务器认可的域名
- 服务器用有时限的 token 验证请求（攻击者只有连用户的 token 也盗取后才能伪造合法请求）







### 通过 CORS 实现跨域访问

> CORS：Cross-Origin Resource Sharing  跨域资源共享

如果站点 A 允许站点 B 的脚本访问其资源，必须在 HTTP 响应中显式的告知浏览器：站点 B 是被允许的

- 访问站点 A 的请求，浏览器应告知该请求来自站点 B
- 站点 A 的响应中，应明确哪些跨域请求是被允许的



合法的跨域请求有两种

- 简单请求
- 非简单请求（简单请求以外的其他请求）



#### 简单请求的跨域访问

> 简单请求的定义
>
> - 请求方法只能是 GET/HEAD/POST 方法之一
> - 仅能使用 CORS 安全的头部：Accept、Accept-Language、Content-Language、Content-Type
> - Content-Type 值只能是： text/plain、multipart/form-data、application/x-www-form-urlencoded 三者其中之一



简单请求的跨域访问流程

- 请求中携带 Origin 头部告知来自哪个域 
- 响应中携带 Access-Control-Allow-Origin 头部表示允许哪些域 
- 浏览器检查并处理请求，详情见[这里](#浏览器如何处理跨域请求)
  - 如果响应的 Access-Control-Allow-Origin 头部包含本域，即允许跨域访问。同时，浏览器正常处理响应
  - 如果 Access-Control-Allow-Origin 头部没有包含本域，不允许跨域访问

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_12-23-13.png" alt="Snipaste_2021-01-30_12-23-13" style="zoom:60%;" />



#### 非简单请求的跨域访问

> 非简单请求的跨域访问分两步
>
> 1. 发送预检请求（访问资源前，需要先发起 prefilght 预检请求（方法为 OPTIONS）询问何种请求是被允许的）
> 2. 发送目标请求

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_12-25-37.png" alt="Snipaste_2021-01-30_12-25-37" style="zoom:60%;" />

预检请求头部

- Access-Control-Request-Method   # 在 preflight 预检请求 (OPTIONS) 中，告知服务器接下来的请求会使用哪些方法
- Access-Control-Request-HEADERSs   # 在 preflight 预检请求 (OPTIONS) 中，告知服务器接下来的请求会传递哪些头部

预检请求响应 

- Access-Control-Allow-Methods    # 在 preflight 预检请求的响应中，告知客户端后续请求允许使用的方法
- Access-Control-Allow-HEADERSs     # 在 preflight 预检请求的响应中，告知客户端后续请求允许携带的头部
- Access-Control-Max-Age    # 在 preflight 预检请求的响应中，告知客户端该响应的信息可以缓存多久



此外还有一些其他响应头部

- Access-Control-Expose-HEADERSs  # 告知浏览器哪些响应头部可以供客户端（前端）使用，默认情况下只有 Cache-Control、Content-Language、 Content-Type、Expires、Last-Modified、Pragma 可供使用
- Access-Control-Allow-Origin    # 告知浏览器允许哪些域访问当前资源，*表示允许所有域。为避免缓存错乱，响应中需要携带 Vary: Origin 
- Access-Control-Allow-Credentials   # 告知浏览器是否可以将 Credentials 暴露给客户端（前端）使用，Credentials 包含 cookie、authorization 类头部、 TLS 证书等



## 缓存的控制



### 条件请求

> 之前在断点续传时用到过条件请求。在请求下一部分数据时，用 If-Match 头部发送上次请求返回的响应中的指纹 ETag 。如果目标资源没有被修改过，条件成立
>
> 这里将完整讲解条件请求，为后续检查缓存是否过期的控制做准备



### 条件请求的定义

目的：由客户端携带条件判断信息，而服务器预执行条件验证过程成功后，再返回资源的表述

常见应用场景

- 使缓存的更新更有效率（如 304 响应码使服务器不用传递包体）
- 断点续传时对之前内容的验证
- 当多个客户端并行修改同一资源时，防止某一客户端的更新被错误丢弃（比如：多人协作 Wiki）



### 资源 URL 和资源表述 Representation

资源 R 可被定义为随时间变化的函数 MR(t) 

- 静态资源：创建后任何时刻值都不变，例如指定版本号的库文件
- 动态资源：其值随时间而频繁地变化，例如某新闻站点首页



### 条件验证器的概念

验证器 validator：根据客户端请求中携带的相关头部，以及服务器资源的信息，执行两端的资源验证 

- 强验证器：服务器上的资源表述只要有变动（例如版本更新或者元数据更新），那么以旧的验证头部访问一定会导致验证不过
- 弱验证器：服务器上资源变动时，允许一定程度上仍然可以验证通过（例如一小段时间内仍然允许缓存有效）



### 验证器响应头部

- Etag 响应头部     （表示响应中的资源描述的指纹）
  - ABNF语义描述：ETag = entity-tag
    - entity-tag = [ weak ] opaque-tag
    - weak = %x57.2F     # 解码后  表示  W/
    - opaque-tag = DQUOTE *etagc DQUOTE
    - etagc = %x21 / %x23-7E / obs-text 
  - 例如：强验证器 ETag: "xyzzy" 弱验证器 ETag: W/"xyzzy"
- Last-Modified 响应头部   （表示对应资源表述的上次修改时间）
  - ABNF语义描述：Last-Modified = HTTP-date
  - Date 头部： Date = HTTP-date  表示响应包体生成的时间，Last-Modified 不能晚于 Date 的值



### 条件请求头部

- If-Match = "*" / 1#entity-tag     （如果匹配，向下执行）
- If-None-Match = "*" / 1#entity-tag     （如果不匹配，向下执行）
- If-Modified-Since = HTTP-date      （如果不晚于这个时间，向下执行）
- If-Unmodified-Since = HTTP-date     （如果不早于这个时间，向下执行。和上一个头部正好相反）
- If-Range = entity-tag / HTTP-date   （断点续传中用的。请求范围内的数据，结果源文件被修改，服务器将返回全部文件/数据）



### 条件请求常用场景

- 首次缓存

![Snipaste_2021-01-30_16-14-17](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-14-17.png)

首次缓存，响应携带资源描述的 Etag 指纹和 Last-Modified

- 基于过期缓存发其条件请求

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-19-42.png" alt="Snipaste_2021-01-30_16-19-42" style="zoom:80%;" />

请求中携带指纹，服务端发现资源没有被修改过，返回 304 。告诉浏览器缓存没有过期，浏览器可直接使用缓存，节约了响应中携带包体的流量

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-21-38.png" alt="Snipaste_2021-01-30_16-21-38" style="zoom:80%;" />

服务器发现资源被修改过，将资源放入包体中，生成新的指纹，并返回 200 响应码

- 增量更新

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-24-26.png" alt="Snipaste_2021-01-30_16-24-26" style="zoom: 67%;" />

当服务器支持 Range 服务时，连接意外中断时已接收到部分数据

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-25-28.png" alt="Snipaste_2021-01-30_16-25-28" style="zoom: 67%;" />

客户端使用 Ranges 请求部分数据，服务器成功发送数据后返回 206 表示成功处理部分请求

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-28-07.png" alt="Snipaste_2021-01-30_16-28-07" style="zoom:67%;" />

客户端发送 Ranges 和 指纹，服务器发现指纹和源文件指纹不匹配，说明源文件已经被修改过。返回 412 响应码，提示客户端重头下载源文件。这个过程中客户端发送了 2 次请求

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-29-44.png" alt="Snipaste_2021-01-30_16-29-44" style="zoom:67%;" />

如果使用 If-Range 头部，可以只发送一次请求就完成上一种发送两次请求的方式

- 更新丢失问题：两个客户端更新同一份资源导致更新丢失

更新操作往往都是先 get 一次资源，用户修改部分数据后，再 put 修改资源

问题描述：多个用户更新同一份资源。A，B用户 get 到的数据相同。A先 put 数据，B后 put 数据。结果A再次查询数据后发现数据和自己提交的数据不一致

乐观锁解决

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-38-04.png" alt="Snipaste_2021-01-30_16-38-04" style="zoom: 67%;" />

只允许第一次修改成功。成功后服务器生成新指纹，这会让第二次修改请求和新指纹不匹配而修改失败

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-42-08.png" alt="Snipaste_2021-01-30_16-42-08" style="zoom: 80%;" />

乐观锁也可解决首次上传问题



### Nginx处理条件请求的规则（not_modified 模块）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_16-43-22.png" alt="Snipaste_2021-01-30_16-43-22" style="zoom:80%;" />



### 缓存的工作原理

- 浏览器第一次发送请求：服务器返回响应，客户端对数据进行返回
- 浏览器再次想要请求数据
  - 先查询浏览器的缓存，命中缓存且没有过期，直接使用缓存而不发送请求
  - 缓存没命中，发送请求
  - 命中缓存，但浏览器中的缓存已经过期，发送请求。如果源文件没有被修改过返回 304 并更新浏览器缓存的有效时间。否重新向源服务器发送请求



代理服务器和浏览器的缓存实现图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_22-02-55.png" alt="Snipaste_2021-01-30_22-02-55" style="zoom: 50%;" />

PS：Nginx 用红黑树保存缓存的 K



### 缓存新鲜度的四种计算方式

> 判断缓存是过期通常使用 age头部判断
>
> 响应中含有 age 头部，说明响应中的数据来自代理服务器的缓存而非源服务器
>
> age 头部的值的含义：缓存保存服务器的现在的时间，单位是秒



response_is_fresh = (freshness_lifetime > current_age)       （ freshness_lifetime 是缓存的有效时间，可以是绝对时间或相对时间。current_age 的值接近 Age 头部的值，其计算方式后面讲）

- freshness_lifetime：按优先级，取以下响应头部的值
  - s-maxage > max-age > Expires > 预估过期时间
  - 例如：
    - Cache-Control: s-maxage=3600
    - Cache-Control: max-age=86400
    - Expires: Fri, 03 May 2019 03:15:20 GMT 
      - Expires = HTTP-date，指明缓存的绝对过期时间



预估的过期时间：RFC7234 推荐：（DownloadTime– LastModified)*10%

- DownloadTime  获取到响应的时间
- LastModified  数据上次被修改的时间



**age头部和current_age的计算**

Age 表示自源服务器发出响应（或者验证过期缓存），到使用缓存的响应发出时经过的秒数（字面意思可能有点难理解，但是说的没毛病）

- 对于代理服务器管理的共享缓存，客户端可以根据 Age 头部判断缓存时间
- Age = delta-seconds

current_age 计算：current_age = corrected_initial_age + resident_time;    （修正后的 age + 一小段时间）

- resident_time = now - response_time(接收到响应的时间);

- corrected_initial_age = max(apparent_age, corrected_age_value); 

  - corrected_age_value = age_value + response_delay;      （ age_value 的值取自 Age 头部，如果没就有取0）

    - response_delay = response_time - request_time(发起请求的时间);

  - apparent_age = max(0, response_time - date_value);   （ date_time 的值取自 Date 头部）

    response_time 是本地接收到请求的本地时间，data_value 是服务器发送响应时的服务器的时间



**缓存新鲜度示例**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-30_22-08-36.png" alt="Snipaste_2021-01-30_22-08-36" style="zoom:60%;" />

 

### 复杂的Cache-Control头部

> Cache-Control = 1#cache-directive 
>
> - cache-directive = token [ "=" ( token / quoted-string ) ]
>   - delta-seconds = 1*DIGIT        RFC 规范中的要求是，至少能支持到 2147483648 (2^31) 

Cache-Control 中的 token 的取值可以是以下几种

- 请求中的头部：`max-age`、`max-stale`、`min-fresh`、no-cache、no-store、no-transform、only-if-cached
- 响应中的头部： `max-age`、`s-maxage` 、 must-revalidate 、proxy-revalidate 、no-cache、no-store、no-transform、public、private



- 以上` `中的 token 的等号后面的值通常是秒数，如：`Cache-Control: max-age=12345`
- no-cache，private 后可以跟 =token 也可单独使用



**Cache-Control 头部在请求中的值**

- max-age：告诉服务器，客户端不会接受 Age 超出 max-age 秒的缓存（如果缓存的 age 超过请求中的 max-age ，那么代理/缓存服务器直接请求源服务器获取新数据，而不使用代理/缓存服务器中的缓存）
- max-stale：告诉服务器，即使缓存不再新鲜，但陈旧秒数没有超出 max-stale 时，客户端仍打算使用。若 max-stale 后没有值，则表示无论过期多久客户端都可使用
- min-fresh：告诉服务器，Age 至少经过 min-fresh 秒后缓存才可使用
- no-cache：告诉服务器，不能直接使用已有缓存作为响应返回，除非带着缓存条件到上游服务端得到 304 验证返回码才可使用现有缓存
- no-store：告诉各代理服务器不要对该请求的响应缓存（实际有不少不遵守该规定的代理服务器） 
- no-transform：告诉代理服务器不要修改消息包体的内容
- only-if-cached：告诉服务器仅能返回缓存的响应，否则若没有缓存则返回 504 错误码



**Cache-Control 头部在响应中的值**

- must-revalidate：告诉客户端一旦缓存过期，必须向服务器验证后才可使用
- proxy-revalidate：与 must-revalidate 类似，但它仅对代理服务器的共享缓存有效
- no-cache：告诉客户端不能直接使用缓存的响应，使用前必须在源服务器验证得到 304 返回码。如果 no-cache 后指定头部，则若客户端的后续请求及响应中不含有这些头则可直接使用缓存
- max-age：告诉客户端缓存 Age 超出 max-age 秒后则缓存过期
- s-maxage：与 max-age 相似，但仅针对共享缓存，且优先级高于 max-age 和 Expires
- public：表示无论私有缓存或者共享缓存，皆可将该响应缓存
- private：表示该响应不能被代理服务器作为共享缓存使用。若 private 后指定头部，则在告诉代理服务器不能缓存指定的头部，但可缓存其他部分 
- no-store：告诉所有下游节点不能对响应进行缓存
- no-transform：告诉代理服务器不能修改消息包体的内容



### 什么样的响应才会被缓存

> 1. 什么样的响应会被缓存
> 2. 怎么设置使用缓存的条件（ Vary ）



**什么样的响应会被缓存**

- 请求方法可以被缓存理解（不只于 GET 方法） 
- 响应码可以被缓存理解（404、206 也可以被缓存）
- 响应与请求的头部没有指明 no-store 
- 响应中至少应含有以下头部中的 1 个或者多个：
  - Expires、max-age、s-maxage、public
  - 当响应中没有明确指示过期时间的头部时，如果响应码非常明确，也可以缓存（这是使用预估的缓存时间）
- 如果缓存在代理服务器上：不含有 private，不含有 Authorization



**其他响应头部**

> Pragma = 1#pragma-directive 
>
> - pragma-directive = "no-cache" / extension-pragma
>   - extension-pragma = token [ "=" ( token / quoted-string ) ]
> - Pragma: no-cache 与 Cache-Control: no-cache 意义相同

> Warning = 1#warning-value 
>
> - warning-value = warn-code SP warn-agent SP warn-text [ SP warn-date ]
>   - warn-code = 3DIGIT     （3个十进制数）
>   - warn-agent = ( uri-host [ ":" port ] ) / pseudonym
>   - warn-text = quoted-string
>   - warn-date = DQUOTE HTTP-date DQUOTE 
>
> 常见的 warn-code 
>
> - Warning: 110 - "Response is Stale"
> - Warning: 111 - "Revalidation Failed"
> - Warning: 112 - "Disconnected Operation"
> - Warning: 113 - "Heuristic Expiration"     （如果缓存使用的是预估时间，响应会携带此 Warning 头部的值返回给浏览器）
> - Warning: 199 - "Miscellaneous Warning"
> - Warning: 214 - "Transformation Applied"
> - Warning: 299 - "Miscellaneous Persistent Warning"





**使用缓存作为当前请求响应的条件**

URI 作为主要的缓存关键字，当一个 URI 同时对应多份缓存时，选择日期最近的缓存 

例如 Nginx 中默认的缓存关键字：proxy_cache_key

$scheme$proxy_host$request_uri; 

当缓存中的响应中的条件允许当前请求的方法使用缓存时，直接返回缓存中的响应，而不向源服务器发送请求



**Vary 条件头部**

缓存中的响应 Vary 头部指定的头部必须与请求中的头部相匹配： 

Vary = “*” / 1#field-name 

- Vary: *意味着一定匹配失败

当前请求以及缓存中的响应都不包含 no-cache 头部（Pragma: no-cache 或者 Cache-Control: no-cache）

缓存中的响应必须是以下三者之一：

- 新鲜的（时间上未过期）
- 缓存中的响应头部明确告知可以使用过期的响应（如 Cache-Control: max-stale=60）
- 使用条件请求去服务器端验证请求是否过期，得到 304 响应



Vary 头部的经典使用示例

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-01_12-28-57.png" alt="Snipaste_2021-02-01_12-28-57" style="zoom:80%;" />

1. Client1发送请求（ Accept-Encoding: * 内容协商，客户端接受响应的包体能所有编码/压缩方式进行编码/压缩）
2. Cache 代理/缓存服务器转发请求，接收源服务器的响应。响应返回给 Client1后，Cache 以 URL 为 key ，将响应缓存起来，并给响应添加 Vary头部（作用在第4步中说明）
3. Client2发送请求（ Accept-Encoding: * 内容协商，客户端只接受 br 方式压缩的包体的响应）
4. Cache 用 URL 查询缓存并命中步骤 2 中的缓存，但是此缓存中的响应的 Vary 头部说：如果你能接收此响应中 Content-Encoding 的值我就返回缓存中的响应（ Client2 能否接受 gzip 方式压缩的包体）。但是 Client2 的请求中说我只能接收 br 压缩方式的包体。所以不使用缓存
5. Cache 重新向源服务器发送请求，并做步骤 2 中的内容
6. Client3 发送请求（ Accept-Encoding: * 内容协商，客户端只接受 br 方式压缩的包体的响应）
7. Cache 用 URL 做 key 命中缓存，且请求能满足缓存中响应的 Vary 头部（第 5 步中缓存的响应）



**验证请求与响应**

验证请求 

- 若缓存响应中含有 Last-Modified 头部 
  - If-Unmodified-Since
  - If-Modified-Since
  - If-Range
- 若缓存响应中含有 Etag 头部 
  - If-None-Match
  - If-Match
  - If-Range



### 如何缓存更新频率不同的资源

场景：

- html文件经常被修改，js和css文件不经常被修改
- js和css文件是通过html标签向指定 URL 发起子请求请求到的

问题：使用什么缓存方式缓存html，js，css文件能更高效地使用缓存

解决方式：如下图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-01_12-48-13.png" alt="Snipaste_2021-02-01_12-48-13" style="zoom: 67%;" />![Snipaste_2021-02-01_12-48-40](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-01_12-48-40.png)<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-01_12-48-13.png" alt="Snipaste_2021-02-01_12-48-13" style="zoom: 67%;" />![Snipaste_2021-02-01_12-48-40](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-01_12-48-40.png)



- html 文件设置的缓存时间极端或不允许缓存（不允许缓存最好。百度首页的html文件就不被允许缓存，它使用 Cache-Control: max-age=0）
- js和css文件设置的缓存时间很长，且请求 js和css 的 URL中含有 js和css的版本号



这样一来，如果html被修改或js和css被修改就会发生以下事件

- html被修改

  由于html从未被浏览器缓存过，所以浏览器每次请求html文件都会向源服务器发送请求

- js或css被修改

  如果页面想修改js和css，只需修改html中请求js和css的 URL 中的js和css的版本号即可。一旦html被修改，请求新的html文件后就会请求新的js和css并应用于当前的html

  新的js和css被浏览器缓存起来，旧的js和css等待被LRU算法淘汰掉即可



### 多种重定向跳转方式的差异

举两个重定向的例子

- 访问 xxx.xxx.com/login 点击登录后，响应中携带重定向的状态码和 Location 头部
- 访问 http://www.baidu.com 时，百度只支持 https，响应中返回携带重定向的状态码 307 和 Location 重定向到 https://www.baidu.com



重定向能解决什么问题

- 提交 FORM 表单成功后需要显示内容页，怎么办？
- 站点从 HTTP 迁移到 HTTPS，怎么办？ 
- 站点部分 URI 发生了变化，但搜索引擎或者流量入口站点只收录了老的 URI ，怎么办？
- 站点正在维护中，需要给用户展示不一样的内容，怎么办？
- 站点更换了新域名，怎么办？



#### 重定向流程

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-01_15-59-52.png" alt="Snipaste_2021-02-01_15-59-52" style="zoom:80%;" />

当浏览器接收到重定向响应码时，需要读取响应头部 Location 头部的值，获取到新的 URI 再跳转访问该页面



- 原请求：接收到重定向响应码的请求这里称为原请求
- 重定向请求：浏览器接收到重定向响应码后，会发起新的重定向请求 



> Location = URI-reference（对 201 响应码表示新创建的资源） 
>
> URI-reference = URI/relative-ref 
>
> - relative-ref = relative-part [ "?" query ] [ "#" fragment ]
>   - relative-part = "//" authority path-abempty / path-absolute / path-noscheme / path-empty



#### 重定向响应返回码



重定向可以从两种角度划分分类

- 重定向请求的响应能否被缓存
  - 永久重定向：能被缓存
  - 临时重定向：不能被缓存
- 重定向请求能否重用原请求的方法和包体



永久重定向，表示资源永久性变更到新的 URI 

- 301（HTTP/1.0）：重定向请求**通常**（由于历史原因一些浏览器会把 POST 改为 GET）会使用 GET 方法，而不管原请求究竟采用的是什么方法 
- 308（HTTP/1.1）：重定向请求必须使用原请求的方法和包体发起访问

临时重定向，表示资源只是临时的变更 URI 

- 302 （HTTP/1.0）：重定向请求**通常**会使用 GET 方法，而不管原请求究竟采用的是什么方法
- 303 （HTTP/1.1）：它并不表示资源变迁，而是用新 URI 的响应表述而为原请求服务，重定向请求会使用 GET 方法
  - 例如表单提交后向用户返回新内容（亦可防止重复提交）
- 307 （HTTP/1.1）：重定向请求必须使用原请求的方法和包体发起访问 

特殊重定向 

- 300：响应式内容协商中，告知客户端有多种资源表述，要求客户端选择一种自认为合适的表述     （被动式内容协商时使用的状态码）
- 304：服务器端验证过期缓存有效后，要求客户端使用该缓存



重定向循环：服务器端在生成 Location 重定向 URI 时，在同一条路径上使用了之前的 URI，导致无限循环出现 

重定向循环会导致 Chrome 浏览器会提示：ERR_TOO_MANY_REDIRECTS    （错误，太多次重定向了）



## 如何通过 tunnel 隧道访问被限制的网络

tunnel 隧道通常用于正向代理

connect 方法在 tunnel 隧道中将会得到应用

VPN 也会用到隧道技术

tunnel 隧道的其他知识暂时忽略（课里将的不清楚，这类知识也不好找，不自行搭建正向代理服务器时也不需要用这个技术，所以暂时忽略）



## 爬虫的工作原理与应对方式

> 网络爬虫模拟人类使用浏览器浏览、操作页面的行为，对互联网的站点进行操作
>
> 网络爬虫获取到一个页面后，会分析出页面里的所有 URI，沿着这些 URI 路径递归的遍历所有页面
>
> - DFS（深度优先搜索）
> - BFS（广度优先算法）

爬虫两大类：搜索引擎的爬虫，其他爬虫

站点对待爬虫的态度：欢迎访问和拒绝访问

通常站点会欢迎搜索引擎的爬虫，拒绝其他带有攻击意图或恶意占用站点网络带宽的爬虫



SEO（Search Engine Optimization），搜索引擎优化 

- “合法”的优化：sitemap、title、keywords、https 等
- “非法”的优化：利用 PageRank 算法漏洞 （如果多个站点都引用了A站点的连接，A站点的权重就会变高）

面向搜索引擎的优化会让站点更容易被搜索引擎搜索到，排名更靠前



拒绝爬虫访问的方案：各种形式的验证。滑动滑块的验证，图片验证码，图像识别点击验证码，短信/邮件验证码



### 爬虫常见的请求头部

- User-Agent：识别是哪类爬虫
- From：提供爬虫机器人管理者的邮箱地址 
- Accept：告知服务器爬虫对哪些资源类型感兴趣
- Referer：相当于包含了当前请求的页面 URI



### robots.txt：告知爬虫哪些内容不应爬取

> 爬虫会爬取站点下的  /robots.txt 告诉爬虫哪些内容不应该爬取
>
> Robots exclusion protocol：http://www.robotstxt.org/orig.html
>
> 但是这个协议不是爬虫必须遵守的。谷歌，百度的搜索引擎爬虫会遵循协议



robots.txt 文件内容

- User-agent：允许哪些机器人
- Disallow：禁止访问特定目录
- Allow：抵消 Disallow 指令
- Crawl-delay：访问间隔秒数
- Sitemap：指出站点地图的 URI



## HTTP Basic 基本认证

> RFC7235，一种基本的验证框架，被绝大部分浏览器所支持 
>
> 基本认证明文传输，如果不使用 TLS/SSL 传输则有安全问题

HTTP Basic 基本认证的认证方式根据 Authorization 头部进行认证

HTTP协议提供的 HTTP Basic 认证方式过于简单，通常自定义的认证方式也使用 Authorization 头部，但是认证逻辑由源服务器/认证服务器规定



> 在请求中传递认证信息：Authorization = credentials 
>
> credentials = auth-scheme [ 1*SP ( token68 / `#auth-param` ) ] 
>
> - auth-scheme = token
> - token68 = 1*( ALPHA / DIGIT / "-" / "." / "_" / "~" / "+" / "/" ) *"=“ 
> - auth-param = token BWS "=" BWS ( token / quoted-string ) 
> - BWS = OWS 
>   - OWS = *( SP / HTAB ) 
>
> 例如：authorization：Basic ZGQ6ZWU= 
>
> 默认 authorization 值的 token 使用 base64 编码
>
> 由代理服务器认证：Proxy-Authorization = credentials



> 在响应头部中告知客户端需要认证：WWW-Authenticate = 1#challenge 
>
> - challenge = auth-scheme [ 1*SP ( token68 / `#auth-param` ) ]
>   - auth-scheme = token
>     - token68 = 1*( ALPHA / DIGIT / "-" / "." / "_" / "~" / "+" / "/" ) *"="
>     - auth-param = token BWS "=" BWS ( token / quoted-string )
>     - BWS = OWS
>       - OWS = *( SP / HTAB ) 
>
> 例如：www_authenticate：Basic realm="test auth_basic" 
>
> 由代理服务器认证：Proxy-Authenticate = 1#challenge 
>
> 认证响应码 
>
> - 由源服务器告诉客户端需要传递认证信息：401 Unauthorized 
> - 由代理服务器认证： 407 Proxy Authentication Required 
> - 认证失败：403 Forbidden



## Wireshark 的基本用法

> 这里讲的是使用 Wireshark 抓取本地端口收发的报文
>
> 关于如何使用 Wireshark 抓取服务器上运行的系统程序收发的报文需要自行查询

### 面板功能介绍

- Wireshark面板

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-06-52.png" alt="Snipaste_2021-02-02_12-06-52" style="zoom:50%;" />

- 工具栏

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-08-33.png" alt="Snipaste_2021-02-02_12-08-33" style="zoom:50%;" />

- 数据包的着色规则

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-09-30.png" alt="Snipaste_2021-02-02_12-09-30" style="zoom:50%;" />

- 设定时间显示格式（绝对时间，相对时间，设定基于某一个数据包的相对时间）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-11-42.png" alt="Snipaste_2021-02-02_12-11-42" style="zoom: 67%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-11-10.png" alt="Snipaste_2021-02-02_12-11-10" style="zoom: 45%;" />

- 数据包列表面板的标记符号

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-12-55.png" alt="Snipaste_2021-02-02_12-12-55" style="zoom:67%;" />

- 四种流跟踪（查询某个数据包的上层协议数据包，如想查询某次 HTTP 数据包的 TCP 报文）

  TCP ，UDP ，SSL ，HTTP

- 文件操作

  - 标记报文 Ctrl+M 
  - 导出标记报文（文件->导出特定分组），亦可按过滤器导出报文
  - 合并读入多个报文（文件->合并）

- 抓取移动设备的报文

  1. 在操作系统上打开 wifi 热点
  2. 手机连接 wifi 热点
  3. 用 Wireshark 打开捕获->选项面板，选择 wifi 热点对应的接口设备抓包

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-19-49.png" alt="Snipaste_2021-02-02_12-19-49" style="zoom:50%;" />

- DNS报文和抓包实战，dig 工具查询（ DNS 协议是基于 UDP 实现的，想抓 DNS 的数据包就使用过滤器只保留 UDP 协议的数据包）
- 捕获过滤器和显示过滤器

![Snipaste_2021-02-02_12-36-36](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-36-36.png)![Snipaste_2021-02-02_12-37-08](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-37-08.png)





wireshark的捕获过滤器使用的语法也广泛用于其他抓包工具

但是wireshark的显示过滤器使用的语法是独有的，其他抓包工具都不能使用



### <a id="BPF 捕获滤器">BPF 捕获滤器</a>

> Berkeley Packet Filter，在设备驱动级别提供抓包过滤接口，多数抓包工具都支持此语法（比如 tcpdump 就支持此语法）
>
> expression 表达式：由多个原语组成



#### Expression 表达式

primitives 原语：由名称或数字，以及描述它的多个限定词组成 

- qualifiers 限定词
  - Type：设置数字或者名称所指示类型，例如 host www.baidu.com
  - Dir：设置网络出入方向，例如 dst port 80  （限定词可连用）
  - Proto：指定协议类型，例如 udp
  - 其他 
- 原语运算符
  - 与：&& 或者 and
  - 或：|| 或者 or
  - 非：! 或者 not
- 例如：src or dst portrange 6000-8000 && tcp or ip6



#### 限定词

Type：设置数字或者名称所指示类型 

- host、port
- net ，设定子网，net 192.168.0.0 mask 255.255.255.0 等价于 net 192.168.0.0/24
- portrange，设置端口范围，例如 portrange 6000-8000

Dir：设置网络出入方向 

- src、dst、src or dst、src and dst
- ra、ta、addr1、addr2、addr3、addr4（仅对 IEEE 802.11 Wireless LAN 有效）这些是链路层的限定词，了解即可

Proto：指定协议类型（捕获过滤器只支持部分基础协议，像是 websocket，http，dns 协议统统不支持。显示过滤器支持）

- ether、fddi、tr、 wlan、 ip、 ip6、 arp、 rarp、 decnet、 tcp、udp、icmp、igmp、icmp、igrp、pim、ah、esp、vrrp

其他限定词

- gateway：指明网关 IP 地址，等价于 ether host ehost and not host host
- broadcast：广播报文，例如 ether broadcast 或者 ip broadcast
- multicast：多播报文，例如 ip multicast 或者 ip6 multicast
- less, greater：小于或者大于



#### 基于协议域过滤 

- 捕获所有 TCP 中的 RST 报文
  - tcp[13]&4==4    （取 TCP 报文中 RST 标志位的值的方法：取第 14 个字节—从0开始计数所以下标是 13 ，与上 0100 就能得到第3位的 bit 值）
- 抓取 HTTP GET 报文
  - port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420
  - 注意：47455420 是 ASCII 码的 16 进制，表示”GET ” 

PS：TCP 报头可能不只 20 字节，data offset 提示了承载数据的偏移，但它以 4 字节为单位

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_17-01-13.png" alt="Snipaste_2021-02-02_17-01-13" style="zoom:80%;" />



### 显示过滤器

- 显示过滤器使用过滤属性作为过滤条件表达式
- 任何在报文细节面板中解析出的字段名，都可以作为过滤属性 
  - 在视图->内部->支持的协议面板里，可以看到各字段名对应的属性名
  - 例如，在报文细节面板中 TCP 协议头中的 Source Port，对应着过滤属性为 tcp.srcport

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_17-06-09.png" alt="Snipaste_2021-02-02_17-06-09" style="zoom:80%;" />



#### 过滤值的比较符号

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_17-07-17.png" alt="Snipaste_2021-02-02_17-07-17" style="zoom:50%;" />

- 使用英文缩写或符号均可
- 可以用正则表达式



#### 过滤值类型

- Unsigned integer：无符号整型，例如 ip.len le 1500
- Signed integer：有符号整型
- Boolean：布尔值，例如 tcp.flags.syn。对于值为布尔类型的属性，使用时可单独使用，而不用使用 k==v 的形式
- Ethernet address：以:、-或者.分隔的 6 字节地址，例如 eth.dst == ff:ff:ff:ff:ff:ff
- IPv4 address：例如 ip.addr == 192.168.0.1
- IPv6 address：例如 ipv6.addr == ::1
- Text string：例如 http.request.uri == "https://www.wireshark.org/"



#### 多个表达式间的组合

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_17-08-45.png" alt="Snipaste_2021-02-02_17-08-45" style="zoom:50%;" />

集合操作符`{}`

例如 tcp.port in {443 4430..4434} ，实际等价于 tcp.port == 443 || (tcp.port >= 4430 && tcp.port ⇐ 4434) 

切片操作符`[]`  （数字的单位为字节）

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
  - eth.src[0:3,1-2,:4,4:,2] ==00:00:83:00:83:00:00:83:00:20:20:83



#### 可用函数

- upper       Converts a string field to uppercase.
- lower        Converts a string field to lowercase.
- len            Returns the byte length of a string or bytes field.
- count       Returns the number of field occurrences in a frame.
- string       Converts a non-string field to a string.



#### 显示过滤器表达式可视化面板

分析 —> Display Filter Expression

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_17-17-01.png" alt="Snipaste_2021-02-02_17-17-01" style="zoom:80%;" />



### 追踪流



## DNS 域名查询服务

DNS 域名结构：根域名服务器保存能解析`.com`，`.cn`，`.xx`的下游域名服务器

下游域名服务器保存能解析`.xxx.com`，`.xxx.cn`等域名的服务器，以此递归直到遇到能将域名完整解析为 IP 地址的服务器



DNS 的递归查询流程

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-25-34.png" alt="Snipaste_2021-02-02_12-25-34" style="zoom:67%;" />

上述递归查询中，发送的 DNS 查询报文都是 DNS 格式的查询请求和结果响应

- query：查询域名
- response：返回 IP 地址

使用`dig domain`也可获取 DNS 查询轨迹



**DSN 报文格式**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-30-50.png" alt="Snipaste_2021-02-02_12-30-50" style="zoom:67%;" />

请求报文中没有 Answer RRs ， Authority RRs ，Additional RRs，响应报文中包含上述所有数据

- QDCOUNT 表示 Questions 的数量
- ANCOUNT 表示 Answer RRs 的数量
- ...



- Questions 的格式

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-33-46.png" alt="Snipaste_2021-02-02_12-33-46" style="zoom: 50%;" />

- Answer 的格式

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-02_12-34-42.png" alt="Snipaste_2021-02-02_12-34-42" style="zoom:50%;" />



# 第 2 部分：Websocket



## Websocket协议的功能

- HTTP/1.1 request-response 模式下，支持单向通信。消息只能由客户端主动发送请求，服务器被动返回响应
- Websocket 下，支持双向通信。客户端和服务器建立 WS 连接后，服务器能主动向客户端发送数据。效率要比客户端轮询/定时查询的效果
  - 只能通过 HTTP/1.1 升级协议的方式建立 Websocket 连接
  - 端口复用，兼容 HTTP 协议。ws 协议默认使用 80 端口，wss （ws +SSL）协议默认使用 443。80 端口和 443 端口也是 HTTP 和 HTTPS 的默认端口
  - 优点：双向通信；缺点：可扩展性差（稍后再说）
  - 会话管理方式：ws 会话由客户端和服务器双向控制，关闭 ws 会话需要两边都发起关闭会话
  - 长连接维护方式：客户端和服务器通过 ping/pong 心跳保持长连接
  - 支持扩展：比如使用插件让 ws 支持压缩数据等



## Websocket的约束

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-03_15-22-53.png" alt="Snipaste_2021-02-03_15-22-53" style="zoom:60%;" />



HTTP 请求数量激增时，可通过搭建集群，将请求分摊到集群中的节点上。能这样做的原因是：HTTP 协议是无状态的协议。在不使用 Session 的前提下，一个请求可以让集群中任意一个节点处理

WS 协议是基于 TCP 连接的协议，客户端需要和单台服务器建立 WS 连接后才能双向通信。服务器使用集群后，客户端建立的 WS 连接意味着同一个 WS 连接下，客户端只能向一台服务器发送数据

为了实现 WS 连接下客户端也能和集群中任意节点进行双向通信，需要设计复杂的系统（牺牲了简单性）

上图中处理 WS 的集群中，WS 请求发送到消息分发系统前的门面服务器时，门面服务器先将消息转换为简单的格式后发送给消息分发系统，消息分发系统将消息传递给源服务器进行处理

消息分发系统可以是：Kafka，RabbitMQ，RocketMQ，Redis 等



## Websocket 的协议格式



Websocket 协议的实际思想是：在 Web 约束下暴露给 TCP 层

- Websocket 要传递的元数据由应用层存放
- Websocket 的传输单位是 frame（帧），一条完整的 message 由若干个 frame（帧）组成。而不是基于流（HTTP、TCP） 
- 浏览器的同源策略对 Websocket 协议同样生效



### Websocket 的帧格式

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-03_15-38-22.png" alt="Snipaste_2021-02-03_15-38-22" style="zoom:60%;" />

PS：每行 32 位 = 4 字节

- 前两个字节的数据必须有
- RSV1/RSV2/RSV3：默认为 0，仅当使用 extension 扩展时，由扩展决定其值
- opcode 决定了帧的类型。比如，帧是 ping/pong 心跳帧、数据帧（文本帧或二进帧）、关闭帧（关闭 WS 连接时需要发送的帧）
  - 持续帧
    - 0：继续前一帧（表示此帧的类型和前一个 WS 协议的帧的类型一样）
  - 非控制帧
    - 1：文本帧（UTF8）
    - 2：二进制帧
    - 3-7：为非控制帧保留
  - 控制帧
    - 8：关闭帧
    - 9：心跳帧 ping
    - A：心跳帧 pong
    - B-F：为控制帧保留
- FIN 位：用于控制 一条 message 的结束，同时隐含下一个 frame 是新 message 的开始。如当前帧的类型不是控制帧，且 FIN 位为 1 ，表示此 message 的结束



### ABNF 语义描述 WS 的帧格式

上图中，有很多数据都是可选的。为清楚描述帧的格式，下面用 ABNF 语义描述的 WS 的帧的格式

> ws-frame = frame-fin ; 1 bit in length        （表示 frame-fin 占用 1 bit/位）
>
> ​					frame-rsv1 ; 1 bit in length 
>
> ​					frame-rsv2 ; 1 bit in length 
>
> ​					frame-rsv3 ; 1 bit in length 
>
> ​					frame-opcode ; 4 bits in length 
>
> ​					frame-masked ; 1 bit in length 
>
> ​					frame-payload-length ; 3 种长度 
>
> ​					[ frame-masking-key ] ; 32 bits in length 
>
> ​					frame-payload-data ; n*8 bits in ; length, where ; n >= 0



### WS 的 URL 格式

> ws-URI = "ws:" "//" host [ ":" port ] path [ "?" query ]
>
> - 默认 port 端口 80 
>
> wss-URI = "wss:" "//" host [ ":" port ] path [ "?" query ]
>
> - 默认 port 端口 443

此外客户端还要发送

 客户端提供信息 

必选

- 握手随机数：Sec-WebSocket-Key

可选

- 选择子协议： Sec-WebSocket-Protocol 
- 扩展协议： Sec-WebSocket-Extensions 
- CORS 跨域：Origin





## HTTP 升级到的 Websocket 的过程



### 发送 HTTP 请求升级协议的过程

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-03_15-55-39.png" alt="Snipaste_2021-02-03_15-55-39" style="zoom: 67%;" />

- 红色头部：必选头部
- 绿色头部：Sec-WebSocket-Key 头部是携带随机数的头部，随机数被 SHA1 和 BASE 64 编码过
- 蓝色头部：跨域相关头部
- 黑色头部：可选头部，比如 Sec-WebSocket-Extensions



### 如何证明握手被服务器接受？预防意外 

请求中 Sec-WebSocket-Key 携带随机数 

- 例如 Sec-WebSocket-Key: A1EEou7Nnq6+BBZoAZqWlg== 

响应中 Sec-WebSocket-Accept 携带随机数的证明值

- 证明值的构造规则：BASE64(SHA1(Sec-WebSocket-KeyGUID))    （随机数+指定字符串后 先进行 SHA1 编码再进行 BASE 64 编码）
  - GUID（RFC4122）：258EAFA5-E914-47DA-95CA-C5AB0DC85B11
  - 拼接值：A1EEou7Nnq6+BBZoAZqWlg==258EAFA5-E914-47DA-95CA-C5AB0DC85B11
  - SHA1 值：713f15ece2218612fcadb1598281a35380d1790f
  - BASE 64 值：cT8V7OIhhhL8rbFZgoGjU4DReQ8=
- 最终头部：Sec-WebSocket-Accept: cT8V7OIhhhL8rbFZgoGjU4DReQ8=



## Websocket 数据帧格式

这里关注的是数据帧中 指定数据长度的 len 和帧中的数据 data

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-03_16-04-26.png" alt="Snipaste_2021-02-03_16-04-26" style="zoom: 67%;" />



使用 WS 连接发送消息时

- 确保 WebSocket 会话处于 OPEN 状态
- 以帧来承载消息，一条消息可以拆分多个数据帧 
- 客户端发送的帧必须基于掩码编码 
- 一旦发送或者接收到关闭帧，连接处于 CLOSING 状态
- 一旦发送了关闭帧，且接收到关闭帧，连接处于 CLOSED 状态
- TCP 连接关闭后，WebSocket 连接才完全被关闭



## 掩码：防御代理缓存污染攻击

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-03_16-27-26.png" alt="Snipaste_2021-02-03_16-27-26" style="zoom:60%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-03_16-27-41.png" alt="Snipaste_2021-02-03_16-27-41" style="zoom:60%;" />

目的：防止恶意页面上的代码，可以经由**浏览器**构造出合法的 GET 请求，使得代理服务器可以识别出请求并缓存响应 

强制浏览器执行以下方法：

- 生成随机的 32 位 frame-masking-key，不能让 JS 代码猜出（否则可以反向构造）
- 对传输的包体按照 frame-masking-key 执行可对称解密的 XOR 异或操作，使代理服务器不识别
- 消息编码算法：
  - j = i MOD 4
  - transformed-octet-i = original-octet-i XOR masking-key-octet-j





## Websocket 保持会话心跳和关闭会话



心跳帧（可以插在数据帧中传输 ）

- ping 帧  （opcode=9，可以含有数据）
- pong 帧   （opcode=A，必须与 ping 帧数据相同）

关闭会话的方式 

- 控制帧中的关闭帧：在 TCP 连接之上的双向关闭
  - 发送关闭帧后，不能再发送任何数据
  - 接收到关闭帧后，不再接收任何到达的数据
  - opcode=8，可以含有数据，但仅用于解释关闭会话的原因（前 2 字节为无符号整型，遵循 mask 掩码规则）
- TCP 连接意外中断（因为 WS 连 是建立在 TCP 连接上的，如果 TCP 连接断掉，WS连接也就断了）



**关闭帧的错误码**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-03_16-32-12.png" alt="Snipaste_2021-02-03_16-32-12" style="zoom:60%;" />



# 第 3 部分：HTTP/2.0



## HTTP/1.1 发展中遇到的问题

**需求**

- 请求携带的数据量变大
- 一个页面上的请求数量变多每个页面小于 10 个资源，到每页面 100 多个资源
- 请求的数据类型多样化。从文本为主的内容，到富媒体（如图片、声音、视频）为主的内容
- 用户对低延迟的要求越来越苛刻。对页面内容实时性高要求的应用越来越多



**问题**

- 随着网络带宽的增加（光纤入户等），延迟并没有显著下降
  - 浏览器发起的并发连接有限
  - HTTP/1.1 要求同一连接同时只能在完成一个 HTTP 事务（请求/响应）才能处理下一个事务
- REST 架构下要求 HTTP/1.1 能做到无状态
  - 导致每个请求都必须尽可能携带完整的头部（各种可能用到的头部都必须有）。据统计，多个请求间会携带大量重复的头部
  - 每次请求都要将域名下的所有 Cookie 放到请求头中。有时 Cookie 的大小会很大
- HTTP/1.1 不支持双向传递消息。只能将协议升级为 Websocket 才能实现双向传递消息



**HTTP/1.1 使用的解决方式**

使用各种拼接合并方案将多张小图片或多个 css 文件或多个小 js 文件合并到一次响应中减少请求次数

将资源分布到不同域名下。以便减缓浏览器对同一个域名只能建立有限个连接的限制





## HTTP/2.0 特性概述

### HTTP/2 对 HTTP/1.1 的适配

HTTP2（RFC7540，2015.5）

- 在应用层上修改，基于并充分挖掘 TCP 协议性能
- 客户端向 server 发送 request ，server 向客户端返回 response 这种基本模型不会变
- 老的 scheme 不会变，没有 http2://
- 使用 http/1.x 的客户端和服务器可以无缝的通过代理方式转接到 http/2 上
- 不识别 http/2 的代理服务器可以将请求降级到 http/1.x



### HTTP/2 的主要特性

- 传输数据量的大幅减少
  - 以二进制方式传输（HTTP/1.1 以 ASC 方式传输）
  - 标头压缩
- 多路复用及相关功能
  - 消息优先级
- 服务器消息推送
  - 并行推送（HTTP/2 充分利用带宽的扩大，使用长肥管道并行发送请求，并行接收响应。而不是 HTTP/1.1 一样，同一个连接只能串行发送请求并等待响应返回后再发送其他请求）



[在这里](https://http2.akamai.com/demo)体验 HTTP/1.1 和 HTTP/2 的响应差别。网站主要通过将大图片被分割成多个图片，使用 HTTP/1.1 和 HTTP/2 请求这些小图片的响应速度反映两者的区别



## Wireshark 解密 TLS/SSL 报文

> HTTP/2 可以运行在 TCP 协议上，也可以运行在 TLS/SSL 协议上



**HTTP/2 应用协议下的 TLS 层**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-06_17-44-53.png" alt="Snipaste_2021-02-06_17-44-53" style="zoom:80%;" />



### TLS1.2 的加密算法

- 常见<a id="加密套件">加密套件</a>

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-06_17-45-54.png" alt="Snipaste_2021-02-06_17-45-54" style="zoom:80%;" />

- 对称加密算法：AES_128_GCM

  - 每次建立连接后，加密密钥都不一样

- 密钥生成算法：ECDHE

  - 客户端与服务器通过交换部分信息，各自独立生成最终一致的密钥



### 使用 Wireshark 解密 TLS 消息

原理：获得 TLS 握手阶段生成的密钥 

- 通过 Chrome 浏览器 DEBUG 日志中的握手信息生成密钥 

步骤

- 配置 Chrome 输出 DEBUG 日志
  - 配置环境变量 SSLKEYLOGFILE
- 在 Wireshark 中配置解析 DEBUG 日志
  - 编辑->首选项->Protocols->TLS/SSL
  - (Pre)-Master-Secret

抓包结果如下。成功抓取被解密过的 HTTP/2 的数据包

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_15-16-53.png" alt="Snipaste_2021-02-07_15-16-53" style="zoom:80%;" />

如果没有为 Wireshark 指定密钥文件日志，Wireshark 抓取的报文的协议将都是 TSL 协议（被加密过的数据包），如下图

![Snipaste_2021-02-07_15-15-57](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_15-15-57.png)



PS：使用 Wireshark 解密 TLSSSL 报文时需要注意。应该先打开 Wireshark 再开始访问网页。如果先访问网页再打开 Wireshark 时，Wireshark 抓取的报文不是从 HTTP/2 长连接的第一条报文开始抓取的，Wireshark 无法正确读取 chrome 的日志文件中的密钥



### 二进制格式与可见性

浏览器默认要求使用 HTTP/2 时必须建立在 SSL 协议上。也就是说使用 HTTP/2 协议的请求都会被加密

TLS/SSL 降低了可见性门槛 

- 代理服务器没有私钥不能看到内容

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-06_18-13-34.png" alt="Snipaste_2021-02-06_18-13-34" style="zoom:50%;" />



## 基于 TCP 或 TLS 协议的 HTTP/2 协议

> [HTTP/2笔记之连接建立](http://www.blogjava.net/yongboy/archive/2015/03/18/423570.html)

### 浏览器对 HTTP/2 的支持

- IETF 标准不要求必须基于 TLS/SSL 协议
- 浏览器要求必须基于 TLS/SSL 协议
- <a id="alpn">在 TLS 层 ALPN (Application Layer Protocol Negotiation)扩展做协商，只认 HTTP/1.x 的代理服务器不会干扰 HTTP/2</a>
- shema：http://和 https:// 默认基于 80 和 443 端口
- h2：基于 TLS 协议运行的 HTTP/2 被称为 h2 
- h2c：直接在 TCP 协议之上运行的 HTTP/2 被称为 h2c



### 协议升级方式

- 协议升级分两个阶段。协议升级阶段，统一建立连接阶段
- 先建立 TCP 连接，再做协议升级

协议升级阶段：左侧为基于 TLS 协议的 HTTP/2 的升级方式，右侧为之间建立在 TCP 协议上运行的 HTTP/2 的升级方式

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-06_18-24-41.png" alt="Snipaste_2021-02-06_18-24-41" style="zoom: 67%;" />

统一建立连接阶段：执行完上述建立连接的流程后，再执行统一建立连接过程。统一建立连接过程后再说



### 在 TCP 上从 HTTP1 升级到 HTTP/2

浏览器强制使用 TLS 协议建立 HTTP/2 ，所以无法使用浏览器搭建基于 TCP 协议的 HTTP/2

解决方案：curl 7.46.0 以上的版本支持基于 TCP 协议的 HTTP/2

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-06_21-56-06.png" alt="Snipaste_2021-02-06_21-56-06" style="zoom: 67%;" />

协议升级流程&不使用 TLS 协议进行协议升级

#### 协议升级阶段

- 客户端发送 HTTP/1.1 请求，使用 Connection ， Upgrade 头部实现升级
- 服务器返回 HTTP/1.1 响应，使用状态码，Connection，Upgrade
- 执行完上述流程后，执行下main的统一建立连接过程

#### <a id="统一建立连接阶段">统一建立连接阶段</a>

不管使用 TCP 还是TLS ，建立连接后都需要执行以下流程

- 客户端发送 Magic 帧
  - Preface（ASCII 编码，12字节）
  - 何时发送？
    - 接收到服务器发送来的 101 Switching Protocols
    - TLS 握手成功后
  - Preface 内容
    - 0x505249202a20485454502f322e300d0a0d0a534d0d0a0d0a
    - PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n
- 客户端和服务器互相发送设置帧（SETTINGS Freme）和设置应答帧（SETTINGS Frame with ACK flag）
  - 客户端发送设置帧，服务器发送设置应答帧
  - 服务器发送设置帧，客户端发送设置应答帧

h2c 抓包示例图

![Snipaste_2021-02-07_15-32-26](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_15-32-26.png)



### <a id="SSL 和 TLS">SSL 和 TLS</a>

> 下面以 HTTP/1.1 下使用 TLS1.2 为例介绍 SSL 和TLS

如果只是用 HTTP 不使用 SSL，报文内容将能被抓取，且能被阅读，甚至篡改。SSL 的目的是对报文进行加密 或者说 是对通信进行加密



> 下面将涉及
>
> - HTTPS 是什么
> - 讲解 SSL 前，先讲解相互交换密钥的加密技术
>   - 共享密钥加密
>   - 公开密钥加密
>   - 混合加密方式
> - 证明公开密钥正确性的证书
> - HTTPS 的安全通信机制



#### HTTPS 是什么

- HTTPS = HTTP + SSL（TSL是以 SSL为原型开发的协议，有时会统一称该协议 为 SSL）

- HTTPS 并非是应用层的一种新协议。意思是HTTP 通信接口部分用 SSL（Secure Socket Layer）和 TLS（Transport Layer Security）协议

- 通常，HTTP 直接和 TCP 通信。当使用 SSL时，则演变成先和 SSL通信，再由 SSL和 TCP 通信了。简言之，所谓 HTTPS，其实就是身披 SSL 协议这层外壳的 HTTP

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_18-01-44.png" alt="Snipaste_2021-02-07_18-01-44" style="zoom:50%;" />
  
  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-12_14-20-22.png" alt="Snipaste_2021-02-12_14-20-22" style="zoom:80%;" />



#### 相互交换密钥的加密技术

- 共享密钥加密。也称对称密钥加密

  - 加密和解密使用同一个密钥
  - 通常发送方发送加密后的数据时需要将密钥一并发过去（因为接收端持有密钥才能解密发送过来的数据）
  - 因为需要安全地将密钥发送给客户端，所以安全性不高（通信不安全的前提下，如果能做到安全传送密钥，也就能安全传送数据，这是矛盾的）

- 公开密钥加密。也称非对称密钥加密

  - 发送方持有公钥，接收方持有私钥。即可做到通信中的数据不会被他人解密
  - 一对公私钥能做到客户端发送数据服务器接收数据是安全的。但是不能做到服务器发送数据客户端接收数据是安全的。即安全的单向通信
  - 需要两对密钥才能做到安全的双向通信

- 混合加密方式

  - 交换密钥环节使用公开密钥     加密方式，之后的建立通信交换报文阶段则使用共享密钥加密方式

    <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_18-16-45.png" alt="Snipaste_2021-02-07_18-16-45" style="zoom:50%;" />

PS：如果密钥被攻击者获得，那加密也就失去了意义



#### 证明公开密钥正确性的证书

在交换密钥阶段，服务器向客户端传送公开密钥时，公开密钥可能会被篡改（因为公开密钥就是公开的，所以无需担心被盗取，但担心被篡改）

建立非对称加密通信的流程：

1. 服务器开发者把自己的公开密钥登录至数字证书认证机构
2. 数字证书认证机构用自己的私有密钥（数字认证机构的私有密钥）对服务器开发者发开的公开密钥签署数字签名并颁发公钥证书
  - 公钥证书 = 公钥 + 数字签名
  - 数字认证机构的公开密钥已事先植入到浏览器里了（目的是能安全向 CA 发送验证请求，验证服务器发来的公钥是否合法）
3. 服务器和客户端交互的起始阶段：客户端拿到服务器的公钥证书后，使用数字证书认证机构的公开密钥，向数字证书认证机构验证公钥证书上的数字签名，已确认服务器的公开密钥的真实性
4. 客户端使用服务器的公开密钥对报文进行加密后发送报文
5. 服务器用私有密钥对报文解密



#### <a id="HTTPS 安全通信连接的建立过程">HTTPS 安全通信连接的建立过程</a>

之前介绍了使用互换密钥建立安全通信的简单流程，下面讲立安全通信的实际具体流程（基于 TLS 1.2）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_18-35-39.png" alt="Snipaste_2021-02-07_18-35-39" style="zoom:80%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_20-37-20.png" alt="Snipaste_2021-02-18_20-37-20" style="zoom: 75%;" />

**补充：**步骤 3 和步骤 4 之间应该还要加以一个 Handshark : Server Key Exchange。右图中 2~5 通常是使用一个 RTT 发送的



步骤 **1**： 客户端通过发送<a id="clienthello"> Client Hello 报文</a>开始 SSL 通信。报文中包含客户端支持的 SSL 的指定版本、加密组件（Cipher Suite）列表（所使用的加密算法及密钥长度等）

步骤 **2**： 服务器可进行 SSL 通信时，会以 Server Hello 报文作为应答。和客户端一样，在报文中包含 SSL版本以及加密组件。服务器的加密组件内容是从接收到的客户端加密组件内筛选出来的

步骤 **3**： 之后服务器发送 Certificate 报文。报文中包含公开密钥证书

步骤 **4**： 最后服务器发送<a id="serverhello"> Server Hello Done 报文</a>通知客户端，最初阶段的 SSL 握手协商部分结束

步骤 **5**： SSL 第一次握手结束之后，客户端以 Client Key Exchange 报文作为回应。报文中包含通信加密中使用的一种被称为 Pre-master secret 的随机密码串。该报文已用步骤 3 中的公开密钥进行加密

步骤 **6**： 接着客户端继续发送 Change Cipher Spec 报文。该报文会提示服务器，在此报文之后的通信会采用 Pre-master secret 密钥加密

步骤 **7**： 客户端发送 Finished 报文。该报文包含连接至今全部报文的整体校验值。这次握手协商是否能够成功，要以服务器是否能够正确解密该报文作为判定标准（因为此次通信以及以后通信都使用 Pre-master secret 密钥加密）

步骤 **8**： 服务器同样发送 Change Cipher Spec 报文

步骤 **9**： 服务器同样发送 Finished 报文

步骤 **10**： 服务器和客户端的 Finished 报文交换完毕之后，SSL 连接就算建立完成。当然，通信会受到 SSL 的保护。从此处开始进行应用层协议的通信，即发送 HTTP 请求

步骤 **11**： 应用层协议通信，即发送 HTTP 响应

步骤 **12**： 最后由客户端断开连接。断开连接时，发送 close_notify 报文。上图做了一些省略，这步之后再发送 TCP FIN 报文来关闭与 TCP 的通信



下面是对整个流程的图解。图中说明了从**仅使用服务器端的公开密钥证书**（服务器证书）建立 HTTPS 通信的整个过程

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_19-26-48.png" alt="Snipaste_2021-02-07_19-26-48" style="zoom:67%;" />

CBC 模式（Cipher Block Chaining）又名密码分组链接模式。在此模式下，将前一个明文块加密处理后和下一个明文块做 XOR 运算，使之重叠，然后再对运算结果做加密处理。对第一个明文块做加密时，要么使用前一段密文的最后一块，要么利用外部生成的初始向量



### 在 TLS 上从 HTTP1 升级到 HTTP/2

- 在 TCP 上从 HTTP1 升级到 HTTP/2 使用的方法是 HTTP Upgrade
- 在 SSL 上从 HTTP1 升级到 HTTP/2 使用的方式是 **ALPN 扩展**（application_layer_protocol_negotiation extension，译：应用层协议协商 扩展），这是 RFC7301 规定的



#### 协议升级流程

[ALPN 是 TLS 层的扩展，所以 ALPN 执行协议协商的过程是在建立 SSL 时一并执行的](#alpn)

关于“扩展”，在第 4 部分再说

- 客户端（浏览器）发送 [Client Hello 报文](#clienthello)时，报文携带展示客户端支持哪些协议的数据（下图中展示，客户端支持 http/1.1 和 h2）

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_20-27-17.png" alt="Snipaste_2021-02-07_20-27-17" style="zoom:50%;" />

- 服务器接收上述报文，从客户端支持的协议中选一个并放到 [Server Hello 报文](#serverhello)响应中发给客户端（下图中展示，服务器选择使用 h2 协议，即基于 SSL 的 HTTP/2）

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_20-32-49.png" alt="Snipaste_2021-02-07_20-32-49" style="zoom: 50%;" />

#### 统一建立连接阶段

基于 TLS 的 HTTP/2 的统一建立连接阶段和直接基于 TCP 的 HTTP/2 的统一连接阶段是一样的

所以看[直接基于 TCP 的 HTTP/2 的统一连接阶段](#统一建立连接阶段)即可



## 帧，消息，流

HTTP/1.1 的 request 和 response 中 ABNF 描述的 HTTP 消息格式为 

`HTTP-message = start-line *(HEADERS-field CRLF) CRLF [message-body]`

在 HTTP/2 中为支持多路复用，除了 message 的概念外又引入了 stream，而实际承载数据的是 frame



### 帧，消息，流的关系

- 连接 Connection：1个 TCP 连接，包含一个或者多个 Stream
- 数据流 Stream：一个双向通讯数据流，包含 1 条或者多条 Message
- 消息 Message：对应 HTTP/1 中的一条请求或者响应，包含一条或者多条 Frame
- 数据帧 Frame：最小单位，以二进制压缩格式存放 HTTP/1 中的内容

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_21-33-36.png" alt="Snipaste_2021-02-07_21-33-36" style="zoom: 67%;" />

PS：浏览器中，对一个站点只建立一个 TCP 连接；一个连接上有多个 Stream 是 HTTP/2 实现多路复用的关键



#### 帧和流的关系

- 每一个 Frame 都有一个 Stream ID 表示自己属于哪一个 Stream
- 但是 Frame 没有标识位表示自己属于哪一个 Message

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_21-36-05.png" alt="Snipaste_2021-02-07_21-36-05" style="zoom: 50%;" />



#### 消息的组成

- 每条 Message （请求或响应）中有多个 Frame ，由 1 个 HEADERS（可能含有 0 个或者多个持续帧构成） 及 0 个或者多个 DATA 帧构成
- Message 在帧格式中比较难找到 HEADERSS 帧和 DATA 帧的对应关系，但在应用层就能比较明确地联系起来
- HEADERS 消息同时包含 HTTP/1.1 中的 start line 与 HEADERSs 部分
- 取消 HTTP/1.1 中的不定长 Chunk 消息

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_21-38-24.png" alt="Snipaste_2021-02-07_21-38-24" style="zoom: 50%;" />

下面一个抓包实例图（Wireshark 抓包抓的 HTTP2 报文，每一条都是 HTTP/2 中一个 Frame）

![Snipaste_2021-02-07_21-45-17](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_21-45-17.png)



#### Frame 传输中无序，接收时组装

- 同一条消息的多个 Frame 在同一个 Stream 中必须有序，但可以不连续
  - 比如：必须先传 HEADERSS 帧再传 DATA 帧
  - 比如：Stream1 中的多个 Frame 可以不连续传递。在 Stream1 传一部分数据后，传一部分 Stream3 的帧再继续传 Stream1 的帧
- 如果一条消息的 Frame 都是连续传递的，那就和 HTTP/1.1 的串行传递一样了

![Snipaste_2021-02-07_21-48-11](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_21-48-11.png)



#### 消息与帧

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-07_21-53-38.png" alt="Snipaste_2021-02-07_21-53-38" style="zoom:50%;" />



### 帧格式



#### 标准帧头部

9 字节标准帧头部 + Fram Payload（帧负载的不定长的数据）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_12-23-32.png" alt="Snipaste_2021-02-08_12-23-32" style="zoom:50%;" />

![Snipaste_2021-02-08_12-24-20](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_12-24-20.png)

- Length：帧的总长度（2^24 位 = 16 MB）
- Type：帧类型
- Flags：根据 Type 的不同，Flags 的含义也不同
- Stream Identifier：Stream ID 流的唯一标识符
- Frame Payload：帧负载的不定长的数据





#### Stream ID 的作用

- 实现多路复用的关键

  - 接收端的实现可据此并发组装消息

  - 同一 Stream 内的 frame 必须是有序的（无法并发）

  - SETTINGS_MAX_CONCURRENT_STREAMS 控制着并发 Stream 数

    <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_12-30-44.png" alt="Snipaste_2021-02-08_12-30-44" style="zoom:67%;" />

- 推送<a id="依赖性请求">依赖性请求</a>的关键

  - 由客户端建立的流必须是奇数

  - 由服务器建立的流必须是偶数

    <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_12-31-49.png" alt="Snipaste_2021-02-08_12-31-49" style="zoom:67%;" />

- 流状态管理的约束性规定

  - 新建立的流 ID 必须大于曾经建立过的状态为 opened 或者 reserved 的流 ID
  - 在新建立的流上发送帧时，意味着将更小 ID 且为 idle 状态的流置为 closed 状态
  - Stream ID 不能复用，长连接耗尽 ID 应创建新连接（Stream ID 是用 Frame 中 31 bit 表示的，如果 31 bit 全为 1 后还要新建 Stream 就要新建连接）

- 应用层流控仅影响数据帧

  - Stream ID 为 0 的流仅用于传输控制帧

- 在HTTP/1 升级到 h2c 中，以 ID 为 1 流返回响应，之后流进入half-closed (local) 状态



#### 不同的帧类型及设置帧的子类型



![Snipaste_2021-02-08_12-24-20](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_12-24-20.png)

9 字节标准帧头部中各部分的详细说明

- Length：Frame Payload 的长度，以下称之为帧长度（帧长度最大大小为 16 MB，但默认最大大小是 16 KB。如果需要发送超过 16 KB 的帧，需要提前发送设置帧设置）

  - 0 至 214 (16,384) -1
    - 所有实现必须可以支持 16KB 以下的帧
  - 214 (16,384) 至 224-1 (16,777,215)
    - 传递 16KB 到 16MB 的帧时，必须接收端首先公布自己可以处理此大小
    - 通过 SETTINGS_MAX_FRAME_SIZE 帧（Identifier=5）告知

- Type：帧类型（每种帧除了标准帧头部外，其他的部分都不一样。HEADERSS，DATA，WINDOW_UPDATE帧最为常见）

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_14-19-50.png" alt="Snipaste_2021-02-08_14-19-50" style="zoom:80%;" />

  - RST_STREAM，终止流。用于通知服务器停止对请求的处理，且不需要再返回响应（比如用户发送请求后突然点击返回）
  - PING。心跳机制，只有 PING 没有 PONG
  - WINDOW_UPDATE。实现流量控制，但不同于 TCP 的流量控制



#### 设置帧



**Setting 设置帧格式（type=0x4）** 

- 设置帧并不是“协商”，而是发送方向接收方通知其特性、能力
- 一个设置帧可同时设置多个对象

设置帧通过以下格式设置 K 的 V

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_14-44-26.png" alt="Snipaste_2021-02-08_14-44-26" style="zoom:67%;" />

- Identifier：设置对象。比如设置：并发流的大小，最大帧的 size 等
- Value：设置值



**设置类型**

- SETTINGS_HEADERS_TABLE_SIZE (0x1): 通知对端索引表的最大尺寸（单位字节，初始 4096 字节）。比如设置头部压缩格式的设置帧
- SETTINGS_ENABLE_PUSH (0x2): Value设置为 0 时可禁用服务器推送功能，1 表示启用推送功能
- SETTINGS_MAX_CONCURRENT_STREAMS (0x3): 告诉接收端允许的最大并发流数量
- SETTINGS_INITIAL_WINDOW_SIZE (0x4): 声明发送端的窗口大小，用于Stream级别流控，初始值2^16-1 (65,535) 字节
- SETTINGS_MAX_FRAME_SIZE (0x5):设置帧的最大大小，初始值 2^14 (16,384)字节
- SETTINGS_MAX_HEADERS_LIST_SIZE (0x6): 知会对端头部索引表的最大尺寸，单位字节，基于未压缩前的头部



设置帧抓包实例图

这是一个空设置帧，因为它没有

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_14-52-05.png" alt="Snipaste_2021-02-08_14-52-05" style="zoom:80%;" />

这是一个含有多个设置项的设置帧

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_14-55-32.png" alt="Snipaste_2021-02-08_14-55-32" style="zoom:80%;" />



## HPACK 算法压缩头部

> HPACK 算法压缩头部是在[ RFC7541 ](https://httpwg.org/specs/rfc7541.html)中规定的
>
> HPACK 采用的压缩方式有以下 3 种
>
> - 静态字典
> - 动态字典
> - 压缩算法：哈夫曼编码（最高压缩比 8 : 5，其压缩比比较有限）
>
> 头部的 value 中会存在随机性很强的整形数字，因此 HPACK 除使用压缩手段压缩头部外还要考虑如何表示整形数字的编码
>
> 报文中表示数字不是像高级语言中直接写字面值或直接使用二进制表示，而是需要二进制表示值还要加一些辅助方式
>
> 下面讲的是： 3中压缩方式，整形数字的编码方式，头部经 HPACK 处理后的表现形式



### 使用 HPACK 的效果

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_19-56-01.png" alt="Snipaste_2021-02-08_19-56-01" style="zoom:67%;" />

上图展示头部空间压缩了 27.42%

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_19-57-09.png" alt="Snipaste_2021-02-08_19-57-09" style="zoom:67%;" />



### 静态字典

静态字典：提前定义索引和对应字符串映射关系的字典。示例如下图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_19-07-33.png" alt="Snipaste_2021-02-08_19-07-33" style="zoom:67%;" />

2 表示 name 为 method ，value 为 GET 的头部

RFC7541 的静态字典只有 66 项数据 [在这里](https://httpwg.org/specs/rfc7541.html#static.table.definition)



### 动态字典

动态字典是动态的，初始为空。它是基于先前的请求或响应中的头部建立的字典

- 先入先出的淘汰策略
- 动态表大小由 SETTINGS_HEADERS_TABLE_SIZE 设置帧定义
- 允许重复项
- 初始为空



### 哈夫曼编码

哈夫曼原理：出现概率较大的符号采用较短的编码，概率较小的符号采用较长的编码 

- 静态 Huffman 编码。RFC7541 基于哈夫曼编码规定了 十进制 0~256 的哈夫曼编码，[见这里](https://httpwg.org/specs/rfc7541.html#huffman.code)
  - 十进制 0~256 对应着 ACS 编码。所以静态哈夫曼编码是对 ACS 编码中的数据进行的编码
  - 其中出现频率高的是 数字，小写字母，大写字母
- 动态 Huffman 编码
  - SPDY（HTTP/2 的前身）使用动态哈夫曼编码
  - 动态哈夫曼编码是基于之前传输过的编码进行编码的，容易被攻击



PS：哈夫曼编码是在构建哈夫曼树后建立的，哈夫曼树的构建比较容易，这里不赘述

哈夫曼编码的数据在报文中表现形式间压缩同步过程



### HPACK 中整型数字的编码

以字节为单元，使用保留位+数据位标识整形数字

- 对于小于 31 的数字的编码：前3位位保留位，后5位用二进制正常表示整形数字（这叫使用 5 位前缀）

  ```
    0   1   2   3   4   5   6   7  	  0   1   2   3   4   5   6   7  
  +---+---+---+---+---+---+---+---+	+---+---+---+---+---+---+---+---+
  | ? | ? | ? |       Value       |	| X | X | X | 0 | 1 | 0 | 1 | 0 |   （这是对 10 进行编码的示例）
  +---+---+---+-------------------+	+---+---+---+---+---+---+---+---+
  ```

  

- 不一定必须是 5 位前缀，也可以是 N 位前缀。整形的编码公式为

  ```
  if I < 2^N - 1, encode I on N bits
  else
  	encode (2^N - 1) on N bits
  	I = I - (2^N - 1)
  	while I >= 128
  		encode (I % 128 + 128) on 8 bits
  		I = I / 128
  	encode I on 8 bits
  ```

  ```
    0   1   2   3   4   5   6   7  
  +---+---+---+---+---+---+---+---+
  | ? | ? | ? | 1 | 1 | 1 | 1 | 1 |	数字超过 2^N-1 时（这里取N=5）为例，使用左侧方式保存整形数字
  +---+---+---+-------------------+	如果是编码的起始部分，0位置的值为1
  | 1 |     Value-(2^N-1) LSB     |	如果是编码的结尾部分，0位置的值为0
  			  ...
  +---+---------------------------+
  | 0 |     Value-(2^N-1) MSB     |
  +---+---------------------------+
  ```

编码和解码示例图如下

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_19-37-57.png" alt="Snipaste_2021-02-08_19-37-57" style="zoom:45%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_19-38-51.png" alt="Snipaste_2021-02-08_19-38-51" style="zoom: 40%;" />



### HPACK 压缩头部的过程

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_19-41-48.png" alt="Snipaste_2021-02-08_19-41-48" style="zoom: 67%;" />

1. 原始头部数据经过静态字典和动态字典编码
2. 对整形数字用整形数字的编码方式或直接用字面量，对其他数据进行哈夫曼编码或直接用字面量



索引表用法示意图

![Snipaste_2021-02-08_19-51-42](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_19-51-42.png)



## HPACK 中 HEADERSS 中的编码格式



### HEADERSS 帧和持续帧的格式

**HEADERSS帧的格式**

```
+----------------+
| Pad Length?(8) |
+-+--------------+---------------------+
|E|       Stream Dependency?(31)       |
+-+--------------+---------------------+
|   Weight?(8)   |
+-+--------------+---------------------+
|     HEADERS Block Fragment (*)      ...
+--------------------------------------+
|           Padding (*)              ...
+--------------------------------------+
```

- ? 表示可选项
- \* 表示可重复单元
- Weight，表示权重
- Deader Block Fragment，表示头部



**CONTINUATION 持续帧(type=0x9)** 

- 跟在 HEADERS 帧或者 PUSH_PROMISE 帧之后，补充完整的 HTTP 头部
- PUSH_PROMiSE 帧也是携带头部的帧
- 持续帧只有 HEADERS Block Fragment

```
+--------------------------------------+
|      HEADERS Block Fragment (*)       |
+--------------------------------------+
```



### 字面编码 

- 组成
  - HEADERS name 和 value 以索引方式编码
  - HEADERS name 以索引方式编码，而 HEADERS value 以字面形式编码
  - HEADERS name 和 value 都以字面形式编码
- 可控制是否进入动态表
  - 进入动态表，供后续传输优化使用
  - 不进入动态表
  - 不进入动态表，并约定该头部永远不进入动态表



### 使用了 HPACK 的 HEADERSS 二进制编码格式

1. 名称和值都在索引表中
    编码方式：首位传1，其余7位传索引号

  ```
    0   1   2   3   4   5   6   7
  +---+---+---+---+---+---+---+---+
  | 1 |        Index(7+)          |
  +---+---------------------------+
  ```

  

2. 名称在索引表中，值需要编码传递，同时新增到动态表中
    编码方式：前2位传01，后6位为索引号

  ```
    0   1   2   3   4   5   6   7
  +---+---+---+---+---+---+---+---+
  | 0 | 1 |      Index(6+)        |
  +---+---------------------------+
  | H |    Value Length (7+)      |
  +---+---------------------------+
  | Value String (Length octets)  |
  +-------------------------------+
  ```

  PS：H位为1表示使用哈夫曼编码，为0表示没用哈夫曼编码

3. 名称、值都需要编码传递，同时新增动态表中
    编码格式：前2位传01，后6位为0

  ```
    0   1   2   3   4   5   6   7
  +---+---+---+---+---+---+---+---+
  | 0 | 1 |          0            |
  +---+---+-----------------------+
  | H |     Name Length (7+)      |
  +---+---------------------------+
  |  Name String (Length octets)  |
  +---+---------------------------+
  | H |     Name Length (7+)      |
  +---+---------------------------+
  |  Name String (Length octets)  |
  +-------------------------------+
  ```

  

4. 名称在索引表，值需要编码传递，且不更新动态表
    编码格式：前4为传0000，后4位为索引号

  ```
    0   1   2   3   4   5   6   7
  +---+---+---+---+---+---+---+---+
  | 0 | 0 | 0 | 0 |   Index(4+)   |
  +---+---------------------------+
  | H |    Value Length (7+)      |
  +---+---------------------------+
  | Value String (Length octets)  |
  +-------------------------------+
  ```

  

5. 名称、值都需要编码传递，且不更新至动态表中
    编码格式：前4为传0000，后4位也为0

  ```
    0   1   2   3   4   5   6   7
  +---+---+---+---+---+---+---+---+
  | 0 | 0 | 0 | 0 |      0        |
  +---+---------------------------+
  | H |     Name Length (7+)      |
  +---+---------------------------+
  |  Name String (Length octets)  |
  +-------------------------------+
  | H |     Name Length (7+)      |
  +---+---------------------------+
  |  Name String (Length octets)  |
  +-------------------------------+
  ```

  

6. 名称在索引表中，值需要编码传递，且永远不更新至动态表中
    编码格式：前4为传0001，后4位为索引号

  ```
    0   1   2   3   4   5   6   7
  +---+---+---+---+---+---+---+---+
  | 0 | 0 | 0 | 1 |   Index(4+)   |
  +---+---------------------------+
  | H |    Value Length (7+)      |
  +---+---------------------------+
  | Value String (Length octets)  |
  +-------------------------------+
  ```

  

7. 名称、值都需要编码传递，且永远不更新至动态表中
    编码格式：前 4 位传 0001，后4位为0

  ```
    0   1   2   3   4   5   6   7
  +---+---+---+---+---+---+---+---+
  | 0 | 0 | 0 | 1 |       0       |
  +---+---------------------------+
  | H |     Name Length (7+)      |
  +---+---------------------------+
  |  Name String (Length octets)  |
  +-------------------------------+
  | H |     Name Length (7+)      |
  +---+---------------------------+
  |  Name String (Length octets)  |
  +-------------------------------+
  ```

报文示例

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_21-50-24.png" alt="Snipaste_2021-02-08_21-50-24" style="zoom:67%;" />



## 服务器端的主动消息推送

HTTP/2 提供的主动推送消息和 Websocket不同

它是提前将资源推送至浏览器缓存 

- 推送可以基于已经发送的请求，例如客户端请求 html，html 中用 src 引入外部 css
- 实现方式
  - 推送资源必须对应一个请求
  - 请求由服务器端 PUSH_PROMISE 帧发送
  - 响应在偶数 ID 的 STREAM 中发送

PS：因为和 websocket 不一样，所以应用场景不是太确定。淘宝，知乎等首页都没有用到此功能

### 主动推送流程

左侧为使用 HTTP/1.1 时请求 html 中的 css 文件的方式。右侧为 HTTP/2 主动推送消息的方式

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_16-24-51.png" alt="Snipaste_2021-02-09_16-24-51" style="zoom:50%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_16-26-51.png" alt="Snipaste_2021-02-09_16-26-51" style="zoom:50%;" />

HTTP/2 主动推送消息的方式为 先发送一个 PUSH_PROMISE 帧通知客户端主动推送的数据即将来临，再发送数据帧（这两个帧可以不在同一个流中）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_16-29-41.png" alt="Snipaste_2021-02-09_16-29-41" style="zoom:67%;" />



### PUSH_PROMISE 帧格式

PUSH_PROMISE 帧，type=0x5，只能由服务器发送

``` 
+----------------+
| Pad Length?(8) |
+-+--------------+------------------------------------------+
|R|                   Promise Stream ID (31)                |	// Promise Stream ID 告诉客户端接下来的数据帧将来自哪个流
+-+--------------------------+------------------------------+	// PUSH 帧相当于服务器主动推送的消息的 HEADERSS 帧
|                  HEADERS Block Fragment (*)              ...	// 包含头部信息
+-----------------------------------------------------------+
|                          Padding(*)                     ...
+-----------------------------------------------------------+
```



### PUSH 的禁用

使用设置帧发送 SETTINGS_ENABLE_PUSH（0x2） 即可禁用主动推送

- 1 表示启用推送功能
- 0 表示禁用推送功能



## Stream 的状态变迁

Stream 流是有状态的，客户端和服务器都会为 Stream 维护一个状态

### Stream 特性 

- 一条 TCP 连接上，可以并发存在多个处于 OPEN 状态的 Stream
- 客户端或者服务器都可以创建新的 Stream
- 客户端或者服务器都可以首先关闭 Stream
- 同一条 Stream 内的 Frame 帧是有序的
- 从 Stream ID 的值可以轻易分辨 PUSH 消息（是否违反客户都安建立的流都是奇数，服务器建立的流都是偶数）
  - 所有为发送 HEADERS/DATA 消息而创建的流，从1、3、5 等递增奇数开始
  - 所有为发送 PUSH 消息而创建的流，从 2、4、6 等递增偶数开始



### Stream 状态的种类和变迁

帧符号 

- H: HEADERSS 帧
- PP: PUSH_PROMISE 帧
- ES: END_STREAM 标志位为 1 的帧
- R: RST_STREAM 帧 

流状态 

- idle：起始状态
- closed：关闭状态
- open：运行状态。可以收发任何帧。如果收发 ES 帧或 R 帧，Stream 的状态将会改变
- half closed 单向关闭 / 半关闭
  - remote：不再接收数据帧
  - local：不能再发送数据帧
- reserved
  - remote
  - local

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_19-16-06.png" alt="Snipaste_2021-02-09_19-16-06" style="zoom:67%;" />

PS：send 表示发送帧，recv 表示接收帧



## RST_STREAM 帧及常见错误码

### RST_STREAM 帧存在的原因

- 能优雅地关闭流
- 为什么要优雅地关闭流，而不是关闭连接
  - HTTP/1.1 上一个连接上同一时间只能发一个请求或收一个响应。想断掉请求或响应只需要关闭连接即可
  - 在HTTP/2 上一条连接上有多个流，每个流上又并行收发请求和响应。想断掉某个请求或响应，但如果直接关闭连接就会影响到其他流，所以使用 RST_STREAM 帧只关闭某一个流更为安全



### RST_STREAM 帧（type=0x3） 

- HTTP2 多个流共享同一连接，RST 帧允许立刻终止一个未完成的流

- RST_STRAM 帧不使用任何 flag （包括标准帧头部中的 flags ）

- RST_STREAM 帧的格式

  ```
  +-------------------------------------------+
  |              Error Code (32)              |
  +-------------------------------------------+
  ```



### <a id="常见错误码">常见错误码</a>

| 错误码                    | 错误原因说明                                                 |
| ------------------------- | ------------------------------------------------------------ |
| NO_ERROR (0x0)            | 没有错误。GOAWAY帧优雅关闭连接时可以使用此错误码             |
| PROTOCOL_ERROR (0x1)      | 检测到不识别的协议字段                                       |
| INTERNAL_ERROR (0x2)      | 内部错误                                                     |
| FLOW_CONTROL_ERROR (0x3)  | 检测到对端没有遵守流控策略                                   |
| SETTINGS_TIMEOUT (0x4)    | 某些设置帧发出后需要接收端应答，在期待时间内没有得到应答则由此错误码表示 |
| STREAM_CLOSED (0x5)       | 当Stream已经处于半关闭状态不再接收Frame帧时， 又接收到了新的Frame帧 |
| FRAME_SIZE_ERROR (0x6)    | 接收到的Frame Size不合法                                     |
| REFUSED_STREAM (0x7)      | 拒绝先前的Stream流的执行                                     |
| CANCEL (0x8)              | 表示Stream不再存在                                           |
| COMPRESSION_ERROR (0x9)   | 对HPACK压缩算法执行失败                                      |
| CONNECT_ERROR (0xa)       | 连接失败                                                     |
| ENHANCE_YOUR_CALM (0xb)   | 检测到对端的行为可能导致负载的持续增加，提醒对方“冷静”一点   |
| INADEQUATE_SECURITY (0xc) | 安全等级不够                                                 |
| HTTP_1_1_REQUIRED (0xd)   | 对端只能接受HTTP/1.1协议                                     |





## Stream优先级与资源分配规则

一个页面中多多个请求，例如请求html文本，css文件，js文件，图片文件等

它们对用户的优先级是不一样的，比如 html > css > js > imgs。因此为流设置优先级很有必要



### Priority 优先级设置帧 

- 帧类型：type=0x2
- 不使用 flag 标志位字段
- Stream Dependency：依赖流，只有依赖的流完成收发请求或响应后再去做本流
- Weight权重：取值范围为 1 到 256。默认权重16
- 仅针对 Stream 流设置优先级，若 ID 为 0 的 Stream 试图发送此帧去影响整个连接，则接收端必须报错
- 在 idle 和 closed 状态下，仍然可以发送 Priority 帧

```
+-+--------------+---------------------+
|E|       Stream Dependency?(31)       |
+-+--------------+---------------------+
|   Weight?(8)   |
+----------------+
```



### HEADERS 帧设置优先级

HEADERS 帧也有 Weight 选项可以指定优先级

下图是某个请求的 HEADERS 帧

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_21-41-17.png" alt="Snipaste_2021-02-09_21-41-17" style="zoom:50%;" />



### 数据流优先级

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_21-46-53.png" alt="Snipaste_2021-02-09_21-46-53" style="zoom:67%;" />



**exclusive 标志位**

exclusive  译：独占

```
+-+--------------+---------------------+
|E|       Stream Dependency?(31)       |
+-+--------------+---------------------+
|   Weight?(8)   |
+----------------+
```

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_21-48-12.png" alt="Snipaste_2021-02-09_21-48-12" style="zoom:67%;" />



## 不同于 TCP 的流量控制

可以从 TCP 层进行流量控制，也可以从 HTTP/2 应用层协议中进行流量控制

HTTP/2 的流量控制可以针对某个流也可以针对整条连接



HTTP/1.1 通过 TCP 层进行流控（HTTP/1.1 的 TCP 连接上没有多路复用），这里只做简介，详情见第 5 部分

- TCP 短链接（每对 request-response 使用一个连接，用完后关闭连接）通过接收窗口，拥塞阻塞进行流控
- TCP 长连接（复用一个 TCP 连接进行多次收发 request-response）通过接收窗口进行流控

PS：TCP 长链接，短链接 是 HTTP 长连接，短链接的另一种叫法。其实指的都是 TCP 的长短链接

HTTP/2 在应用层进行流控

- 多路复用下，多个 Stream 争夺 TCP 的流控制，互相干扰可能造成 Stream 阻塞

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_21-59-53.png" alt="Snipaste_2021-02-09_21-59-53" style="zoom:67%;" />

- 代理服务器内存有限，上下游网速不一致时，通过流控管理内存

- **HTTP/2 通过发送 WINDOW_UPDATE 帧设置窗口大小实现流控**。由接收端告诉发送端，接收端的窗口能接收多少字节

- **HTTP/2 还能通过发送设置帧设置 SETTINGS_MAX_CONCURRENT_STREAMS ，限制并发流数量实现流控**



### HTTP/2 的流控定义

HTTP/2 中的流控制既针对单个 Stream，也针对整个 TCP 连接 

- 客户端与服务器都具备流量控制能力
- 单向流控制：发送和接收独立设定流量控制
- 以信用为基础：接收端设定上限，发送端应当遵循接收端发出的指令
- 流量控制窗口（流或者连接）的初始值是 65535 字节
- 只有 DATA 帧服从流量控制。像是其他 HEADERS帧，设置帧等帧都不受此控制
- 流量控制不能被禁用



### WINDOW_UPDATE 帧格式

- type=0x8，不使用任何 flag
- 窗口范围 1 to 231-1 (2,147,483,647)字节
  - 0 是错误的，接收端应返回 PROTOCOL_ERROR
- 当 Stream ID 为 0 时表示对连接流控，否则为对 Stream 流控
- 流控仅针对直接建立 TCP 连接的两端
  - 代理服务器并不需要透传 WINDOW_UPDATE 帧（代理服务器可自行处理此帧）
  - 接收端的缩小流控窗口会最终传递到源发送端

```
+-+------------------------------------+
|R|     Window Size Increment (31)     |
+-+------------------------------------+
```



### 流控窗口如何实现流控

- 窗口大小由接收端告知
- 窗口随 DATA 帧的发送而减小

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_22-09-52.png" alt="Snipaste_2021-02-09_22-09-52" style="zoom:67%;" />

### 限制并发流数

设置帧设置 SETTINGS_MAX_CONCURRENT_STREAMS

- 并发仅统计 open 或者 half-close 状态的流（不包含用于推送的 reserved 状态）
- 超出限制后的[错误码](#常见错误码)
  - PROTOCOL_ERROR
  - REFUSED_STREAM

下图是某个设置帧中设置的最大并发流数

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-09_22-15-11.png" alt="Snipaste_2021-02-09_22-15-11" style="zoom:67%;" />



## HTTP/2 和 gRPC 框架

RPC 框架需要使用某种格式序列化要传递的数据。gRPC，Dubbo 等 RPC 框架经常使用 protobuf 作为序列化工具

此节以 gRPC 框架为例，抓包分析 RPC 协议的报文中由 Protocol Buffers 编码的消息结构

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_12-14-51.png" alt="Snipaste_2021-02-11_12-14-51" style="zoom:67%;" />

protobuf 将数据以一个 Field 为单位进行编码，一个消息结构中可以有多个 field。Field 的组成如下

![Snipaste_2021-02-11_12-20-54](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_12-20-54.png)

上述结构中有以下两个主要组成部分

- Tag    由以下两部分组成
  - field_number    field 的 ID 号。第一个 field 的 field_number 的值为 1 ，依次递增
  - wire_type    用于描述 field 的类型
- Value    field 的值



wire_type 的类型如下（ Java 中的数据类型和 protobuf 中的数据类型对应关系见[官网](https://developers.google.com/protocol-buffers/docs/proto)，官网在谷歌的多级域名下，得翻墙）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_12-21-58.png" alt="Snipaste_2021-02-11_12-21-58" style="zoom:67%;" />



PS：视频中使用 gRPC 和 protobuf 作为序列化方式编写示例代码和抓包（包的协议名为 "GRPC"），GRPC 包中 protobuf 编码的数据如下图（自己可以尝试使用 Netty + protobuf 编写示例代码和抓包）

````
Data: 00000000050a03796f75    # 这是 GRPC 协议包中携带的数据，由 protobuf 编码，解码后值为 "you"
````

![Snipaste_2021-02-11_12-29-11](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_12-29-11.png)



**Protocol Buffers 字符串编码举例**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_15-54-09.png" alt="Snipaste_2021-02-11_15-54-09" style="zoom: 67%;" />



如果抓包时端口未被识别为 http2，需要手动右击未被正确识别的报文（未被正确识别为 HTTP/2 的报文被显示为它的上层协议—TCP协议）点击 Decode As 设置指定端口的 TCP 报文被识别为 HTTP/2报文。下述的 GQUIC 协议包也是如此



## HTTP2 和 HTTP3



### HTTP2 的问题和 HTTP3 的意义

**1. TCP 和 TLS 建立连接握手过多的问题**

HTTP/1.1 ，HTTP/2 基于 TCP。HTTP/2 使用了 TLS

TCP 存在 3 次握手建立连接，TLSv1.2 也存在 2 次握手（协商安全套件握手和交换密钥握手）建立安全连接（ TLSv1.3只需要 1 次握手）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_16-55-33.png" alt="Snipaste_2021-02-11_16-55-33" style="zoom:67%;" />

**2. 多路复用和 TCP 的队头阻塞问题**

HTTP/2 的多路复用采用一个连接上在多个流传递帧

- 帧在发送端的操作系统上被转为 TCP 报文并有序发送出去
- 帧在网络中是无需的，帧发送到目标主机后又变为有序的

TCP 的队头阻塞问题：

如果这些帧的第一个帧丢失，目标主机即使接收到其他帧，也会先等待接收第一个帧后才会对这些帧进行重组（将同一个流中的帧组合起来）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_16-57-13.png" alt="Snipaste_2021-02-11_16-57-13" style="zoom: 67%;" />

PS：

- 这里举例使用的是SPDY，但其多路复用的执行流程和 HTTP2 的执行流程是一样的
- 这里的小色块表示帧，同一个流的帧的小色块颜色是一样的



**3. TCP的问题**

**TCP 由操作系统内核实现**，更新缓慢



### HTTP3-QUIC协议

> QUIC 有 IETF 的 QUIC 和谷歌的 QUIC（GQUIC）
>
> 谷歌的GQUIC已经发布，IETF 的 QUIC 协议原计划在 2019 年 7月份发布（待其被广泛使用又要花一些时间）
>
> 基于 HTTP3 未别广泛使用的前提，这里只介绍 HTTP3 的目的和设计原则
>
> 附带 IETF QIUC 草案
>
> - IETF draft 20: https://tools.ietf.org/html/draft-ietf-quic-http-20
> - https://datatracker.ietf.org/doc/draft-ietf-quic-transport/



#### HTTP3 在协议层中的位置

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_17-05-47.png" alt="Snipaste_2021-02-11_17-05-47" style="zoom:67%;" />

- HTTP3 基于 QUIC 协议
- HTTP3 的对外表现和 HTTP2 类似，这是为了兼容 HTTP2。也就是说 HTTP3 实现了 HTTP2 的功能
- HTTP3 基于 UDP 协议，而非 TCP
- QUIC 协议包含了 TLSv1.3



#### HTTP3 体验

HTTP3 尚未被广泛使用且不稳定，浏览器默认关闭 QUIC 协议。可以进行以下操作开启浏览器的 QUIC

- [chrome://flags/#enable-quic](chrome://flags/#enable-quic) Chrome 默认关闭，需要手动开启

- 开启 QUIC 前，打开B站任意一个视频，Chrome 调试工具的抓包面板中基本都是 "h2" 和 "http1" 的请求

- 打开 QIUC 协议，重新打开B站任意一个视频。一些请求变为 "http/2 + quic/46"。如下图

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_17-47-38.png" alt="Snipaste_2021-02-11_17-47-38" style="zoom:67%;" />

  ![Snipaste_2021-02-11_18-03-59](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_18-03-59.png)



#### HTTP/3 和 QUIC协议



**对比 HTTP/2 和 HTTP/3**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_17-49-00.png" alt="Snipaste_2021-02-11_17-49-00" style="zoom:67%;" />

- TCP 协议中包含有拥塞控制
- UDP 协议不包含拥塞控制。所以 QUIC 协议包含拥塞控制作为补充



**HTTP/3 的连接迁移**

连接迁移：HTTP/3 新增功能。允许客户端更换 IP 地址、端口后，仍然可以复用前连接

用于解决例如移动端会经常切换连接的wifi，导致IP地址频繁更换的问题。如果是 HTTP/2，由于设备切换网络被分配新的 IP 地址，导致客户端需要重新建立和服务器的连接



如图，HTTP/3 使用 Connection ID 和 Packet Number 解决连接迁移的问题

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_17-55-45.png" alt="Snipaste_2021-02-11_17-55-45" style="zoom:67%;" />



**解决了队头阻塞的问题**

因为 HTTP/3 协议基于 UDP ，而 UDP 没有队列的概念。即使发送端发送的第一个帧丢失也不影响接收端直接接收其他帧

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_18-07-12.png" alt="Snipaste_2021-02-11_18-07-12" style="zoom:50%;" />



**HTTP/3 的少次数握手**

RTT RTT(Round-Trip Time): 往返时延，在计算机bai网络中它也是一个重要的性能指标，它表示从发送端发送数据开始，到发送端收到来自接收端的确认（接收端收到数据后便立即发送确认），总共经历的时延

- HTTP/3 的 1RTT 完全握手

  - TCP + TLS1.2 需要进行 3 次 RTT 握手，HTTP3只需要进行 1 次 RTT 握手

    原因：HTTPS的QUIC基于UDP，UDP不需要握手。HTTPS 的 TLSv1.3 仅支持数个安全套件，所以认为不需要再进行安全套件的协商的握手（即认定对方一定至少支持这些安全套件中的一个）HTTPS直接互发生成的密钥

- HTTP/3 的 0RTT 握手和 0RTT 恢复会话握手

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_18-10-58.png" alt="Snipaste_2021-02-11_18-10-58" style="zoom:50%;" />

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-11_18-11-39.png" alt="Snipaste_2021-02-11_18-11-39" style="zoom:50%;" />



## 四层/七层负载均衡

> 参考[什么是四层和七层负载均衡？他们之间的区别是什么？](https://kb.cnblogs.com/page/188170/)，[linux负载均衡总结性说明（四层负载/七层负载）](https://www.cnblogs.com/kevingrace/p/6137881.html)

负载均衡服务器除了四层，七层负载均衡外的其他功能见 Nginx



> - 二层负载均衡，基于MAC地址。通过虚拟MAC地址接受请求，再将请求分配给真实的MAC地址
> - 三层负载均衡，基于IP地址。通过虚拟IP 地址接收请求，再将请求分配给真实的IP 地址
> - 四层负载均衡，基于端口地址。通过虚拟IP+端口接收请求，再将请求分配给真实的服务器
> - 七层负载均衡，通过虚拟URL或主机名接收请求，再将请求分配给真实的服务器
>
> 四层交换机主要分析IP层及TCP/UDP层
> 七层交换机除了支持四层负载均衡外还能分析应用层协议的信息
>
> 四层负载均衡服务器有两种转发请求的方式
> 负载均衡服务器接收到 TCP 的 SYN 包后
>
> - 只修改报文中目标IP地址和端口。客户端和代理服务器建立单独的TCP连接，代理服务器和源服务器建立单独的TCP连接
> - 修改报文中目标IP地址和端口，修改报文中的源地址。使客户端直接与源服务器建立TCP连接
> 显然第二种方式效率更高
>
> 但如果是七层负载均衡代理服务器。因为代理服务器要分析应用层的报文，所以代理服务器必须和客户端建立TCP连接



# 第 4 部分：TLS/SSL

> TSL/SSL 的初步认识见[这里](#SSL 和 TLS)
>
> 下面的章节主要围绕 HTTP/3 使用的 TLSv1.3 详细讲述 TLS/SSL
>
> PS：[新浪](www.sina.com.cn)用 TLS1.3 ，[陶辉个人博客站点](www.taohui.pub)用 TLS1.2



> 常见的加密算法分为
>
> - 对称加密算法（DES、3DES、DESX、Blowfish、IDEA、RC4、RC5、RC6和AES）
> - 非对称加密算法（RSA、ECC（移动设备用）、Diffie-Hellman、El Gamal、DSA（数字签名用））
> - Hash算法（MD2、MD4、MD5、HAVAL、SHA、SHA-1、HMAC、HMAC-MD5、HMAC-SHA1）
>
> 参考博客[常用加密算法概述](https://www.cnblogs.com/colife/p/5566789.html)，[浅谈常见的七种加密算法及实现](https://blog.csdn.net/baidu_22254181/article/details/82594072)



## TLS 概述

SSL 3.0 给网络通信带来安全性

TLS 1.2 是 TLS 的一个比较完善的版本

TLS 1.3 优化了 TLS 1.2 的握手，减少了握手次数

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-12_16-08-26.png" alt="Snipaste_2021-02-12_16-08-26" style="zoom: 50%;" />



**TLS 协议的主要组成**

TLS 主要由 Handshake 握手协议和 Record 记录协议组成

- Record 记录协议用于定义 TLS 中通信的发送端如何加密信息，接收端如何解密信息（接收端使用的解密密钥需要用 Handshake 握手协议获取密钥）
- Handshake 握手协议（用于安全地交换密钥，让接收端使用密钥进行解密）
  - 验证通讯双方的身份（通过证书实现验证身份）
  - 交换加解密的安全套件
  - 协商加密参数



**TLS 安全密码套件解读**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-12_19-18-36.png" alt="Snipaste_2021-02-12_19-18-36" style="zoom:50%;" />

PS：上图中的安全套件是 TLSv1.2 常用的安全套件

## 对称加密

```
		   密钥加密				  密钥解密
原始文档  ——————————>  加密文档  ——————————>  原始文档
```

**AES 对称加密算法在网络中的应用**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-12_19-21-31.png" alt="Snipaste_2021-02-12_19-21-31" style="zoom:50%;" />

### 对称加密的工作原理

**对称加密流程**

1. 对明文进行分组
2. 对所有组内的明文进行加密
3. 并合所有分组的加密结果产生最后的密文

- 加密函数：对称加密需要对明文进行可逆操作将其转为密文。下面**以对明文和密钥进行异或操作为加密方式为例**进行讲解
  - 异或操作：双目操作。两位相同结果为 0 ；两位不同结果为 1
- 分组加密（ Block cipher ）：将明文分成多个等长的 Block 模块，对每个模块分别加解密
  - 分组的必要性：密钥和加密函数能处理的明文长度有限，而明文的长度不可预测，且通常明文长度大于密文。所以需将明文分组，一次对这些组中的明文使用加密函数和密钥进行加密
- 填充策略：分组后最后一组数据长度可能短于密钥的长度，需要对最后一组数据进行填充。有多种填充方式
  - 位填充：以 bit 位为单位来填充。按照某种规则进行填充
  - 字节填充：以字节为单位为填充
    - 补零：... | DD DD DD DD DD DD DD DD | DD DD DD DD **00 00 00 00** |
    - ANSI X9.23：... | DD DD DD DD DD DD DD DD | DD DD DD DD **00 00 00 04** |  （除最后一个字节外其余填充字节均为 0 ，最后一个字节的值为填充的 0 的字节个数）
    - ISO 10126：... | DD DD DD DD DD DD DD DD | DD DD DD DD **81 A6 23 04** |
    - **PKCS7 （RFC5652）**：... | DD DD DD DD DD DD DD DD | DD DD DD DD **04 04 04 04** |  （要填充 n 个字节，每个填充的字节值就是 n  AES 就使用此方式作为填充策略）

````
1010  # 密钥序列
 ⊕
0110  # 明文
 =
1100  # 密文
````

PS：对明文进行怎样的加密操作取决于加密算法的加密函数



**分组工作模式（ block cipher mode of operation ）**

分组工作模式及分组策略

下面还是以**对明文和密钥进行异或操作为加密方式为例**进行讲解

- ECB（Electronic codebook）模式（下图是解密流程）

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-17_10-51-25.png" alt="Snipaste_2021-02-17_10-51-25" style="zoom: 50%;" />

- CBC（Cipher-block chaining）模式（下图是加密流程）

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-17_10-53-47.png" alt="Snipaste_2021-02-17_10-53-47" style="zoom:67%;" />

- CTR（Counter）模式（下图是加密流程）

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-17_10-59-31.png" alt="Snipaste_2021-02-17_10-59-31" style="zoom:67%;" />

  

**基于哈希算法的 MAC （Message Authentication Code）算法：验证数据完整性的算法**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_11-33-46.png" alt="Snipaste_2021-02-18_11-33-46" style="zoom:67%;" />

发送方发送消息前用 key 计算消息的哈希值，接收方用相同的 key 计算接收到的消息的哈希值。这两个哈希值相同表示消息是完整的，哈希值不同表示消息被修改过或消息是不完整的



- GCM（Galois/Counter Mode）

  GCM = CTR + GMAC

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_11-37-43.png" alt="Snipaste_2021-02-18_11-37-43" style="zoom:67%;" />



TLS 1.2，1.3 中主要使用 AES 算法

TLS 1.2，1.3 使用BCM（可能存在手误，原文可能是GCM）分组工作模式



### 详解 AES 对称加密算法



AES 常用填充算法：PKCS7 ，常用分组工作模式：GCM

| AES 加密方式 | 密钥长度（单位：32比特） | 分组长度（单位：32比特） | 加密轮数 |
| ------------ | ------------------------ | ------------------------ | -------- |
| AES-128      | 4                        | 4                        | 10       |
| AES-192      | 6                        | 4                        | 12       |
| AES-256      | 8                        | 4                        | 14       |

**加密流程**

1. 把明文按照 128bit（16 字节）拆分成若干个明文块，每个明文块是 4*4 矩阵（分组）
2. 按照选择的填充方式来填充最后一个明文块（填充）
3. 每一个明文块利用 AES 加密器和密钥，加密成密文块（多次加密）
4. 拼接所有的密文块，成为最终的密文结果（合并加密结果）

其中第 3 步要进行多次，轮数由上述表格中指定。其中加密轮分 3 种：初始轮，普通轮，最终轮

加密时：执行 1 次初始轮，执行若干次普通轮，执行 1 次最终轮。轮数相加等于加密方式中指定的轮数

解密时，轮的执行和加密时执行的顺序相反。执行 1 次最终轮，执行若干次普通轮，执行 1 次最终轮



**加密轮概述**

C = E(K,P)，E 为每一轮算法，每轮密钥皆不同 

- 初始轮
  - AddRoundKey 轮密钥加
- 普通轮
  - AddRoundKey 轮密钥加 —> SubBytes 字节替代 —>ShiftRows 行移位 —> MixColumns 列混合
- 最终轮
  - SubBytes 字节替代 —> ShiftRows 行移位 —> AddRoundKey 轮密钥加

AES 的加密流程中加密轮的详细步骤见 [PPT](D:\编程学习资料\Web协议与抓包实战\PDF课件\第4部分 TLS协议.pdf)



## 非对称加密与 RSA 算法

非对称加密中，客户端使用的公钥由两种方式获取

- 在和服务器建立 TLS 握手时，服务器把公钥传递给客户端
- 客户端直接从 PKI 基础设施中获取

早期常用 RSA 传递对称加密中的密钥，现常用 RSA 算法生成 CA 证书



**RSA 算法中公私钥的产生**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_13-36-27.png" alt="Snipaste_2021-02-18_13-36-27" style="zoom: 67%;" />

- 明文中的数字都必须小于 n
- 欧拉函数。v = 小于 n 的正整数中与 n 互质的数的个数。用 p 和 q 能直接计算出 v 的值
- 问：无法或很难从公钥中推出私钥的原因
  - 答：欲知私钥须知 d ，欲知 d 须知 v 。欲知 v 须知 n 或 p 和 q 。欲知 p 和 q 需将 n 进行因式分解。n是大数，对大数进行因式分解的效率极低，甚至可视为无法在可接受的时间内解出正解

PS：RSA 算法生成公私钥即为：使用随机数 p ，q ，k 生成公私钥的过程

**RSA 算法加解密流程**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_13-40-47.png" alt="Snipaste_2021-02-18_13-40-47" style="zoom:67%;" />

- 加解密时都需要进行多次较为耗时的计算，所以类比可推出：非对称加密算法相对于对称加密算法更为耗时



## 基于 openssl 实战验证 RSA

目标：使用基于 RSA 算法的 openssl 生成公私密钥，并对一个文本文件进行加解密

准备工作：准备一个名为`hello.txt`的文本。文件内容为`Hello World!`



生成私钥

```shell
$ openssl genrsa -out private.pem
```

生成公钥/从私钥中提取出公钥（公钥格式参见 RFC 3447 ）

```shell
$ openssl rsa -in private.pem -pubout -out public.pem
```

查看 ASN.1 格式的私钥（ASN.1 格式的私钥稍后再说）

```shell
$ openssl asn1parse -i -in private.pem
```

查看 ASN.1 格式的公钥

```shell
$ openssl asn1parse -i -in public.pem
```

解析公钥中的 n 和 k （参数 19 是公钥中 n 和 k 所在位置的偏移量，需要指定才能解析 n 和 k 的值，否则默认不解析这部分数据）

```shell
$ openssl asn1parse -i -in public.pem -strparse 19
```

加密文件 

```shell
$ openssl rsautl -encrypt -in hello.txt -inkey public.pem -pubin -out hello.en
```

解密文件

```shell
$ openssl rsautl -decrypt -in hello.en -inkey private.pem -out hello.de
```



**openssl 基于 RSA 生成的私钥的格式**

```
RSAPrivateKey ::= SEQUENCE { 
    version Version, 
    modulus INTEGER, -- n 
    publicExponent INTEGER, -- k 
    privateExponent INTEGER, -- d 
    prime1 INTEGER, -- p 
    prime2 INTEGER, -- q 
    exponent1 INTEGER, -- d mod (p-1) 
	exponent2 INTEGER, -- d mod (q-1) 
	coefficient INTEGER, -- (inverse of q) mod p 
	otherPrimeInfos OtherPrimeInfos OPTIONAL 
}
```

**openssl 基于 RSA 生成的公钥的 n 和 k 的格式**

```
RSAPublicKey ::= SEQUENCE {
	modulus INTEGER, -- n
	publicExponent INTEGER -- k
}
```



## 非对称密码应用



### PKI 证书体系

公钥数字证书的运作体系：PKI

公私钥可由openssl等工具生成/创建，openssl 创建的公私钥是数字证书的一部分

公钥证书需要 PKI 进行管理

- 基于私钥加密，只能使用公钥解密：起到身份认证的使用
- 公钥的管理：Public Key Infrastructure（PKI）公钥基础设施
  - 由 Certificate Authority（CA）数字证书认证机构将用户个人身份与公开密钥关联在一起
  - 公钥数字证书组成 ：CA 信息、公钥用户信息、公钥、权威机构的签字、有效期 



**签发证书流程**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_17-14-10.png" alt="Snipaste_2021-02-18_17-14-10" style="zoom:67%;" />



**签名与验签流程**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_17-15-04.png" alt="Snipaste_2021-02-18_17-15-04" style="zoom:80%;" />

场景：某网站站长申请一个数字证书并部署到自己的网站上

签名：

1. 站长将个人信息和自己的公钥发给 CA
2. CA 对站长的个人签名用哈希算法进行加密签名得到数字签名
3. CA 将数字签名和公钥整合，得到数字证书



验签：（使用浏览器的人访问站点前，浏览器自行请求网站证书并向 CA 提出验签请求）

1. 请求网站证书，并向 CA 发起验签请求
2. CA 获取到库中的数字签名后与数字证书中的数字签名进行对比



**证书信任链**

![Snipaste_2021-02-18_17-24-11](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_17-24-11.png)

**证书类型**

- 域名验证证书 DV
- 组织验证证书 OV
- 扩展验证证书 EV

从上到下，验证严谨程度递增，价格递增。但单从加密程度上来说，以上 3 种 证书的加密程度都是相同的



### DH 密钥交换协议

通信通常先用非对称加密传递对称加密的密钥（这个过程叫做密钥交换），此后的通信用对称加密进行通信

这里讲下密钥交换的 2 种实现：RSA 密钥交换，DH 密钥交换

- RSA 密钥交换

  缺点：存在没有向前保密性的问题（如果有人保存了某次通信的所有报文，并再之后破解得到 Server 的私钥就能破解所有报文）

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_17-28-04.png" alt="Snipaste_2021-02-18_17-28-04" style="zoom:50%;" />

  1. 在建立 TLS 握手时，Server 向 Client 发送非对称加密算法的公钥
  2. Client 接收到公钥后，用公钥加密一个对称加密算法的密钥并发送给 Server
  3. Server 接收到数据后用私钥解密，得到对称加密算法的密钥
  4. 此后通信双方使用对称加密算法进行安全的通信

- DH 密钥交换协议（非对称加密的一个应用）：TLS 握手时，通讯双方独立生成相同密钥的协议

  能保证向前保密性。因为 DH 密钥交换协议可以让双方在完全没有对方任何预先信息的条件下通过不安全信道创建起一个密钥，且每次建立的通信使用的密钥都是新的且保存在内存中，用完即弃

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_17-34-41.png" alt="Snipaste_2021-02-18_17-34-41" style="zoom:67%;" />

  Client 基于Private key 2 和 Public key 1 生成一个密钥

  Server 基于Private key 1 和 Public key 2 生成一个密钥

  这两个密钥是相同的。它被用于以后的对称加密通信



**DH 密钥交换协议的问题** 

- 中间人伪造攻击
  - 向 Alice 假装自己是 Bob，进行一次 DH 密钥交换
  - 向 Bob 假装自己是 Alice，进行一次 DH 密钥交换

解决方案：使用PKI。强制要求证明自己的身份，中间人就无法伪装为Alice或Bob

DH 的问题：交换密钥过程中涉及大量的乘法运算。耗时耗能



[DH算法在混合加密中起什么作用？](https://www.zhihu.com/question/35137387)



## DH 协议的升级：基于椭圆曲线的 ECDH 协议

DH 协议的问题

- 有大量的乘法运算
- DH 的安全性是基于大数进行因式分解十分困难，所以其密钥位数过长

目前互联网中主要使用的密钥交换协议是 ECDH，它是 DH 协议的升级，它是基于 ECC 椭圆曲线实现的

无论是 DH 协议还是 ECDH 协议，它们都是在 TLS/SSL 协议的握手中执行的。比如在TLS1.2 中在Client Hello和Server Hello中协商安全套件，在Server Key Exchange 中指定使用的椭圆曲线，P（基点），和Q（生成的公钥）

在TLS1.3 中直接在Change Cipher Spec 中交换椭圆曲线类型，P点和Q点

实际使用时，通常不传递P点。通常每个椭圆曲线的基点都是固定的，



### ECC 椭圆曲线

> 椭圆曲线函数在 ECDH 协议中的应用和 大数在 DH 中的应用类似
>
> 椭圆曲线函数在 ECDH 中的应用体现为：一个常数，一个变量 a，就能计算出变量 b。但是已知常数和变量 b 很难计算出 a。这表示 a 适合作为私钥，b 适合做公钥。能通过 a 生成 b，b 在网络中传递，攻击者获取 b 后却不能计算出 a

ECC 椭圆曲线图形并不像椭圆，但在计算其弧长等属性时，其计算方式和椭圆的计算方式相像，所以称为椭圆曲线



> 椭圆曲线的表达式： y<sup>2</sup> = x<sup>3</sup> + ax + b , 4a<sup>3</sup> + 27b<sup>2</sup> ≠ 0



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_19-37-32.png" alt="Snipaste_2021-02-18_19-37-32" style="zoom:50%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_19-38-02.png" alt="Snipaste_2021-02-18_19-38-02" style="zoom:50%;" />



**ECC 的关键原理** 

- Q =  K * P = K 个 P相加
  - 已知 K 与 P，能快速计算出  Q
  - 已知 Q 与 P，计算 K 的逆向运算非常困难
  - 所以 P 通常是常数；K 通常是密钥，且通常是大数；Q 是公钥



### TLS1.2 与 TLS1.3 中的 ECDH 协议

> ECDH 密钥交换协议
>
> - DH 密钥交换协议使用椭圆曲线后的变种，称为 Elliptic Curve Diffie–Hellman key Exchange，缩写为 ECDH，优点是比 DH 计算速度快、同等安全条件下密钥更短
> - ECC（Elliptic Curve Cryptography）：椭圆曲线密码学
> - 魏尔斯特拉斯椭圆函数（Weierstrass‘s elliptic functions）：y<sup>2</sup>=x<sup>3</sup>+ax+b



**ECDH 的步骤**

1. Alice 选定大整数 Ka 作为私钥（ Alice 创建私钥 a ）
2. 基于选定曲线及曲线上的共享 P 点，Alice 计算出 Qa=Ka.P（ Alice 创建公钥 a）
3. Alice 将 Qa、选定曲线、共享 P 点传递点 Bob（Alice 传递公钥 a 和曲线，共享点给 Bob）
4. Bob 选定大整数 Kb 作为私钥，将计算了 Qb=Kb.P，并将 Qb 传递给 Alice（ Bob 创建私钥 b，计算公钥 b，把公钥 b 传给 Alice ）
5. Alice 生成密钥 Qb.Ka = (X, Y)，其中 X 为对称加密的密钥（ Alice 创建新密钥 c ）
6. Bob 生成密钥 Qa.Kb = (X, Y)，其中 X 为对称加密的密钥（ Bob 创建新密钥 c ）

通信双方创建的两个新密钥能相等的理由：Qb.Ka = Ka.(Kb.P) = Ka.Kb.P = Kb.(Ka.P) = Qa.Kb



**ECDH 在 TLSv1.2 和 TLSv1.3 报文级别上的表现**

- TLSv1.2 下

  - 通信双方通过 Client Hello 和 Server Hello 进行安全套件协商

  - 通过 Server Key Exchange 和 Client Key Exchange 传递双方生成的公钥。下图只展示 Client Hello 和 Server Key Exchange 的报文

    （ Server Key Exchange 为什么没有被包含到[HTTPS 安全通信连接的建立过程](#HTTPS 安全通信连接的建立过程)中？）

    <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_20-31-56.png" alt="Snipaste_2021-02-18_20-31-56" style="zoom:47%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_20-31-12.png" alt="Snipaste_2021-02-18_20-31-12" style="zoom:45%;" />

  - TLS1.2 建立安全通信通道的详情[见这里](#HTTPS 安全通信连接的建立过程)

- TLS 1.3 下

  - TLS1.2 中安全套件的协商和公钥互换是分离的，TLS 1.3 将这两个阶段合并以减少一次 RTT

    - Client 发起握手前为每种安全套件都生成Private key 和 Public key，并在 Client Hello 中并将支持的套件类型信息全发送给 Server

    - Server 选择一种安全套件后返回创建椭圆曲线类型和自己的生成的 Public key 和并通过 Server Hello 返回给 Client

    - 因为 TLS1.3 支持的安全套件有限，所以才敢为支持的所有套件都创建公私钥并把所有公钥通过 Client Hello 发给 Server

      <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_21-02-57.png" alt="Snipaste_2021-02-18_21-02-57" style="zoom:50%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_21-05-10.png" alt="Snipaste_2021-02-18_21-05-10" style="zoom:50%;" />

  - TLS 1.3 常用的椭圆曲线是 X25519


<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_21-00-55.png" alt="Snipaste_2021-02-18_21-00-55" style="zoom:50%;" />

### TLS1.2 中的FREAK 攻击

512 位以下 RSA 密钥可轻易破解

中间人篡改客户端发送的 Client Hello 中支持的安全套件，让服务器只能选择一些简单的安全套件。如果使用的安全套件生成的RSA密钥长度少于512位，密钥将很容易破解



问题：中间人还能篡改客户端发出的Client Hello报文？？TLS1.3怎么克服了这个问题？？



### openssl 1.1.1 版本对 TLS1.3 的支持情况

Ciphersuites 安全套件 

- TLS13-AES-256-GCM-SHA384
- TLS13-CHACHA20-POLY1305-SHA256
- TLS13-AES-128-GCM-SHA256
- TLS13-AES-128-CCM-8-SHA256
- TLS13-AES-128-CCM-SHA256



### 测试 TLS 站点支持情况

**https://www.ssllabs.com/ssltest/index.html**





## 握手的优化：session 缓存、ticket 票据及 TLS1.3 的0-RTT

TLS1.2和1.3中有通用的手段优化握手，提高性能（session，ticket票据，0-RTT）



**Session 缓存**

TLS 1.2 中建立 TLS 链接时存在Server Key Exchange 和 Client Key Exchange 报文。这两个报文用于 Client 和 Server 互发公钥

Session 缓存是：浏览器和服务器建立会话时，浏览器保存一个SessionID，服务器用 map & kv 形式保存所有浏览器的 Session 会话状态

会话状态可保存 客户端和服务器建立连接时使用的两个公钥

当同一个客户端不是首次建立会话时，如果Session没过期，Client Hello 会携带 Session ID ，服务器命中 map 中的 session 后便不再发送Server Key Exchange 和 Client Key Exchange 报文，而发送 Change Ciper Spec 报文告知浏览器：使用上次的公钥即可

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_21-35-44.png" alt="Snipaste_2021-02-18_21-35-44" style="zoom:45%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-18_21-38-21.png" alt="Snipaste_2021-02-18_21-38-21" style="zoom:50%;" />

缺点：

- Session 存在共享问题
- Session 需要服务器保存在内存中，所以需要为 Session 设置合适的过期时间



**Session Ticket**

解释的不清楚，没听懂



**0-RTT**

0-RTT 是TLS 握手的报文和 HTTP 请求报文合并到一块。这样看起来就像是只发送 HTTP 请求报文而没有发送 TLS 报文

没有进行完整的 TLS 握手就发送 HTTP 请求，是因为 HTTP 请求报文是直接使用上次使用的加密方式进行加密的。客户端认为上次使用的密钥还能直接使用，如果服务器认可报文的加密方式，也认可密钥没有过期，则服务器直接返回 HTTP 响应



**优化方式所面临的重放攻击**

上述 3 种优化方式都面临着重放攻击。举个例子

POST 请求通常都会让接口程序执行修改用户在 DB 中的数据。黑客可能会获取某次 POST 请求后将其保存下来，在不破解报文的情况下直接使用此报文多次发送请求导致接口程序不断修改 DB 中的数据



## 量子通讯

计算机网络中的量子通讯和物理学中的量子力学之间没有关系，量子通讯知识接用了量子力学上的一些理念

量子通讯是理论上绝对安全的通讯方式。它基于概率学

### TLS 与量子通讯的原理

克劳德·艾尔伍德·香农的信息论证明了 one-time-pad（OTP）的绝对安全性。如果密钥满足以下条件，那么理论上是绝对安全的

- 密钥是随机生成的（伪随机的不行。然而通常的随机数生成函数都依赖于操作系统的时钟周期，这是伪随机）
- 密钥的长度大于等于明文长度（通常做不到。只能对明文分组，填充）
- 相同的密钥只能使用一次（session id ，session ticket ，0-RTT 都违反了此原则。这导致重放攻击称为可能）



虽然密钥很难做到上述 3 条，但是能尽量向其靠拢

假设现在有个密钥能满足上述 3 条，但是要绝对安全地传递密钥是困难的。量子通讯可以用于传递密钥，量子通讯是理论上绝对安全的通讯方式。它基于概率学





量子密钥分发 quantum key distribution，简称 QKD 

- 量子力学：任何对量子系统的测量都会对系统产生干扰
- QKD：如果有第三方试图窃听密码，则通信的双方便会察觉
  - 接收方接收到的数据的正确性会随着第三方的窃听而降低，如果通信被窃听，通信双方便能察觉到



### 量子通讯 BB84 协议的执行流程



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_11-18-15.png" alt="Snipaste_2021-02-19_11-18-15" style="zoom:67%;" />

场景：Alice 用量子通讯给 Bob 传递密钥。Alice 的 Basis 和 Bob 的 Basis 都是随机生成的

![Snipaste_2021-02-19_11-19-00](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_11-19-00.png)

1. 数据经过 Alice 的光栅变为不同的箭头
2. Bob 接收到箭头后，用自己的光栅进行解密
3. Bob将解密后的箭头发给 Alice
4. Alice 告诉 Bob 它解密的正确率并进行纠错（如果通讯没有被窃听，理论上错误率在 75%，如果低于 75%，Alice 会认为通讯被窃听）
5. Bob 接收纠错后，解出正确的密钥
6. 双方使用密钥进行对称加密通信





# 第 5 部分：TCP

> TCP 是传输层协议
>
> 以 TCP 链接的生命周期为主线介绍 TCP
>
> - 如何管理 TCP 连接
>   - 如何建立 TCP 连接
>   - 如何用 TCP 连接传递数据（传递数据）
>   - 拥塞控制是怎样进行的（拥塞控制）
>   - 如何关闭 TCP 连接（关闭连接）
> - TCP 的其他特性（keepalive，校验，外带数据，多路复用，四层负载均衡）
>
> 最后在使用层面对 TCP 进行一个总结



TCP 将消息拆分成多个 TCP segment 进行发送



## TCP 的设计哲学和作用

- TCP/IP 的前身 ARPA：NCP 协议
  - ARPA网络：原美国国防部建立的网络。是军工网络，容错率低
  - NCP协议不遵守OSI的分层，NCP协议柔和了多层的功能
- TCP/IP 协议发展
  - TCPv1 ~ TCPv3 都将 IP 功能包含了进来，直到 TCPv4 才将 IP 功能分离到网络层。于是才有了 TCPv4 + IPv4，这也是没有 IPv1~v3 的原因
- TCP 通常由操作系统的内核实现



### TCPv4 协议分层后的互联网世界

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_12-56-32.png" alt="Snipaste_2021-02-19_12-56-32" style="zoom:67%;" />

- 网络层的作用是：协助报文跨越不同的网络寻找目标主机
- 传输层的作用是：报文找到主机后如何向主机发送数据



### TCP/IP 的 7 个设计理念

PS：这些设计理念有优先级，1~7 优先级递减

> **David D Clark：《The Design Philosophy of The DARPA Internet Protocols》** 
>
> 1. Internet communication must continue despite loss of networks or gateways.
> 2. The Internet must support multiple types of communications service.
> 3. The Internet architecture must accommodate a variety of networks.
> 4. The Internet architecture must permit distributed management of its resources.
> 5. The Internet architecture must be cost effective.
> 6. The Internet architecture must permit host attachment with a low level of effort.
> 7. The resources used in the internet architecture must be accountable.
>
> TCP 的主要目的和设计原则是前 3 条
>
> - 能容错。例如滑动窗口，拥塞控制都是容错机制
> - 要求能支持不同类型的设备。比如：华为，中兴，小米等不同类型的手机和电脑
> - 能连接不同类型的网络。比如：wifi，海底光缆



### TCP 协议的分层

TCP：面向连接的、可靠的、基于字节流的传输层通信协议

- 面向连接：一台主机和另一台主机进行1对1的通信
- 基于字节流：以字节为单位，不限制数据长度/大小
- 可靠的：TCP 由队列的概念。属于同一个数据流的字节需要按队列排队接收，只有当接收前一个字节后才会接收下一个字节，如果先接收到下一个字节而没接收到前一个字节，就会一直等待前一个字节



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_14-31-55.png" alt="Snipaste_2021-02-19_14-31-55" style="zoom:50%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_14-35-25.png" alt="Snipaste_2021-02-19_14-35-25"  />

协议的分层结构使得 TCP 只要在报文中添加一个 TCP 头部即可实现其功能

PS：网络设备，主机，物理链路都是不可靠的网络传输，TCP 承担了保证网络传输安全的责任





### TCP 协议的特点

- 在 IP 协议之上，解决网络通讯可依赖问题
- 点对点（不能广播、多播），面向连接
- 双向传递（全双工）   HTTP/1.1 是半双工的，而 Websocket 是全双工的因为TCP将全双工功能暴露给了WS
- 字节流：打包成报文段、保证有序接收、重复报文自动丢弃
- 缺点：不维护应用报文的边界（对比 HTTP、GRPC）
- 优点：不强制要求应用必须离散的创建数据块，不限制数据块大小
- 流量缓冲：解决速度不匹配问题（通信双方所在的设备的性能是不同的 ，TCP 需要解决双方设备性能不同的问题）
- 可靠的传输服务（保证可达，丢包时通过重发进而增加时延实现可靠性）
- 拥塞控制



## <a id="TCP 报文格式">TCP 报文格式</a>

TCP Segment 报文段格式

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_18-38-25.png" alt="Snipaste_2021-02-27_18-38-25" style="zoom:50%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_16-04-40.png" alt="Snipaste_2021-02-19_16-04-40" style="zoom:67%;" />

- Sequence Number 和 Acknowledgment Number 都是 TCP Segment 报文段的唯一标识符
- Acknowledgment Number 还有标识报文可达性的作用。通常 SYN 报文中此值置 0 ，ACK 报文中此值才有意义
- 关于 Offset ，TCP Flags ，Window 在后面再说
- TCP Segment 报文段除固定的 20 字节外还有不定长的 Options，下图是 Option 的类型和一个 TCP 报文中 Options 的示意图
  - 每个 Option 都有一个 Length ，其值为整个 Option 的长度（以字节为单位），长度将 Length 值的长度也包括在内

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_16-07-08.png" alt="Snipaste_2021-02-19_16-07-08" style="zoom: 40%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_16-12-20.png" alt="Snipaste_2021-02-19_16-12-20" style="zoom:55%;" />



**如何标识一个连接**

TCP 报文使用四元组（源地址，源端口，目的地址，目的端口）

- 对于 IPv4 地址，单主机最大 TCP 连接数为 2(32+16+32+16) 

QUIC 协议使用 Connection ID （8 字节）



## tcpdump 分析网络报文

> Windows 下使用 Wireshark 进行抓包。在学习使用 Wireshark 进行远程抓包前先学习用 Linux 的 tcpdump 直接在应用服务器上进行本地抓包
>
> tcpdump 使用的捕获过滤器语法和 Wireshark 使用的过滤器语法相同，关于捕获过滤器的语法[看这里](#BPF 捕获滤器)即可

tcpdump 的使用主要包括

- 捕获过滤器语法
- 列举或选择网卡设备
- 捕获停止条件（限定只抓取多少条报文）
- 文件操作（将报文保存到文件中或读取文件中的报文；拆分报文文件，防止文件过大；设置报文中的时间戳格式）
- 分析报文详情（显示报文中的详细信息；以 16 进制/ Ascii 方式 显示报文；显示/不显示数据链路层/网络层的报文）

PS：报文文件可以被 Wireshark 读取

tcpdump 的选项和参数详情可见课件或 Utools 的 Linux 命令手册



**tcpdump 使用示例**

```shell
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

```shell
$ tcpdump host www.baidu.com
```



```shell
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





## TCP 的三次握手



### 三次握手建立链接

> 这里讲 3 次握手建立连接的
>
> - 目的
> - SYN 和 ACK 报文的格式，SYN 和 ACK 如何同步序列号

**握手的目的**

- 同步 Sequence 序列号
  - 初始序列号 ISN（Initial Sequence Number）
- 交换 TCP 通讯参数
  - 如 MSS、窗口比例因子、选择性确认、指定校验和算法



**SYN 报文和 ACK 报文的格式**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_17-38-29.png" alt="Snipaste_2021-02-19_17-38-29" style="zoom:50%;" />

三次握手（1）：SYN 的报文

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_17-39-22.png" alt="Snipaste_2021-02-19_17-39-22" style="zoom:67%;" />

- 将 Flags 中的 SYN 位 置 1
- Seq 中填上 Client 的 ISN

三次握手（2）：SYN/ACK 报文

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_17-45-31.png" alt="Snipaste_2021-02-19_17-45-31" style="zoom: 67%;" />

- 将 Flags 中的 SYN 位和 ACK 位 置 1
- Seq 中填上 Server 的 ISN
- Ack 中填上 Client 发送的 SYN 报文中的 ISN+1 的值

ACK 的报文

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_17-41-10.png" alt="Snipaste_2021-02-19_17-41-10" style="zoom:67%;" />

- 将 Flags 中的 ACK 位 置 1
- Ack 中填上 Server 发送的 SYN/ACK 报文中的 ISN+1的值

下图是 tcpdump 在抓包 3 次握手的 TCP 报文示例图

![Snipaste_2021-02-19_17-52-43](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_17-52-43.png)



问题

- 为什么 Client 和 Server 的 ISN 是不一样的？
- 为什么 Client 和 Server 的 INS 不是从 0 开始递增的？

答：网络通信中存在复制重发，延迟，丢失。为防止造成通信双方互相影响，初始的 ISN 采用随机值



### 三次握手过程中的状态变迁

TCP 连接有 5 种状态

- CLOSED（起始状态都是关闭状态）
- LISTEN（监听状态。服务器创建 socket 后监听端口便会进入此状态等待收发报文）
- SYN-SENT（发送 SYN 时进入此状态但时间极短，很难观察到）
- SYN-RECEIVED（服务器接收到 SYN 后进入此状态，直到接收到 ACK 后 ESTABLISHED）
- ESTABLISHED（成功建立连接后进入的状态）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-19_17-57-53.png" alt="Snipaste_2021-02-19_17-57-53" style="zoom:67%;" />



可以使用 netstat 查看本机 tcp 连接的状态

```shell
$ netstat -anp | grep tcp  # -a 查看所有套接字 -n 将域名或hostname替换为ip地址 -p 显示连接使用的Socket的程序识别码或程序名
```



**TCB**： Transmission Control Block，保存连接使用的源端口、目的端口、目的 ip、序号、 应答序号、对方窗口大小、己方窗口大小、tcp 状态、tcp 输入/输出队列、应用层输出队列、tcp 的重传有关变量等。是服务器通信时保存对端信息的数据结构



下面讲：在建立好 TCP 连接上如何可靠地传输数据



### 三次握手中的性能优化和安全问题

> - 调整超时时间和缓冲队列
> - Fast Open 降低时延
> - 如何应对 SYN 攻击
>   - 调整 Linux 内核参数，设置新的拒接策略
>   - tcp_syncookies
> - TCP_DEFER_ACCEPT



#### 操作系统内核中三次握手的流程



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-21_10-53-28.png" alt="Snipaste_2021-02-21_10-53-28" style="zoom:80%;" />

SYN 队列 和 ACCEPT 队列的长度体现了服务器能同时处理的连接数和请求数



#### 超时时间和缓冲队列

- 应用层 connect 超时时间调整
- 操作系统内核限制调整（ Linux 的内核参数可通过 /etc/sysctl.conf 文件进行修改）
  - 服务器端 SYN_RCV 状态
    - net.ipv4.tcp_max_syn_backlog：SYN_RCVD 状态连接的最大个数
    - net.ipv4.tcp_synack_retries：被动建立连接时，发SYN/ACK的重试次数（服务器收不到 ACK 后会尝试重发 SYN/ACK ）
  - 客户端 SYN_SENT 状态
    - net.ipv4.tcp_syn_retries = 6 主动建立连接时，发 SYN 的重试次数
    - net.ipv4.ip_local_port_range = 32768 60999 建立连接时的本地端口可用范围
  - ACCEPT队列设置（可通过 backlog 进行设置）



#### Fast Open 降低时延

Fast Open ：使用缓存方式减少同一个客户端和服务器进行非首次建立 TCP 连接的 TCP 握手次数

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-21_19-01-09.png" alt="Snipaste_2021-02-21_19-01-09" style="zoom:80%;" />

上图左侧为：Client 第 1 次和 Server 建立 TCP 连接时执行 3 次握手，Client 第 2 次建立连接时还要进行 3 次握手

右侧为：开启 Fast Open 后。Client 首次和 Server 建立 TCP 连接，Server 返回 Cookie ，其中保存了建立连接所协商的内容。当下次建立连接时，客户端直接复用 Cookie 中的设置（即默认上次协商的内容这次还能使用）。Client 发送应用层报文时直接把 Cookie 中的内容一并携带过去。即省略 3 次握手，直接发送应用层协议报文



net.ipv4.tcp_fastopen：系统开启 TFO 功能 

- 0：关闭
- 1：作为客户端时可以使用 TFO
- 2：作为服务器时可以使用 TFO
- 3：无论作为客户端还是服务器，都可以使用 TFO



#### 应对 SYN 攻击的手段



SYN 攻击：攻击者短时间伪造不同 IP 地址的 SYN 报文，快速占满 backlog 队列，使服务器不能为正常用户服务

**Linux 内核参数调优**

- net.core.netdev_max_backlog
  - 接收自网卡、但未被内核协议栈处理的报文队列长度
- net.ipv4.tcp_max_syn_backlog
  - SYN_RCVD 状态连接的最大个数
- net.ipv4.tcp_abort_on_overflow
  - 超出处理能力时，对新来的 SYN 直接回包 RST，丢弃连接



**tcp_syncookies**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-21_20-17-30.png" alt="Snipaste_2021-02-21_20-17-30" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-21_20-17-54.png" alt="Snipaste_2021-02-21_20-17-54" style="zoom:80%;" />

net.ipv4.tcp_syncookies = 1

当 SYN 队列满后，新的 SYN 不进入队列，计算出 cookie 再以 SYN+ACK 中的序列号返回客户端，正常客户端发报文时， 服务器根据报文中携带的 cookie 重新恢复连接 

- 由于 cookie 占用序列号空间，导致此时所有 TCP 可选功能失效，例如扩充窗口、时间戳等



#### TCP_DEFER_ACCEPT

TCP_DEFER_ACCEPT：延迟接收套接字

开启此功能后，Server 的操作系统和 Client 建立好 TCP 连接后，应用程序如 Nginx 或 Java 网络编程的 accept 就不会立即取出套接字。等到 Client 第一次发送应用层数据后 Server 的应用程序的 accept 才会生效（从 ACCEPT 队列中取出套接字）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-21_10-53-28.png" alt="Snipaste_2021-02-21_10-53-28" style="zoom:80%;" />



## TCP 协议上的数据传输



虽然 TCP 不限制数据流的长度，但是 TCP 层之下的网络层和数据链路层发送报文时所使用的长度是有限的，所以 TCP 会将从 应用层传来的任意长度数据流切分成若干报文段



这节讲拆分数据流的依据是什么，以及如何拆分报文段

**数据流分段的依据**

- MSS：防止 IP 层分段（如果 TCP 不对数据进行分段，那么分段的工作会交给 IP 层，而 IP 协议的分段效率低下）
- 流控：接收端的能力



MSS：Max Segment Size 

- 定义：仅指 TCP 承载数据，不包含 TCP 头部的大小，参见 RFC879
- MSS 选择目的
  - 尽量每个 Segment 报文段携带更多的数据，以减少头部空间占用比率
  - 防止 Segment 被某个设备的 IP 层基于 MTU 拆分（ MTU 会在 IP 协议中说）
- 默认 MSS：536 字节（默认 MTU576 字节，20 字节 IP 头部，20 字节 TCP 头部）
- 握手阶段协商 MSS
- MSS 分类
  - 发送方最大报文段 SMSS：SENDER MAXIMUM SEGMENT SIZE
  - 接收方最大报文段 RMSS：RECEIVER MAXIMUM SEGMENT SIZE

MMS 是在 TCP 3 次握手的报文中 Options 中设置的。Options 是 [TCP 报文格式](#TCP 报文格式)中的内容

Client 发送 SYN 报文中携带的 Options 会指定推荐的 MSS 值，Server 发送 SYN/ACK 的 Options 中返回实际使用的 MSS 值

Client 建议 MSS 的值为 1460

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-21_21-58-23.png" alt="Snipaste_2021-02-21_21-58-23" style="zoom:80%;" />

Server 决定实际使用的 MSS 的值为 1400

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-21_21-59-45.png" alt="Snipaste_2021-02-21_21-59-45" style="zoom:80%;" />



## 重传与确认

发送方发送的报文可能会在发送途中丢失，因此发送方需要有重发机制和确认接收端接收到报文的机制

> - 发送端发送数据后，如何确认接收端是否接收到？
> - 如果发送端觉得接收端没有接收到，发送端如何重发数据？
> - 重传与确认是怎么演化为滑动窗口？
> - 演化为滑动窗口后，TCP 报文中的序列号和确认序列号是什么？
> - 序列号的设计理念是什么？



### PAR 重发和确认机制

PAR：Positive Acknowledgment with Retransmission

设计思路：发送发发送报文后，启动定时器。接收端接收到报文后返回 ACK 通知发送端报文已经收到了。如果超出定时器指定的时间范围（超时）后，发送端还未接收到 ACK ，发送端会重发报文。**只有发送端接收到 ACK 后才会发送下一个报文**

缺点：串行发送报文导致效率低下

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_10-53-10.png" alt="Snipaste_2021-02-22_10-53-10" style="zoom:67%;" />



### 提升并发能力的 PAR 改进版

相对于 PAR 的改进之处：

- 发送方无需收到 ACK 就能发送下一个报文
- 为确认发送端收到的 ACK 是对哪条报文的确认响应，Message 和 Ack 报文都会通过序列号确认关系。使用的序列号是 Seq Number 和 Ack Number
- 并发传送导致接收方接收数据的压力增大，为确保接收方是否有能力再接收报文，ACK 报文会指出接收端还能接收多少字节（TCP 报文格式中，用 Window 表示接收端还能接收多少字节）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_10-57-58.png" alt="Snipaste_2021-02-22_10-57-58" style="zoom: 80%;" />

这个解决方案并不完美，逐渐优化方案，直到演化出滑动窗口，更好地解决了接收端接收大量报文的问题



### Sequence 序列号/Ack 序列号

- 设计目的：解决应用层字节流的可靠发送

  - 跟踪应用层的发送端数据是否送达
  - 确定接收端有序的接收到字节流

- 序列号的值针对的是字节而不是报文（序列号的值不是每发一个报文就 +1）

  下一个序列号 = 当前序列号 + TCP Segment Len    如下图：2030 = 644 + 1386

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_11-11-32.png" alt="Snipaste_2021-02-22_11-11-32" style="zoom:80%;" />

- 序列号占用的位数有限。当发送的字节数超过序列号能表示的最大值后，序列号从 ISN 开始复用

  - 缺点：在长肥网络下，存在 PAWS（ Protect Against Wrapped Sequence numbers ）问题
  - PAWS 问题：某次发送报文时使用序列号 a ，但是发送失败。由于序列号复用，之后某次发送报文再次用到序列号 a ，接收端可能会误以为这次的报文是上次报文的重发
  - PAWS 问题的解决方案：为每个报文添加时间戳。TCP timestamp 是 TCP 报文中的可选 Option



### RTO重传定时器的计算

改进版的 PAR 使用定时器设置超时时间，超时时间的设定比较重要

使用 TCB 计算 RTT 的时间

如果 RTT 时间在 RTO 范围内，则认定报文发送成功。所以下面说下 RTT 的测量

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_11-23-30.png" alt="Snipaste_2021-02-22_11-23-30" style="zoom:80%;" />

测量方式：通过时间戳（ TCP timestamp ）测量



如果 RTO 设置的较小或较大就会出现上图中的问题



RTO 的计算方式（看不懂，也不需要懂）

- 平滑 RTO：RFC793，降低瞬时变化（多数操作系统都不使用这个方案）
  - SRTT （smoothed round-trip time） = ( α * SRTT ) + ((1 - α) * RTT) 
  - α 从 0到 1（RFC 推荐 0.9），越大越平滑
  - RTO = min[ UBOUND, max[ LBOUND, (β * SRTT) ] ]
  - 如 UBOUND为1分钟，LBOUND为 1 秒钟， β从 1.3 到 2 之间
  - 不适用于 RTT 波动大（方差大）的场景
- 追踪 RTT 方差（Linux 使用这个方案）
  - RFC6298（RFC2988），其中α = 1/8， β = 1/4，K = 4，G 为最小时间颗粒： 
  - 首次计算 RTO，R为第 1 次测量出的 RTT
    - SRTT（smoothed round-trip time） = R
    - RTTVAR（round-trip time variation） = R/2
    - RTO = SRTT + max (G, K*RTTVAR) 
  - 后续计算 RTO，R’为最新测量出的 RTT
    - SRTT = (1 - α) * SRTT + α * R’
    - RTTVAR = (1 - β) * RTTVAR + β * |SRTT - R’|
    - RTO = SRTT + max (G, K*RTTVAR)



### 滑动窗口：发送窗口和接收窗口

滑动窗口源于改进版的 PAR，但是要比它复杂很多，功能也很强大



举例：滑动窗口：发送窗口快照 

1. 已发送并收到 Ack 确认的数据：1-31 字节
2. 已发送未收到 Ack 确认的数据：32-45 字节
3. 未发送但总大小在接收方处理范围内：46-51 字节
4. 未发送但总大小超出接收方处理范围：52-字节

![Snipaste_2021-02-22_13-39-05](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_13-39-05.png)



#### 发送窗口

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_14-18-22.png" alt="Snipaste_2021-02-22_14-18-22" style="zoom:80%;" />

PS：上图中，黑框代表窗口，框内的字节代表窗口内的数据

发送窗口中有 3 个重要的变量

- SND.WND   窗口能容纳的字节数
- SND.UNA    指向发向对端但还未收到 ACK 的数据的第一个字节的指针
- SND.NXT     指向将要发送但还未发送的数据的第一个字节的指针



#### 接收窗口

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_14-22-53.png" alt="Snipaste_2021-02-22_14-22-53" style="zoom:80%;" />

接收窗口中有 2 个重要的变量

- RCV.WND    接收窗口能容纳的字节数
- RCV.NXT      指向下一个将要接收数据的地址的指针



#### 滑动窗口与流量控制

实际演示例子展示发送窗口和接收窗口

为简化过程，设定如下限制：MSS 不产生影响，窗口不变

场景：客户端发送一个请求，服务器分别返回响应头部和包体

1. 客户端发送 140 字节
2. 服务器接收 140 字节数据，发送 80 字节响应头部和 ACK （稍后发送 280 字节的包体）
3. 客户端接收 80 字节响应头部和 ACK，并返回 ACK
4. 服务器发送包体的前 140 字节
5. 客户端接收前 140 字节的包体并返回 ACK
6. 服务器接收到第 2 步发送的头部的 ACK
7. 服务器接收到第 4 步发送的一部分包体的 ACK
8. 服务器发送包体的后 160 字节数据
9. 客户端接收包体后 160 字节的数据，并返回 ACK
10. 服务器接收到到第 8 步的 ACK

![Snipaste_2021-02-22_14-34-22](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_14-34-22.png)



左图为客户端消息的发送，右图为服务器消息的发送

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_14-43-13.png" alt="Snipaste_2021-02-22_14-43-13" style="zoom:80%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_14-43-20.png" alt="Snipaste_2021-02-22_14-43-20" style="zoom:80%;" />



#### 操作系统缓冲区与滑动窗口的关系

- 滑动窗口存放在操作系统的缓冲区中
- 操作系统的缓冲区会被操作系统调整
- 应用进程无法读取数据时也会对缓冲区造成影响，比如：窗口收缩

这节讲，操作系统的缓冲区时如何影响发送窗口和接收窗口的



Server 的窗口接收到数据后，应用层程序如果没有及时读取缓存将会导致窗口收缩

窗口收缩后，ACK 报文中使用 Window 字段指明接收端当前的窗口的大小



问题：为什么应用程序不及时读取缓存会导致窗口收缩，而不是窗口的可用大小减小？



**收缩窗口导致的丢包**

窗口收缩后还未来得及通知对端，对端就发来新的数据

由于对端发送数据时还不知道接收端窗口收缩，新的数据长度可能大于当前窗口。窗口无法容纳这些数据，操作系统将直接丢弃包中数据

操作系统为解决这种问题，采用如下手段解决（这些手段保证了上述方式的丢包情况将不会发生）

- 先收缩窗口，再减少缓存
- 接收端窗口关闭后，发送端定时发送探测包探测接收端窗口是否开启



**窗口设置为多少比较合适**

设置为 bps * RTT

比如带宽（bps）为100M，时延的平均值在 1s 左右，窗口设置为 100M



缓存不仅要为窗口分配空间，还要为应用缓存分配空间

```
net.ipv4.tcp_adv_win_scale = 1  # 这是应用缓存和窗口空间的比例
```

应用缓存 = buffer / （2^tcp_adv_win_scale）



**Linux中对TCP缓冲区的调整方式**

- net.ipv4.tcp_rmem = 4096 87380 6291456
  - 读缓存最小值、默认值、最大值，单位字节，覆盖 net.core.rmem_max
- net.ipv4.tcp_wmem = 4096 16384 4194304
  - 写缓存最小值、默认值、最大值，单位字节，覆盖net.core.wmem_max
- net.ipv4.tcp_mem = 1541646 2055528 3083292
  - 系统无内存压力、启动压力模式阀值、最大值，单位为页的数量
- net.ipv4.tcp_moderate_rcvbuf = 1
  - 开启自动调整缓存模式



## 减少小报文提高网络效率

这里说的小报文既有发送端主动发送多个小报文，也有因对端滑动窗口空间不足导致发送端不得不将数据拆分成多个小报文，还有为每个非 ACK 报文回复一个 ACK（单纯的 ACK 报文作用单一，最少 20 字节）



### SWS（ Silly Window syndrome ）糊涂窗口综合症

场景：Server 工作繁忙时，Client 用 TCP 发送一段较大的数据

Client 根据 Server 的窗口大小拆出一段数据发送，由于 Server 工作繁忙，应用层程序无法及时读取数据导致窗口收缩。Client 每次发送数据都会引起 Server 的窗口收缩导致 Client 发送的数据越来越少。这段数据别拆分到多个报文中，且报文中的数据量越来越少

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-22_17-11-08.png" alt="Snipaste_2021-02-22_17-11-08" style="zoom:80%;" />



### SWS 避免算法

避免算法用于解决 SWS 问题

接收方

- 采用David D Clark 算法：窗口边界移动值小于 min（MSS, 缓存/2）时，通知窗口为 0

发送方

- 采用 Nagle 算法
  - 没有已发送未确认报文段时，立刻发送数据
  - 存在未确认报文段时，直到：没有已发送未确认报文段，或数据长度达到 MSS 时再发送
  - TCP_NODELAY 用于关闭 Nagle 算法



### TCP delayed acknowledgment 延迟确认

- 当有响应数据要发送时，ack 会随着响应数据立即发送给对方

- 如果没有响应数据，ack 的发 送将会有一个延迟，以等待看是否有响应数据可以一起发送（等待时间和服务器的时钟周期有关）

  ```shell
  cat /boot/config-`uname -r` | grep '^CONFIG_HZ=' # 可用此命令查看
  ```

- 如果在等待发送 ack 期间，对方的第二个数据段又到达了，这时要立即发送 ack



### Nagle 和 delayed ACK 的矛盾

Nagle 需要等接收到 ACK 后再发送数据，而延迟 ACK 会延迟返回 ACK。为解决矛盾，需要关闭其中一个

- 关闭 delayed ACK：TCP_QUICKACK
- 关闭 Nagle：TCP_NODELAY





## 拥塞控制

> - 什么是拥塞，拥塞控制是什么
> - 慢启动
> - 拥塞避免
> - 快速重传
> - 快速恢复

由于 TCP 不限制传输的字节流的长度。所以当大量 TCP 连接发送大量字节流时会造成网络拥塞

**拥塞控制历史**

- 以丢包作为依据
  - New Reno：RFC6582
  - BIC：Linux2.6.8 – 2.6.18
  - CUBIC（RFC8312）：Linux2.6.19
- 以探测带宽作为依据
  - BBR：Linux4.9



### 慢启动

开启慢启动后：

- 发送窗口和接收窗口的初始大小为若干个 MSS （当前初始值大概是 10 个 MSS ）
- 窗口大小很小，所以前几次发送的数据量很小。如果发送的数据都顺利得到 ACK，慢启动会尝试扩大窗口大小
- 通俗点讲：慢启动就是刚开始传数据时，“悠着点”传少量数据。如果数据都能返回 ACK 表示网络良好，发送端将尝试发送更多数据



- 拥塞窗口cwnd（congestion window）
  - 通告窗口 rwnd（receiver‘s advertised window）
  - 发送窗口 swnd = min(cwnd，rwnd)
- 每收到一个 ACK，cwnd 扩充一倍

缺点：cwnd 扩充一倍后，传输的数据量激增，可能出现大量丢包

慢启动如果出现问题会导致丢包数量很大，拥塞避免能解决这个问题



### 拥塞避免

拥塞避免：为 cwnd 设置一个阈值，当 cwnd 超过阈值后，cwnd 开始线性式增长（之前是指数式增长）。如果出现大量丢包，阈值减半，cwnd 大幅减小并恢复指数式增长（重新进入慢启动）

慢启动阈值 ssthresh（slow start threshold）：

- 达到 ssthresh 后，以线性方式增加 cwnd
- cwnd += SMSS*SMSS/cwnd

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-23_11-06-35.png" alt="Snipaste_2021-02-23_11-06-35" style="zoom:80%;" />



### 快速重传和快速回复



失序数据段：连续发送 3 个数据段，结果数据段 2 发送失败，那么数据段 3 就是失序数据段。因为数据段 3 前应该存在的数据段 2 丢失了

接收到失序数据段的原因

- 若报文丢失，将会产生连续的失序 ACK 段（拥塞控制主要解决的是丢包问题，主要解决这种情况）
- 若网络路径与设备导致数据段失序，将会产生少量的失序 ACK 段
- 若报文重复，将会产生少量的失序 ACK 段

接收端接收到数据段后，返回 ACK ，ACK 中会告诉发送端接下来要发送哪个数据段



**快速重传（RFC2581）**

接收方： 

- 当接收到一个失序数据段时，立刻发送它所期待的缺口 ACK 序列号
- 当接收到填充失序缺口的数据段时，立刻发送它所期待的下一个 ACK 序列号
- 快速重传要求接收方满足上述 2 种情况就立即返回 ACK，所以不能使用 TCP 的延迟发送

发送方：

- 当接收到 3 个重复的失序 ACK 段（4 个相同的失序 ACK 段）时，不再等待重传定时器的触发，立刻基于快速重传机制重发报文段

![Snipaste_2021-02-23_11-16-26](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-23_11-16-26.png)



快速重传下一定要进入慢启动吗？ 

- 收到重复 ACK，意味着网络仍在流动。（没有收到指定的 ACK 只能说明出现少量丢包，而不是大量丢包）
- 慢启动会突然减少数据流

所以没必要进入慢启动



**快速恢复**

使用条件：启动快速重传且正常未失序 ACK 段到达前，启动快速恢复

通俗点讲：出现丢包时进行快速重传，在重传期间降低网速（减小 cwnd）就是快速回复的作用

- 将 ssthresh 设置为当前拥塞窗口 cwnd 的一半，设当前 cwnd 为 ssthresh 加上 3*MSS
- 每收到一个重复 ACK，cwnd 增加 1 个 MSS
- 当新数据 ACK 到达后，设置 cwnd 为 ssthresh

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-23_11-32-16.png" alt="Snipaste_2021-02-23_11-32-16" style="zoom:80%;" />



## SACK 与选择性重传算法

TCP 的序列号采用累计确认

快速重传中，接收端返回的 ACK 会携带它所期待的缺口 ACK 序列号。比如接收端期望 a 报文段

在重复接收 a 报文段缺失的消息后，发送端知道了 a 报文段传送失败。到那时发送端并不知道 a 后的 b，c 报文段是否发送成功

### 选择性重传算法

当发送端重发报文段时有两种选择

- 只重发 a（乐观）
- 重发 a，b，c（悲观。浪费带宽，大量丢包时效率低下）



### SACK

SACK：TCP Selective Acknowledgment 

接收端返回 ACK 时，除携带希望发送端发送的下一个报文段序号外还携带接收端已经接收到了哪些报文段。这样发送端就不会重复发送成功接收到的报文段

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_10-37-20.png" alt="Snipaste_2021-02-27_10-37-20" style="zoom: 67%;" />



下面是一次丢包和 SACK 的示意图

- ACK Number 表示希望对端下次发送的报文序列号
- SACK 的 left edge 和 right edge 是左闭右开的区间
- 在 Wireshark 中第 3 个报文是 TCP Previous segment not captured。是发送端发送报文时，Wireshark 发现发送端上次发送的报文没被抓到
- 在 Wireshark 中第 5,6 个报文是 TCP Dup ACK 。是接收端发送的 SACK 报文

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/%E6%9C%AA%E5%91%BD%E5%90%8D%E7%BB%98%E5%9B%BE.png" alt="未命名绘图" style="zoom:80%;" />



SACK 也可以用于拥塞控制的快速重传



## 基于测量的 BBR 拥塞控制算法



之前的拥塞控制是基于丢包执行的。当出现丢包后，系统将降低发包速率，快速重发，快速恢复等

BBR 拥塞控制通过测量最佳速率进行发包。操作系统按 FIFO 顺序发送 TCP 报文，等待发送的报文被放在队列中，在队列中的报文的 RTT 要比一个直接发送的报文的 RTT 长，所以最佳的发包速率所处的状态为队列尽可能保持为空的状态

网络状态发生改变或 TCP 连接使用的数据链路改变都会引起 RTT的大幅改变，因此 BBR 算法提供动态测量最佳发包速率的功能，以便随时修改发包速度



## TCP 的 4 次握手关闭连接和状态变迁



关闭连接时使用 4 次握手的目的：防止数据丢失；通知应用层优雅关闭连接（优雅关闭 Socket）

- FIN 包：结束包
- ACK 包：确认包



### 正常关闭连接

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_12-32-33.png" alt="Snipaste_2021-02-27_12-32-33" style="zoom:80%;" />

1. Client 发送 FIN 报文，进入 FIN-WAIT-1 状态
2. Server 接收到 FIN 包后告诉应用层准备关闭连接并进入 CLOSE-WAIT 状态，返回 ACK 报文。如果应用程序一直不对 FIN 包做处理，TCP 连接将一直处于 CLOSE-WAIT 状态
3. Client 接收到 ACK 报文后进入 FIN-WAIT-2 状态
4. Srever 的应用层程序调用`socket.close()`后发送 FIN 报文，进入 LAST-ACK 状态
5. Client 接收到 FIN 报文后返回 ACK 报文并进入 TIME-WAIT ，并开始计时，等待 2 个 MSL 后关闭连接并进入 CLOSED 状态
6. Server 收到 ACK 报文后关闭连接

PS：MSL 是一个报文在网络中最大的存活时间

上述握手都有支持重发和确认流程



### 通信双方同时主动关闭连接

双方会进入一个 CLOSING 中间状态

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_13-05-54.png" alt="Snipaste_2021-02-27_13-05-54" style="zoom:80%;" />



### TCP 状态机 & TCP 3 次握手和 4 次握手总结

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_13-08-10.png" alt="Snipaste_2021-02-27_13-08-10" style="zoom:80%;" />



### TIME-WAIT 优化

TIME-WAIT 最长可达两个 MSL，对于处理并发的服务器来说需要减少处于 TIME-WAIT 状态的端口。所以 TIME-WAIT 不能设置太长/大

问：为什么 TIME-WAIT 要设置为两个 MSL，而不能在短些

答：如下解释。如果 TIME-WAIT 很短或不存在会发生以下严重错误

1. 第一个 TCP 连接中有个报文发送出现延迟
2. 连接关闭后应用程序又**立刻复用**端口建立了 TCP 连接
3. 之前的报文由于延迟，接收端现在才收到报文

但是这个迟到的报文是第一个连接的报文，不是第二个连接的。这导致了两次连接上的报文混合，出现错乱。所以 TIME-WAIT 是有保护作用的

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_17-31-55.png" alt="Snipaste_2021-02-27_17-31-55" style="zoom:80%;" />

- MSL(Maximum Segment Lifetime)
  - 报文最大生存时间
- 维持 2MSL 时长的 TIME-WAIT 状态
  - 原因：保证至少一次报文的往返时间内端口是不可复用



**Linux 下能对 TIME-WAIT 的优化操作**

Linux 对 TCP 层的优化都是通过配置文件实现的

- net.ipv4.tcp_tw_reuse = 1
  - 开启后，作为客户端时新连接可以使用仍然处于 TIME-WAIT 状态的端口
  - 由于 timestamp 的存在，操作系统可以拒绝迟到的报文。需配置 net.ipv4.tcp_timestamps = 1
  - 此选项危险性小
- net.ipv4.tcp_tw_recycle = 0
  - 开启后，同时作为客户端和服务器都可以使用 TIME-WAIT 状态的端口
  - 不安全，无法避免报文延迟、重复等给新连接造成混乱。所以不建议开启此功能
- net.ipv4.tcp_max_tw_buckets = 262144
  - time_wait 状态连接的最大数量
  - 超出后直接关闭连接
- 使用 RST 复位报文关闭连接，而不使用 4 次握手关闭连接
  - 使用场景：进程突然关掉或其他严重问题



## keepalive

这里只讲 Keep-Alive 功能在 Linux 内核中的参数说明和 keepalive 断开连接的依据

- 发送心跳周期
  - net.ipv4.tcp_keepalive_time = 7200   # 单位为秒
- 探测包发送间隔
  - net.ipv4.tcp_keepalive_intvl = 75
- 探测包重试次数
  - net.ipv4.tcp_keepalive_probes = 9

如果 TCP 连接上 keepalive_time 内没有发送任何数据就启动探测，根据探测结果决定是否关闭连接

发送端将发送多个探测包，如果对端能返回应答，这个连接将再等待一个 keepalive_time

如果没有应答，发送端将重试。超过重发次数后还没收到响应就断开连接



## PUSH 报文

比如发送端发送10MB数据，数据被分成多个Segment，最后一个报文会将PSH位设为1

含义是数据已发送完毕，通知操作系统尽快读取缓冲区的数据，因为已经接收到了一个完整的数据

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_21-43-58.png" alt="Snipaste_2021-02-27_21-43-58" style="zoom:67%;" />



## 校验和

TCP 报文中提供校验和，它会校验 TCP Segment 中的 Data，Header，甚至会校验 IP 层的报文。这违反了分层原则

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_18-38-25.png" alt="Snipaste_2021-02-27_18-38-25" style="zoom:50%;" />



## 带外数据

也叫带外数据，但通常不叫它带外数据。**功能**：用于紧急处理数据

使用场景：

- 比如在Linux中执行一些远程通信的指令，执行时用户输入 Ctrl +c，就会发送USR，对端优先执行某些操作（停止远程通信并终止命令的执行）
- 在下载文件时点击取消下载，就会发送 USR ， 发送端会优先执行一些操作（比如终止传输文件数据）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_21-44-52.png" alt="Snipaste_2021-02-27_21-44-52" style="zoom:50%;" />



## TCP 实现多路复用

TCP 是面向不定长的字节流的协议，因此对它进行多路复用是较为困难的

多路复用：在一个信道上传输多路信号或数据流的过程和技术



- HTTP2 的多路复用针对的是 htpp 请求

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_21-49-17.png" alt="Snipaste_2021-02-27_21-49-17" style="zoom: 67%;" />

- TCP 上的多路复用是针对编程而言的。非阻塞 Socket 用于同一时间处理多个 TCP 连接，但还需要借助 epoll 模型。这里不对 epoll 模型作介绍和分析



非阻塞 Socket：同时处理多个 TCP 连接

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_21-52-30.png" alt="Snipaste_2021-02-27_21-52-30" style="zoom: 50%;" />



## epoll 模型



### epoll 模型和操作系统实现的 TCP Socket

[epoll 模型参考博客](https://my.oschina.net/u/4299156/blog/3233158)

- epoll 出现：linux 2.5.44
- 进程内同时刻找到缓冲区或者连接状态变化的所有 TCP 连接
- epoll 对外提供 3 个 API 供用户（操作系统程序员，编程语言维护人员）使用
  - epoll_create
  - epoll_ctl
  - epoll_wait

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_21-57-35.png" alt="Snipaste_2021-02-27_21-57-35" style="zoom:67%;" />



### epoll 高效的原因

epoll 认为：服务器可能同一时间建立并管理了大量的 TCP 连接，但是同一时间内活跃的连接（进行收发数据或状态变迁的连接）数只占一小部分

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_22-01-25.png" alt="Snipaste_2021-02-27_22-01-25" style="zoom: 50%;" />

epoll 的两个核心数据结构

- 红黑树：存放所有连接的 socket
- 队列：存放所有发生变化的连接/活跃的连接



epoll 模型把连接的套接字放到红黑树中。当连接活跃时，操作系统把连接放到队列里等待激活线程后让连接对应的应用层程序处理连接中的数据



有了非阻塞 socket ，异步编程看起来就比较麻烦了。因此常使用 非阻塞 + epoll + 同步编程 = 协程/NIO



下图代码是 OpenResty 中的一段代码。建立连接时需等待对端返回 ACK ，但是线程不会真的等待而是切换到其他地方执行任务，等到收到 ACK 后 epoll 提醒线程回来继续执行这段代码

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-27_22-07-49.png" alt="Snipaste_2021-02-27_22-07-49" style="zoom:67%;" />



下节讲 TCP 层怎么做 LoadBlance



## 四层负载均衡

下面介绍工作在传输层的伪 TCP，UDP 服务的四层负载均衡



### OSI 模型下的七层 LB 与四层 LB

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-28_14-58-12.png" alt="Snipaste_2021-02-28_14-58-12" style="zoom:67%;" />

第 3 部分介绍过七层负载均衡



### 四层负载均衡与表示层的 TLS 卸载

许多四层负载均衡还能解析到表示层，因为有解析 TLS 的需求

企业内网中天然保证安全性，所以请求进入内网的负载均衡后卸载掉 TLS 能减少负担

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-28_14-58-59.png" alt="Snipaste_2021-02-28_14-58-59" style="zoom:67%;" />



### 四层负载均衡与连接五元组

TCP 使用四元组确定通信两端的主机和端口，四层负载均衡/传输层 通过五元确定通信两端

- Source IP
- Destination IP
- Source Port
- Destination Port
- Protocol  （传输层使用的协议，比如：TCP，UDP）



### 三层路由器与四层负载均衡

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-28_15-04-39.png" alt="Snipaste_2021-02-28_15-04-39" style="zoom:67%;" />



### 多层 LB

LB 可以在外网和内网中同时部署，使用。内网中的 LB 就是信任的边界，因为内网相对安全，LB 课卸载请求中的 TLS 以减少性能消耗



# 第 6 部分：IP 层和以太网

> 目的不是彻底明白 OSI 模型的第 2 ，3 层协议。而是了解网络中的节点是怎么连接起来的，报文的传输成本有多大（尤其是细腰结构下的3 层 IP 协议）
>
> 课程分两部分
>
> - 前半部分：报文如何在网络中传输，交换机和路由器在网络中起什么作用，如何应对 IPv4 地址短缺问题
> - 后半部分：解析 IPv4 协议和 ICMP 协议，了解 IP 层的分片是怎么编码的，多播和广播是怎么进行的，IPv6 有什么新特性

> 下面的网络层的`IP`协议会讲这些内容
>
> - `IP`报文与路由
> - 网络层其他常用协议：`ICMP`、`ARP`、`RARP`
> - `IPv6`的区别



## 网络层和数据链路层的功能



### 网络层和数据链路层的作用

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-01-27_12-51-13.png" alt="Snipaste_2021-01-27_12-51-13" style="zoom:80%;" />

**网络层的作用**：网络层用于将本地网络连接起来并组成 Internet 网络

- IP寻址：用 IP 地址寻找主机所在的网络和主机
- 选路：到达目标主机的路径有很多，如何选择最短路径
- 封装打包：添加，解析 IP 头部
- 分片：如果传输层传来的报文大小超过 MPU，对报文进行分片

**数据链路层的作用**：数据链路层和网卡，电缆等物理设备直接打交道

- 逻辑链路控制
- 媒体访问控制
- 封装链路层帧
- MAC 寻址
- 差错检测与处理
- 定义物理层标准

数据链路层从网络中接收到的是一段字节组成的 package，但是数据在链路层中是一个比特位一个比特位传输的所以他需要把报文分装成帧



### IP 协议的细腰结构

在 OSI 分层协议模型中，网络层实质上只有 IP 协议。网络层以上的协议都是基于 IP 协议的。因此 IP 协议追求性能

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-01_08-18-31.png" alt="Snipaste_2021-03-01_08-18-31" style="zoom:67%;" />

IP 协议通过：无连接，非可靠，无确认提高性能



### 多播：广播与组播

之前说的都是单播

多播能指定范围：本机作用域，本地链路层，场点内，组织内，全球作用域（广域网）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-01_08-24-34.png" alt="Snipaste_2021-03-01_08-24-34" style="zoom:67%;" />



### 路由器与交换机

工作在网络层的路由器

- 连接不同网络的设备

工作在数据链路层的交换机

- 同一个网络下连接不同主机的设备

网络传输中不关心 IP 地址时，我们就工作在数据链路层，交换机把不同的主机连接在一起，交换机工作在数据链路层。如果需要跨网络传数据，就需要网络层，需要路由器跨越网络，主要与 IP 地址，IP 协议打交道



### 网络传输示例

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-01_08-35-45.png" alt="Snipaste_2021-03-01_08-35-45" style="zoom: 50%;" />

场景：Client Host 要发报文给 Server

1. 获取本地网关找到一台路由器：10.2.14.1
2. 路由器进入骨干网，XXX协议在骨干网中找最短路径
3. 到达其内网，通过其路由器找到 Server



## IP 地址分类

IP 地址很充裕时，通过将 IP 地址划分为网络地址 + 主机地址，方便路由器寻找主机所在的网络



### IP 地址的分配机构

IP 地址由 IANA 机构管理，IANA 将指定范围的 IP 地址分配给某个区域，比如把 120.xxx.xxx.xxx 分配给亚洲

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-01_08-39-10.png" alt="Snipaste_2021-03-01_08-39-10" style="zoom:80%;" />



### IPv4 分类地址

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-01_08-40-24.png" alt="Snipaste_2021-03-01_08-40-24" style="zoom:80%;" />



### CIDR 子网掩码无分类地址

CIDR（ Classless Inter-Domain Routing ）

子网掩码用于表示 IP 地址的前几位是网络地址，子网掩码有两种表现形式

- 表示形式和Ipv4一样，32位二进制，8位一组，用.分割。是 1 的位对应的IP地址中的位是网络标识，是 0 的标识是主机标识
- 子网掩码另一种表示方法是 0~32 的一个十进制数字，表示 IP 地址的前 n 位是网络地址



CIDR 子网掩码 71.94.0.0/15 多级子网划分示意图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-01_08-47-48.png" alt="Snipaste_2021-03-01_08-47-48" style="zoom:67%;" />



### 预留 IP 地址

全 0 或者全 1 的特殊含义

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-01_08-49-47.png" alt="Snipaste_2021-03-01_08-49-47" style="zoom:80%;" />

全 0 地址经常被用于 bind 操作

预留 IP 地址（RFC1918）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-01_08-51-04.png" alt="Snipaste_2021-03-01_08-51-04" style="zoom: 80%;" />

预留 IP 地址中，分类地址常用于企业内网

- A 类地址，比如阿里的内网中为阿里云提供 A 类地址
- B 类地址，比如公有云服务器中给我们分配的内网地址，我的阿里云服务器的内网地址就是 172.24.61.41
- C 类地址，常用于家用网络中的内网地址。比如家用路由器的默认 IP 就是 192.168.0.0 或 192.168.0.1



## IP 地址与链路地址的转换：APR 与 RAPR 协议

将 MAC 地址和 IP 地址连接起来



### 链路层 MAC 地址

链路层地址 MAC（Media Access Control Address） 

- 实现本地网络设备间的直接传输 

网络层地址 IP（Internet Protocol address） 

- 实现大型网络间跨网络的传输

查看 MAC 地址的指令

- Linux：ifconfig
- Windows: ipconfig /all 



### 2.5 层协议 ARP：从 IP 地址寻找 MAC 地址

- 动态地址解析协议 ARP （ Address Resolution Protocol ）（RFC826） 
- 因为用于 IP 地址（第 3 层）和 MAC 地址（第 2 层）的转换，所以 ARP 被视为 2.5 协议

#### APR 寻址流程

使用场景：已知目标主机的 IP 地址，想要获取目标的 MAC 地址。以主机 A 想要获取主机 B 的 MAC 地址为例

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_14-21-52.png" alt="Snipaste_2021-03-02_14-21-52" style="zoom:67%;" />

1. A 向网络中发送广播，询问主机 B 的 MAC 地址，广播报文中携带 B 的 IP 地址
2. B 收到广播后返回响应报文（单播），报文中携带有自己的 MAC 地址
3. C 和 D 收到广播后不返回响应，因为广播报文中的 IP 不是自己的 IP
4. A 得到 B 的 MAC 地址后会将其缓存起来

PS：

- 得到的 MAC 地址被用于向其进行网络通信时组建报文，在数据链路内寻找主机
- Linux 中查询本地 APR 缓存可使用 `apr -nv`



#### APR 报文格式：FrameType=0x0806 

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_14-30-59.png" alt="Snipaste_2021-03-02_14-30-59" style="zoom:80%;" />

- Hardware Type：硬件类型，如 1 表示以太网
- Protocol Type：协议类型，如 0x0800 表示 IPv4
- Hardware Address Length：硬件地址长度，如 6 能标识 MAC 地址使用的 6 个字节
- Protocol Address Length：协议地址长度，如 4 表示 IPv4 使用的 4 个字节
- Opcode：操作码，如 1 表示请求，2 表示应答
- Sender Hardware Address：发送方硬件地址
- Sender Protocol Address：发送方协议地址
- Target Hardware Address：目标硬件地址
- Target Protocol Address：目标协议地址



#### APR 寻址抓包示例



Wireshark 使用 "arp" 作为捕获过滤器

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_14-49-57.png" alt="Snipaste_2021-03-02_14-49-57" style="zoom: 67%;" />

发送广播查找 IP 为 192.168.0.114 的目标主机（因为不知道目标的 MAC 地址，所以报文中目标的 MAC 地址全置零）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_14-44-47.png" alt="Snipaste_2021-03-02_14-44-47" style="zoom: 67%;" />

目标主机使用单播返回报文，报文中会携带自己的 MAC 地址

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_14-47-08.png" alt="Snipaste_2021-03-02_14-47-08" style="zoom:67%;" />



#### 补充：硬件类型和操作码的取值

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_15-09-55.png" alt="Snipaste_2021-03-02_15-09-55" style="zoom: 67%;" />



### 2.5 层协议 RARP：从 MAC 地址中寻找 IP 地址

- 动态地址解析协议 RARP（ Reverse Address Resolution Protocol ）（RFC903）
- 也被视为 2.5 层协议



#### RARP 寻址流程

使用场景：动态分配 IP 地址。联网设备启动时需要向服务器申请一个 IP 地址。这个过程就会涉及收发报文

1. 设备启动后发送广播，请求能分配 IP 的设备返回一个响应。广播报文中携带有自己的 MAC 地址
2. 能分配 IP 的设备为其分配 IP 并通过单播返回响应



#### RARP 报文格式：FrameType=0x8035

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_15-16-37.png" alt="Snipaste_2021-03-02_15-16-37" style="zoom:80%;" />

RARP 的报文格式和 ARP的报文格式基本相同



### ARP 欺骗/攻击

场景：在一个本地网络中 Alice 要和 Bob 进行通信，因此需要先使用 ARP 协议交换 MAC 地址



Alice 和 Bob 发送广播 ARP 报文互相查找其MAC地址时，Charlie 本不应该返回响应，但其主动返回响应并将自己的 MAC 地址给它们

此后 Alice 和 Bob 本地保存的 arp 列表中 MAC 地址就是 Charlie 的地址

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_15-18-26.png" alt="Snipaste_2021-03-02_15-18-26" style="zoom:67%;" />



## NAT 和 NAPT 地址转换

NAT & NAPT 用于解决 IPv4 地址短缺问题。IPv6 下 IP 地址短缺问题基本得到解决，但 IPv6 还未普及。因此地址转换技术仍在广泛使用

内网中主机的 IP 不在公网公开，所以在公网无法使用内网的 IP 找到目标主机。因此内网中可以使用任意内网中未使用过的 IP 地址

内网中的主机通过持有公网 IP 的对外路由器使用**地址转换协议**和公网中的主机通信

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_15-26-48.png" alt="Snipaste_2021-03-02_15-26-48" style="zoom:67%;" />



### 单向（向外）转换 NAT：动态映射

下图中，左边是内网，右边是公网

场景：内网主机 A 的内网 IP 为 10.0.0.207，路由器的公网 IP 是 194.54.21.11。内网主机向公网主机发送请求并得到响应

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_15-31-16.png" alt="Snipaste_2021-03-02_15-31-16" style="zoom: 67%;" />

1. A 组建报文并发送给路由器
2. 路由器将自己的一个公网 IP 和 A 的内网 IP 绑定到一块
3. 路由器将 A 发来的报文中 Source IP 改为 2 中 A 绑定到的公网 IP，并将报文发到网络中
4. 公网主机收到请求报文，并返回响应
5. 路由器收到响应，并将响应报文中 Destination IP 改为 A 的内网 IP，将响应发给 A



NAT（IP Network Address Translator）应用的前提 

- 内网中主要用于客户端访问互联网
- 同一时间仅少量主机访问互联网（NAT 中需要为内网中活跃主机分配绑定一个公网 IP，但是公网 IP 有限）
- 内网中存在一个路由器负责访问外网

基于上述第 2 条限制，NAT 难以在有多台主机的内网中工作。NAPT 作为 NAT 的升级版，解决了这个问题



### NAPT 端口映射：Network Address Port Translation

NAPT 的公网 IP + 一个端口号能绑定一个内网主机 IP + 内网主机的一个端口号

这让一个公网 IP 能绑定到很多台内网主机

下图中，左边是内网，右边是公网

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_15-42-09.png" alt="Snipaste_2021-03-02_15-42-09" style="zoom:67%;" />



### 双向（向内）NAT：IP 地址静态映射

之前的单向 NAT 和 NAPT 都是提供内网主机访问公网主机。很难做到公网主机访问内网主机

双向 NAT 通过静态映射：有限时间内将路由器的公网 IP 一直绑定到一台内网主机上。对这个公网 IP 的访问就是对指定内网主机的访问

缺点：静态导致不灵活。比如运营商使用拨号网络会导致路由器对外的公网IP是动态的。因为这个缺点，所以很难使用/不使用这个技术



### NAPT 端口映射抓包实例演示

场景：在 Linux 上部署一个简单的 SpringBoot 程序，用连 WiFi 的笔记本访问程序提供的 HTTP 接口。Wireshark 抓包分析 NAPT 地址转换过程

- 需要在 本地上用 Wireshark 抓包。在 Linux 上用 tcpdump 抓包并输入到文件里，在本地用 Wireshark 打开 tcpdump 抓取的报文
- 同时分析上述两处报文



本机内网地址：192.168.0.114

阿里云服务器公网地址：39.107.101.13   内网地址：172.24.61.41



笔记本本地 Wireshark 抓包

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_16-36-35.png" alt="Snipaste_2021-03-02_16-36-35" style="zoom:80%;" />

阿里云服务器 tcpdump 抓包

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_16-38-16.png" alt="Snipaste_2021-03-02_16-38-16" style="zoom:80%;" />

本地发报文时

- Src：192.168.0.114:10599
- Dst：39.107.101.13:8084

服务器收报文时

- Src：223.90.2.95:44077
- Dst：172.24.61.41:8084



## LVS/NAT 工作模式

- LVS（Linux Virtual Server）是一种三层负载均衡
- LVS 有 3 种工作模式，这里说的 NAT 工作模式是性能最差的。但与 4 层，7 层负载均衡来说，LVS/NAT 更好
- LVS 相当于一台 NAT 路由器

场景：客户端访问服务器

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-02_21-59-11.png" alt="Snipaste_2021-03-02_21-59-11" style="zoom:67%;" />

1. 请求发到 LVS 服务器后，LVS 使用 LB 算法选择一台内网中后端服务器
2. LVS 修改报文中的目标 IP ，转发报文
3. 后端服务器收到报文，返回响应给 LVS 服务器
4. LVS 服务器收到响应后，修改报文中的目标地址为客户端的地址（原报文中的地址是 LVS 服务器的地址）

LVS 维护了一张表，表保存了上述客户端和服务器的地址映射关系。这样 LVS 在转发请求和响应时才能正确修改报文中的目标地址

因为请求和响应报文都要经过 LVS 服务器，导致 LVS 服务器负载较大，所以才说 LVS/NAT 相对其他两种工作模式性能更差



## IP 选路协议

选路协议主要被路由器使用。在[图解网络硬件](D:\pdf\图解系列\图解网络硬件.竹下隆史.pdf)中有对下面选路协议更为详细的说明



### 传输报文的选路协议

- 直接传输（LH1 和 LH2 的通信。在同一条数据链路上）
- 本地网络间接传输：内部选路协议（LH4 和 LS1 的通信。在同一网络内，但不再同一数据链路上）
  - RIP
  - OSPF
- 公网间接传输：外部选路协议，在公网和骨干网上使用的选路协议（LH1 和 RS2 的通信。在不同网络内的主机）
  - BGP

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_11-33-51.png" alt="Snipaste_2021-03-03_11-33-51" style="zoom:67%;" />



### 集线器，交换机和路由器

**集线器**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_11-12-34.png" alt="Snipaste_2021-03-03_11-12-34" style="zoom: 50%;" />

- 集线器的作用是把内网中的网络设备连接起来
- 集线器不是智能的，不过滤数据，也没有数据应该发到哪里的任何功能。集线器只知道它的以太网连接端口上是否连接有设备
- 数据包进入集线器的某个端口后，集线器将数据包复制一份并广播到其他端口。缺点：浪费流量，不安全（因为其他计算机也会收到报文）

**交换机**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_11-12-53.png" alt="Snipaste_2021-03-03_11-12-53" style="zoom:60%;" />

- 交换机也用以太网连接端口接受网络设备的连接，但交换机有学习能力&是智能的，它会记录连接自己端口的网络设备的 MAC 地址，维护一张 MAC 表
- 当交换机的一个端口收到报文后，交换机根据自己维护的 MAC 表，将报文只发给指定的网络设备（单播），而不像集线器一样广播报文

**路由器**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_11-13-35.png" alt="Snipaste_2021-03-03_11-13-35" style="zoom:60%;" />

集线器和交换机被用来在一个局域网内交换数据，比如在自己家或公司内网，它们不能被用来和外网交换数据，比如因特网

要和外网（比如因特网）交换数据，设备需要能够读取 IP 地址，而集线器和交换机不读取 IP 地址，所以需要引入路由器

- 路由器，是一个能指引数据路径的设备，从一个网络到另一个网络，基于它们的 IP 地址

- 当一个数据包被路由器接收时，路由器检查数据的 IP 地址，比判断这个包是发送给自己的网络还是其他网络。所以路由器实际上就是网络的出入口（Gateway，又叫网关）

  场景：12 网段的主机向 14 网段的主机发送报文

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_11-56-45.png" alt="Snipaste_2021-03-03_11-56-45" style="zoom:67%;" />

**网络拓扑图**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_11-14-19.png" alt="Snipaste_2021-03-03_11-14-19" style="zoom: 50%;" />

三者非常类似，只是工作方式不同

- 集线器和交换机用来创建网络，路由器用来连接网络

PS：集线器和交换机是同等级的设备，但交换机更高级



路由器维护的路由表中宏观上来看是一张点到点最短路径的地图

选路协议的目的就是绘制这张地图

### RIP 内部选路协议

Routing Information Protocol 

- 特点：
  - 基于跳数确定路由（无权重的双向最短路径问题）
  - UDP 协议向相邻路由器通知路由表
- 问题
  - 跳数度量（而不是基于传输耗时）
  - 慢收敛（RIP 基于 UDP 协议，UDP 广播开销大，网络中新增一台路由器会造成网路变化，重新使用 RIP 协议更新路由表会比较费时）
  - 选路环路（双向最短路径问题都会涉及环路）

下图是 RIP 更新路由表的示意图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_12-06-05.png" alt="Snipaste_2021-03-03_12-06-05" style="zoom:67%;" />



### OSPF 内部选路协议

OSPF：Open Shortest Path First（开源的，基于传输耗时的选路协议。就是考虑了权重的最短路径问题）

- 多级拓扑结构：同级拓扑中的每台路由器都具有最终相同的数据信息（LSDB），即每台路由器都知道网络中路由器分布和传输耗时的全貌
- 直接使用 IP 协议（协议号 0x06 为 TCP，0x11 为 UDP，而 0x59 为 OSPF）传递路由信息

OSPF 最短路径树构造时需要遵守的原则

- 只有路由器到达网络有开销，网络到达路由器没有开销

  也就是路由器到达交换机有时间开销，交换机到达路由器没有时间开销

问题&猜想：路由器会连接大量的同级的路由器和下游网络的交换机，而交换机连接少量设备。因此交换机找路由器快，路由器找交换机慢



OSPF 最短路径树构造示例

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_12-16-58.png" alt="Snipaste_2021-03-03_12-16-58" style="zoom: 50%;" />



### BGP 外部选路协议

BGP：Border Gateway Protocol（BGP 是 EPG（Exterior Gateway Protocol，外部网关协议）的一种实现）

BGP 分为 EBGP 和 IBGP

一个网络内有多个 BGP 路由器时，它们为对等路由器，它们之间使用IBGP选路。网络间的BGP路由器用EBGP选路

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_12-19-46.png" alt="Snipaste_2021-03-03_12-19-46" style="zoom:67%;" />



路由跟踪工具（Windows: tracert ，Linux/Mac: traceroute）主要使用 ICMP 协议实现

```shell
$ traceroute www.baidu.com
```

路由跟踪工具用于查看发送报文时会经过哪些路由器

路由跟踪工具基本都是基于 ICPM 协议，ICPM 协议基于 IP 协议，在讲 ICMP 前先讲 IP 协议



## IP 报文格式和 IP 报文分片



### IP 报文格式

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_16-46-11.png" alt="Snipaste_2021-03-03_16-46-11" style="zoom:67%;" />

- Version：IP 协议的版本。值为 4 表示使用的是 IPv4 协议
- IHL：头部长度，单位字。一个字等于 4 个字节。头部长度 = 20 字节的固定长度 + 不定长的 IP Optionn
- TL：总长度，单位字节。总长度包括 IP 报文头部和 IP报文承载的所有数据
- 分片相关字段
  - Id：分片标识
  - Flags：分片控制，占 3  个比特位
    - DF 为 1：不能分片
    - MF 为 1：中间分片，为 0：结尾分片
  - FO：分片内偏移，单位 8 字节
- TTL：路由器跳数生存期（报文每经过一个路由器 TTL 的值 -1， TTL 值为 0 时还未到达目标主机时，路由器放弃报文并返回一个发送失败的报文）
- Protocol：承载的上层协议，比如 TCP 协议
- HC：校验和



### 基于 MTU 的 IP 报文分片

MTU（Maximum Transmission Unit）最大传输单元（ RFC791 ：>=576 字节）

- 报文传递到传输层时，如果报文长度超过 MTU ，IP 协议将会对其分片

- ping 命令可用于帮助学习 IP 报文分片。ping 命令会向指定主机发送 ICMP 报文，ICMP 报文承载有 IP 报文

  - Windows 下 -f：设置 DF 标志位为 1，-l：指定负载中的数据长度（ping 命令会使用随机字符填充至指定的数据长度）

  - Linux 下 -s：指定负载中的数据长度

    ```shell
    $ ping 39.107.101.13 -l 2000 # ping 百度，结果数据长度稍大就 ping 不通
    ```

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-14-02.png" alt="Snipaste_2021-03-03_17-14-02" style="zoom:80%;" />



**常见的 MTU**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-14-45.png" alt="Snipaste_2021-03-03_17-14-45" style="zoom:67%;" />



### 分片的缺点和分片流程

之前说过 TCP 协议尽力在网络层进行分片，因为网络层分片存在一些问题，多次分片导致传输报文，重组报文带来额外开销

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-18-47.png" alt="Snipaste_2021-03-03_17-18-47" style="zoom:67%;" />

IP 分片示例

MF = 1 表示属于同一组报文分片未结束。MF = 0 表示属于同于组报文的分片结束了。此外属于同一组的分片报文中 IP 报文的 Id 字段相同

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-21-29.png" alt="Snipaste_2021-03-03_17-21-29" style="zoom: 80%;" />

分片主体：源主机，路由器 

重组主体：目的主机



### 分片抓包演示示例

```shell
$ ping 39.107.101.13 -s 4000 # 这是 Linux 格式的 ping，实际演示使用的是 Windows 的 ping 命令
```

![Snipaste_2021-03-03_17-37-51](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-37-51.png)

同组的分片报文中 IP 报文的 Id 字段相同，最后一个分片报文 MF 位为 0。下面 3 张图为同组的分片报文

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-39-22.png" alt="Snipaste_2021-03-03_17-39-22" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-39-38.png" alt="Snipaste_2021-03-03_17-39-38" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-39-53.png" alt="Snipaste_2021-03-03_17-39-53" style="zoom:80%;" />



## IP 协议的助手：ICMP 协议

- ICMP：Internet Control Message Protocol（ RFC792 ）
- ICMP 协议基于 IP 协议，ICMP 报文中也承载着 IP 报文。但着不意味着 ICMP 协议 在 IP 协议之上，它们不像 TCP 协议与 IP 协议的上下关系
- ICMP 协议是  IP 协议的辅助协议
- IP 协议重视性能，只能完整简单的功能和工作。ICMP 协议用于告知错误，传递信息等功能



### ICMP 报文格式

- ICMP 格式简单，因为其目的是告知 IP 传递的数据的类型
- Type 字段表示主类型，Code 字段表示子类型

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-46-55.png" alt="Snipaste_2021-03-03_17-46-55" style="zoom: 80%;" />

常见主类型解释如下

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_17-49-44.png" alt="Snipaste_2021-03-03_17-49-44" style="zoom:50%;" />

目的地不可达报文：Type=3 的常用子类型 Code 

0：网络不可达 | 1：主机不可达 | 2：协议不可达 | 3：端口不可达 | 4：要分片但 DF 为1 | 10：不允许向特定主机通信 | 13：管理受禁



ping 命令发送的 ICMP 报文中。ping request 的 Type 是 8 ，ping reply 的 Type 是 0



### 路由跟踪工具和 ICMP & IP 的 TTL

路由跟踪工具通常发送基于 IP 协议的 ICMP 报文并设置 TTL 进行路由跟踪

原理：报文到达目标主机前会经过多个路由器，通过发送 TTL = n 的报文，当报文的 TTL = 0 时路由器放弃报文并返回响应报文。接收端便能得知路由器的信息（TTL 超限的报文中 Type = 11 ）

第一次发报文设置 TTL = 1，第二次设置 TTL = 2 ...

**演示示例**

捕获过滤器语法（路由跟踪工具除发 ICMP 报文外也会发 UDP 报文）

```
icmp or udp
```

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_18-18-34.png" alt="Snipaste_2021-03-03_18-18-34" style="zoom:80%;" />



## 多播与 IGMP 协议

广播和组播都是多播。多数编程语言可以比较容易在编程层面实现组播

- 广播：发送给同一个数据链路中的所有主机
- 组播：发送给同一个数据链路中的多台指定主机

IP和UDP都支持广播和组播。实现组播需要基于 IGMP 协议



### 广播

有两种广播：

-  广播报文到同一数据链路上的所有主机 ：目标 MAC 地址应置为：ff:ff:ff:ff:ff:ff

- 广播报文到同一网络上的所有主机：目标 IP 地址置为 255.255.255.255

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_21-05-35.png" alt="Snipaste_2021-03-03_21-05-35" style="zoom:80%;" />

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_11-13-48.png" alt="Snipaste_2021-03-04_11-13-48" style="zoom:80%;" />



### 组播



<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_21-11-02.png" alt="Snipaste_2021-03-03_21-11-02" style="zoom:80%;" />

预留组播地址 

- 224.0.0.1：子网内的所有系统组
- 224.0.0.2：子网内的所有路由器组
- 224.0.1.1：用于 NTP 同步系统时钟
- 224.0.0.9：用于 RIP-2 协议



组播以太网地址

- 以太网地址：01:00:5e:00:00:00 到 01:00:5e:7f:ff:ff
- 低 23 位：映射 IP 组播地址至以太网地址

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_11-40-04.png" alt="Snipaste_2021-03-04_11-40-04" style="zoom:80%;" />

**演示示例**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_11-41-43.png" alt="Snipaste_2021-03-04_11-41-43" style="zoom:80%;" />

```
IP 地址  	                  11101111 11111111 11111111 11111010
MAC 地址	00000001 00000000 01011110 01111111 11111111 11111010
```





### 组播使用的 IGMP 协议

- IGMP（Internet Group Management Protocol）协议
- IGMP 协议定义的报文用于告诉路由器报文应该发给哪些主机



**IGMP 报文格式**

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_21-20-56.png" alt="Snipaste_2021-03-03_21-20-56" style="zoom:80%;" />

Type 类型（ IGMP 协议有以下多个版本，目前多用 v3 版本）

- 0x11 Membership Query [RFC3376]
- 0x22 Version 3 Membership Report [RFC3376]
- 0x12 Version 1 Membership Report [RFC-1112]
- 0x16 Version 2 Membership Report [RFC-2236]
- 0x17 Version 2 Leave Group [RFC-2236]

下图为 v3 版本的 IGMP 协议报文，右图是内部 Group Record 的结构

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_21-38-01.png" alt="Snipaste_2021-03-03_21-38-01" style="zoom:80%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-03_21-38-24.png" alt="Snipaste_2021-03-03_21-38-24" style="zoom:80%;" />

Record Type 类型 

- 当前状态
  - 1: MODE_IS_INCLUDE
  - 2: MODE_IS_EXCLUDE
- 过滤模式变更（如从 INCLUDE 奕为 EXCLUDE）
  - 3: CHANGE_TO_INCLUDE
  - 4: CHANGE_TO_EXCLUDE
- 源地址列表变更（过滤模式同时决定状态）
  - 5: ALLOW_NEW_SOURCES
  - 6: BLOCK_OLD_SOURCES

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_11-49-25.png" alt="Snipaste_2021-03-04_11-49-25" style="zoom:80%;" /><img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_11-49-36.png" alt="Snipaste_2021-03-04_11-49-36" style="zoom:80%;" />



## IPv6 协议

> - IPv6 定义
> - IPv6 的单播和多播
> - IPv6 与数据链路的 MAC 地址是如何映射的
> - IPv6 的分片机制



### IPv6 地址

- IPv6 由 128位二进制组成。用冒分十六进制表示法表示：2 个字节为一组，用冒号分隔，如果中间全是0，可以省略0，用::表示（只允许出现一次::）

- IPv6 允许所有联网设备都有一个唯一的公网 IP。IPv6 不存在地址短缺，所以不使用 NAT 技术

- IPv6 地址分配情况如下图

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_14-32-06.png" alt="Snipaste_2021-03-04_14-32-06" style="zoom: 67%;" />

  

### IPv6 的多播

IPv6 使用多播地址的前 16 位可设置多播的类型和作用范围（数据链路，本地网络，跨网络，全网络）

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_14-38-17.png" alt="Snipaste_2021-03-04_14-38-17" style="zoom:80%;" />

Scope ID

1：本机作用域 | 2：本地链路作用域 | 5：场点作用域 | 8：组织作用域 | 14：全局作用域

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_14-41-48.png" alt="Snipaste_2021-03-04_14-41-48" style="zoom:67%;" />



### IPv6 和 MAC 地址的映射规则

IPv6 和 IPv4 一样将 IP 地址划分为网络地址和主机地址两部分。IPv6 将前 64 位作为网络地址，后 64 位作为主机地址

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_14-43-41.png" alt="Snipaste_2021-03-04_14-43-41" style="zoom:80%;" />

- Global Routing Prefix（全局路由前缀）：48 位
  - 可任意划分为多级
- 子网ID：16 位
  - 可任意划分为多级
- 接口ID：64 位
  - 直接映射 MAC 地址



目前有两种 MAC 地址：48 位 MAC 地址（老旧的 MAC 地址），64 位 MAC 地址（现代的 MAC 地址）

对于 64 位的 MAC 地址，可直接将 IPv6 的后 64 位作为 MAC 地址，对于 48 位 MAC 地址

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_14-48-01.png" alt="Snipaste_2021-03-04_14-48-01" style="zoom:67%;" />

- 取 OUI（组织唯一标识）放在前 24 比特，主机标识放在后
- 中间 16 比特置为 FFFE
- 置 OUI 第 7 位为 1 表示全局



上述映射方式过于简单，会导致 MAC 地址对公网暴露，不是太安全。因此有解决方案：对报文中的 MAC 地址进行加工

在 Windows 下使用以下命令开启或关闭对 MAC 地址的隐藏

```powershell
> netsh interface ipv6 set global randomizeidentifiers=disabled # 关闭隐藏 MAC 地址

> netsh interface ipv6 set privacy state=disabled # 开启隐藏 MAC 地址
```



### IPv6 报文格式

和 IPv4 的报文格式相比

- IPv6 去掉了 IHL：头部长度，单位字；HC：校验和
- 将和分片有关的字段（Id：分片标识，Flags：分片控制，FO：分片内偏移）放到可选字段（IP Option）中
- 其他字段不做修改或稍作修改，功能不变但名字可能改变

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_14-58-46.png" alt="Snipaste_2021-03-04_14-58-46" style="zoom:67%;" />

- Version
- 流量控制
  - Traffic Class（TOS） ：可控制报文的优先级
  - Flow Label：QOS 控制
- Payload Length（Total Length）：去掉 IPv6 报文报文头部长度后的长度。40 字节的固定头部
- Next Header
  - 如果使用分片，此字段的作用就是指向下一个头部
  - 不过没有分片，这个字段的作用和 IPv4 报文中的 Protocol 字段作用一致，用于指明上层使用的协议
- HopLimit（ TTL）



PS：IPv6 报文 = 40 字节的固定主头部 + 不定长的可选扩展头部 + Data 负载



### IPv6 分片



下图是关于 Next Header 的示意图

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_15-11-25.png" alt="Snipaste_2021-03-04_15-11-25" style="zoom:67%;" />

分片报文中，分片相关的字段和 IPv4 中的相关字段用法相同

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_15-13-30.png" alt="Snipaste_2021-03-04_15-13-30" style="zoom:67%;" />



- 不可分片部分：主首部，部分扩展首部
- 可分片部分：数据，部分扩展首部

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-04_16-20-44.png" alt="Snipaste_2021-03-04_16-20-44" style="zoom:67%;" />



## Wireshark 统计分析功能

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Wireshark%20%E7%BB%9F%E8%AE%A1%E5%88%86%E6%9E%90%E6%8A%A5%E6%96%87.png" alt="Wireshark 统计分析报文" style="zoom: 33%;" />



# 问题

前文种的内容基本都是分析 HTTP 或 TCP 报文，如何发送自定义的 HTTP 请求等

后端如何实现这些功能？

比如：断点续传，多线程下载，随机播放，防盗链，跨域等

**这里的 TCP 没有讲 TCP 的拆包、粘包**

wss = ws + ssl/tls 怎么玩

什么是拨号网络？拨号网络下使用的公网IP是动态的吗？

待补充 DNS 请求流程和抓包实例演示

问题：

- 缓存。缓存代理服务器和浏览器缓存之间如何交互

- [正向代理和反向代理的区别](https://www.cnblogs.com/taostaryu/p/10547132.html)

- [TLS 加密套件是什么](#加密套件)   在 Nginx 核心 100 讲中讲的比此课程中讲的更详细

- HTTP/2 的 request，response 是由多个 HTTP/1 中的内容组成的，是什么意思？

- 长肥通道/管道是什么意思

- 为什么一个HTTP/2报文中有两个Stream的数据

  <img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-02-08_12-18-05.png" alt="Snipaste_2021-02-08_12-18-05" style="zoom:80%;" />

- 什么叫[依赖性请求](#依赖性请求)

- （TLS 1.2 和 1.3）与安全套件的关系，或者说 TLS 和 密钥互换协议的关系。比如 Session 缓存能去掉 Server Key Exchange 和 Client Key Exchange 报文。但是这俩报文只有在 TLS1.2 中才有。而且Session使用以前的公钥，这违反了 ECDH 协议对密钥用完即弃的原则，因此只能通过这是过期时间进行补救

- 在本地使用 tracert 时会看到报文会经过某个路由器，这个路由器的 IP 可以看出。路由器和之前 NAPT 协议实战中本机内网 IP 使用的公网 IP 与其在同一网段（但不是同一个  IP，那为啥不是同一个 IP 呢？我tm哪知道）

- IPv4的广播和组播有范围限制吗？比如只在自己的数据链路中多播或跨网络多播

- 在进行 ICMP 和 TCP/UDP 以外的 IP 通信时，由于不存在端口号这个概念，因此需要直接根据 IP 首部的协议号来生成会话信息？这就不用端口号了？



网络通信流程猜想

- 同一个网段内，使用 MAC 地址即可从数据链路层找到目标主机（交换机保存 MAC 地址和主机的映射，即交换机知道 MAC 地址后就能找到同一数据链路中对应的主机）

- 不同网段内，使用 IP 地址寻址，路由器维护的路由表中记录 kv，k 是 IP 地址，v 是另一个路由器的 IP 地址

  意思是：如果你想找 k 主机就去找 v 路由器，v 路由器知道 k主机在哪。一直循环这个过程直到，路由器查 k 时查到的是交换机的地址。交换机用 报文中的 MAC 地址在自己的数据链路中找目标主机

场景：主机 host1 向 主机 host2 发送报文

过程：

1. host1 组建报文。报文包含双方的 MAC 地址和四元组
2. host1 把报文发给自己的上游交换机 exchange1
3. exchange1查表，发现 host2 不在自己管理的数据链路/网段中（本地网络中发送报文）
4. exchange1 把报文发给路由器 router1。（开始跨网络发送报文）
5. router1 查表，命中。结果是想找 host2 先找 router2。把报文发给 router2
6. router2 查表，命中。结果是想找 host2 先找 router3。把报文发给 router3
7. router3 查表，命中。结果是想找 host2 先找 exchange2。把报文发给 exchange2
8. exchange2 查表，命中。在自己的数据链路中找到 host2 ，把报文发给 host2。**END**

上述交换机和路由器使用的**表**。主要指路由器使用的表，是使用选路协议后构建的



交换机好像是路由器的一种   还是说路由器是交换机的一种

一个网络到底是怎么定义的？之前以为一台路由器下的所有交换机组建的数据链路构成一个网络。路由器连路由器代表着网络连接网络，但好像不是这个意思，因为一个网络中可以有多态路由器



下图是 IGMP 的报文格式（图太长了就不放在正文里了）

``` 
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Type = 0x22  |    Reserved   |            Checksum           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Reserved           |  Number of Group Records (M)	|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|																|
.						Group Record [1]						.  // Group Record 的格式在下面
|																|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|																|
.						Group Record [2]						.
|																|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|																|
.							  ....   							.
|																|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|																|
.						Group Record [M]						.
|																|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

```
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Record Type  |    Aux Data Len   |   Number of Sourves (N)   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                      Multicast Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                      Source Adress [1]                        |
+-															   -+
|                      Source Adress [2]                        |
+-															   -+
.		                 ..........								.
+-															   -+
|                      Source Adress [2]                        |
+-															   -+
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|																|
.						Auxiliary Data							.
|																|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```







# 后期补充

## OSI 分层架构详细协议图

![](http://cdn.processon.com/5da97644e4b0893e99345abf?e=1571390548&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:n1Jq_gqNPwTHkq6xLLCgD9xhZek=)



## 资料

### REST架构论文

https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm

### Chrome 抓包工具

第一部分课程主要使用Chrome 开发者工具的Network面板，主要参考资料如下：https://developers.google.com/web/tools/chrome-devtools/network/

### Wireshark 抓包工具

Wireshark是本课程的主要抓包工具

#### 常用协议抓包示例

https://wiki.wireshark.org/SampleCaptures

#### 官方用户手册

https://www.wireshark.org/docs/wsug_html_chunked/

### RFC(Request for Comments)文档

#### URI格式

* URL格式
  [RFC1738](https://tools.ietf.org/html/rfc1738 "RFC1738")
* URN格式
  [RFC2141](https://tools.ietf.org/html/rfc2141 "RFC2141")
* URI格式
  [RFC1630](https://tools.ietf.org/html/rfc1630 "RFC1630")、[RFC3986](https://tools.ietf.org/html/RFC3986 "RFC3986")

#### HTTP消息格式

* 基本格式 [RFC7230](https://tools.ietf.org/html/rfc7230 "RFC7230")、[RFC7231](https://tools.ietf.org/html/rfc7231 "RFC7231")
* Range请求 [RFC7233](https://tools.ietf.org/html/rfc7233 "RFC7233")
* 条件请求 [RFC7232](https://tools.ietf.org/html/rfc7232 "RFC7232")
* 缓存 [RFC7234](https://tools.ietf.org/html/rfc7234 "RFC7234")
* WEBDAV [RFC2518](https://tools.ietf.org/html/RFC2518 "RFC2518")
* Content-Disposition头部 [RFC6266](https://tools.ietf.org/html/RFC6266 "RFC6266")
* Cookie状态管理 [RFC6265](https://tools.ietf.org/html/RFC6265 "RFC6265")
* 同源策略 [RFC6454](https://tools.ietf.org/html/RFC6454 "RFC6454")

#### Websocket消息格式

* Websocket格式 [RFC6455](https://tools.ietf.org/html/rfc6455 "rfc6455")
* Websocket压缩扩展 [RFC7692](https://tools.ietf.org/html/rfc7692 "rfc7692")

#### HTTP2消息格式

* HTTP2格式 [RFC7540](https://tools.ietf.org/html/rfc7540 "rfc7540")
* HPACK头部压缩 [RFC7541](https://tools.ietf.org/html/rfc7541)
* ALPN（Application-Layer Protocol Negotiation Extension）扩展 [RFC7301](https://tools.ietf.org/html/rfc7301 "rfc7301")

#### 其他文档：

* MIME扩展类型https://www.iana.org/assignments/media-types/media-types.xhtml

#### TLS协议：

* TLS1.3 [RFC8446](https://tools.ietf.org/html/rfc8446 "rfc8446")
* 椭圆曲线安全性 [RFC7748](https://tools.ietf.org/html/rfc7748 "rfc7748")

#### TCP协议：

* TCP [RFC793](https://tools.ietf.org/html/rfc793 "rfc793")
* TCP窗口确认策略 [RFC813](https://tools.ietf.org/html/rfc813 "rfc813")
* TCP最大报文段长度MSS [RFC879](https://tools.ietf.org/html/rfc879 "rfc879")
* TCP拥塞控制 [RFC896](https://tools.ietf.org/html/rfc896 "rfc896")
* 主机实现TCP协议细节 [RFC1122](https://tools.ietf.org/html/rfc1122 "rfc1122")
* TCP校验和 [RFC1146](https://tools.ietf.org/html/rfc1146 "rfc1146")
* TCP高性能扩展 [RFC1323](https://tools.ietf.org/html/rfc1323 "rfc1323")
* TCP选择性重传报文段 [RFC2018](https://tools.ietf.org/html/rfc2018 "rfc2018")
* TCP拥塞控制 [RFC2581](https://tools.ietf.org/html/rfc2581 "rfc2581")
* 重传定时器 [RFC6298](https://tools.ietf.org/html/rfc6298 "rfc6298")
* TCP FAST OPEN [RFC7413](https://tools.ietf.org/html/rfc7413 "rfc7413")

#### IP协议：

* ARP协议 [RFC826](https://tools.ietf.org/html/rfc826 "rfc826")
* RARP协议 [RFC903](https://tools.ietf.org/html/rfc826 "rfc903")
* 路径MTU发现 [RFC1191](https://tools.ietf.org/html/rfc1191 "rfc1191")
* 私有网络IP地址分配 [RFC1918](https://tools.ietf.org/html/rfc1918 "rfc1918")

### DNS 协议

DNS协议 https://www.inacon.de/ph/data/DNS/



## 会话连接管理图文描述

> 取自《图解网络硬件》的第 5 章 “防护墙功能与防范威胁的对策” 第 7 节 “防火墙种搭载的各种功能‘ 的会话管理。里面有 TCP 管理连接的详细描述

### 会话与数据流

会话（session）是指两个系统之间通信的逻辑连接从开始到结束的过程。

- 在 TCP 中某个服务器与客户端成对进行通信时，会完成 3 次握手来确认建立 1 个 TCP 连接，在从连接建立开始至连接结束的时间里，客户端发送请求（request）和服务器进行应答（response）这一交互过程即可称为进行了 1 个会话
- 在 UDP 中，客户端与服务器之间只要发送源的端口和目的地端口的配对一致，随后的一系列通信均可以称为会话
- 在 ICMP 中，例如 Echo 和对应的 Echo reply 的组合就可以称为会话

一个会话存在“客户端→服务器”（c2s 或 client to server）和“服务器→客户端”（s2c 或 server to client）两个数据流（flow）。数据流是指发往通信对方的多个分组序列

<center>HTTP 通信中的数据流和会话示例</center>

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-20_14-02-01.png" alt="Snipaste_2021-03-20_14-02-01" style="zoom:80%;" />



### TCP 连接管理

一个 TCP 的连接需要通过 3 次握手来确认建立

1. 最初由客户端发送 SYN 消息，即发送首部中 SYN 比特信息设置为“1”的 TCP 数据段。SYN 读作 /`sin/，表示同步的意思，取自 Synchronization 这个单词的前三个字母。SYN 相当于一个开始信号，与打电话时先拨号码的行为类似
2. 当服务器收到来自客户端的 SYN 消息后，将返回表示确认的 ACK 消息，同时也会发送一个SYN 消息至客户端。ACK 表示确认的意思，取自 Acknowledgement 这个单词的前三个字母
3. 客户端返回 ACK

TCP 连接使用端口号表示不同的网络服务（应用程序）。例如，HTTP 使用 80 号端口，TELNET 使用 23 号端口。提供 HTTP 服务的服务器必须接收和处理客户端发送至 80 号端口的 TCP 数据段。能够处理分组的状态一般表示为 listen 状态（ listen 意为“侦听”，也称为 listening ）



<center>TCP 的 3 次握手 & 建立TCP连接/数据传输/结束步骤</center>

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-20_14-06-04.png" alt="Snipaste_2021-03-20_14-06-04" style="zoom:80%;" />

① 客户端发送 SYN 标志位设置为 On 的 TCP 数据段。

② 服务器接收到带有 SYN 标志位的消息后，将 SYN 与 ACK 的标志位设置为 On，并设置 Ack 编号为“发送方的 Seq+1”后进行回复

③ 客户端将 ACK 标志位设置为 On，将 Ack 编号设置为“接收的 Seq 编号 +1”的 TCP 数据段发送回服务器，确认建立 TCP 连接

**序列号**

TCP 中用序列号（sequence）来表示应用程序数据发送至何处，TCP 连接所使用的初始序列在 3 次握手的过程中确定

序列号分为两类，一类用于从客户端发往服务器端（c2s）的上行 TCP 数据段，另一类用于从服务器端发往客户端（s2c）的下行 TCP 数据段。上行和下行两种数据流在建立时，各自使用不同的随机数作为初始序列号 ISN（Initial Sequence Number）



### 防火墙对建立 TCP 连接的检查

上述是客户端与服务器正常建立 TCP 连接的过程。客户端和服务器之间加上防火墙时，防火墙（开启后）再让其建立 TCP 连接会做如下事情



- SYN 检查

  TCP 会话开始时客户端必会发送一个 SYN 消息。如果是没有附带会话信息（或尚未建立会话），即非 SYN 消息的 TCP 数据段到达防火墙，防火墙就会将其视作非法而整个丢弃。但也可以根据不同的情形（双活冗余或会话超时等）关闭（OFF）防火墙的这个功能，使不带有会话信息的、非 SYN 消息的 TCP 数据段也能够通过防火墙

- ACK 检查

  在根据 SYN Cookie（参考表 5-27）信息防范 SYN Flood 攻击时，通过对 SYN-ACK 的 ACK消息进行检查，能够确认进行中的 3 次握手是否为非法尝试

- 同一数据段检查

  终端再次发送 TCP 数据段时，对于和之前收到的 TCP 数据段含有相同序列号或数据的 TCP 数据段，可以指定防火墙的处理方式，即指定是使用新接收到的重复数据段还是丢弃该重复数据段

- 窗口检查

  检查 TCP 首部内的序列号和滑动窗口大小（Window Size），拦截超过滑动窗口容量数据的序列号

- 数据段重组

  即使各数据段的顺序出现变化，TCP 数据段也能根据序列号调整为正确顺序。在防火墙进行这一工作，可以验证 TCP 数据段序列号是否完整



### 经过防火墙建立的 TCP 会话



#### 会话建立的处理

防火墙按照以下步骤处理从网络接口接收到的分组，从而完成会话建立。

① 检索会话表，确认表内是否存在相同会话（若存在相同会话，则禁止会话建立的后续流程）

② 若不存在相同会话，则检查该分组是否可以通过 L3 路由选择或 L2 转发来输出。如果可以输出，确定对应的网络输出接口和目的地区域（若不能输出，则丢弃该分组）

③ 分组转发时，如果目的地址需要进行 NAT 则先完成 NAT，确定 NAT 后的网络输出接口和目的地区域

④ 根据分组的发送源信息（发送源网络接口、发送源区域和发送源地址）以及经过②、③步骤后得到的目的地信息（目的地网络接口、目的地区域、目的地址）进行安全策略检查，发现有符合的安全策略时，则根据该策略（允许通信或拒绝通信）决定是继续转发还是丢弃分组。如果没有符合的安全策略，则根据“默认拒绝”的设定丢弃该分组

⑤ 当分组被允许通信时，会话表中就会生成该会话的相关信息



#### 会话的生存时间

会话表中记录的会话信息有一定的生存时间。会话建立后，如果在一定时间内一直处于无通信状态，防火墙将会判断该会话的生存时间已到，进而将该会话记录项从会话表内删除

如果无条件地任由会话记录留在会话表中，这些会话信息则很有可能会被用于恶意攻击等行为。另外，由于会话表的记录项在数量上也有一定的限制，因此长期保留会话记录也会导致资源的长期占用，从而影响新会话记录的生成

会话时间能够根据 TCP、UDP 或其他 IP 协议的不同分别进行设置。对于 TCP 而言，会话的超时时间一般为 30 分钟 ~1 小时，UDP 则为 30 秒左右。

例如，某Telnet 会话通过防火墙完成了连接，若在 1 个小时内没有进行任何通信，防火墙会自动将该会话记录从会话表中删除。此后，客户端想要继续该 Telnet 会话时，也会被防火墙拒绝（图 5-12），因此客户端需要重新建立 Telnet 会话。会话生存时间的调整可以参考本书 07.04 节

<center>会话生存时间与超时的概念图</center>

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-20_14-24-42.png" alt="Snipaste_2021-03-20_14-24-42" style="zoom:80%;" />



#### 会话终止处理

也就是断开 TCP 连接

会话终止处理

TCP 连接一般通过下面的步骤终止会话。

① 客户端在完成收发数据后，会发送 FIN 标志位设置为 On 的 TCP 数据段（FIN）

② 服务器接收到 FIN 消息后，会在回复消息中将 FIN 与 ACK 标志位设置为 On，并将 Ack编号设置为“接收的 Seq 编号 +1”

③ 客户端同样在回复的 TCP 消息中将 ACK 标志位设为 On，将 Ack 编号设置为“接收的Seq 编号 +1”，连接就此结束

④ 这时，客户端会进入 TIME_WAIT 的 TCP 状态。一定时间后本次连接所使用的 TCP 端口号（来自客户端的通信发送源端口号）将会禁用。这一时间段称为 2MSL（Maximum Segment Lifetime，MSL 的 2 倍），根据实现的不同，大约在 1 分钟到几分钟之间不等



如果客户端或服务器在确认连接建立时发生了故障，那么将只有能够通信的一方进入侦听状态，这种情形称为半侦听或是半关闭。如果这时通信的故障方从故障中恢复，并接收到故障前交互的 TCP 数据段，便会向通信对方回复一条 TCP 响应数据段，该数据段中 RST 标志位设为ON，通过这条响应消息强制终止 TCP 连接。

终止连接有时会通过 FIN 和 RST 两个标志位来完成，不过当防火墙接收到来自通信方的 FIN 或 RST 时，还可以启动另一个 30 秒左右的定时器。如果在该时间段内 FIN → FIN-ACK → ACK 的终止过程仍未完成，防火墙中的会话表项会被强制删除

<center>在接收 SYN、FIN 消息时强制删除会话信息的时机</center>

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2021-03-20_14-32-39.png" alt="Snipaste_2021-03-20_14-32-39" style="zoom:80%;" />



