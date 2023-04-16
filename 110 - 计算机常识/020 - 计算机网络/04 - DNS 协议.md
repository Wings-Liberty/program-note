# DNS 基本概念

DNS 协议用于把域名转为 IP 地址，由于域名服务器存在树形关系，所以要定位到某个域名和 IP 地址的映射需要进行递归查询

不过目前采用的是**递归 + 迭代方式做域名解析**

## 递归+迭代方式的域名解析

![Pasted image 20220911153100](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911153100.png)

本地域名服务器帮助客户机查询

1. 本地域名服务器查询根域名服务器，根域名服务器告诉本地域名服务器**这个顶级域名应该去哪个顶级域名服务器去查**
2. 本都域名服务器查询顶级域名服务器，顶级域名服务器告诉本地域名服务器**这个二级域名因该去哪个域名服务器去查**
3. 以此类推，知道得到域名的 IP 地址后再把结果返回给客户机


![Pasted image 20220911153011](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911153011.png)


DNS 服务默认用 53 端口，以下报文来自一次 DNS 查询的响应报文

![Pasted image 20220911155638](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911155638.png)

报文表示，DNS 服务器用 53 端口发送了响应报文


## dig 域名解析命令

Linux 中 `dig domain` 也可获取 DNS 查询轨迹

![Pasted image 20220911154254](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911154254.png)


# DNS 报文格式



![Pasted image 20220911154634](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911154634.png)


DNS 其实就是请求域名解析报文和响应域名 IP 地址报文，重点关注 Query 和 Answer 两部分即可

下面是查询 githhub 域名的一次 DNS 请求和响应

这是请求报文
![Pasted image 20220911155413](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911155413.png)

这是响应报文

![Pasted image 20220911155456](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911155456.png)


