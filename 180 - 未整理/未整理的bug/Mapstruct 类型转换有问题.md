## 问题描述


场景：MsgContext 转 proto 发送给 A，A 收到 proto 后还原为 MsgContext

操作：

期望结果：MsgContext 能正确被还原

实际结果：原 MsgContext 为 COMMON_MSG 但 A 最终将其转为 ACK_MSG


## 解决方案

这是 proto 的问题

Message 的 parseFrom 不是严格的反序列化

假设字节数组是 CommonMsg 的序列化结果，CommonMsg 中有 AckMsg 没有的属性

AckMsg::parseFrom 可能也能把字节数组转换为 AckMsg

所以只能用类似 code \[+length\] + bytes 自定义报文解决
