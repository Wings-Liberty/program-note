- 需要阅读的文章==全部==被标注在 todo 框中。文章的评论也是值得看的
- IM 多与移动端有关，所以可能需要学下 Android
- IM系统大都采用 C/S、B/S、P2P 等技术来实现即时通信的功能



学习 IM 的主要目的是希望通过理论结合实际地学习计算机网络，和了解 IM，而不是真的想从事 IM 方向（暂时是这样）



# 系列文章记录

社区中有许多以某个主题为主线的系列文章，只有少数的文章单独成军。这里以学习顺序为排序，列出需要阅读的系列文章的入口

系列文章中，有些文章需要有一定的基础知识才能阅读，所以不是一上手就直接读完整个系列的文章

- [ ] [零基础入门 IM 系列](http://www.52im.net/thread-3065-1-1.html)



# 零基础入门 IM

> 目的：
>
> - 了解 IM 是什么？
> - 学习 IM 需要计网的哪些知识？

简答：

Q：IM 是什么？

A：即时通讯

Q：学习 IM 需要计网的哪些知识？

A：包括但不限于计网，网络编程基础（包括但不限于 Netty）、通信协议的选型、IM的架构设计



阅读以下文章，将能了解到 “IM系统是什么？”

- [ ] [零基础IM开发入门(一)：什么是IM系统？](http://www.52im.net/thread-3065-1-1.html)（零基础 IM 开发入门系列的其他文章在后续学习中再读）





# 计算机网络理论准备

学习 IM 需要计网的理论知识（协议+硬件）和网络编程的实操

先上计网，这是[计算机网络的通讯协议关系图](http://www.52im.net/thread-180-1-1.html)。以下文章，除经典外都是选读



> 即时通讯技术归根结底还是网络编程技术的应用，如果您很好奇承载这些网络协议的物理设备是怎么工作的，可以先看看《[网络编程懒人入门(六)：史上最通俗的集线器、交换机、路由器功能原理入门](http://www.52im.net/thread-1629-1-1.html)》



## TCP 和 UDP 理论知识

由于编程层面能接触到的最底层的网络层是传输层，所以网络编程以传输层的 TCP 和 UDP 的学习为核心



**系列文章**

- [ ] [TCP/IP详解 卷1：协议](http://www.52im.net/topic-tcpipvol1.html)（经典，必读，全读）
- [ ] [脑残式网络编程入门](http://www.52im.net/thread-1729-1-1.html)（基础，且阅读难度最低）
- [ ] [网络编程懒人入门系列](http://www.52im.net/thread-1095-1-1.html)（基础）
- [ ] [不为人知的网络编程](http://www.52im.net/thread-1003-1-1.html)（进阶）



**零散文章**

深入了解 TCP

- [ ] [理论经典：TCP协议的3次握手与4次挥手过程详解](http://www.52im.net/thread-258-1-1.html)
- [ ] [理论联系实际：Wireshark抓包分析TCP 3次握手、4次挥手过程](http://www.52im.net/thread-275-1-1.html)
- [ ] [通俗易懂-深入理解TCP协议（上）：理论基础](http://www.52im.net/thread-513-1-1.html)
- [ ] [通俗易懂-深入理解TCP协议（下）：RTT、滑动窗口、拥塞处理](http://www.52im.net/thread-515-1-1.html)

深入了解 UDP

- [ ] [UDP中一个包的大小最大能多大？](http://www.52im.net/thread-29-1-1.html)
- [ ] [NAT详解：基本原理、穿越技术(P2P打洞)、端口老化等](http://www.52im.net/thread-50-1-1.html)



## 基于广域网的互联网编程知识

广域网因物理网络的复杂性，要编写能适应各种网络拓扑的程序代码，需要过硬的广域网络基础知识，比如NAT路由转发、P2P打洞等等



**NAT原理、点对点通信系列文章**

- [ ] [P2P技术详解](http://www.52im.net/thread-50-1-1.html)



**零散文章和资源**

- [ ] [通俗易懂：快速理解P2P技术中的NAT穿透原理](http://www.52im.net/thread-1055-1-1.html)（* 适合入门）
- [ ] [最新收集NAT穿越(p2p打洞)免费STUN服务器列表](http://www.52im.net/thread-608-1-1.html)
- [ ] [一款用于P2P开发的NAT类型检测工具](http://www.52im.net/thread-609-1-1.html)



## 高性能网络编程

- [ ] [高性能网络编程系列](http://www.52im.net/thread-561-1-1.html)

现时的网络编程为了解决高性能问题，有很多成型的Socket应用层模式存在，比如：NIO、AIO等

- [ ] [Java新一代网络编程模型AIO原理及Linux系统AIO介绍](http://www.52im.net/thread-306-1-1.html)简单介绍了传统的阻塞式IO、NIO，并着重介绍了最新的AIO技术
- [ ] [NIO 编程精选专题](http://www.52im.net/forum.php?mod=collection&action=view&ctid=9)

关于 Java 的网络编程和 Netty 的深入学习会在下一个标题中展开



## 移动互联网络的特性

IM 多用于移动端，移动互联网络因为无线通信的复杂性：慢、高延迟、易受干扰、带宽窄等特点，使得与传统的有线网络下的应用功能实现有明显差异，您很有必要详细了解一下无线网络方面的特性

- [ ] [移动端IM开发者必读(一)：通俗易懂，理解移动网络的“弱”和“慢”](http://www.52im.net/thread-1587-1-1.html)
- [ ] [移动端IM开发者必读(二)：史上最全移动弱网络优化方法总结](http://www.52im.net/thread-1588-1-1.html)
- [ ] [现代移动端网络短连接的优化手段总结：请求速度、弱网适应、安全保障](http://www.52im.net/thread-1413-1-1.html)



## 深入学习通信技术物理层（高阶知识，选读！）

高阶知识。IM/推送技术开发的边界知识，有助于你从物理层理解各种网络问题

- [ ] [IM开发者的零基础通信技术入门系列文章](http://www.52im.net/thread-2354-1-1.html)



# 网络编程

> 目标：计网理论结合编程实现 IM。理论联系实际理解 Socket 通信的原理和实践

NIO框架的流行，使得开发大并发、高性能的互联网服务端成为可能。这其中最流行的无非就是MINA和Netty了，MINA目前的主要版本是[MINA2](http://docs.52im.net/extend/docs/src/mina2/)、而Netty的主要版本是[Netty3](http://docs.52im.net/extend/docs/src/netty3/)和[Netty4](http://docs.52im.net/extend/docs/src/netty4/)。下面的学习均以用 Netty 作为通信框架



学习顺序：先复习输入输出流，BIO，NIO，Netty 的使用。再看下面的文章



**有关 TCP 的 Socket 通信 Demo 文章和代码（BIO 实现）**

- [ ] [网络编程懒人入门(八)：手把手教你写基于TCP的Socket长连接](http://www.52im.net/thread-1722-1-1.html)
- [ ] [Android端与服务端基于TCP协议的Socket通讯](http://blog.csdn.net/ryantang03/article/details/8274517)
- [ ] iOS平台的 [CocoaAsyncSocket](https://github.com/52im/CocoaAsyncSocket) 托管代码中有许多TCP的官方Demo代码，值得一看

以上文章仅是随手找的Demo代码，网络上有关TCP数据通信的演示性代码很容易找到



**本文作者专门编写的有关跨移动端平台的 UDP Socket 通信 Demo（Netty 实现）**

- [ ] [NIO框架入门系列文章](http://www.52im.net/thread-367-1-1.html)

上述系列文章都是 Demo 演示，而非系统学习 NIO，所以仅对在不同场景下使用 NIO 的学习有帮助



Netty 源码级别的学习可参考[宇道源码 Netty 分析篇](http://svip.iocoder.cn/categories/Netty/)



# IM 传输层协议选型

> 根据业务的需求，从 TCP 和 UDP 中选择一个合适的传输层协议

TCP 可靠性高，UDP 可靠性低。相同的任务量，采用 TCP 方案所需要的硬件配置高，采用 UDP 方案为提高可靠性需要自行保证可靠性

以下文章用于描述两种传输协议的不同，可在选型时做参考

- [ ] [简述传输层协议TCP和UDP的区别](http://www.52im.net/thread-580-1-1.html)
- [ ] [为什么QQ用的是UDP协议而不是TCP协议？](http://www.52im.net/thread-279-1-1.html)
- [ ] [移动端即时通讯协议选择：UDP还是TCP？](http://www.52im.net/thread-33-1-1.html)
- [ ] [网络编程懒人入门(五)：快速理解为什么说UDP有时比TCP更有优势](http://www.52im.net/thread-1277-1-1.html)
- [ ] [微信对网络影响的技术试验及分析（论文全文）](http://www.52im.net/thread-195-1-1.html)
- [ ] [为何基于TCP协议的移动端IM仍然需要心跳保活机制？](http://www.52im.net/thread-281-1-1.html)



# IM 数据通信格式

根据不同的应用场景，会使用不同的数据格式

比如：XMPP、Protobuf、JSON、私有2进制、MQTT、定格化XML、Plain text等



下列文章可在选择数据通讯格式时做参考

- [ ] [如何选择即时通讯应用的数据传输格式](http://www.52im.net/thread-276-1-1.html)
- [ ] [Protobuf通信协议详解：代码演示、详细原理介绍等](http://www.52im.net/thread-323-1-1.html)
- [ ] [强列建议将Protobuf作为你的即时通讯应用数据传输格式](http://www.52im.net/thread-277-1-1.html)
- [ ] [全方位评测：Protobuf性能到底有没有比JSON快5倍？](http://www.52im.net/thread-772-1-1.html)
- [ ] [移动端IM开发需要面对的技术问题（含通信协议选择）](http://www.52im.net/thread-133-1-1.html)
- [ ] [简述移动端IM开发的那些坑：架构设计、通信协议和客户端](http://www.52im.net/thread-289-1-1.html)
- [ ] [理论联系实际：一套典型的IM通信协议设计详解](http://www.52im.net/thread-283-1-1.html)
- [ ] [58到家实时消息系统的协议设计等技术实践分享](http://www.52im.net/thread-298-1-1.html)
- [ ] [金蝶随手记团队分享：还在用JSON? Protobuf让数据传输更省更快(原理篇)](http://www.52im.net/thread-1510-1-1.html)
- [ ] [扫盲贴：认识MQTT通信协议](http://www.52im.net/thread-318-1-1.html)



# 移动端IM的心跳保活和后台消息推送

此专题主要涉及 Android 和 IOS 中的心跳保活和后台消息推送，所以暂时只了解为什么需要心跳包括即可

可参考[基于TCP协议的移动端IM仍然需要心跳保活机制](http://www.52im.net/thread-281-1-1.html)（此文在前面提到过，所以不加 todo 标志）



# 移动端IM系统的架构设计

IM其本质是一套消息发送与投递系统，或者说是一套网络通信系统，归根结底就是两个词：存储与转发

但一个成熟的移动端IM系统要想正常运转，涉及的内容则远不止这些，而最考验技术功底的就是服务端架构的设计与实现

- [ ] [浅谈IM系统的架构设计](http://www.52im.net/thread-307-1-1.html)
- [ ] [简述移动端IM开发的那些坑：架构设计、通信协议和客户端](http://www.52im.net/thread-289-1-1.html)
- [ ] [一套海量在线用户的移动端IM架构设计实践分享(含详细图文)](http://www.52im.net/thread-812-1-1.html)
- [ ] [一套原创分布式即时通讯(IM)系统理论架构方案](http://www.52im.net/thread-151-1-1.html)
- [ ] [从零到卓越：京东客服即时通讯系统的技术架构演进历程](http://www.52im.net/thread-152-1-1.html)
- [ ] [蘑菇街即时通讯/IM服务器开发之架构选择](http://www.52im.net/thread-31-1-1.html)
- [ ] [腾讯QQ1.4亿在线用户的技术挑战和架构演进之路PPT](http://www.52im.net/thread-158-1-1.html)
- [ ] [微信技术总监谈架构：微信之道——大道至简(演讲全文)](http://www.52im.net/thread-200-1-1.html)
- [ ] [如何解读 微信技术总监谈架构：微信之道——大道至简》](http://www.52im.net/thread-201-1-1.html)
- [ ] [快速裂变：见证微信强大后台架构从0到1的演进历程（一）](http://www.52im.net/thread-168-1-1.html)
- [ ] [17年的实践：腾讯海量产品的技术方法论](http://www.52im.net/thread-159-1-1.html)
- [ ] [移动端IM中大规模群消息的推送如何保证效率、实时性？](http://www.52im.net/thread-1221-1-1.html)
- [ ] [现代IM系统中聊天消息的同步和存储方案探讨](http://www.52im.net/thread-1230-1-1.html)
- [ ] [WhatsApp技术实践分享：32人工程团队创造的技术神话](http://www.52im.net/thread-1542-1-1.html)
- [ ] [微信朋友圈千亿访问量背后的技术挑战和实践总结](http://www.52im.net/thread-1569-1-1.html)
- [ ] [王者荣耀2亿用户量的背后：产品定位、技术架构、网络方案等](http://www.52im.net/thread-1595-1-1.html)
- [ ] [腾讯资深架构师干货总结：一文读懂大型分布式系统设计的方方面面](http://www.52im.net/thread-1811-1-1.html)
- [ ] [一套高可用、易伸缩、高并发的IM群聊、单聊架构方案设计实践](http://www.52im.net/thread-2015-1-1.html)
- [ ] [即时通讯新手入门：一文读懂什么是Nginx？它能否实现IM的负载均衡？](http://www.52im.net/thread-2600-1-1.html)
- [ ] [从游击队到正规军：马蜂窝旅游网的IM系统架构演进之路](http://www.52im.net/thread-2675-1-1.html)
- [ ] [一套亿级用户的IM架构技术干货(上篇)：整体架构、服务拆分等](http://www.52im.net/thread-3393-1-1.html)（* 新）
- [ ] [一套亿级用户的IM架构技术干货(下篇)：可靠性、有序性、弱网优化等](http://www.52im.net/thread-3445-1-1.html)（* 新）
- [ ] [从新手到专家：如何设计一套亿级消息量的分布式IM系统](http://www.52im.net/thread-3472-1-1.html)（* 新）



专题：[IM架构篇](http://www.52im.net/forum.php?mod=collection&action=view&ctid=7)



# 移动端 IM 的通信安全

IM系统大都采用C/S、B/S、P2P等技术来实现即时通信的功能

当今的计算机密码学的主要作用有：加密（ Encryption）、认证（Authentication），鉴定（Identification）



- [ ] [即时通讯安全篇系列](http://www.52im.net/thread-216-1-1.html)
- [ ] [传输层安全协议SSL/TLS的Java平台实现简介和Demo演示](http://www.52im.net/thread-327-1-1.html)
- [ ] [理论联系实际：一套典型的IM通信协议设计详解（含安全层设计）](http://www.52im.net/thread-283-1-1.html)
- [ ] [微信新一代通信安全解决方案：基于TLS1.3的MMTLS详解](http://www.52im.net/thread-310-1-1.html)
- [ ] [来自阿里OpenIM：打造安全可靠即时通讯服务的技术实践分享](http://www.52im.net/thread-215-1-1.html)
- [ ] [简述实时音视频聊天中端到端加密（E2EE）的工作原理](http://www.52im.net/thread-763-1-1.html)
- [ ] [移动端安全通信的利器——端到端加密（E2EE）技术详解](http://www.52im.net/thread-764-1-1.html)
- [ ] [Web端即时通讯安全：跨站点WebSocket劫持漏洞详解(含示例代码)](http://www.52im.net/thread-793-1-1.html)
- [ ] [通俗易懂：一篇掌握即时通讯的消息传输安全原理](http://www.52im.net/thread-970-1-1.html)



专题：[IM安全篇](http://www.52im.net/forum.php?mod=collection&action=view&ctid=6)



# IM 中的实时音视频技术

IM 应用中的实时音视频技术，几乎是 IM 开发中的最后一道高墙

原因在于：实时音视频技术 = 音视频处理技术 + 网络传输技术 的横向技术应用集合体，而公共互联网不是为了实时通信设计的（？？？）

实时音视频技术上的实现内容主要包括：音视频的采集、编码、网络传输、解码、播放等多个环节

简单地说：音视频技术综合性强



## 音视频技术开篇

- [ ] [即时通讯音视频开发系列](http://www.52im.net/thread-228-1-1.html)
- [ ] [实时语音聊天中的音频处理与编码压缩技术简述](http://www.52im.net/thread-825-1-1.html)
- [ ] [网易视频云技术分享：音频处理与压缩技术快速入门](http://www.52im.net/thread-678-1-1.html)
- [ ] [学习RFC3550：RTP/RTCP实时传输协议基础知识](http://www.52im.net/thread-590-1-1.html)
- [ ] [基于RTMP数据传输协议的实时流媒体技术研究（论文全文）](http://www.52im.net/thread-273-1-1.html)
- [ ] [声网架构师谈实时音视频云的实现难点(视频采访)](http://www.52im.net/thread-399-1-1.html)
- [ ] [浅谈开发实时视频直播平台的技术要点](http://www.52im.net/thread-475-1-1.html)
- [ ] [还在靠“喂喂喂”测试实时语音通话质量？本文教你科学的评测方法！](http://www.52im.net/thread-507-1-1.html)
- [ ] [实现延迟低于500毫秒的1080P实时音视频直播的实践分享](http://www.52im.net/thread-528-1-1.html)
- [ ] [移动端实时视频直播技术实践：如何做到实时秒开、流畅不卡](http://www.52im.net/thread-530-1-1.html)
- [ ] [如何用最简单的方法测试你的实时音视频方案](http://www.52im.net/thread-535-1-1.html)
- [ ] [技术揭秘：支持百万级粉丝互动的Facebook实时视频直播](http://www.52im.net/thread-541-1-1.html)
- [ ] [简述实时音视频聊天中端到端加密（E2EE）的工作原理](http://www.52im.net/thread-763-1-1.html)
- [ ] [移动端实时音视频直播技术详解系列](http://www.52im.net/thread-853-1-1.html)
- [ ] [理论联系实际：实现一个简单地基于HTML5的实时视频直播](http://www.52im.net/thread-875-1-1.html)
- [ ] [IM实时音视频聊天时的回声消除技术详解](http://www.52im.net/thread-939-1-1.html)
- [ ] [浅谈实时音视频直播中直接影响用户体验的几项关键技术指标](http://www.52im.net/thread-953-1-1.html)
- [ ] [如何优化传输机制来实现实时音视频的超低延迟？](http://www.52im.net/thread-1008-1-1.html)
- [ ] [首次披露：快手是如何做到百万观众同场看直播仍能秒开且不卡顿的？](http://www.52im.net/thread-1033-1-1.html)
- [ ] [Android直播入门实践：动手搭建一套简单的直播系统](http://www.52im.net/thread-1154-1-1.html)
- [ ] [网易云信实时视频直播在TCP数据传输层的一些优化思路](http://www.52im.net/thread-1254-1-1.html)
- [ ] [实时音视频聊天技术分享：面向不可靠网络的抗丢包编解码器](http://www.52im.net/thread-1281-1-1.html)
- [ ] [P2P技术如何将实时视频直播带宽降低75%？](http://www.52im.net/thread-1289-1-1.html)
- [ ] [专访微信视频技术负责人：微信实时视频聊天技术的演进](http://www.52im.net/thread-1201-1-1.html)
- [ ] [腾讯音视频实验室：使用AI黑科技实现超低码率的高清实时视频聊天](http://www.52im.net/thread-1308-1-1.html)
- [ ] [微信团队分享：微信每日亿次实时音视频聊天背后的技术解密](http://www.52im.net/thread-1311-1-1.html)
- [ ] [近期大热的实时直播答题系统的实现思路与技术难点分享](http://www.52im.net/thread-1369-1-1.html)
- [ ] [福利贴：最全实时音视频开发要用到的开源工程汇总](http://www.52im.net/thread-1395-1-1.html)
- [ ] [七牛云技术分享：使用QUIC协议实现实时视频直播0卡顿！](http://www.52im.net/thread-1406-1-1.html)
- [ ] [实时音视频聊天中超低延迟架构的思考与技术实践](http://www.52im.net/thread-1465-1-1.html)
- [ ] [理解实时音视频聊天中的延时问题一篇就够
- [ ] [](http://www.52im.net/thread-1553-1-1.html)
- [ ] [实时视频直播客户端技术盘点：Native、HTML5、WebRTC、微信小程序](http://www.52im.net/thread-1564-1-1.html)
- [ ] [写给小白的实时音视频技术入门提纲](http://www.52im.net/thread-1620-1-1.html)
- [ ] [微信多媒体团队访谈：音视频开发的学习、微信的音视频技术和挑战等](http://www.52im.net/thread-1746-1-1.html)
- [ ] [腾讯技术分享：微信小程序音视频技术背后的故事](http://www.52im.net/thread-1799-1-1.html)



专题：[实时音视频开发](http://www.52im.net/forum.php?mod=collection&action=view&ctid=4)



## 开源及时通讯标准：WebRTC

> **WebRTC**，**网页即时通信**（Web Real-Time Communication）的缩写，是支持浏览器进行实时语音对话或视频对话的 API。已被纳入 W3C 推荐标准



- [ ] [开源实时音视频技术WebRTC的现状](http://www.52im.net/article-126-1.html)
- [ ] [简述开源实时音视频技术WebRTC的优缺点](http://www.52im.net/thread-225-1-1.html)
- [ ] [访谈WebRTC标准之父：WebRTC的过去、现在和未来](http://www.52im.net/thread-227-1-1.html)
- [ ] [良心分享：WebRTC 零基础开发者教程（中文）](http://www.52im.net/thread-265-1-1.html)
- [ ] [WebRTC实时音视频技术的整体架构介绍](http://www.52im.net/thread-284-1-1.html)
- [ ] [新手入门：到底什么是WebRTC服务器，以及它是如何联接通话的？](http://www.52im.net/thread-356-1-1.html)
- [ ] [WebRTC实时音视频技术基础：基本架构和协议栈](http://www.52im.net/thread-442-1-1.html)
- [ ] [浅谈开发实时视频直播平台的技术要点](http://www.52im.net/thread-475-1-1.html)
- [ ] [[观点\] WebRTC应该选择H.264视频编码的四大理由](http://www.52im.net/thread-488-1-1.html)
- [ ] [基于开源WebRTC开发实时音视频靠谱吗？第3方SDK有哪些？](http://www.52im.net/thread-510-1-1.html)
- [ ] [开源实时音视频技术WebRTC中RTP/RTCP数据传输协议的应用](http://www.52im.net/thread-589-1-1.html)
- [ ] [简述实时音视频聊天中端到端加密（E2EE）的工作原理](http://www.52im.net/thread-763-1-1.html)
- [ ] [实时通信RTC技术栈之：视频编解码](http://www.52im.net/thread-1034-1-1.html)
- [ ] [开源实时音视频技术WebRTC在Windows下的简明编译教程](http://www.52im.net/thread-1125-1-1.html)
- [ ] [网页端实时音视频技术WebRTC：看起来很美，但离生产应用还有多少坑要填？](http://www.52im.net/thread-1282-1-1.html)



# 移动端IM开发的其它热点问题

- IM 基础知识补课
- IM 消息送达保证机制
- IM 消息 ID 技术
- ...



# 动手实践

- [ ] [跟着源码学 IM 系列](http://www.52im.net/thread-2663-1-1.html)
- [ ] [一种Android端IM智能心跳算法的设计与实现探讨（含样例代码）](http://www.52im.net/thread-783-1-1.html)
- [ ] [详解Netty的安全性系列](http://www.52im.net/thread-426-1-1.html)
- [ ] [微信本地数据库破解版(含iOS、Android)，仅供学习研究 ](http://www.52im.net/thread-710-1-1.html)（* 强烈推荐）
- [ ] [Java NIO基础视频教程、MINA视频教程、Netty快速入门视频](http://www.52im.net/thread-1244-1-1.html)
- [ ] [轻量级即时通讯框架MobileIMSDK的iOS源码（开源版）](http://www.52im.net/thread-352-1-1.html)
- [ ] [开源IM工程“蘑菇街TeamTalk”2015年5月前未删减版完整代码](http://www.52im.net/thread-777-1-1.html)
- [ ] [微信本地数据库破解版(含iOS、Android)，仅供学习研究 ](http://www.52im.net/thread-710-1-1.html)
- [ ] [用于IM中图片压缩的Android工具类源码，效果可媲美微信](http://www.52im.net/thread-701-1-2.html)
- [ ] [高仿Android版手机QQ可拖拽未读数小气泡源码](http://www.52im.net/thread-922-1-2.html)
- [ ] [一个WebSocket实时聊天室Demo：基于node.js+socket.io](http://www.52im.net/thread-516-1-2.html)
- [ ] [Android聊天界面源码：实现了聊天气泡、表情图标(可翻页)](http://www.52im.net/thread-409-1-2.html)
- [ ] [高仿Android版手机QQ首页侧滑菜单源码](http://www.52im.net/thread-923-1-2.html)
- [ ] [开源libco库：单机千万连接、支撑微信8亿用户的后台框架基石](http://www.52im.net/thread-623-1-2.html)
- [ ] [分享java AMR音频文件合并源码，全网最全](http://www.52im.net/thread-397-1-3.html)
- [ ] [微信团队原创Android资源混淆工具：AndResGuard](http://www.52im.net/thread-140-1-3.html)
- [ ] [一个基于MQTT通信协议的完整Android推送Demo](http://www.52im.net/thread-315-1-3.html)
- [ ] [Android版高仿微信聊天界面源码](http://www.52im.net/thread-418-1-3.html)
- [ ] [高仿手机QQ的Android版锁屏聊天消息提醒功能](http://www.52im.net/thread-1233-1-1.html)
- [ ] [高仿iOS版手机QQ录音及振幅动画完整实现](http://www.52im.net/thread-1301-1-1.html)
- [ ] [Android端社交应用中的评论和回复功能实战分享](http://www.52im.net/thread-1584-1-1.html)
- [ ] [Android端IM应用中的@人功能实现：仿微博、QQ、微信，零入侵、高可扩展](http://www.52im.net/thread-2165-1-1.html)
- [ ] [仿微信的IM聊天时间显示格式(含iOS/Android/Web实现)](http://www.52im.net/thread-2371-1-1.html)
- [ ] [Android版仿微信朋友圈图片拖拽返回效果](http://www.52im.net/thread-2673-1-1.html)
- [ ] [IM开发宝典：史上最全，微信各种功能参数和逻辑规则资料汇总](http://www.52im.net/thread-3008-1-1.html)（* 强烈推荐）





# 补充

- [ ] [从根上理解高性能、高并发系列](http://www.52im.net/thread-3272-1-1.html)
- [ ] [WebSocket 详解系列](http://www.52im.net/thread-331-1-1.html)
