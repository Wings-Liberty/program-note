#正在复习 






# IDEA 创建 Docker 镜像

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

