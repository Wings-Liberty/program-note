# 安装Docker

## 快速安装

```shell
$ sudo apt install docker.io
$ sudo systemctl start docker
```

查看是否安装成功，显示版本号即安装完成

```shell
$ docker -v
Docker version 18.09.7, build 2d0083d
```


## 阿里云加速

docker下载镜像的默认地址在国外，下载速度慢。所以需要把下载地址转到国内

进入阿里云开发者平台的容器镜像服务，[看这里](https://cr.console.aliyun.com/instances/mirrors)

<img src="https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/%E6%AF%94%E7%89%B9%E6%88%AA%E5%9B%BE2020-06-15-21-23-15.png" alt="比特截图2020-06-15-21-23-15" style="zoom:80%;" />

按照提示执行命令即可完成加速

## 运行 hello world

```shell
$ docker run hello-world
```

若能成功显示 “Hello World !” 内容，表示 docker 可正常使用

# Docker常用命令

## 命令格式


```shell
$ docker 指令 选项 参数
```


## 帮助命令

```shell
# 查看 docker 版本
$ docker version
$ docker -v

# 查看 docker 版本详情
$ docker info

# 查看命令手册
$ docker [options] --help
```


## Docker 的常用名词

`镜像`，`容器`，`仓库`


举个例子：从【仓库】里下载 MySQL【镜像】，根据 MySQL【镜像】启动一个 MySQL 【容器】，启动成功后这个 MySQL 【容器】就是一个可被访问的 MySQL 服务


不过这个 MySQL 服务是运行在宿主机上的一个虚拟机上，这个虚拟机有自己的 IP 和端口，启动的 MySQL 【容器】必须经过虚拟机的 IP 和端口才能访问到


MySQL【镜像】是一个安装有 MySQL 软件的虚拟机模板，根据同一个 MySQL【镜像】启动多个 MySQL【容器】等同于运行多个 MySQL 服务


Docker 的各个容器之间是相互隔离的运行时虚拟机，尽管他们在同一台宿主机上运行


## 镜像的增删查

默认从中央仓库查询和下载镜像


**查镜像**

```shell
# 查本地镜像信息 - a 列出所有镜像(“一个完整的镜像其实是由多个镜像组成的”) q 只显示镜像ID 
# --digests 显示镜像摘要 --no-trunc 显示完整镜像信息
$ docker images
```

```shell
# 搜索远程库中的镜像 - s 后再加数字n列出收藏数不小于n的镜像
# --no-trunc 显示镜像完整信息 --automated 显示auto类型镜像
$ docker search XXX
```

**增**

```shell
# 下载镜像 版本号可指定也可不指定，默认下载latest
$ docker pull XXX[:tag]
```

**删**

```shell
# 单个删除 - f 强制删除 可指定要删除的镜像名或镜像ID
$ docker rmi -f ID/Name

# 批量删除
$ docker rmi -f 镜像名1:tag 镜像名2:tag 镜像名3:tag

# 全部删除； $(docker images -qa) 此命令将列出所有镜像的ID
$ docker rmi -f $(docker images -qa)

# 注：rmi中的i是指镜像(images)
```


## 容器启动，查看，退出，删除

> 启动的容器就像 C 语言中创建的对象，docker 的容器不使用时需要先停止运行（stop），后删除对象（rm，释放资源）

**查容器**

```shell
# 列出近期的容器 -a 列出正在运行和历史上运行过的容器
# -l 最近创建的容器 -n 后加数字m显示最近创建的m个容器
# -q 只显示容器编号 --no-trunc 不截断输出
$ docker ps
```

**启动**

```shell
# 启动容器 默认运行latest版本
# -d 后台运行 -i 交互模式运行 -t 重新分配一个终端 -p 指定运行端口
# -P 随机分配端口映射 --name 指定容器名称
$ docker run -it XXX[:tag]

# 访问本机8888端口即可访问到容器的8080端口
$ docker run -p 8888:8080 --name mycontainer XXX[:tag] 

# 执行此命令后，你会发现当前终端的命令提示符变了。你启动并进入了另一个ubuntu系统的终端，在这里你能像在外面一样执行bash指令（不过只有少数命令可以用）
$ docker run -it -p 3306:3306 --name mymysql mysql
```

**退出**

```shell
# 退容器的终端并关闭容器。容器关闭后不会被立即删除，会进入关机 / 挂起 状态
$ exit

# 退出容器的终端，但不关闭容器
Ctrl + p + q
```

**重启和停止**

```shell
# 启动之前停止的容器
$ docker start 容器ID/Name

# 重启容器
$ docker restart 容器ID/Name

# 停止容器
$ docker stop 容器ID/Name

# 强制停止
$ docker kill 容器ID/Name
```

**删除已停止的容器**

```shell
# 单个删除 批量删除 全部删除 和 删除镜像的操作基本相同
$ docker rm 容器ID/Name
```

## 从宿主机返回到容器的终端


```shell
# 进入退出容器终端前的状态
$ docker attach 容器ID/Name

# 在宿主机向容器终端发送shell命令
$ docker exec 容器ID/Name shell命令

# 如果shell命令是/bin/bash，那么就是进入容器并打开一个新的容器终端
$ docker exec -it 容器ID/Name /bin/bash
```


## 查询容器状态

```shell
# 查看容器详情，如容器名，镜像名，容器使用的数据卷等（数据卷一会再说）
$ docker inspect 容器ID/Name

# 查看容器日志，挺有用的，因为有时会出现启动容器后，容器自动停止的问题，可通过此命令查错 - t 添加时间戳 f 跟随打印日志 --tail 指定打印日志条数
$ docker logs 容器ID/Name

# 查看容器内的进程
$ docker top 容器ID/Name
```


## 宿主机和容器的文件传输


```shell
# 容器和宿主机间的文件拷贝。将前者的文件拷贝到后者指定的路径中。容器文件路径最好使用绝对路径，宿主机路径可以使用相对路径
$ docker cp 容器ID/Name:文件路径 宿主机文件路径
$ docker cp 宿主机文件路径 容器ID/Name:文件路径 
```

注意

- 不管是宿主机传给容器还是容器传给宿主机，都是覆盖方式传递。新文件会覆盖旧文件，且没有覆盖提示
- 不管是宿主机传给容器还是容器传给宿主机，目的地的路径可以是一个不存在的文件（create if absent），也可以是一个已存在的目录
- 不管是宿主机传给容器还是容器传给宿主机，两个文件相互独立，对一个文件的修改不会同步到另一个文件里