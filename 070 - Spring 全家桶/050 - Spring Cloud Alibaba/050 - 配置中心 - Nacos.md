#还没有复习 

nacos 配置中心的核心功能

1.  一个服务可以指定读取多个配置文件，nacos 根据什么优先级规则读取文件？
    
2.  nacos client 能实时感知 nacos server 中的配置变化，这是如何实现的
    

1.  本地文件 < 远程文件
    

nacos client 被要求配置 application-name，profile，extend-filename...

nacos会根据上述可选配置用内置的优先级顺序读取文件，覆盖原有配置项

2.  client 长轮询获取更新
    

client 定时发送请求给 server，请求超时时间为 30s

server 收到请求后，如果和上次请求相比，配置无变化，则 server 在 29.5s 后再给 client 返回结果；否则立即返回结果

在 nacos server 中，server 会把配置信息保存到数据库中，同时还会把配置写入本地磁盘

每次读时都读磁盘

每次写配置时采用双写，写磁盘，写数据库

这样的话能加快读配置时间