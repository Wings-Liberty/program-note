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

![比特截图2020-06-15-21-23-15](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/%E6%AF%94%E7%89%B9%E6%88%AA%E5%9B%BE2020-06-15-21-23-15.png)

阿里云分配的加速地址是不同的

按照提示执行命令即可完成加速

## 运行hello world

```shell
$ docker run hello-world
```

若显示以下内容，表示docker可正常使用

```shell
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:fc6a51919cfeb2e6763f62b6d9e8815acbf7cd2e476ea353743570610737b752
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

# Docker常用命令



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



## Docker的常用名词

`镜像`，`容器`，`仓库`

容器，镜像，仓库的关系就像java中的 对象，类，maven仓库的关系

java中：对象是类的实例化，类是自己写的或第三方提供的

docker中：容器是镜像的实例化，镜像是自己构建的或第三方提供的



## 镜像的增删查

**查即镜像**

```shell
# 查本地镜像信息 - a 列出所有镜像(“一个完整的镜像其实是由多个镜像组成的”) q 只显示镜像ID 
# --digests 显示镜像摘要 --no-trunc 显示完整镜像信息
$ docker images
```

```shell
# 搜索远程库中的镜像 - s 后再加数字n列出收藏数不小于n的镜像
# --no-trunc 显示镜像完整信息 --automated 显示auto类型镜像
$ docker search XXX
$ docker search ubuntu
```

**增**

```shell
# 下载镜像 版本号可指定也可不指定，默认下载latest
$ docker pull XXX[:tag] 
$ docker pull ubuntu
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

> 启动的容器就像C语言中创建的对象，docker的容器不使用时需要先停止运行（stop），后删除对象（rm，释放资源）

**查容器**

```shell
# 查询正在运行的容器 -a 列出正在运行和历史上运行过的容器
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
$ docker run -it ubuntu
```

**退出**

```shell
# 退容器的终端并关闭容器
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

> 宿主机：安装docker的机器

```shell
# 进入退出容器终端前的状态
$ docker attach 容器ID/Name

# 在宿主机向容器终端发送shell命令
$ docker exec 容器ID/Name shell命令

# 如果shell命令是/bin/bash，那么就是进入容器并打开一个新的容器终端
$ docker exec -it 容器ID/Name /bin/bash
```



## 杂项

```shell
# 容器和宿主机间的文件拷贝。将前者的文件拷贝到后者指定的路径中。容器文件路径最好使用绝对路径，宿主机路径可以使用相对路径
$ docker cp 容器ID/Name:文件路径 宿主机文件路径
$ docker cp 宿主机文件路径 容器ID/Name:文件路径 

# 查看容器详情，如容器名，镜像名，容器使用的数据卷等（数据卷一会再说）
$ docker inspect 容器ID/Name

# 查看容器日志，挺有用的，因为有时会出现启动容器后，容器自动停止的问题，可通过此命令查错 - t 添加时间戳 f 跟随打印日志 --tail 指定打印日志条数
$ docker logs 容器ID/Name

# 查看容器内的进程
$ docker top 容器ID/Name
```



# Docker镜像

- Docker镜像分层是什么意思？

例如：一个tomcat镜像，它还需要java环境，java环境又需要系统环境，操作系统又需要linux的内核，镜像一层一层的堆叠起来。所以一个镜像会分好多层，而用户直接接触到的只是最外层的tomcat

## 创建docker镜像

> 创建docker镜像有两种方式。用`docker commit`构建镜像和用DockerFile文件构建镜像。这里只说第一种方式，第二种方式会在DockerFile部分说明

在容器中一顿操作后（例如修改了某个文件，在容器中下载了某个软件包等），想把操作后的容器的状态保留下来，把**容器当前的状态**变成镜像即可

```shell
# commit后容器就会变成镜像并添加到宿主机的镜像列表
$ docker commit -m=“提交的描述信息” -a=“作者名” 容器ID/Name 要创建的目标镜像名:[标签名]
# 数据卷中的文件不会被放到新镜像中，但是数据卷的文件夹会在新镜像中（等看完数据卷再看这个吧）
```



怎么看创建镜像时的提交信息？（这里的提交信息指的是上述`docker commit`命令中的 `-m`携带的描述信息）

```shell
$ docker inspect ImageName/ImageID | grep Comment
# grep -i 忽略大小写
$ docker inspect ImageName/ImageID | grep -i comm
```



# Docker容器数据卷

> 容器和宿主机共享文件

## 数据卷



### 命令行实现

此时，两个目录中的文件资源共享。如果停止容器，主机修改文件，重启容器，文件可同步共享

```shell
# :ro 指定主机对共享文件夹只读不可写，可不加
$ docker run -it -v 宿主机中文件或目录的路径:容器内的文件或目录的路径[:ro] 镜像名
```



### DockerFile实现

> DockerFile是一个文件，没有文件拓展名，使用一个DockerFile和一个镜像可以构建一个新的镜像。先做一个了解，DockerFile的具体内容后面

1. 编写一个DockerFile文件

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
   

==注：Docker挂载主机目录Docker访问出现cannot open directory .: Permission denied解决办法：在挂载目录后多加一个--privileged=true参数即可==

## 数据卷容器

> 以上内容是实现了宿主机和容器的资源共享
>
> 下面是容器之间的资源共享

1. 启动一个有DockerFile的镜像（我这里使用的是修改过的ubuntu镜像）容器dc01

   ```shell
   $ docker run -it --name doc01 cx/ubuntu
   ```

2. 启动另一个镜像容器dc02，它共享了dc01的资源

   ```shell
   $ docker run -it --name dc02 --volumes-from dc01 cx/ubuntu
   ```

共享后只要还有容器在使用这个数据卷容器，数据卷容器里的资源就会持续存在。数据卷容器具有传递性

# DockFile

> DockerFile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本

知识补充：'scratch'   源镜像   就像是java中的Object类，所有镜像都会默认继承它



 从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段。

- Dockerfile是软件的原材料
- Docker镜像是软件的交付品
- Docker容器则可以认为是软件的运行态

Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

1. Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等; 
2. Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务;
3. Docker容器，容器是直接提供服务的。



## DockerFile的常用保留字

| 保留字 | 简述 |
| :------: | :----: |
| FROM | 基础镜像，当前新镜像是基于哪个镜像的 |
| MAINTAINER | 镜像维护者的姓名和邮箱地址 |
| RUN | 容器构建时需要运行的命令 |
| EXPOSE | 当前容器对外暴露的端口号 |
| WORDIR | 指定在容器创建后，终端默认登录进来工作目录，一个落脚点 |
| ENV | 用来在构建镜像过程中设置环境变量  举例：ENV MY_PATH /usr/mytestENV这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前指定了环境变量前缀一样；也可以在其他指令中直接使用这些环境变量，比如WORKDIR $MY_PATH |
| ADD | 将宿主机目录下的文件拷贝到镜像里面并且ADD命令会自动处理URL和解压tar压缩包 |
| COPY | 类似ADD,拷贝文件和目录到镜像中，但是它只是拷贝，不会自动处理URL和解压tar压缩包。COPY将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置。 |
| VOLUME | 容器数据卷，用于数据保存和持久化工作 |
| CMD | 指定一个容器启动时要运行的命令。dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被docker run之后的参数替换。 |
| ENTRYPOINT | 指定一个容器启动时要运行的命令。ENTRYPOIT的目的和CMD一样，都是在指定容器启动程序及参数。 |
| ONBUILD | 当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后，父镜像的onbuild被触发。 |

CMD和ENTRYPOINT 的区别

都是指定一个容器启动时要运行的命令，但是 如果dockerfile中可以有多个CMD指令，只有最后一个生效 ，CMD会被docker run之后的参数替换 ；而有多个ENTRYPOINT指令，每个指令都生效，这就是CMD和ENTRYPOINT的区别。



这里有一个Dockerfile的文件例子

```dockerfile
FROM ubuntu
MAINTAINER zzyy<zzyy167@126.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN apt -y install vim
RUN apt -y install net-toolsEXPOSE 80CMD echo $MYPATHCMD echo "success--------------ok"
CMD /bin/bash
```





## Docker原理

> DockerFile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本

在执行DockFile的时候

先实例化一个容器，然后从上到下执行命令。每执行一条命令创建一个镜像 



# 案例mysql

需要做的事：

- 创建数据卷

- 进入mysql，并创建一个数据库和一张表
- 将刚创建的数据库的备份存到数据卷中

1. 安装一个mysql

   ```shell
   # 这里默认下载最新版本的mysql镜像
   $ docker pull mysql
   ```

2. 运行mysql镜像

   ```shell
   # -d 后台运行
   $ docker run -d mysql
   ```

   ```shell
   2020-02-29 13:54:45+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.19-1debian9 started.
   2020-02-29 13:54:45+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
   2020-02-29 13:54:45+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.19-1debian9 started.
   2020-02-29 13:54:45+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
   	You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
   ```

   这里抛出了一些日志，其中有一个ERROR级别的日志导致无法创建容器

   ```shell
   2020-02-29 13:54:45+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
   	You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
   ```

   - `MYSQL_ROOT_PASSWORD` ：**强制要求**的，会在容器创建时将root用户密码设置成改变量的值。没有启动会报错
   - `MYSQL_DATABASE`：可选项。会在容器创建的时候创建改数据库。如果有指定用户名和密码，会将该用户设置成改数据库的超级用户
   - `MYSQL_USER`, `MYSQL_PASSWORD`：可选项。容器创建时创建该用户

   创建mysql容器的时候需要指明root用户的密码

   那么就补上root用户的密码

   ```shell
   # 这里就指明root用户的密码为'password'
   $ docker run -d - v /test/databasesdata:/test/data -e MYSQL_ROOT_PASSWORD=password mysql
   ```

3. 进入容器的命令行

   由于启动了mysql服务，该终端会一直停留在mysq服务打印日志的状态。使用docker attach命令会进到这个无法写命令的终端，如果误进，使用Ctrl+p+q执行不停止退出

   ```shell
   # 查找出容器的ID
   $ docker ps
   
   # 使用exec命令进入容器并打开一个新终端
   $ docker exec -it 容器ID /bin/bash
   ```

4. 进入mysql

   ```shell
   $ mysql -uroot -p
   # 密码使用之前设置的密码
   
   # 进入mysql...建库，建表
   ```

5. 回到宿主机并保存sql文件

   ```shell
   docker exec ffbe3d38223d sh -c ' exec mysqldump --all-databases -uroot -p"password"  ' > /test/all-databases.sql
   ```



下面这行指令作为启动 mysql 容器的参考

```bash
$ docker pull mysql:5.7
# 找到一份 mysql5.7 的配置文件，修改文件中 data 目录的位置
$ docker run -d  -p 3306:3306 --name mysql -v /etc/mysql/mysql.conf.d:/etc/mysql/conf.d -v /opt/docker/data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=1434948003Cx! --restart=always mysql:5.7
```



# 案例 - RocketMQ（单节点启动）

[参考这里](https://blog.csdn.net/qq_34125999/article/details/117332370)



```bash
docker pull foxiswho/rocketmq:server-4.3.2
docker pull foxiswho/rocketmq:broker-4.3.2
docker pull styletang/rocketmq-console-ng:1.0.0
```





```bash
docker run -p 9876:9876 --name nameserver -e "JAVA_OPT_EXT=-server -Xms256m -Xmx256m -Xmn256m" -e "JAVA_OPTS=-Duser.home=/opt" -v /usr/local/rocketmq/rmqserver/logs:/opt/logs -v /usr/local/rocketmq/rmqserver/store:/opt/store -d foxiswho/rocketmq:server-4.3.2

docker run -p 10911:10911 -p 10909:10909 --name broker -e "JAVA_OPTS=-Duser.home=/opt" -e "JAVA_OPT_EXT=-server -Xms128m -Xmx128m -Xmn128m" -v /usr/local/rocketmq/rmqbroker/conf/broker.conf:/etc/rocketmq/broker.conf -v /usr/local/rocketmq/rmqbroker/logs:/opt/logs -v /usr/local/rocketmq/rmqbroker/store:/opt/store -d foxiswho/rocketmq:broker-4.3.2

docker run --name rocketmq-console -e "JAVA_OPTS=-Drocketmq.namesrv.addr=124.223.70.38:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" -p 8082:8080 -t -d styletang/rocketmq-console-ng:1.0.0

docker run --name rocketmq-console -e "JAVA_OPTS=-Drocketmq.namesrv.addr=124.223.70.38:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" -p 8082:8080 -t -d styletang/rocketmq-console-ng:1.0.0
```



# 本地镜像发布到阿里云

我们都知道有一个github，也知道有一个gitee。

docker有一个dockerhub，多放置各大官方的docker镜像，因为是外网速度慢管理不方便。国内阿里云提供docker镜像仓库服务（就像是国内的dockerhub），我们用这个

1. 有一个完整的镜像

2. 进入阿里云[容器镜像服务](https://cr.console.aliyun.com/instances/repositories)，创建一个命名空间，创建一个镜像仓库（需要命名空间）

3. 将镜像推送到Registry

   ```shell
   # 登陆阿里云的容器镜像服务
   $ sudo docker login --username=刻舟求虐 registry.cn-beijing.aliyuncs.com
   # 打版本号。生成新的镜像
   $ sudo docker tag [ImageId] registry.cn-beijing.aliyuncs.com/wings-liberty/myubuntu:[镜像版本号]
   # 推送
   $ sudo docker push registry.cn-beijing.aliyuncs.com/wings-liberty/myubuntu:[镜像版本号]
   ```

   注意：要推送到远程库的镜像，镜像名必须有前面类似`registry.cn-beijing.aliyuncs.com/wings-liberty`的前缀，这是用来指明推送的地址（命名空间/仓库名）

4. 在[容器镜像服务](https://cr.console.aliyun.com/instances/repositories)中搜索并pull自己的镜像



# Docker基础网络

> 参考博客[docker基础网络](https://www.jianshu.com/p/2d02bb9a9da5)，[为docker容器指定ip](https://www.jianshu.com/p/b8625ccb5e9c)

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



## host网络

介绍：容器创建时通过 --network=host 指定使用宿主机网络，此时容器与宿主机共享网络栈，容器内的网络配置和宿主机完全一样。

使用场景：选用host网络的容器，其网络栈和宿主机一摸一样，它的优势在于网络性能强于其他网络模式。如果对网络传输有很大需求可以选用host网络。

## none网络

介绍：容器创建时通过 --network=host 指定容器不创建任何网卡，此时容器里只有lo（lo表示localhost）

使用场景：none没有网卡的网络，能做到更加封闭，可以更好的保护重要数据，所以最适合对安全性要求高并且不需要联网的容器。

==以上内容为简书博客copy来的，none网络为什么是用--network=host指定none网络？是不是作者笔误==

## bridge网络

介绍：容器创建时不指定--network，那么容器默认使用bridge网络。bridge网络是由docker创建的linux bridge -- docker0提供。

使用场景：bridge网络是通过容器上虚拟网络设备和网桥上虚拟网络设备组成一组veth（相当于虚拟的网线）进行连接的，然后通过docker0从172.17.0.0/16分配ip给容器使用。显而易见bridge网络适用于日常需要连接网络的容器，例如http容器、web容器...

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

由于docker0 网桥是安装docker时就默认创建的，我们无法在创建容器的时候指定容器ip，只能由docker0自动分配ip。

如果想要创建容器的时候指定ip，需要自己再建一个bridge网络。



## 创建docker容器时给容器分配ip

上面介绍了docker自带网络的三种模式。
其中提到过bridge（docker0）网络由于是docker安装时就创建的，无法在创建容器的时候指定容器ip。
那么在实际部署中，我们需要指定容器ip，不允许其自行分配ip，防止容器ip混乱。

有什么办法可以在创建容器时指定容器ip呢？很简单，自己创建一个新的bridge网络bridge1，在创建bridge1的时候同时创建子网，那么在创建容器的时候指定网络为bridge1并指定ip即可。

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

创建好网桥后，运行容器是即可使用以下命令指定ip，此ip需在上述创建网桥时设定的范围内。上述创建网桥时设置的ip范围为`172.16.12.0/16`，即0~16

```shell
$ docker run -it --name ubunut --network=cxbridge --ip 172.16.12.12 centos
```

若指定了网桥（假设指定的网桥是cxbridge），但没有指定ip，将会自动在`172.16.12.0/16`选一个ip分配给容器



# IDEA创建Docker镜像

以下操作均在IDEA和云服务器中进行

IDEA构建Docker镜像的本质是IDEA连接到Docker的Client后在IDEA上进行可视化操作



## IDEA连接云服务器的Docker服务



### 云服务器开启Docker的2375端口

1. 创建文件（无脑复制粘贴执行即可）

   ```shell
   mkdir -p /etc/systemd/system/docker.service.d
   cat > /etc/systemd/system/docker.service.d/tcp.conf <<EOF
   [Service]
   ExecStart=
   ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
   EOF
   ```

2. 重启Docker服务

   ````shell
   systemctl daemon-reload
   systemctl restart docker
   ````

3. 查看2375端口开启情况

   ```shell
   ps aux | grep dockerd | grep -v grep
   ```

如果想关掉2375端口，删除步骤1创建的文件，重启Docker服务（和步骤2的操作相同）即可。



### IDEA连接云服务器的Docker服务

一般来说下载IDEA后会自动下载Docker的插件

![比特截图2020-06-11-18-54-33](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/%E6%AF%94%E7%89%B9%E6%88%AA%E5%9B%BE2020-06-11-18-54-33.png)

输入地址后IDEA会自动连接



## 配置Docker的Configurations

![Snipaste_2020-06-11_20-47-45](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2020-06-11_20-47-45.png)



## 写DockersFile

![比特截图2020-06-11-20-32-08](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/%E6%AF%94%E7%89%B9%E6%88%AA%E5%9B%BE2020-06-11-20-32-08.png)

问题：

- 上述的dockerfile文件中，的`COPY target/login.jar login.jar`显然`target/login.jar`指的是`Login`目录下的`target/login.jar`。即为什么docker会以`Login`为“根目录”？

  答：在配置Docker的Configurations时，有一个参数`context`即在指定“根目录”



## 构建Docker镜像

![Snipaste_2020-06-11_20-52-16](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Snipaste_2020-06-11_20-52-16.png)



# Docker 限制容器对宿主机的资源使用情况

[Docker 限制容器对宿主机的资源使用情况](https://www.cnblogs.com/zhuochong/p/9728383.html)



# Docker的架构图

- Client
- DOCKER_HOST
- Registry

![image-20200611213645315](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/image-20200611213645315.png)

了解Docker架构图的目的

- 助于理解`docker build`命令。由上图可知，`docker xxx`的命令都是在终端/可视化界面执行的。之前`docker build`的结尾要加一个`.`，原因就在这里。dockerfile中执行的文件移动，复制等，可能文件移动指的是两台机器间的文件移动。因此需要指定当前终端所在机器的一个文件夹作为“根目录”，这样可以方便指定本机文件（省的指定文件的时候加一堆前缀，例如指定张图片要加D:\workspace\Typora-workspace\imgs\xx\xx\xx\xx\sfasf.jpg）



# 实用指令

启动容器时，命令加上--restart=always 表示以后启动 docker 后都会执行这行指令启动容器，相当于 docker 启动后的自动启动程序的指令配置

