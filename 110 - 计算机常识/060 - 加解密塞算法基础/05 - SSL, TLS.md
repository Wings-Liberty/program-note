#还没有复习 

# 提供密码套餐的 SSL/TLs

SSL/TLS 作为计算机网络协议，提供网络数据的安全传输工作

SSL/TLS 之上能承载各种应用层协议，可以为 HTTP，SMTP，POP3，FTP 等协议提供安全的数据传输功能

SSL/TLS 提供了一种密码通信的框架，在正式通信前，通信双方先协商好通信时使用的密码学组件。比如用哪种对称加密算法，非对称加密算法，密钥交换方式等

且 SSL/TLS 就像事先搭配好的盒饭一样， 规定了一些密码技术的 “推荐套餐”， 这种推荐套餐称为**密码套件**

![Pasted image 20220627151641](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220627151641.png)

# SSL/TLS 工作方式
目标：Alice 和 Bob 希望通过 SSL/TLS 进行安全的通信

1. 双方安全地共享对称密钥
2. 双方用对称密钥加解密数据保证机密性

TLS 1.2，1.3 用 ECDHE 协议（DH 协议的升级）共享对称密钥

再用对称密钥传输数据