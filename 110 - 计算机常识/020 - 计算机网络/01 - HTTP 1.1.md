
# 浏览器发送HTTP请求的典型场景


![Pasted image 20220704182936](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220704182936.png)

用户在地址栏中输入`url`时，浏览器会联想出用户曾访问过的`url`。因为浏览器中有一个轻量级数据库缓**存了用户的浏览和输入历史**


输入地址后，按下回车键。会发生以下事件

![[../../91 - 静态资源/Pasted image 20220704183028.png|475]]


上图表明，不管是从 URL 中解析域名，向 DNS 发送查询域名的请求，还是构造 HTTP 请求，**这都是浏览器的工作**，而不是 HTML 标记语言和 JS 语言做的事


# HTTP 消息格式


## 口语化描述

口语上不规范地表述 HTTP 消息格式通常是这样的

- `start-line` 起始行（必须有）
	- `request-line` 请求的起始行——请求行
	- `status-line` 响应的起始行——状态行
- `HEADERS-field` 头部/首部（0或多个首部）
- `message-body` 包体（可选）



## 实践举例

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



## 报文头部示意图

![Pasted image 20220704195900](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220704195900.png)





# 协议的通信规则


## URL 的基本格式

![image.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/20230416110221.png)

## 为什么要对 URI 进行 Base64 编码？

在 ABNF 语义描述的 URL/URI 中，用`: @ / ? # @ `等作为分割符。这些字符称为保留字符

问题：问什么要对URI进行编码？

答：防止 URI 中其他地方出现以下几种产生歧义的数据

- 上述的保留字/分割符
- 不在 ASCⅡ 编码范围内的字符
- ASCⅡ 中不可显示的字符
- 不安全的字符（传输环节可能会被不正确处理），如空格，引号，尖括号等


## 常用方法

GET，POST，PUT，DELETE，HEAD，OPTIONS，TRACE，CONNECT


> [!NOTE] 方法的详细介绍
> 见 utools 手册，或 MDN 文档介绍


## HTTP 的状态码

- 1xx：请求已接收到，需要进一步处理才能完成，`HTTP/1.0` 不支持
- 2xx：成功处理请求
- 3xx：重定向
- 4xx：客户端错误
- 5xx：服务器错误


[HTTP状态码](https://baike.baidu.com/item/HTTP状态码/5053660?fr=aladdin)


如果客户端识别不了返回的响应码（例如返回了 555 但是没有此响应码），客户端默认将其作为 x00 处理（比如将 555 当作 500 处理）


# 连接与消息的路由

HTTP 1.1 用 Connection 头部管理连接

## 长短连接

HTTP/1.1 默认支持长连接，HTTP 协议本身作为**应用层协议不存在长短连接的概念**，但因为 HTTP 建立在 TCP 之上，所以**这里的长短连接指一个 TCP 连接**是否在发送一个 HTTP 请求后就断开

并用 Connection: keep-alive 表示长连接

- 请求包含此头部，表示客户端请求建立长连接
- 响应包含此头部，表示服务器支持建立长连接


Connection: Close 表示短链接

## 如何用 Connection 管理跨代理服务器的长短连接？

**Connection 仅针对当前连接有效**：当客户端携带 `Connection: Keep-Alive` 时会尝试和请求的服务器建立长连接，如果目标是代理服务器，那么客户端和代理服务器间可能会建立长连接，代理服务器和上游服务器建立的连接不一定是长连接


如果代理服务器陈旧，不能正确的处理请求的 Connection 头部，会将客户端请求中的`Connection: Keep-Alive`原样转发给上游服务器

客户端和上游服务器会误以为和代理服务器建立了长连接，但实际上和代理建立了短链接


> [!warning] 目前 HTTP 不能很好地支持管线化 / 流水线
> 参考[这里](https://blog.csdn.net/qq_44918090/article/details/120757316)和 MDN。


## Host 头部如何控制 HTTTP 报文在服务端的路由

```
Host = uri-host [ ":" port ]
```

HTTP/1.1 规范要求，不传递 Host 头部则返回 400 错误响应码


`Host` 是 HTTP 1.1 协议中新增的一个请求头，主要用来实现虚拟主机技术。

虚拟主机（virtual hosting）即共享主机（shared web hosting），可以利用虚拟技术把一台完整的服务器分成若干个主机，因此可以在单一主机上运行多个网站或服务。

举个例子，ip 地址为 `61.135.169.125` 的服务器，在这台服务器上部署着谷歌、百度、淘宝的网站。为什么我们访问`https://www.google.com` 时，看到的是 Google 的首页而不是百度或者淘宝的首页？原因是 `Host` 请求头决定着访问哪个虚拟主机。


以 `nginx` 为例。客户端和服务端建立 TCP 连接后，`nginx` 会匹配 Host 头部与域名寻找虚拟主机（server），再匹配 url 定位 location 块


## 代理服务器转发消息时的相关头部


### 代理服务器向源服务器传递客户端 IP

如果客户端和源服务器之间有代理服务器，代理服务器和源服务器建立的 TCP 连接中，发送的 HTTP 协议包中的源 IP 地址是代理服务器的地址的而不是客户端的地址

问题：源服务器怎么获取客户端的IP地址

- RFC 协议规定，代理服务器可以在向源服务器转发请求时在请求中添加 `X-Forward-For` 头部传递客户端的 IP
- `nginx` 的 `realip` 模块通常会使用自定义的头部—— `X-Real-IP` 传递客户端的 IP


### 消息的转发

#### Max-Forwards

限制 Proxy 代理服务器的最大转发次数，仅对 TRACE/OPTIONS 方法有效

#### Via

指明经过的代理服务器名称及版本

#### Cache-Control

控制缓存行为，比如 Cache-Control: no-transform，表示禁止代理服务器修改响应包体


### 请求与响应的上下文


#### User-Agent

指明客户端的类型信息，服务器可以据此对资源的表述做抉择。其值包含了使用的浏览器名&版本号，操作系统名&版本号等

例如：

```
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:66.0) Gecko/20100101 Firefox/66.0

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36
```


#### Referer

此头部表示发起当前请求的 js 代码所属页面的源地址，所以可能暴露用户的浏览历史，涉及到用户的隐私问题


浏览器对来自某一页面的请求自动添加的头部。非浏览器的客户端发送的请求未必会添加这个头部


Referer 不会被添加的场景：

- 来源页面采用的协议为表示本地文件的 "file" 或者 "data" URI 
- 当前请求页面采用的是 http 协议，而来源页面采用的是 https 协议 

服务器端常用 Referer 统计分析、缓存优化、防盗链等功能

#### Server


指明服务器上所用软件的信息，用于帮助客户端定位问题或者统计数据 

例如： 

```
Server: nginx

Server: openresty/1.13.6.2
```


#### Allow 与 Accept-Ranges


Allow：告诉客户端，服务器上该 URI 对应的资源允许哪些方法的执行 

例如:

```
Allow: GET, HEAD, PUT
```



Accept-Ranges：告诉客户端，服务器上该资源是否允许 range 请求。这和多线程下载和断点续传有关

例如：

```
Accept-Ranges: bytes（开启）

Accept-Ranges: none（关闭）
```


# 内容协商与传输

内容协商：每个 URI 指向的资源可以是任何事物，可以有多种不同的表述，例如一份文档可以有**不同语言的翻译**（国际化）、**不同的媒体格式**（文件格式）、可以针对不同的浏览器提供**不同的压缩编码**（传递数据时使用的压缩方式）等


### 协商要素和资源表述

请求中携带 `Accept`/`Accept-*` 表示希望服务器能返回客户端要求形式的内容

响应中携带 `Content`/`Content-*` 表示服务器实际返回的协商内容的响应

```
Cntent-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Language: de-DE, en-CA
```

Content-Type 也能用在请求的头部里，告诉服务器请求体里的数据类型是什么



由于内容协商的存在，不同客户端访问同一个 URL 得到的响应结果可能时不同的。这是浏览器与服务器内容协商的结果

![Pasted image 20220911104013](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911104013.png)

## 主动式 & 被动式内容协商

Proactive **主动式内容协商**： 

指由客户端先在请求头部中提出协商内容，提出客户端需要哪种形式的资源

![Pasted image 20220911104027](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911104027.png)

Reactive **响应式内容协商**： 

服务器返回的响应头部中标明协商结果，以及返回的资源

服务器返回 300 Multiple Choices 或者 406 Not Acceptable，由客户端选择一种表述 URI 使用，这也是一种协商结果的表现



被动式内容协商（由于 RFC 没有明确规定 Client 如何从多个选择中选一个内容的规则，所以不同浏览器的选择策略不同。没有明确的规范导致实际情况下很少使用被动式内容协商）

![Pasted image 20220911104101](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911104101.png)


常见的协商要素主要包含：媒体资源类型，内容编码语言等

```
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3

Accept-Encoding: gzip, deflate, br

Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
```


字符编码：由于 UTF-8 格式广为使用， 所以 Accept-Charset 已被废弃




## HTTP包体的传输方式

请求或者响应都可以携带包体


### 定长包体传输


发送 HTTP 消息时已能够确定包体的全部长度，就用 **Content-Length 头部明确指明包体长度**，单位是字节，且必须与实际传输的包体长度一致 

优点：接收端处理更简单


如果 Content-Length 小于实际包体，抓包后可知，TCP 连接中能找到完整的数据。但客户端只会读取是 HTTP 响应中 Content-Length 指定大小的数据，**多余的数据抛弃掉**

如果 Content-Length 大于实际包体，浏览器因无法正确读取数据而直接断开连接并报错


### 不定长包体传输

发送 HTTP 消息时不能确定包体的全部长度 

使用 Transfer-Encoding 头部指明使用 Chunk 传输方式

- 含 Transfer-Encoding 头部后 Content-Length 头部应被忽略
- 包体将以 chunk 为单位发送多个 chunk ，直到包体全部发送完毕



优点

- 基于长连接持续推送动态内容
- 压缩体积较大的包体时，不必完全压缩完（计算出头部）再发送，可以边发送边压缩
- 传递必须在包体传输完才能计算出的 Trailer 头部




### MIME 媒体数据类型

> MIME（ Multipurpose Internet Mail Extensions ） 

```
Content-Type: type/subtype
```

type 作为主类型，有独立媒体类型和复合媒体类型两种

常见取值有

- "text"，"image"，"audio"，"video"，"application"，extension-token（文本，图片，音频，视频，应用程序 如js，扩展类型）

- "message"，"multipart"，extension-token（复合媒体类型）


subtype  子类型，每种主类型下都有众多子类型，例如 text/plain   

大小写不敏感，但通常是小写 

例如：

```
Content-type: text/plain; charset=utf-8
```


[更多MIME媒体数据类型](https://www.iana.org/assignments/media-types/media-types.xhtml)



### HTML form 表单提交时的协议格式


**HTML 的 form 表单**

- HTML 是一个结构化的文本文档，没有交互能力和发送请求的能力但是 HTML 的表单却可以通过提交按钮直接发送 HTTP 请求

- FORM 表单：HTML 中的元素，提供了交互控制元件用来向服务器通过 HTTP 协议提交信息，常见控件有

  - Text Input Controls：**文本输入**控件 
  - Checkboxes Controls：**复选框**控件
  - Radio Box Controls ：**单选按钮**控件
  - Select Box Controls：**下拉列表**控件
  - File Select boxes：选取**文件**控件
  - Clickable Buttons：可点击的**按钮**控件
  - **Submit** and Reset Button：**提交或者重置**按钮控件

form 表单提交请求时的关键属性

| 属性名  | 用途                             |
| ------- | -------------------------------- |
| action  | 提交时发起 HTTP 请求的 URI       |
| method  | 提交时发起 HTTP 请求的 http 方法 |
| enctype | 表单提交时的编码方式                                 |


对表单内容在请求包体中的编码方式有两种

- application/x-www-form-urlencoded 数据被编码成以 ‘&’ 分隔的键值对，同时以 ‘=’ 分隔键和值，字符以 URL 编码方式编码
- multipart/form-data 包括 1个或多个分隔符 + 资源表示 + 表示包体结束的结束分隔符    

PS：资源表示 = 资源描述+数据

在 application/x-www-form-urlencoded 编码方式下，GET 请求会把 kv 数据追加到 url 后，数据以 query param 格式被传递，且请求不会标明 Content-Type；POST 请求会把 kv 数据放在请求体里，且请求会标明 Content-Type

> [!NOTE] get 请求的表单，只能用 application/x-www-form-urlencoed 


> [!NOTE] HTML form 标签默认 enctype 值是 "application/x-www-form-urlencoded"



### Multipart 包体格式（RFC822）

一个 multipart/form-data 编码类型的包体可以包含 1 个或多个独立的资源表示，也就是有 1个或多个 encapsulation


每一个 encapsulation 都对应 1 个资源表示。例如：1 个文件、图片、单选框、输入框等


例如：某次响应中指定的分隔符是 ----WebKitFormBoundaryRRJKeWfHPGrS4LKe

multipart/form-data 的响应头会用 Content-type 标明分隔符：

```
Content-type: multipart/form-data; boundary=----WebKitFormBoundaryRRJKeWfHPGrS4LKe\r\n
```

[xhr multipart boundary分隔符](https://www.cnblogs.com/ksyy/p/11506361.html)

### 表单提交实例演示

![Pasted image 20220705171815](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220705171815.png)

![Pasted image 20220705171843](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220705171843.png)


上图是 4 中表单示例


> [!NOTE] GET 请求中的 application/x-www-form-urlencoed 方式编码
> get 请求的表单，只能使用 application/x-www-form-urlencoed 方式编码，且表单数据将拼接在 url 后，而不是包体中
> 

get 请求的表单能将参数拼接到 url 后的原因
- applicarion/x-www-form-urlcoded 编码方式和 url 后的参数的编码方式刚好相同
- get 请求不能携带包体，而 applicarion/x-www-form-urlcoded 编码方式的参数刚好又能拼接到 url 后


get 请求的表单不能使用 multipart/form-data，因为这种编码方式必须将数据放在包体中


post 请求的 applicarion/x-www-form-urlcoded 表单抓包示意图

![Pasted image 20220705172130](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220705172130.png)


![Pasted image 20220705172139](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220705172139.png)



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



#### 内容协商，客户端发送 Accept-Ranges 询问是否允许 Range

- 允许服务器基于客户端的请求只发送响应包体的一部分给到客户端，而客户端自动将多个片断的包体组合成完整的体积更大的包体 

- 服务器通过 Accept-Range 头部表示是否支持 Range 请求




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



通过 Range 头部传递请求范围，如：

```
Range: bytes=0-499
```



#### Range 条件请求 If-Range


如果客户端已经得到了 Range 响应的一部分，并想在这部分响应未过期的情况下，获取其他部分的响应 


- 常与 If-Unmodified-Since 或者 If-Match 头部共同使用
- If-Range = entity-tag / HTTP-date 
  - 可以使用 Etag 或者 Last-Modified

**`If-Range`** 头字段通常用于断点续传的下载过程中，用来自从上次中断后，确保下载的资源没有发生改变

每次请求数据后，响应中会携带一个 ETag 头部作为请求的指纹，请求下一部分资源时，将指纹的值放到 If-Range 头部中。示例如下图

![Pasted image 20220705173823](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220705173823.png)


如果服务器的资源被修改，那么资源的指纹就会被改变，自然就匹配补上 If-Range 上的指纹，服务器就会返回 412 响应码


#### 服务端对 Range 的响应

- 206 已处理部分请求
- 416 请求返回不合法
- 200 OK


**206 Partial Content**

Content-Range 头部：显示当前片断包体在完整包体中的位置 

```
Content-Range: <unit> <range-start>-<range-end>/<size>
```


如果文件大小是未知的，就用 * 号替代，例如：

```
Content-Range: bytes 42-1233/1234
Content-Range: bytes 42-1233/*
```

![Pasted image 20220705180040](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220705180040.png)


**416 Range Not Satisfiable**

请求范围不满足实际资源的大小

![Pasted image 20220705175923](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220705175923.png)


**200 OK**

服务器不支持 Range 请求时，则以 200 返回完整的响应包体




#### 简单的多范围请求与 multipart 响应示例

请求

```
Range: bytes=0-50, 100-150
```

响应

```
Content-Type: multipart/byteranges; boundary=...
```

其包体保存数据的方式和上节表单提交请求使用的包体格式基本相同，因为它们的 content-type 的主类型都是 multipart




# HTTP Cookie

Cookie 是被保存在客户端的 kv 键值对

### Cookie 的工作原理

Cookie 被保存在客户端，由客户端维护，由服务端创建

Cookie 值存放在内存或者磁盘中

服务器收到请求后，在响应头里添加 Set-Cookie 头部，浏览器收到响应后通常会保存下 Cookie

之后对该服务器每一次请求中都通过 Cookie 请求头部将 Cookie 信息发送给服务器

另外，Cookie 的过期时间、域、路径、有效期、适用站点都可以根据需要来指定



## Cookie 头部和 Set-Cookie 头部的定义

Cookie 的头部格式如下

```
Cookie: <cookie-list>
```

举个例子

```
Cookie: PHPSESSID=298zf09hf012fh2; csrftoken=u32t4o3tb3gg43; _gat=1;
```

Set-Cookie 头部格式如下

```
Set-Cookie: <cookie 名>=<cookie 值>[;选项];
```

选项内容可以包含 Cookie 的过期时间、域、路径、有效期、适用站点等内容

举个例子

```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly

[页面内容]
```



## Cookie 的使用限制

RFC 规范对浏览器使用 Cookie 的要求如下

- 每条 Cookie 的长度（包括 name、value 以及描述的属性等总长度）的最大值由浏览器规定，但也不能小于 4KB
- 每个域名下至少支持 50 个 Cookie
- 至少要支持 3000 个 Cookie

这个首部可能会被完全移除，例如在浏览器的隐私设置里面设置为禁用 cookie

代理服务器传递 Cookie 时会有限制

`Domain` 和 `Path` 标识定义了 Cookie 的作用域：即允许 Cookie 应该发送给哪些 URL

## 第三方 Cookie

> 第三方 Cookie 常用于收集用户的轨迹信息或其他信息


跨域访问资源时，请求的地址是第三方服务器。响应中可能会携带有 Set-Cookie 头部。浏览器会将这些数据保存到本地 Cookie 中

在用户后续访问第三方服务器时，请求将自行携带上这些 Cookie

比如：

1. 用户访问 A 网站并购买商品
2. 用户在 B 网站浏览时，能在 B 网站查询到他在 A 网站的购买记录

说明用户的资源在访问 A 网站时被保存到了 B 网站


# 浏览器的同源策略

同源策略的目的是：防止不同站点间互相读取信息

举个例子：用户访问 www.a.com 后返回了一个 html 文件，这个文件里包含了 html，js 代码

因为 html 的某些标签引用了外部资源，所以用户访问完 www.a.com 并得到 html 文件后就立即访问了 www.b.com 上的资源

如果允许了跨域访问，那么 www.a.com 的 html 文件里就能正常访问 www.b.com

如果不允许跨域访问，那么 www.a.com 的 html 文件向 www.b.com 发起的请求就会被拒绝


### 同源策略的定义

同源策略只允许 协议，主机，端口相同的站点互相访问 url。比如在 A 站点的脚本中访问 B 站点的接口就是不行的

同源策略是浏览器规定的。浏览器规定不能进行跨域访问，所以在使用 postman 等工具进行接口测试时不会遇到跨域问题


## 跨源资源共享（CORS）的工作原理

规范要求，对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 GET 以外的 HTTP 请求，或者搭配某些 MIME 类型 的 POS 请求），浏览器必须首先使用 OPTIONS 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨源请求。服务器确认允许之后，才发起实际的 HTTP 请求

参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)就够了


 
# 缓存的控制

缓存需要合理配置，因为并不是所有资源都是永久不变的：重要的是对一个资源的缓存应截止到其下一次发生改变（即不能缓存过期的资源）

## 缓存的分类

缓存大致可归为两类：私有与共享缓存。共享缓存存储的响应能够被多个用户使用，私有缓存只能用于单独用户

一般**私有缓存由浏览器缓存实现**，**共享缓存由缓存服务器实现**



## HTTP 缓存的工作原理

发送 HTTP 请求后，请求会携带上是否需要对响应缓存

如果需要服务器进行共享缓存，服务器会缓存；如果需要客户端进行私有缓存，响应会用头部告诉客户端

假设是私有缓存，响应会返回数据并告诉客户端，服务器上数据在多长时间内一定不会更新

**客户端在缓存有效期内需要再次请求数据时，请求会被拦截并直接用本地的缓存数据**

**如果过了缓存有效期**，客户端重新发送请求就不会被拦截。但请求上会携带过期缓存的指纹，**缓存服务器校验本地数据的指纹和客户端过期缓存的指纹是否一样**。

如果指纹一样，说明服务器数据并没有被修改，服务器只需要用状态码告诉客户端 “你的缓存没有过期，因为数据没有更新过”，所以响应也不必要再返回被请求的数据

和缓存相关的头部有很多个，下面就介绍下


## 缓存控制

Cache-Control 头可用在请求或响应里，通过指定指令来实现缓存机制


**比如用在请求里时**

```
Cache-Control: max-age=<seconds> // 希望服务器能缓存此次结果多长时间
Cache-control: no-cache // 要求缓存服务器返回响应前，向源服务器验证缓存是否过期
Cache-control: no-store // 希望服务器不要缓存数据
```

**比如用在响应里时**

```
Cache-Control: public // 共享缓存。告诉客户端，服务器完成了缓存
Cache-Control: private // 私有缓存。告诉客户端，自己缓存吧
Cache-Control: max-age=<seconds> // 服务器会缓存多长时间
```

> [!NOTE] Cache-Control 的值还有很多
> 更多值和含义见手册


## 新鲜度

HTTP 是 C/S 模式的协议，服务器修改数据后不能主动通知客户端丢弃脏缓存。所以就如[[01 - HTTP 1.1#HTTP 缓存的工作原理|之前所说]]，HTTP 缓存中，服务器返回的响应除携带数据外还会告知客户端，服务器上的数据在多长时间内一定不会更新

现在需要说明如何用 HTTP 请求响应头完成这两件事

1. 怎么用指纹判断客户端缓存是否失效？
2. 怎么计算缓存是否过期？


有很多实现方式

### 如何得知缓存有效期，计算缓过期时间

服务器返回 200 响应并携带头部告知缓存有效期，客户端会根据响应头里的信息计算出缓存过期时间，当客户端重新发送请求时根据之前计算的缓存过期时间判断缓存是否过期

**Cache-control: max-age=N**

N 是相对时间，时间起点是获取到携带有数据响应的时间


**Expires**

检查缓存是否过期时，如果没有 Cache-control 头部，再检查是否有 Expires 头部

Expires 的值是绝对时间，表示缓存过期时间

客户端会拿请求体里的 Date 头的时间和 Expires 的值进行对比判断缓存是否过期

**Last-Modified**

如果没有 max-age 和 expires，就找 Last-Modified

缓存有效时间 = ( Date - Last-Modified ) / 10



### 条件请求判断本地缓存是否过期

如果浏览器发送请求前发现缓存可能过期，需要**询问服务器，“服务器的数据被修改了吗？”**，如果服务器数据没有被改过，说明浏览器缓存还能用

用 If-None-Match 或 If-Modified-Since 头实现

条件请求头 If-None-Match 携带客户端缓存的 ETag，等待服务器的校验结果

**如果服务器上的数据的 ETag 和请求头的一致**，就返回 304 **表示服务器上的数据没有更新过，客户端的缓存还能用**

而 If-Modified-Since 携带的是客户端第一次获取到数据的时间，服务器对比此时间和服务器上数据最近的修改时间，如果数据没被改过就返回 304

先检查是否由 Cache-control: max-age=N，再检查是否有 Expires 属性，如果还是没有就检查是否有 Last-Modified 属性


> [!NOTE] 优先级问题
> If-None-Match > If-Modified-Since

### Vary 响应

刚才介绍了 If-None-Match 或 If-Modified-Since 头判断本地缓存是否过期

前者采用数据指纹检查，后者采用文件修改日期检查

还有种检查，不检查缓存是否过期，而是根据请求头检查是否处理过相同的请求

参考[这里](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching#%E7%BC%93%E5%AD%98%E9%AA%8C%E8%AF%81:~:text=%E7%9A%84%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4%E3%80%82-,Vary%20%E5%93%8D%E5%BA%94,-Vary%20HTTP%20%E5%93%8D%E5%BA%94)



# 多种重定向跳转方式的差异

举两个重定向的例子

- 访问 xxx.xxx.com/login 点击登录后，响应中携带重定向的状态码和 Location 头部
- 访问 http://www.baidu.com 时，百度只支持 https，响应中返回携带重定向的状态码 307 和 Location 重定向到 https://www.baidu.com



重定向能解决什么问题

- 提交 FORM 表单成功后需要显示内容页，怎么办？
- 站点从 HTTP 迁移到 HTTPS，怎么办？ 
- 站点部分 URI 发生了变化，但搜索引擎或者流量入口站点只收录了老的 URI ，怎么办？
- 站点正在维护中，需要给用户展示不一样的内容，怎么办？
- 站点更换了新域名，怎么办？



## 重定向的工作原理


![Pasted image 20220911112630](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911112630.png)



当浏览器接收到重定向响应码时，需要读取响应头部 Location 头部的值，获取到新的 URI 再跳转访问该页面


- 原请求：接收到重定向响应码的请求这里称为原请求
- 重定向请求：浏览器接收到重定向响应码后，会发起新的重定向请求 



## 重定向响应返回码分类

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

- 300：被动式内容协商时使用的状态码
- 304：Not Modified，是 HTTP 缓存机制相关的状态码，说明无需再次传输请求的内容



重定向循环：服务器端在生成 Location 重定向 URI 时，在同一条路径上使用了之前的 URI，导致无限循环出现

重定向循环会导致 Chrome 浏览器会提示：ERR_TOO_MANY_REDIRECTS    （错误，太多次重定向了）



# HTTP Basic 基本认证

> RFC7235，一种基本的验证框架，被绝大部分浏览器所支持 
>
> 基本认证明文传输，如果不使用 TLS/SSL 传输则有安全问题

HTTP Basic 基本认证的认证方式根据 Authorization 头部进行认证

HTTP 协议提供的 HTTP Basic 认证方式过于简单，通常自定义的认证方式也使用 Authorization 头部，但是认证逻辑由源服务器/认证服务器规定
