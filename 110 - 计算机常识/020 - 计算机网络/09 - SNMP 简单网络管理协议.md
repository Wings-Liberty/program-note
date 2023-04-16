# SNMP 简单网络管理协议


> [!NOTE] 参考
> - [SNMP介绍及使用](https://www.cnblogs.com/chegxy/p/14020233.html)
> - [SNMP介绍教程](https://bbs.huaweicloud.com/blogs/364441)
> - [SNMP学习笔记之SNMP 原理与实战详解](https://cloud.tencent.com/developer/article/1366136)



SNMP 是一个 C/S 架构的协议

简单地说，服务器能发送 **get 指令获取客户端的信息**，服务器发送 **set 命令让客户端修改自己的信息**，客户端向服务器发送 **trap 命令给服务器发通知**

# 服务器主动获取客户端信息

Q：get 命令能获取什么信息？

A：很多信息，客户端的 OS 信息，内存总量，已使用量，可是用量，客户端温度，磁盘使用情况，网络情况等各种硬件和运行时信息

Q：get 命令怎么指定需要获取客户端的哪些信息

A：通过一个字符串定位到一个信息，这个字符串被称为 `OID`，比如 `.1.3.6.1.2.1.1.5.0` 就能定位到一个资源信息


## OID

SNMP 使用 OID 定位资源信息，OID 就像是一个树，每个节点都有一个 value 值，每个叶子节点对应一个具体的资源，中间节点表示资源的所属分类

比如，服务器想要获取某台客户端的【系统正常运行时间】，这个资源名叫 "sysUpTime"


它在 OID 树中的这个位置

![Pasted image 20220914163130](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220914163130.png)

那对象 sysUpTime 的唯一 OID 是 `.1.3.6.1.2.1.1.3.0`

更多的 OID 参考[这里](http://oid-info.com/get/)

# 客户端主动推送自身信息


Q：trap 命令能干什么？

A：客户端修改一些变量、客户端出现异常等希望通知一下服务器，就能用 trap 命令


