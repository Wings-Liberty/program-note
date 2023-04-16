# 反向代理实例


## 访问指定端口，转发到指定地址


```
server {
	listen 80;
	location / {
		proxy_pass http://127.0.0.1:8080;
	}
}
```

## 访问指定端口和路由前缀，转发到指定地址


[访问 http://127.0.0.1:9001/edu/ 直接跳转到 127.0.0.1:8081](<nginx 监听端口为 9001，
访问 http://127.0.0.1:9001/edu/ 直接跳转到 127.0.0.1:8081
访问 http://127.0.0.1:9001/vod/ 直接跳转到 127.0.0.1:8082

```
server {
	listen 88;
	location /test/ {
		proxy_pass  http://127.0.0.1:8081/
	}
	location /dev/ {
		proxy_pass  http://127.0.0.1:8082/
	}
}
```

`/test` 或 `/dev` 都会被替换成 `/`

如果访问 `/test/hello` 或 `/dev/hello` 最后都会被替换成 `/hello`

但以下配置方式都是错误的

```
location /test {
	proxy_pass  http://127.0.0.1:8081
}
location /dev {
	proxy_pass  http://127.0.0.1:8082
}
```

```
location ~ /test/ {
	proxy_pass  http://127.0.0.1:8081
}
location ~ /dev/ {
	proxy_pass  http://127.0.0.1:8082
}
```

加了 `~` 后，不管 `proxy_pass` 后面加 `/` 没，都不好用

```
location ^~ /test {
		proxy_pass      http://101.43.143.178:8081;
}
location ^~ /dev {
		proxy_pass      http://101.43.143.178:8082;
}
```

也不行。公司用的是 `rewrite` 指令把 /api/xx 换成了 /xx

```
location ^~ /api {
	rewrite ^/api/(.*)$ /$1 break;
	proxy_pass https://gateway;
}
```

如果 nginx 在容器中启动，那 proxy_pass 不要用 127.0.0.1，因为容器会把请求转发给容器本身，而不是宿主机。

##  负载均衡

```
http {
	upstream myserver {
		ip_hash;
		server 101.43.143.178:8082 weight=1;
		server 101.43.143.178:8081 weight=1;
	}

	server {
		listen 80;
		location / {
			proxy_pass  http://myserver;
		}
	}
}
```

- 默认用轮询
- 用 `weight` 调整权重。轮询就是 `weight` 都等于 1 的策略
- ip_hash 根据 ip 为每个客户端 ip 分配一个固定的 server ip，weight 会失效
