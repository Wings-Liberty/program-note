
# Docker 容器数据卷

> 数据卷：容器和宿主机共享文件和文件夹


## 数据卷


### 命令行实现

此时，两个目录中的文件资源共享。如果停止容器，主机修改文件，重启容器，文件可同步共享

```shell
# :ro 指定主机对共享文件夹只读不可写，可不加
$ docker run -it -v 宿主机中文件或目录的路径:容器内的文件或目录的路径[:ro] 镜像名
```

注意：

假设把宿主机的 /home/cx/data 目录 挂在到容器的 /home/tmp 目录下



|情况|效果|
|:--|:--|
|宿主机目录，文件不存在|不会报错，且一律视为指定了一个目录并在宿主机自动创建指定的目录 |
|容器指定的目录不存在|不会报错，且自动创建目录保存指定的宿主机的目录/文件|
|容器中已存在指定目录，宿主机的文件 A 是否会把容器的同名文件覆盖掉|是的|
|容器中已存在指定目录，宿主机没有文件 A，但容器里有 是否会把容器的文件 A 删除掉|是的 |
|容器自己已存在的文件是否会反过来覆盖掉宿主机的 |没试过|


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

上述命令中最后的 '.' 的作用[见这里](http://blog.haohtml.com/archives/18711)

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

> 创建 docker 镜像有两种方式。用 `docker commit` 构建镜像和用 DockerFile 文件构建镜像。这里只说第一种方式，第二种方式会在 DockerFile 部分说明

在容器中一顿操作后（例如修改了某个文件，在容器中下载了某个软件包等），想把操作后的容器的状态保留下来，把**容器当前的状态**变成镜像即可

```shell
# commit 后容器就会变成镜像并添加到宿主机的镜像列表
$ docker commit -m="提交的描述信息" -a="作者名" 容器ID/Name 要创建的目标镜像名:[标签名]
# 数据卷中的文件不会被放到新镜像中，但是数据卷的文件夹会在新镜像中
```


怎么看创建镜像时的提交信息？（这里的提交信息指的是上述 `docker commit` 命令中的 `-m` 携带的描述信息）

```shell
$ docker inspect ImageName/ImageID | grep Comment
# grep -i 忽略大小写
$ docker inspect ImageName/ImageID | grep -i comm
```


# DockFile

> DockerFile 是用来构建 Docker 镜像的构建文件，是由一系列命令和参数构成的脚本

知识补充：'scratch'   源镜像   就像是 java 中的 Object 类，所有镜像都会默认继承它


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
| CMD |指定一个容器启动时要运行的命令。dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换。|
| ENTRYPOINT |指定一个容器启动时要运行的命令。ENTRYPOIT 的目的和 CMD 一样，都是在指定容器启动程序及参数。|
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


# 用 Docker Compose 管理镜像和容器

Docker Compose 是以 yml 文件方式配置管理多个容器的配置和启动

使用方式如下

1. 编写 docker-compose。这是一个 yml 格式的文件
2. 运行 `docker compose up`


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
