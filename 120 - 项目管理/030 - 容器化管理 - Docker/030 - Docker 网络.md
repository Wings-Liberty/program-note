
# Docker 基础网络

> 参考博客 [Docker 基础网络](https://www.jianshu.com/p/2d02bb9a9da5)，[为 Docker 容器指定 IP](https://www.jianshu.com/p/b8625ccb5e9c)

## 前言

在安装 `docker` 时，`docker`  创建了三种网络模式。`host`、`none`、`bridge`

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

使用场景：选用 host 网络的容器，其网络栈和宿主机一摸一样，它的优势在于网络性能强于其他网络模式。如果对网络传输有很大需求可以选用 host 网络



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

