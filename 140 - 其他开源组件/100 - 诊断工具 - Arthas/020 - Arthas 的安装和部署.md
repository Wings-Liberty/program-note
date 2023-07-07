
# 在线安装

一条命令即可

```
curl -O https://arthas.aliyun.com/arthas-boot.jar
```


# 离线安装

1. 下载离线包

[下载地址](https://arthas.aliyun.com/download/latest_version?mirror=aliyun)

2. 上传到服务器的指定路径下

推荐把离线包安装到 `$HOME/opt/arthas/`，因为 IDEA-arthas 执行热部署脚本时会去这个目录里找 `arthas`

如果 docker 容器需要用还需要复制到容器里或放到数据卷里

```sh
docker cp /tmp/arthas container-name:/root/opt/arthas/
```

![[../../020 - 附件文件夹/Pasted image 20230630185742.png]]

3. 安装到本地

```sh
sh ./install-local.sh
```


# 如何让 arthas 在缺少部分依赖的环境里运行

arthas 有部分功能需要依赖 JDK 环境和 unzip 命令。如果开发环境里没有，arthas 有些功能就不能用

受影响功能和解决方案如下

1. 启动 arthas-boot 时，会用 jps 命令获取 Java 进程

解决方案：启动 arthas-boot 前提前获取 Java 进程的 PID，启动时指定 PID

```bash
# 找目标进程的 PID
ps -ef | grep java
# 启动 arthas 时指定 PID
java -jar arthas-boot.jar <pid>
```

2. 执行 idea-arthas 生成的热部署脚本时，会用 jps 命令查询进程，如果 arthas 不在指定目录下

无法解决，必须用 jps 命令