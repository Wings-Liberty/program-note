#还没有复习 

# Network 抓包面板

![Pasted image 20220911095446](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911095446.png)


# 控制器

- Preserve log：打开另一个页面时保存原有请求列表
- Disbale cache：停用浏览器缓存
- 手动清除浏览器 Cookie，Cache：在请求列表右键获取操作列表



# 过滤器


## 按类型过滤

请求类型包含：XHR、JS、CSS、Img、Media、Font、Doc、WS (WebSocket)、Manifest 或 Other

多类型，按住 Command (Mac) 或 Ctrl（Windows、Linux）

![Pasted image 20220911100359](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911100359.png)


隐藏 Data URLs：CSS 图片等小文件以 BASE64 格式嵌入 HTML 中，以减少 HTTP  
请求数


## 属性过滤

![Pasted image 20220911100601](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911100601.png)


- 根据请求的域名过滤：`domain:`，可以用通配符字符 `*`
- 根据请求是否含有指定请求头过滤：`has-response-header:`
- `is:` 使用 `is:running` 可以查找 WebSocket 资源，`is:from-cache` 可查找缓存读出的资源
- 根据请求方式过滤：`method:`
- 根据 MIME 类型过滤：`mime-type:`
- 根据协议类型过滤：`scheme:`，可以设置值为 `http`，`https`
- 根据 Cookie 过滤
	- 显示具有 Set-Cookie 标头并且 Domain 属性与指定值匹配的资源：`set-cookie-domain:`
	- 显示具有 Set-Cookie 标头并且名称与指定值匹配的资源：`set-cookie-name:`
	- 显示具有 Set-Cookie 标头并且值与指定值匹配的资源：`set-cookie-value:`
- 根据响应码过滤：`status-code:`

多属性间通过空格实现 AND 操作


## 请求发起者链

此功能用于查询**本请求引起了哪些请求**，**本请求被哪个行为发起的**


比如，/devtools 请求引起了以下多个子请求，子请求又引起了其他子请求

![Pasted image 20220911101743](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911101743.png)


又比如，本请求被哪个 HTML 引起的请求，本请求又引起了哪些请求

![Pasted image 20220911101859](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911101859.png)


又比如，本请求是被哪个脚本启动的

![Pasted image 20220911102013](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911102013.png)


## 分析 WebSocket

过滤器设置：按类型：`WS`，属性过滤：`is: running`


![Pasted image 20220911234659](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220911234659.png)


