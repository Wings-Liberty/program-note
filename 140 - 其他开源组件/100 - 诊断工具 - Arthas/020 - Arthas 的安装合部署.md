#还没有复习 

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


# 如何应对