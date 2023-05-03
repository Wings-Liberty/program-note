#正在复习 

> 以下操作均在Ubuntu18下进行





# Docker Compose

Docker Compose 是以 yml 文件方式配置管理多个容器的配置和启动

使用方式如下

1. 为需要 Dockerfile 的容器编写 Dockerfile
2. 编写 docker-compose。yml 文件
3. 运行 `docker compose up`



举个例子，现在需要启动一个 webapp 和一个 redis，下面是一个比较简洁的配置

```yaml
version: '3'
services:
	web:
        build: .
        ports:
            - "5000:5000"
        volumes:
            - .:/code
            - logvolume01:/var/log
        links:
            - redis
	redis:
		image: redis
```



docker compose 相当于用一个 yml 文件保存了多个 docker 命令，执行 `docker compose up` 相当于运行了多个 docker 命令

只不过把 docker 命令改成了 yml 的写法



docker compose 更多指令参考

- [菜鸟教程](https://www.runoob.com/docker/docker-compose.html)
- [Docker Compose 模板文件](https://yeasy.gitbook.io/docker_practice/compose/compose_file#image)
- [docker-compose 命令说明](https://www.breword.com/yeasy-docker_practice/compose/commands)

# 搭建公有云仓库 方式

先根据第一步，在服务器上下载 docker，然后搭建 portainer，教程参考[这里](https://blog.csdn.net/m0_67900727/article/details/123550536)

然后用 portainer 连接上阿里云镜像服务作为云仓库

阿里云镜像服务使用方式参考下一个标题

