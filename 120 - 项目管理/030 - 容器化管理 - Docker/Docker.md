> 
>
> 以下操作均在Ubuntu18下进行

# 安装Docker



## 简易安装

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

## 运行hello world

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
# 查看docker版本
$ docker version
$ docker -v

# 查看docker版本详情
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



## 宿主机和容器的交互



```shell
# 容器和宿主机间的文件拷贝。将前者的文件拷贝到后者指定的路径中。容器文件路径最好使用绝对路径，宿主机路径可以使用相对路径
$ docker cp 容器ID/Name:文件路径 宿主机文件路径
$ docker cp 宿主机文件路径 容器ID/Name:文件路径 
```





# Docker 容器数据卷

> 数据卷：容器和宿主机共享文件和文件夹



## 数据卷



### 命令行实现

此时，两个目录中的文件资源共享。如果停止容器，主机修改文件，重启容器，文件可同步共享

```shell
# :ro 指定主机对共享文件夹只读不可写，可不加
$ docker run -it -v 宿主机中文件或目录的路径:容器内的文件或目录的路径[:ro] 镜像名
```



### DockerFile 实现

> DockerFile 是一个文件，没有文件拓展名，使一个 DockerFile 和一个镜像可以构建一个新的镜像。先做一个了解，DockerFile 的具体内容后面

1. 写一个 DockerFile 文件

   ```dockerfile
   # volume test
   FROM ubuntu
   VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
   CMD echo "finished,--------success1"
   CMD /bin/bash
   ```

2. 构建新镜像

   ```shell
   # -f 指明DockerFile文件的路径 t 镜像空间 . 在当前目录下（指定本机的一个目录作为“根目录”，便于DockerFile中指定本地文件路径）
   $ docker build -f DockerFile文件路径 -t 新的镜像名 .
   ```

   上述命令中最后的'.'的作用[见这里](http://blog.haohtml.com/archives/18711)

   ```shell
   # 下面是输出结果，表示成功
   Sending build context to Docker daemon  3.072kB
   Step 1/4 : FROM ubuntu
    ---> 72300a873c2c
   Step 2/4 : VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
    ---> Running in 6c4df6475927
   Removing intermediate container 6c4df6475927
    ---> 803f33bd827c
   Step 3/4 : CMD echo "finished,--------success1"
    ---> Running in d8b63b3ce071
   Removing intermediate container d8b63b3ce071
    ---> 7b22c0dfa779
   Step 4/4 : CMD /bin/bash
    ---> Running in bfb1c6bf79f5
   Removing intermediate container bfb1c6bf79f5
    ---> ad89086f12d0
   Successfully built ad89086f12d0
   Successfully tagged cx/ubuntu:latest
   ```

3. 运行容器，并检查容器内的容器卷

   ```shell
   # 我这里上一步构建的新的镜像名为'cx/ubuntu'
   $ docker run -it cx/ubuntu
   $ ll
   
   # 看到容器卷即为成功
   drwxr-xr-x   2 root root 4096 Feb 27 03:41 dataVolumeContainer1/
   drwxr-xr-x   2 root root 4096 Feb 27 03:41 dataVolumeContainer2/
   ```

4. 检查宿主机内的容器卷

   ```shell
   $ docker inspect 容器ID/Name | grep /var/lib/docker/volumes
   
   # 查找以下内容
   "Mounts": [
               {
                   "Type": "volume",
                   "Name": "8f5aa115de378d169c05ac4dcb430fb822f8df165797fb0caa796b7b9324d428",
                   "Source": "/var/lib/docker/volumes/8f5aa115de378d169c05ac4dcb430fb822f8df165797fb0caa796b7b9324d428/_data",
                   "Destination": "/dataVolumeContainer1",
                   "Driver": "local",
                   "Mode": "",
                   "RW": true,
                   "Propagation": ""
               },
               {
                   "Type": "volume",
                   "Name": "37828dfbfef2c2576382a52308b72ab6f721d0446a84f1b78bb765ca77f60d4e",
                   "Source": "/var/lib/docker/volumes/37828dfbfef2c2576382a52308b72ab6f721d0446a84f1b78bb765ca77f60d4e/_data",
                   "Destination": "/dataVolumeContainer2",
                   "Driver": "local",
                   "Mode": "",
                   "RW": true,
                   "Propagation": ""
               }
           ]
   # 在/var/lib/docker/volumes目录下即为宿主机默认的容器卷
   ```

==注：Docker 挂载主机目录 Docker 访问出现 cannot open directory .: Permission denied 解决办法：在挂载目录后多加一个 --privileged=true 参数即可==

## 数据卷容器

> 以上内容是实现了宿主机和容器的资源共享
>
> 下面是容器之间的资源共享

1. 启动一个有 DockerFile 的镜像（我这里使用的是修改过的 ubuntu 镜像）容器 dc01

   ```shell
   $ docker run -it --name doc01 cx/ubuntu
   ```

2. 启动另一个镜像容器 dc02，它共享了 dc01 的资源

   ```shell
   $ docker run -it --name dc02 --volumes-from dc01 cx/ubuntu
   ```

共享后只要还有容器在使用这个数据卷容器，数据卷容器里的资源就会持续存在。数据卷容器具有传递性



# Docker 镜像



## FAQ



Q：Docker 镜像分层是什么意思？

A：例如一个 tomcat 镜像，它需要 java 环境，java 环境又需要系统环境，操作系统又需要 linux 的内核，镜像一层一层的堆叠起来。所以一个镜像会分好多层，而用户直接接触到的只是最外层的 tomcat



Q：分层有什么好处？

A：举个例子，tomcat 的镜像，MySQL 的镜像，redis 的镜像都需要 linux 的内核。如果这 3 个镜像能用同一个 linux 内核分层就能提高层的复用率。如果当前宿主机已经有 MySQL 镜像，下载 redis 时需要先下一个 linux 内核，结果发现 MySQL 镜像用的 linux 层版本和 redis 镜像所需的版本一致，就可以直接复用本地已有的层



## 创建 docker 镜像

> 创建 docker 镜像有两种方式。用`docker commit`构建镜像和用 DockerFile 文件构建镜像。这里只说第一种方式，第二种方式会在 DockerFile 部分说明

在容器中一顿操作后（例如修改了某个文件，在容器中下载了某个软件包等），想把操作后的容器的状态保留下来，把**容器当前的状态**变成镜像即可

```shell
# commit后容器就会变成镜像并添加到宿主机的镜像列表
$ docker commit -m="提交的描述信息" -a="作者名" 容器ID/Name 要创建的目标镜像名:[标签名]
# 数据卷中的文件不会被放到新镜像中，但是数据卷的文件夹会在新镜像中（等看完数据卷再看这个吧）
```



怎么看创建镜像时的提交信息？（这里的提交信息指的是上述`docker commit`命令中的 `-m`携带的描述信息）

```shell
$ docker inspect ImageName/ImageID | grep Comment
# grep -i 忽略大小写
$ docker inspect ImageName/ImageID | grep -i comm
```



# DockFile

> DockerFile 是用来构建 Docker 镜像的构建文件，是由一系列命令和参数构成的脚本

知识补充：'scratch'   源镜像   就像是 java 中的Object类，所有镜像都会默认继承它



 从应用软件的角度来看，Dockerfile、Docker 镜像与 Docker 容器分别代表软件的三个不同阶段

- Dockerfile 是软件的原材料
- Docker 镜像是软件的交付品
- Docker 容器则可以认为是软件的运行态

Dockerfile 面向开发，Docker 镜像成为交付标准，Docker 容器则涉及部署与运维，三者缺一不可，合力充当 Docker 体系的基石。

1. Dockerfile，需要定义一个Dockerfile，Dockerfile 定义了进程需要的一切东西。Dockerfile 涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程（当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制）等等
2. Docker 镜像，在用 Dockerfile 定义一个文件之后，docker build时会产生一个 Docker 镜像，当运行 Docker 镜像时，会真正开始提供服务
3. Docker 容器，容器是直接提供服务的



## DockerFile 的常用保留字

| 保留字 | 简述 |
| :------: | :----: |
| FROM | 基础镜像，当前新镜像是基于哪个镜像的 |
| MAINTAINER | 镜像维护者的姓名和邮箱地址 |
| RUN | 容器构建时需要运行的命令 |
| EXPOSE | 当前容器对外暴露的端口号 |
| WORDIR | 指定在容器创建后，终端默认登录进来工作目录，一个落脚点 |
| ENV | 用来在构建镜像过程中设置环境变量  举例：ENV MY_PATH /usr/mytestENV 这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前指定了环境变量前缀一样；也可以在其他指令中直接使用这些环境变量，比如 WORKDIR $MY_PATH |
| ADD | 将宿主机目录下的文件拷贝到镜像里面并且 ADD 命令会自动处理 URL 和解压 tar 压缩包 |
| COPY | 类似 ADD，拷贝文件和目录到镜像中，但是它只是拷贝，不会自动处理 URL 和解压 tar 压缩包。COPY 将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置。 |
| VOLUME | 容器数据卷，用于数据保存和持久化工作 |
| CMD | 指定一个容器启动时要运行的命令。dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换。 |
| ENTRYPOINT | 指定一个容器启动时要运行的命令。ENTRYPOIT 的目的和 CMD 一样，都是在指定容器启动程序及参数。 |
| ONBUILD | 当构建一个被继承的 Dockerfile 时运行命令，父镜像在被子继承后，父镜像的 onbuild 被触发。 |



各个保留字的具体使用方式可以参考[这里](https://www.runoob.com/docker/docker-dockerfile.html)



CMD 和 ENTRYPOINT 的区别

都是指定一个容器启动时要运行的命令，但如果 dockerfile 中可以有多个 CMD 指令，只有最后一个生效 ，CMD 会被 docker run 之后的参数替换 ；而有多个 ENTRYPOINT 指令，每个指令都生效，这就是 CMD 和 ENTRYPOINT 的区别



这里有一个 Dockerfile 的文件例子

```dockerfile
FROM ubuntu
MAINTAINER zzyy<zzyy167@126.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN apt -y install vim
RUN apt -y install net-toolsEXPOSE 80CMD echo $MYPATHCMD echo "success--------------ok"
CMD /bin/bash
```





## DockerFile 原理

> DockerFile 是用来构建 Docker 镜像的构建文件，是由一系列命令和参数构成的脚本

在执行 DockFile 的时候

先实例化一个容器，然后从上到下执行命令。每执行一条命令创建一个镜像 





# 本地镜像发布到阿里云

我们都知道有一个github，也知道有一个gitee。

docker 有一个 dockerhub，多放置各大官方的 docker 镜像，因为是外网速度慢管理不方便。国内阿里云提供 docker 镜像仓库服务（就像是国内的dockerhub），我们用这个

1. 有一个完整的镜像

2. 进入阿里云[容器镜像服务](https://cr.console.aliyun.com/instances/repositories)，创建一个命名空间，创建一个镜像仓库（需要命名空间）

3. 将镜像推送到 Registry

   ```shell
   # 登陆阿里云的容器镜像服务
   $ sudo docker login --username=xxx registry.cn-beijing.aliyuncs.com
   # 打版本号。生成新的镜像
   $ sudo docker tag [ImageId] registry.cn-beijing.aliyuncs.com/wings-liberty/myubuntu:[镜像版本号]
   # 推送
   $ sudo docker push registry.cn-beijing.aliyuncs.com/wings-liberty/myubuntu:[镜像版本号]
   ```

   注意：要推送到远程库的镜像，镜像名必须有前面类似`registry.cn-beijing.aliyuncs.com/wings-liberty`的前缀，这是用来指明推送的地址（命名空间/仓库名）

4. 在[容器镜像服务](https://cr.console.aliyun.com/instances/repositories)中搜索并 pull 自己的镜像



# Docker 基础网络

> 参考博客 [Docker 基础网络](https://www.jianshu.com/p/2d02bb9a9da5)，[为 Docker 容器指定 IP](https://www.jianshu.com/p/b8625ccb5e9c)

## 前言

在安装`docker`时，`docker`创建了三种网络模式。`host`、`none`、`bridge`

可使用下面的命令查看网络模式列表

```shell
$ docker network ls
```

```shell
NETWORK ID          NAME                DRIVER              SCOPE
f9005c78a892        bridge              bridge              local
16d3185c14a4        host                host                local
04bd503cccdf        none                null                local
```



## host 网络

介绍：容器创建时通过 --network=host 指定使用宿主机网络，此时容器与宿主机共享网络栈，容器内的网络配置和宿主机完全一样。

使用场景：选用 host 网络的容器，其网络栈和宿主机一摸一样，它的优势在于网络性能强于其他网络模式。如果对网络传输有很大需求可以选用host网络



## none 网络

介绍：容器创建时通过 --network=none 指定容器不创建任何网卡，此时容器里只有 lo（lo 表示 localhost）

使用场景：none 没有网卡的网络，能做到更加封闭，可以更好的保护重要数据，所以最适合对安全性要求高并且不需要联网的容器



## bridge 网络

介绍：容器创建时不指定 --network，那么容器默认使用 bridge 网络。bridge 网络是由 docker 创建的 linux bridge -- docker0 提供

使用场景：bridge 网络是通过容器上虚拟网络设备和网桥上虚拟网络设备组成一组 veth（相当于虚拟的网线）进行连接的，然后通过 docker0 从172.17.0.0/16 分配 ip 给容器使用。显而易见 bridge 网络适用于日常需要连接网络的容器，例如 http 容器、web 容器...

可用以下命令查看网桥

```shell
$ brctl show
```

```shell
bridge name	bridge id		    STP enabled	   interfaces
docker0		8000.0242b4c83297	no
```

使用以下命令查看网桥的网关

```shell
# 第一条命令只会打印一行数据，要想查看所有数据，需要使用第二条命令，其中docker0开头的数据是docker的网桥和网关
$ ifconfig | grep docker0
$ ifconfig
```

```shell
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:b4:c8:32:97  txqueuelen 0  (Ethernet)
        RX packets 4377  bytes 278845 (278.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 15086  bytes 34924384 (34.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

由于 docker0 网桥是安装 docker 时就默认创建的，我们无法在创建容器的时候指定容器 ip，只能由 docker0 自动分配 ip

如果想要创建容器的时候指定 ip，需要自己再建一个 bridge 网络



## 创建 docker 容器时给容器分配 ip

上面介绍了 docker 自带网络的三种模式

其中提到过 bridge（docker0）网络由于是 docker 安装时就创建的，无法在创建容器的时候指定容器 ip

那么在实际部署中，我们需要指定容器 ip，不允许其自行分配 ip，防止容器 ip 混乱。

Q：怎么可以在创建容器时指定容器 ip ？

A：自己创建一个新的 bridge 网络 bridge1，在创建 bridge1 的时候同时创建子网，那么在创建容器的时候指定网络为 bridge1 并指定 ip 即可

```shell
# 最后的cxbridge是网桥的名字
$ docker network create --driver bridge --subnet=172.16.12.0/16 --gate=172.16.1.1 cxbridge
```

```shell
$ docker network ls
```

查看网桥详情

```shell
$ docker network inspect cxbridge
```

创建好网桥后，运行容器是即可使用以下命令指定 ip，此 ip 需在上述创建网桥时设定的范围内。上述创建网桥时设置的 ip 范围为`172.16.12.0/16`，

```shell
$ docker run -it --name ubunut --network=cxbridge --ip 172.16.12.12 centos
```

若指定了网桥（假设指定的网桥是 cxbridge），但没有指定 ip，将会自动在`172.16.12.0/16`选一个 ip 分配给容器



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

