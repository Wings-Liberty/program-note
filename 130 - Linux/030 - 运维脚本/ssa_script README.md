
运维脚本包括脚本文件，软件文件，说明文件等。用于快速在空环境里搭建比较全的开发环境

要求脚本文件的可阅读性强（方便改和写），易用性强（命令简洁易懂，交互简单），健壮性强（能包错误输入，并展示友好的提示）

把和系统相关的指令封装到唯一的统一方法，方便修改。比如把联网下载软件包的指令封装到一个函数里。防止大量地方都有 apt，导致想把脚本在 centos 里用就得全部替换


个人运维脚本（`ssa_script`）如下

```
L README.md（说明文件）
L ssa_software.sh（软件管理命令，安装，启动，停止等命令）
L ssa_cluster.sh（集群管理命令，支持在指定主机上安装软件，分发软件，分发执行命令输出结果，启动或停止集群）
L software/ 
	L package/ （离线软件包，比如 arthas）
	L config/ （常用配置文件，比如挂在给 docker 容器的配置文件）
	L docker_volumes/ （docker 数据卷空目录。这个可能不需要）
```


# software


# cluster

```
--sendfile=<path>/-f # 分发文件到写死的地址。可以输入多个文件
--cmd='command' 所有集群执行指令（底层函数/功能，用于提供灵活的控制方式）
--start=<software_name> 
--install # 让所有集群执行 ssa_software
```


