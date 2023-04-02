#还没有复习 

# 先设密码

没有密码很容易被人攻击服务器

修改配置文件中的`requirepass`字段

重启 redis

每次建立连接后，输入`auth mypassword`否则不能执行任何操作



# 常用命令

> 关键字忽略大小写，Redis的命令手册见[redis中文网](https://www.redis.net.cn/order/)

redis 中常用的数据类型 `string`, `map`, `list`, `set`, `sortedset`

常用命令

**String** 

`set key value`, `get key`, `del key` （set 命令集增改命令于一体）

**list**

`lpush key value`, `rpush key value`, `lrange key start end`, `lpop key`, `rpop key `

**hashmap** （注：下面的 field 是字段名）

`hset key field value`, `hget key field`, `hdel key field`, `hgetall key` 

**set** （无序集合，元素不重复。key 是一个 set 的名字，一个 key 中可以有好多元素）

`sadd key value`, `smembers key`， `srem key value`

**sortedset**（score 是排序的依据，默认从小到大排序）

`zadd key score value`, `zrange key start end [withscores]`, `zrem key value`

**全局命令**

`keys *`, `type key`, `del key`

`select dbId` dbId 是数据库的 id（默认0~15）

`dbsize` 当前库中 key 数量

`fulshdb` 清空当前库

`fulshall` 清空所有库

`EXPIRE key "seconds"`	为key设置过期时间（单位是秒）

`PEXPIRE key "milliseconds"` 为key设置过期时间（单位是毫秒）

`TTL key`，`PTTL key`返回剩余生存时间（前者返回的时间的单位是秒，后者是毫秒）如果key是永久的，返回-1；如果key 不存在或者已过期，返回 -2。

`PERSIST key`	移除 key 的过期时间，将其转换为永久状态。如果返回 1，代表转换成功。如果返回 0，代表 key 不存在或者之前就已经是永久状态。

`SETEX key "seconds" "value"` 等价于SET和EXPIRE合并的操作，区别之处在于SETEX是一条命令，而命令的执行是原子性的，所以不会出现并发问题





# 配置文件



以下是经过简单翻译的`Redis v=5.0.5`配置文件。[转自这篇博客](https://my.oschina.net/u/3049601/blog/3163953)

```
# Redis configuration file example. Redis配置文件示例
#
# Note that in order to read the configuration file, Redis must be
# started with the file path as first argument:
# 如果要使用自定义的Redis配置文件，则需要将配置文件的路径（绝对/相对）跟在"./redis-server"命令后的第一个参数，如：
# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
# 下面是对Redis内存申请单位的注释，比如1k代表1000字节，而1kb代表1024字节，依次类推，并且Redis并不区分大小写，1k 1K都是1000字节
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.

################################## INCLUDES 包含 ###################################

# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
# 
# 可以将公共配置抽取成模板，然后在主配置中使用"include"选项来外挂它，"include"可以在文件开始或者结束的时候使用
# 如果在文件开始，那么主配置文件中指定的key会覆盖"include"挂进来的key,比如"include"挂进来配置文件port为6380，而主配置文件port=6379，那么这种情况Redis启动之后，监听的端口依然是6379
# 如果在文件结束，上面举例的情况，Redis启动之后，监听的端口是6380
# 其实在集群中可以使用这个特性，可以减少配置，如果使用共享文件(NFS)，还可以做到一处修改处处修改的效果，但是后者是通过分发文件实现的，这里是通过共享磁盘
# include /path/to/local.conf
# include /path/to/other.conf

################################## MODULES 模块 #####################################

# Load modules at startup. If the server is not able to load modules
# it will abort. It is possible to use multiple loadmodule directives.
# 挂载官方或者其他大牛写的module，有哪些module可以在Redis官网中的module中查看https://redis.io/modules，比如
# redis-cell(漏洞限流) RedisBloom(布隆过滤器) RedisSearch(全文检索) rediSQL(SQL操作redis)等等module

# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so

################################## NETWORK 网络 #####################################

# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all the network interfaces available on the server.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
# 如果没有指定"bind"配置，则任何机器都可以连接到该Redis服务器，但也可以通过配置"bind"，让一个或者多个地址可以连接该Redis服务器
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
# 通过配置可以发现，该配置是可以支持范围的，另外如果配置是某一个IP，其实整个网段都可以访问该Redis服务器
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only into
# the IPv4 loopback interface address (this means Redis will be able to
# accept connections only from clients running into the same computer it
# is running).
# 如果没有指定"bind"，那么将Redis服务器将暴露在互联网上，这是非常危险的，因此在生产系统上应该禁止这样的设置
# 默认情况下是"bind"指定到本机IPV4的回环地址上，因此只有本机上运行的程序才可以访问该Redis服务器

# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 192.168.10.20
# 虽然我配置的本地IP地址，但是我192.168.10.12主机一样访问，整个192.168.10网段都可以访问

# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
# "protected mode"是一个安全保护层，可以避免Redis服务器被互联网上的机器访问和利用
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
# 当"protected mode"被设置为on(即设置为"protected-mode yes")，且没有显示用bind指定ip地址集合或者没有设置密码，那么Redis服务器只能被本机访问
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
# 默认"protected mode"是开启的，如果确定自己的服务器需要暴露在互联网上，且不存在安全问题，可以将"protected mode"关闭掉
protected-mode yes

# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
# Redis Server的监听端口
port 6379

# TCP listen() backlog.
#
# In high requests-per-second environments you need an high backlog in order
# to avoid slow clients connections issues. Note that the Linux kernel
# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
# in order to get the desired effect.
# 在高并发场景下，为了避免客户端连接缓慢问题，需要高的backlog，默认值是511。但是真正使用的值依赖于LINUX内核参数somaxconn，而somaxconn默认值是128，所以即便这里设置了511，最终生效的128。
# 所以如果公司没有主机工程师一定要记得在安装新机器操作系统时就将一些内核参数改大一些，比如将somaxconn修改为20480
tcp-backlog 511

# Unix socket.
#
# Specify the path for the Unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
# 利用Unix socket可以提升同一服务器上的进程间通信速度，而且是数量级的提升，但通常Redis服务器和应用服务器是分开的，所以下面的两个参数可以不管
# unixsocket 指定一个文件作为通信的媒介
# unixscoketperm 对unixsocket指定文件的访问权限(读-写-执行)，如果真的用了这个特性，该值应该根据系统用户进行权限设置
# unixsocket /tmp/redis.sock
# unixsocketperm 700

# Close the connection after a client is idle for N seconds (0 to disable)
# 当连接变得空闲了之后多少秒关闭连接，默认设置为0，表示禁用这个选项带来的效果--连接不关闭
timeout 0

# TCP keepalive.
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers.
# 2) Take the connection alive from the point of view of network
#    equipment in the middle.
# 如果tcp-keepalive值不为0，那么在客户端和服务器缺乏通信的情况下使用SO_KEEPALIVE，每隔"tcp-keepalive"指定的时间发送"TCP ACKS"给客户端
# 这么做有两个原因：
# 1.可以检测客户端是否还存活
# 2.保持网络连接是活着的，这样避免客户端反复与服务器建立链接导致性能低下
# 
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 300 seconds, which is the new
# Redis default starting with Redis 3.2.1.
# 默认值是300S，我个人觉得300S还是太长了，即使大的集群60S发送一次也不会造成大的网络流量
tcp-keepalive 300

################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
# 将"daemonize"设置为yes，Redis会以守护进程的方式运行，并且会在/var/run目录下生成一个redis.pid文件
daemonize no

# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# 使用Linux系统的upstart或者systemd两种方式来管理redis的启动，需要结合linux的版本来决定，centos7设置为systemd，而ubuntu设置为upstart
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised no

# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
# 如果设置了pid文件，那么Redis启动时会写该pid文件到指定的目录下，退出时删除该pid文件
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
# 如果redis是以守护进程方式运行的，如果没有指定"pidfile"的值，默认生成一个/var/run/redis.pid文件，否则使用指定的"pidfile"
# 如果redis是以非守护进程方式运行，如果没有指定"pidfile"的值，则不会产生pid file
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid

# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
# 指定Redis服务的详细日志级别，有debug\verbose\notice\warning四种级别，debug当然是不推荐的，日志太多了，除非有特殊情况，开发环境可以试试
loglevel notice

# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
# 如果以非守护进程的方式运行，且没有指定"logfile"，那么日志会发送到/dev/null(空文件)，我们无法通过它看到任何日志,但如果指定了"logfile"，则输出到配置的文件当中
# 如果以守护进程的方式运行，且没有指定"logfile"，那么日志会输出到标准输出(控制台)，但如果指定了"logfile"，则输出到配置的文件当中
logfile ""

# 以下3个(syslog-enabled/syslog-ident/syslog-facility)参数感觉不需要关注，它们的目的就是将日志输出使用系统自带的logger，而且可以修改syslog的参数来实现自己特殊的需求
# 我自己没有去测试过，感觉应该很少会用到，也许大企业专门负责Redis集群的会使用它来定制Redis的日志输出格式，然后使用程序来统计最后通过UI来展示
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# 如果要使用system logger，则将"syslog-enabled"设置为yes
# syslog-enabled no

# Specify the syslog identity.
# 指定syslog的id，应该是随便指定吧，起到唯一标识的作用？？？
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# 指定localxxx，需要配合/etc/rsyslog.conf文件使用，意思就是将日志文件输出导出到rsyslog.conf指定的文件中。如果开启了syslog-enable,也许自己指定的logfile就失效了，需要通过rsyslog.conf指定localxxx将日志导出指定的文件中
# syslog-facility local0

# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
# 设置redis有多少个db，默认是16个，可以超过16，但是最大值是多少我也不知道。
# 客户端连接上服务器之后，可以通过"select databases-1"来选择使用的db，比如要使用第15个db，则使用"select 14"
databases 16

# By default Redis shows an ASCII art logo only when started to log to the
# standard output and if the standard output is a TTY. Basically this means
# that normally a logo is displayed only in interactive sessions.
#
# However it is possible to force the pre-4.0 behavior and always show a
# ASCII art logo in startup logs by setting the following option to yes.
# 是否启动的时候输出(显示)Redis的ASCII LOGO，这个没事就不要去动他了吧，看看也不错至少晓得它正在启动了
always-show-logo yes

################################ SNAPSHOTTING 持久化 ################################
# 下持久化有三种，RDB\AOF\RDB+AOF混合，简答提一下对应的实现原理
# RDB：将数据库以二进制存放在磁盘文件中，持久化的时间间隙比较大，丢失的数据比较多，单独只使用这种方式不推荐
# AOF：将操作数据库的指令(包括协议信息)以文本方式存放在磁盘文件中，根据配置最多会丢失1S的数据，这种方式还可以
# 混合：推荐这种方式，但是4.0开始才有此功能，混合持久化结合了RDB快速恢复数据和AOF丢失数据少的优点，而且减少了磁盘开销。。。关于它们更详细的介绍请查看
# Redis设计与实现-RDB持久化 https://my.oschina.net/u/3049601/blog/3153571
# Redis设计与实现-AOF持久化 https://my.oschina.net/u/3049601/blog/3153678
# Redis设计与实现-混合持久化 https://my.oschina.net/u/3049601/blog/3158904
#
# Save the DB on disk: 
# 保存DB到磁盘中
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   15分钟内至少有一个KEY改变了
#   after 300 sec (5 min) if at least 10 keys changed
#   5分钟内至少有10个KEY改变了
#   after 60 sec if at least 10000 keys changed
#   1分钟内至少有10000个KEY改变了
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""
#   如果想禁止RDB持久化，可以将下面的三个save配置项使用"#"注释掉。还可以使用save ""来代替使用"#"来注释掉三个save配置项

save 900 1
save 300 10
save 60 10000

# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
# 如果RDB开启了且最近的BGSAVE失败了，那么Redis默认是不会再接收新增或者修改请求了，但如果BGSAVE又恢复工作，那么新增和修改操作可以继续（表示可以自动恢复）
# 如果公司有自己的监控系统可以很好的检测Redis服务和持久化情况，那么可以将此功能关闭，这样可以提高系统的可用性
# 如果使用集群，且从节点够的情况下有自己的监控，真的可以将这个功能关闭掉
stop-writes-on-bgsave-error yes

# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
# 默认会采用LZF压缩dump出来的数据库并写入到xxx.rdb文件中。压缩会增加CPU的开销，如果想节约CPU的开销，可以将"rdbcompression"设置为"no"，但是会占用更多的磁盘
rdbcompression yes

# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
# 从Redis5版本开始，默认在RDB文件末尾有一个CRC64（一个随机算法，生成信息指纹用的）校验和，它可以让文件格式可以更强的抵抗风险，但是它会带来10左右的性能损失，我们可以禁止它以获得最大的性能输出
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
# 如果禁用掉"checksum"，那么生成的RDB文件结尾的校验和为"0"，那么加载程序则会跳过校验
# 针对大企业（不差钱）个人觉得就保留默认设置应该比较好
rdbchecksum yes

# The filename where to dump the DB
# 指定RDB文件的名字，建议使用ip+port来指定，运维可以更好的分辨，甚至可以通过程序扫描展示到UI上
dbfilename dump.rdb

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
# 指定RDB和AOF文件的存储目录，注意：这里只能指定到目录，不要带文件名称
dir ./

################################# REPLICATION 主从 #################################

# Master-Replica replication. Use replicaof to make a Redis instance a copy of
# another Redis server. A few things to understand ASAP（as soon as possible） about Redis replication.
# Redis的主从复制，使用"replicaof"从一个Redis Server复制到另外一个去，下面有几个要点需要理解
#   +------------------+      +---------------+
#   |      Master      | ---> |    Replica    |
#   | (receive writes) |      |  (exact copy) |
#   +------------------+      +---------------+
#
# 1) Redis replication is asynchronous, but you can configure a master to
#    stop accepting writes if it appears to be not connected with at least
#    a given number of replicas.
#    主从复制虽然是异步进行的，但是可以通过配置(min-replicas-to-write)让从节点小于特定值时，主节点不接受"write"请求
#
# 2) Redis replicas are able to perform a partial resynchronization with the
#    master if the replication link is lost for a relatively small amount of
#    time. You may want to configure the replication backlog size (see the next
#    sections of this file) with a sensible value depending on your needs.
#    Redis的老版本没有复制重同步，但是从2.8开始支持部分同步（使用复制挤压缓冲区实现），解决了断线后老版本"完全同步"低效、阻塞、循环同步的问题
#    如果从节点与主节点断开联系一小段时间，则会发起部分同步，但是复制挤压缓冲区也有大小，可以设置缓冲区的大小来减少完全同步出现
#
# 3) Replication is automatic and does not need user intervention. After a
#    network partition replicas automatically try to reconnect to masters
#    and resynchronize with them.
#    复制时自动进行的，且不需要人工介入。在出现网络分区后，从节点会自动尝试去重连主节点，连接成功之后发起部分同步，如果复制积压缓冲区中的数据丢失了，则会发起完全同步
# 
# 这段英文解释如果是入门学习Redis，可能不会太看得懂，推荐先看看Redis设计与实现这本书，会对这段描述有比较深刻的认识
# replicaof <masterip> <masterport>
# replicaof 主节点IP  主节点端口，注意一定要保证主从节点网络是通的，检查本机防火墙和第三方防火墙

# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the replica to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the replica request.
# 如果主节点设置了密码，那么在从节点的redis.conf中要设置masterauth配置项，将密码写在这里，如果没有
# 设置，那么主节点会拒绝从节点的复制请求
#
# masterauth <master-password>
# masterauth 主节点密码

# When a replica loses its connection with the master, or when the replication
# is still in progress, the replica can act in two different ways:
# 如果从节点与主节点失去连接或者正在从主节点同步数据，那么从节点根据配置可以工作在两种模式下：
#
# 1) if replica-serve-stale-data is set to 'yes' (the default) the replica will
#    still reply to client requests, possibly with out of date data, or the
#    data set may just be empty if this is the first synchronization.
#	 如果"replica-serve-stale-data"设置为"yes"，这也是默认设置，那么从节点将会回复客户端的请求，但是得到的数据可能出现下面两种情况
#     1.如果是与主节点失去连接，那么得到的数据可能是过时的
#     2.如果是第一次从主节点同步数据，那么得到的数据集会是空的
#
# 2) if replica-serve-stale-data is set to 'no' the replica will reply with
#    an error "SYNC with master in progress" to all the kind of commands
#    but to INFO, replicaOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG,
#    SUBSCRIBE, UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB,
#    COMMAND, POST, HOST: and LATENCY.
#    如果"replica-serve-stale-data"设置为"no"，从节点将回复客户端"SYNC with master in progress"错误
#    但是INFO, replicaOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG等命令是可以成功执行并得到相应结果
#
replica-serve-stale-data yes

# You can configure a replica instance to accept writes or not. Writing against
# a replica instance may be useful to store some ephemeral data (because data
# written on a replica will be easily deleted after resync with the master) but
# may also cause problems if clients are writing to it because of a
# misconfiguration.
# 我们可以配置从节点是否可以接受write请求，非常不建议将从节点设置为可接受write请求，因为同步可能会导致数据丢失
# 因此从Redis2.6就将"replica-read-only"默认设置为"yes"了
#
# Since Redis 2.6 by default replicas are read-only.
#
# Note: read only replicas are not designed to be exposed to untrusted clients
# on the internet. It's just a protection layer against misuse of the instance.
# Still a read only replica exports by default all the administrative commands
# such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
# security of read only replicas using 'rename-command' to shadow all the
# administrative / dangerous commands.
# 从节点(只读)在架构时建议不要设计为暴露给互联网上的不可信任客户端，它可以起到滥用实例的保护作用
# 从节点依然支持所有的管理命令，比如CONFI,DEBUG等等，为了提高从节点的安全性，可以使用"rename-command"来屏蔽所有的"管理命令"
replica-read-only yes

# Replication SYNC strategy: disk or socket.
# 同步策略：通过磁盘(Disk-backed)或者通过SOCKET(Diskless)
#
# -------------------------------------------------------
# WARNING: DISKLESS REPLICATION IS EXPERIMENTAL CURRENTLY
# -------------------------------------------------------
# 很遗憾通过SOCKET同步还处在试验阶段
#
# New replicas and reconnecting replicas that are not able to continue the replication
# process just receiving differences, need to do what is called a "full
# synchronization". An RDB file is transmitted from the master to the replicas.
# The transmission can happen in two different ways:
# 新的从节点和重连接的从节点(从节点的最新偏移量不在主节点的复制积压缓冲区中)，则会执行"full synchronization"操作，一个RDB文件会采用以下两种方式中的一种传输给从节点：
#
# 1) Disk-backed: The Redis master creates a new process that writes the RDB
#                 file on disk. Later the file is transferred by the parent
#                 process to the replicas incrementally.
#                 主节点fork一个子进程出来将RDB文件生成到磁盘上，然后父进程将RDB文件逐渐发给从节点
# 2) Diskless: The Redis master creates a new process that directly writes the
#              RDB file to replica sockets, without touching the disk at all.
#              主节点fork一个子进程，然后创建一个和从节点的SOCKET连接，直接将数据发送给从节点，而不借助磁盘
#
# With disk-backed replication, while the RDB file is generated, more replicas
# can be queued and served with the RDB file as soon as the current child producing
# the RDB file finishes its work. With diskless replication instead once
# the transfer starts, new replicas arriving will be queued and a new transfer
# will start when the current one terminates.
# 如果使用磁盘(disk-backed)，当子进程生成RDB文件后，多个从节点立即就可以使用RDB文件进行复制操作
# 如果使用SOCKET(diskless)，一旦复制开始，当新的节点复制请求必须等已开始复制完成之后才能进行。
#
# When diskless replication is used, the master waits a configurable amount of
# time (in seconds) before starting the transfer in the hope that multiple replicas
# will arrive and the transfer can be parallelized.
# 当使用SOCKET(diskless)时，主节点可以等一段时间（可配置），这段时间内过来的复制请求可以并行开始
#
# With slow disks and fast (large bandwidth) networks, diskless replication
# works better.
# 如果磁盘效率低，而网络速度快且带宽也大的情况下，Diskless方式完复制效果更好
repl-diskless-sync no

# When diskless replication is enabled, it is possible to configure the delay
# the server waits in order to spawn the child that transfers the RDB via socket
# to the replicas.
# 如果"repl-diskless-sync"设置为yes，就需要配置"repl-diskless-sync-delay"让主节点等待更多的复制请求过来，并让他们并发复制
#
# This is important since once the transfer starts, it is not possible to serve
# new replicas arriving, that will be queued for the next RDB transfer, so the server
# waits a delay in order to let more replicas arrive.
# 因为一旦有复制开始进行，新来的复制请求就会排队，因此设置了延迟时间就可以让更多的复制同时执行
#
# The delay is specified in seconds, and by default is 5 seconds. To disable
# it entirely just set it to 0 seconds and the transfer will start ASAP.
# 延迟时间单位是"秒"，默认值是5秒。可以将"repl-diskless-sync-delay"设置为0，这样复制就会立即执行
repl-diskless-sync-delay 5

# Replicas send PINGs to server in a predefined interval. It's possible to change
# this interval with the repl_ping_replica_period option. The default value is 10
# seconds.
# 从节点使用"repl-ping-replica-period"指定的时长(单位：秒)定期发送"pings"给主节点，通过这个操作可以检测从节点是否和主节点失联。该值默认是10秒
#
# repl-ping-replica-period 10

# The following option sets the replication timeout for:
# "repl-timeout"会影响复制过程中一下三种情况的超时时间
#
# 1) Bulk transfer I/O during SYNC, from the point of view of replica.
# 2) Master timeout from the point of view of replicas (data, pings).
# 3) Replica timeout from the point of view of masters (REPLCONF ACK pings).
#
# It is important to make sure that this value is greater than the value
# specified for repl-ping-replica-period otherwise a timeout will be detected
# every time there is low traffic between the master and the replica.
# 一定要确保"repl-timeout"的值大于"repl-ping-replica-period"的值，否则当主从节点之间通信量很低时，每次判断超时都是成功的，默认值是60秒
#
# repl-timeout 60

# Disable TCP_NODELAY on the replica socket after SYNC?
#
# If you select "yes" Redis will use a smaller number of TCP packets and
# less bandwidth to send data to replicas. But this can add a delay for
# the data to appear on the replica side, up to 40 milliseconds with
# Linux kernels using a default configuration.
# 如果将"repl-disable-tcp-nodelay"设置为"yes"，那么主节点会使用更小的TCP packet和更少的带宽发送数据到从节点
# 但是这会让从节点的数据延迟40毫秒(LINUX默认配置，也许可以通过tcp_delack_min修改)，关于tcp-nodelay可以看博客：https://blog.csdn.net/bytxl/article/details/17677495
#
# If you select "no" the delay for data to appear on the replica side will
# be reduced but more bandwidth will be used for replication.
# 如果将"repl-disable-tcp-nodelay"设置为no，那么从节点接收数据的延迟会减少，但是要求更多的带宽来完成复制工作
#
# By default we optimize for low latency, but in very high traffic conditions
# or when the master and replicas are many hops away, turning this to "yes" may
# be a good idea.
# 默认我们使用低延迟的选项，也就是"repl-disable-tcp-nodelay"设置为no，但是在非常高的通信量情况下或者从主节点到从节点会经过很多次转发，将"repl-disable-tcp-nodelay"设置为yes是可能是更好的选择
repl-disable-tcp-nodelay no

# Set the replication backlog size. The backlog is a buffer that accumulates
# replica data when replicas are disconnected for some time, so that when a replica
# wants to reconnect again, often a full resync is not needed, but a partial
# resync is enough, just passing the portion of data the replica missed while
# disconnected.
# backlog设置复制积压缓冲区的大小，用以存放从节点与主节点断开连接后这段时间的write等命令，当从节点重新连接上来时，通常不需要做"完全同步"，只需要做部分同步（要求从节点偏移量在复制挤压缓冲区可以找到，表示数据可以使用偏移量后的命令进行恢复），将偏移量后面的命令发送给从节点执行
#
# The bigger the replication backlog, the longer the time the replica can be
# disconnected and later be able to perform a partial resynchronization.
# 更大的backlog意味着从节点可以断开连接更长时间，然后才可以执行部分同步
#
# The backlog is only allocated once there is at least a replica connected.
# 一旦有从节点连接到主节点，复制积压缓冲区就会创建，此时并没有数据。
#
# 这个值到底设置多少，有一个计算公式：2*平均断线时间*每秒写入数据大小，因此和业务量强相关
# repl-backlog-size 1mb

# After a master has no longer connected replicas for some time, the backlog
# will be freed. The following option configures the amount of seconds that
# need to elapse, starting from the time the last replica disconnected, for
# the backlog buffer to be freed.
# 当主节点在最后一个从节点断线之后的一段时间后(repl-backlog-ttl设置)，会将复制积压缓冲区释放
#
# Note that replicas never free the backlog for timeout, since they may be
# promoted to masters later, and should be able to correctly "partially
# resynchronize" with the replicas: hence they should always accumulate backlog.
# 从节点永远都不会释放，因为它必须保存自己接收的最新偏移量，当出现断线重连时将这个偏移量发给主节点，主节点决定使用完全同步还是部分同步
#
# A value of 0 means to never release the backlog.
# 如果设置为0，表示永不释放复制积压缓冲区
#
# repl-backlog-ttl 3600

# The replica priority is an integer number published by Redis in the INFO output.
# It is used by Redis Sentinel in order to select a replica to promote into a
# master if the master is no longer working correctly.
# "replica-priority"是一个整数值，在哨兵的集群模式下，当"主事哨兵"被选举(选举采用过半原则)出来之后，由它决定挂掉主节点下的某一个从节点作为新的主节点，当其他条件都相同的情况下，"replica-priority"值越小的从节点会被选中作为新的主节点
#
# A replica with a low priority number is considered better for promotion, so
# for instance if there are three replicas with priority 10, 100, 25 Sentinel will
# pick the one with priority 10, that is the lowest.
# 总结起来就是值越小优先级越高，其对应的从节点会优先被选择作为新的主节点
#
# However a special priority of 0 marks the replica as not able to perform the
# role of master, so a replica with priority of 0 will never be selected by
# Redis Sentinel for promotion.
# 如果将"replica-priority"设置为0，则该从节点永远都不会被选择为新的主节点，根本就不参与选举。可以减少选举过程中过多的网络通信，加快选举过程
# 我还只是一个理论派，没实战经验，个人感觉如果机器硬件够好，且机器所在的网络质量够好，可以将其优先级设置得高一些
#
# By default the priority is 100.
# 默认值是100
replica-priority 100

# It is possible for a master to stop accepting writes if there are less than
# N replicas connected, having a lag less or equal than M seconds.
# 如果连接的从节点小于N且它们的滞后时间小于或者等于M秒，那么主节点可能会停止接收写操作
# 网上说这两个条件中一个不满足就可能导致主节点不能接收写操作，是很准确，必须是两个条件同时满足才会触发
#
# The N replicas need to be in "online" state.
# 要求这N个从节点是"online"状态，还有在哨兵和Cluster模式下节点还有主观下线和客观下线状态
#
# The lag in seconds, that must be <= the specified value, is calculated from
# the last ping received from the replica, that is usually sent every second.
# 滞后时间的秒数必须小于指定值，滞后时间=当前时间-最后一次接收到的从replica发过来的ping时间
#
# This option does not GUARANTEE that N replicas will accept the write, but
# will limit the window of exposure for lost writes in case not enough replicas
# are available, to the specified number of seconds.
# 这个选项并不能保证N个从节点接收写操作，但是可以将丢失的数据限制在指定秒数内
#
# For example to require at least 3 replicas with a lag <= 10 seconds use:
#
# min-replicas-to-write 3
# min-replicas-max-lag 10
#
# Setting one or the other to 0 disables the feature.
# 如果将这两个选项中的任何一个设置为0，表示禁用这个特征
#
# By default min-replicas-to-write is set to 0 (feature disabled) and
# min-replicas-max-lag is set to 10.
# 默认"min-replicas-to-write"被设置为0，即禁止了这个特征，"min-replicas-max-lag"默认值为10秒

# A Redis master is able to list the address and port of the attached
# replicas in different ways. For example the "INFO replication" section
# offers this information, which is used, among other tools, by
# Redis Sentinel in order to discover replica instances.
# Another place where this info is available is in the output of the
# "ROLE" command of a master.
#
# The listed IP and address normally reported by a replica is obtained
# in the following way:
#
#   IP: The address is auto detected by checking the peer address
#   of the socket used by the replica to connect with the master.
#
#   Port: The port is communicated by the replica during the replication
#   handshake, and is normally the port that the replica is using to
#   listen for connections.
#
# However when port forwarding or Network Address Translation (NAT) is
# used, the replica may be actually reachable via different IP and port
# pairs. The following two options can be used by a replica in order to
# report to its master a specific set of IP and port, so that both INFO
# and ROLE will report those values.
#
# There is no need to use both the options if you need to override just
# the port or the IP address.
# 总结起来说：在主节点中使用info replication可以列出所有从节点的IP+PORT，本来主节点可以使用SOCKET拿到从节点的IP+PORT，但如果使用端口转发(docker,k8s)和NAT或者因为使用了代理，从节点不能直接通过IP+PORT到达，下面这两个从节点选项才有用，它可以将设置的IP和PORT报告给主节点，INFO和ROLE命令会显示设置的值
#
# replica-announce-ip 5.5.5.5
# replica-announce-port 1234

################################## SECURITY ###################################

# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
# 为了让不信任的客户端访问Redis Server，可以要求客户端在执行任何命令之前先校验密码
#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
# 如果客户端和Redis server运行在同一个机器，我们也可以将"requirepass"注释掉，客户端就不需要校验密码。
# 可以推广到：如果在一个局域网里面，如果安全做得足够好，则都可以不设置"requirepass"
#
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
# 因为Redis可以每秒可以验证150K个密码，因此如果要设置密码，一定要设置一个非常强壮的密码，否则很容易被破解
#
# requirepass foobared

# Command renaming.
# 重命名command，可以保护我们的管理员命令和一些会导致Redis卡顿的命令
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
# 在共享环境中，可以将一些危险的命令进行重命名，这样可以让普通的客户端不可以使用那些被重命名的命令，只有内部工具可以使用
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
# 可以将危险的命令重命名为一个空串，彻底的禁止该命令
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to replicas may cause problems.
# 因为主节点的会将write命令使用缓冲区记录下来并传播给从节点执行，因此如果重命名的命令在从节点没有同步修改的话，这可能带来一些意想不到的问题，因此一定要小心这一点。
# 比如将set命令重命名为myset，那么在主节点执行myset foo Messi之后，从节点并不会有foo这个key,因为从节点并不认识myset这个命令

################################### CLIENTS ####################################

# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus 32 (as Redis reserves a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
# 设置同时可以连接到服务器的客户端数量，默认值是10000，但如果主机的最大文件打开数并没有比"maxclients"大，那么"maxclients"=最大文件打开数-32，这个32是提供给Redis内部使用的，比如集群之间的通信等也需要连接数
#
# maxclients 10000

############################## MEMORY MANAGEMENT ################################

# Set a memory usage limit to the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
# 设置Redis的工作最大内存为某一个特定的限制值。当内存使用达到限制值，根据设置的淘汰策略删除keys
#
# If Redis can't remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
# 当Redis无法根据设置的淘汰策略删除keys时或者淘汰策略被设置为"noeviction"，像set lpush等命令会收到报错,此时管理员就应该特别注意了，及时的增加内存，但是此时读命令还是可以继续正常使用的
#
# This option is usually useful when using Redis as an LRU or LFU cache, or to
# set a hard memory limit for an instance (using the 'noeviction' policy).
# 如果将Redis作为一个LRU或者LFU的缓存，再或者将Redis作为hard memory limit for an instance使用时，这个选项就非常有用
#
# WARNING: If you have replicas attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the replicas are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of replicas is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied.
# 如果一个设置了"maxmemory"的主节点连接了一个从节点，那么用于主从复制传递命令的输出缓冲区占用的内存也在maxmemory当中，如果当内存被占满时而出现大量的删除key的操作写到缓冲区，而缓冲区又不够，又会触发删除更多的key,这样就会造成一个死循环，直到整个数据库变成空的。
# 因此
#
# In short... if you have replicas attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for replica
# output buffers (but this is not needed if the policy is 'noeviction').
# 因为used memory是可以大于maxmemory的，只不过出现这种情时会导致内存回收而触发删除KEY的操作。因此，如果在主从模式下，主节点的maxmemory在设置得足够大的情况下，还要给输出缓冲区留出一点空间来，避免出现死循环而导致数据库被清空。不要物理内存有多少就设置多少，况且还有操作系统和其他程序在运行,一般设置为3/4。
#
# maxmemory <bytes>

# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
# volatile-lru -> Evict using approximated LRU among the keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key among the ones with an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
# 关于这几个策略的讲解百度可以找到非常好的描述，在这里就不详细描述了，篇幅也不够
# 推荐一个：https://cloud.tencent.com/developer/article/1530553 讲了原理和使用说明
#
# LRU means Least Recently Used   最近没有被使用的
# LFU means Least Frequently Used 最近使用频率最小的
#
# Both LRU, LFU and volatile-ttl are implemented using approximated randomized algorithms
# LRU LFU 和volatile-ttl使用了较接近的随机算法
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
#       选择上面的任何一种策略，如果没有适合的KEY被淘汰，那么下面的这些写操作就会报错
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
# 默认设置是noeviction
#
# maxmemory-policy noeviction

# LRU, LFU and minimal TTL algorithms are not precise algorithms but approximated
# algorithms (in order to save memory), so you can tune it for speed or
# accuracy. For default Redis will check five keys and pick the one that was
# used less recently, you can change the sample size using the following
# configuration directive.
# LRU LFU TTL三种方式并不是精准的算法，这是为了提高速度和节省内存，同时达到了近似的效果。。。很妙
# 我们可以基于速度或者精准度的要求去调整采样的数据大小，"maxmemory-samples"值越大精准度越高，速度越慢，消耗的内存也越多，反之速度快，但是精准度低，内存开销少
#
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs more CPU. 3 is faster but not very accurate.
# 默认值是5，如果设置为10就非常接近真正的LRU算法了，但是CPU开销也越多了。如果设置为3，速度快了，但是没那么准确
#
# maxmemory-samples 5

# Starting from Redis 5, by default a replica will ignore its maxmemory setting
# (unless it is promoted to master after a failover or manually). It means
# that the eviction of keys will be just handled by the master, sending the
# DEL commands to the replica as keys evict in the master side.
# 从Redis 5开始，从节点默认是忽略掉maxmemory设置的，除非从节点在故障转移时变成了主节点
# 正常情况下，从节点的Key淘汰是通过从主节点发送del命令过来实现的
#
# This behavior ensures that masters and replicas stay consistent, and is usually
# what you want, however if your replica is writable, or you want the replica to have
# a different memory setting, and you are sure all the writes performed to the
# replica are idempotent, then you may change this default (but be sure to understand
# what you are doing).
# "replica-ignore-maxmemory"可以保证主从的数据一致性，除非你真的知道自己把"replica-ignore-maxmemory"
# 设置为no带来的副作用，那建议你不要做着骚的操作，坑人哦
#
# Note that since the replica by default does not evict, it may end using more
# memory than the one set via maxmemory (there are certain buffers that may
# be larger on the replica, or data structures may sometimes take more memory and so
# forth). So make sure you monitor your replicas and make sure they have enough
# memory to never hit a real out-of-memory condition before the master hits
# the configured maxmemory setting.
# 由于从节点默认情况下是不主动删除KEY的，它可能比主节点消耗更多的内存(可能buffer更大，可能数据结构消耗的内存更多等等)，所以要使用你的monitor实时监控你的从节点，并保证主节点达到maxmemory时间先于从节点的内存超过真正的物理内存
#
# replica-ignore-maxmemory yes

############################# LAZY FREEING 惰性回收 ####################################

# Redis has two primitives to delete keys. One is called DEL and is a blocking
# deletion of the object. It means that the server stops processing new commands
# in order to reclaim all the memory associated with an object in a synchronous
# way. If the key deleted is associated with a small object, the time needed
# in order to execute the DEL command is very small and comparable to most other
# O(1) or O(log_N) commands in Redis. However if the key is associated with an
# aggregated value containing millions of elements, the server can block for
# a long time (even seconds) in order to complete the operation.
# Redis提供了两个命令来手动删除keys，其中一个是大家熟知的del，另外一个是unlink
# "del"命令：删除是阻塞式(执行删除时，后续的命令就要排队等待)删除以便释放空间，如果一个Key比较小则删除很快，影响小，但如果这个Key对应的对象非常大，那么删除会很耗时，在高并发的系统里面会阻塞后面的请求，如果系统架构设计不合理则可能导致整个业务系统不可供，造成严重的生产事故
# "unlink"命令：是异步的尽可能快的逐步删除，它所需的时间复杂度是O(1)，Redis会启动另外一个线程来执行真正的删除并回收内存的操作，它不会阻塞后续命令。比如flushall flushdb命令也是异步执行的。
#
# For the above reasons Redis also offers non blocking deletion primitives
# such as UNLINK (non blocking DEL) and the ASYNC option of FLUSHALL and
# FLUSHDB commands, in order to reclaim memory in background. Those commands
# are executed in constant time. Another thread will incrementally free the
# object in the background as fast as possible.
#
# DEL, UNLINK and ASYNC option of FLUSHALL and FLUSHDB are user-controlled.
# It's up to the design of the application to understand when it is a good
# idea to use one or the other. However the Redis server sometimes has to
# delete keys or flush the whole database as a side effect of other operations.
# Specifically Redis deletes objects independently of a user call in the
# following scenarios:
# 除了用户可以使用del,unlink,flushall,flushdb删除key，Redis Server在某些情况下不得不删除Key，甚至清空整个db以保证服务的可用性，下面列举了4种情况
#
# 1) On eviction, because of the maxmemory and maxmemory policy configurations,
#    in order to make room for new data, without going over the specified
#    memory limit.
#    为了避免Redis使用的内存超过"maxmemory"，且一直在这种状态下运行，Redis Server会根据选择的删除策略去自动删除一些Key，以释放空间给其他数据使用。
#
# 2) Because of expire: when a key with an associated time to live (see the
#    EXPIRE command) must be deleted from memory.
#    Key设置的过期时间到了，当用户访问这个Key会自动删除，或者Redis Server定期将这种Key删除。
#
# 3) Because of a side effect of a command that stores data on a key that may
#    already exist. For example the RENAME command may delete the old key
#    content when it is replaced with another one. Similarly SUNIONSTORE
#    or SORT with STORE option may delete existing keys. The SET command
#    itself removes any old content of the specified key in order to replace
#    it with the specified string.
#    一些命令的底层实现就是先删除再新增，所以再使用这些命令的时候会执行删除操作，比如SET,SORT,RENAME
#
# 4) During replication, when a replica performs a full resynchronization with
#    its master, the content of the whole database is removed in order to
#    load the RDB file just transferred.
#    主从模式下，如果断网重连后触发了"完全同步"，也会将整个DB数据删除掉，然后再从RDB文件/SOCKET中加载所有数据
#
# In all the above cases the default is to delete objects in a blocking way,
# like if DEL was called. However you can configure each case specifically
# in order to instead release memory in a non-blocking way like if UNLINK
# was called, using the following configuration directives:
# 上面的4种情况，Redis Server删除数据都是阻塞式删除，就像"del"命令。我们可以将这4种情况的设置为异步删除，就像命令"unlink"一样

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
# Redis的"bgsave"可以异步的将数据集导出到RDB文件中，这种持久化方式满足了大多数的应用，但是有一种情况是当因为一些情况挂掉，比如断电，根据"save xxx"的配置可能会导致几分钟的数据丢失，在一些要求高的系统中这种情况是不被允许的。
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
# Redis提供了"Append Only File"新的持久化技术，该技术理论上可以做到当发生断电时让丢失的数据小于等于1秒，或者服务器本身没有挂，只是Redis Server程序挂了，甚至只有一个single write丢失
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
# AOF和RDB两种持久化技术可以同时开启，如果AOF开启了，那么启动Redis时，是从AOF文件中加载数据的，因为它保存的数据更完整，提供更好的持久化功能
#
# Please check http://redis.io/topics/persistence for more information.
# 更多的信息请出门左转到：http://redis.io/topics/persistence for more information
# 开启AOF，"appendonly"设置为yes
appendonly no

# The name of the append only file (default: "appendonly.aof")
# 指定AOF文件名，此文件存放的目录和RDB是共用的，使用"dir"进行指定
appendfilename "appendonly.aof"

# 对于没有OS知识的朋友，接下来的appendfsync功能可以先要去百度找操作系统写文件缓冲区的知识点，fsync不同的选项决定了写入缓冲区的数据什么时候真正写到磁盘上
# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
# 系统调用"fsync()"告诉OS要真正的将数据写入到磁盘上，而不是写入到缓冲区当中。一些OS会立即写到磁盘，一些OS可能会尽可能快的藏尸将数据写到磁盘
#
# Redis supports three different modes:
# Redis支持三种不同的模式：
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# 模式1-"no"：不调用OS的fsync函数，让OS自己决定什么时候将缓冲区的数据写入到磁盘上，该模式对Redis来说速度最快
#
# always: fsync after every write to the append only log. Slow, Safest.
# 模式2-"always"：每次"写操作"都会调用一次fsync函数，这种方式最安全，但是速度是最慢的
#
# everysec: fsync only one time every second. Compromise.
# 模式3-"everysec"：每一秒钟调用一次fsync，这是一种这种折中方案。
# 
# 看到这里顺便提一下，在Redis中随处可见这种思想，比如前面近似LRU的随机算法，有序集合底层数据结构中结合Hash表和跳跃表实现高效的单个和范围查询，过期key的惰性删除等等
# 在我们自己设计系统、开发模块、甚至生活中也可以将这个思想好好运用
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
# 默认模式是"everysec"的，这是结合速度和安全性的这种方案。如果你不考虑系统DOWN可能带来的数据丢失，可以将模式设置为"no"，而如果你想数据完全不丢，且愿意牺牲性能，可以将模式设置为"always"
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
# 更多的细节请出门左转：http://antirez.com/post/redis-persistence-demystified.html
# 另外大牛"antirez"还开发了基于Redis的神经网络训练模块(neural-redis)和分布式作业队列(Disque)
# If unsure, use "everysec".
# 如果自己不确定到底使用哪一种，就使用默认值everysec

# appendfsync always
appendfsync everysec
# appendfsync no

# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
# 当AOF模式设置为"everysec"或者"always"，执行后台保存AOF文件操作或者AOF文件重写(可以单独百度一下，有的面试官会问这个问题)会产生大量的IO，而一些LINUX OS的fsync调用会被阻塞很长时间(目前还未解决这个问题)，这种情况会阻塞另外线程的同步写操作
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
# 为了减轻这个问题带来的影响，可以使用"no-appendfsync-on-rewrite"配置，一旦有BGSAVE和BGREWRITEAOF在执行，阻止fsync函数调用
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
# 简单点说就是：当"no-appendfsync-on-rewrit"设置为no,那么有一个进程在执行SAVE操作，AOF持久化模式相当于被设置成了"no"，也就是说根据OS的设置，糟糕的情况下可能丢失30秒以上的数据
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.
# 如果你知道上面说的潜在风险，可以将"no-appendfsync-on-rewrite"设置为yes，否则就不要瞎搞，就保持为no

no-appendfsync-on-rewrite no

# AOF文件重写是Redis面试的一个点，也是优化Redis的一个点，将它设置得足够大，可以保存更多日志数据
# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
# 当AOF文件大小超过指定值"auto-aof-rewrite-min-size"，就会发生AOF文件重写
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
# Redis会记住AOF重写后的AOF文件大小，如果重启后还未发生重写，那么记住的就是刚开始加载AOF文件的大小
# 这个文件大小值会与下面的配置项值进行比较，决定什么时候做AOF文件重写
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
# 如果当"当前大小/最后一次重写大小"的比值大于"auto-aof-rewrite-percentage"指定的值，则会触发AOF重写
# 为了避免AOF已经很小还进行AOF重写的尴尬情况，因此需要设置一个AOF重写最小AOF文件大小
# 比如"auto-aof-rewrite-min-size"设置为64M，只有当AOF文件超过64M，且"当前大小/最后一次重写大小">"auto-aof-rewrite-percentage"才会触发AOF重写
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.
# 如果将"auto-aof-rewrite-percentage"设置为0，表示不允许执行自动AOF重写

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
# 如果运行Redis的OS崩溃掉，特别是ext4格式的文件系统使用"data=ordered"选项执行mount操作，在这些情况下
# AOF文件可能是截断(损坏)的，重启Redis时如果"aof-load-truncated"被设置为yes，那么AOF文件在加载时可能会丢失掉崩溃前的一些数据
#
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
# 针对损坏的AOF文件，在重启Redis的时候，支持两种方式
# 1.发现文件损坏，直接报错
# 2.尽可能的从找到的截断(损坏)文件中恢复数据到内存中，这是Redis的默认方式
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# 如果"aof-load-truncated"被设置为yes，且发现了被截断的AOF文件，那么在启动Redis时日志或者控制台中会输出日志，让运维人员或者监控看到这条信息
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
# 如果将"aof-load-truncated"设置no,且发现了被截断的AOF文件，重启Redis会报错，这个时候就需要借用redis-check-aof工具修复AOF文件
# 其实在主从模式下，是否可以到从节点拿AOF文件进行恢复，好像这个方法是多想了，因为哨兵、Codis、Cluster模式会自动进行故障转移，只有单机和纯主从模式也许这种方式可以尝试，但是现在的企业至少应该是哨兵模式了，大企业都用Cluster了或者豌豆荚搞的Codis
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
# 如果AOF文件在文件中间损坏了，即使"aof-load-truncated"设置为yes，重启Redis一样会报错且退出启动
# 这个选项只适合AOF被截断的情况，也就是AOF没有足够的字节
aof-load-truncated yes

# 混合持久化，Redis 4提供的新功能
# When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
# 如果"aof-use-rdb-preamble"设置为yes，那么AOF文件由"rdb file"+"aof tail"两部分组成，这种组合方式可以发挥RDB持久化加载速度快和压缩存储使用空间小的优势，与AOF持久化丢失数据小于1S的优势
# 
#   [RDB file][AOF tail]
#
# When loading Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, and continues loading the AOF
# tail.
# 该混合持久化方式下的AOF文件用"REDIS"字符串区分，前面是RDB内容，后面是AOF内容

aof-use-rdb-preamble yes

################################ LUA SCRIPTING LUA脚本 ###############################
# LUA脚本我没有研究过，简单说下这个配置项是设置LUA脚本最大执行时间
# 另外LUA脚本执行是原子的，因此可以用它做一些特殊的实现，不过就像Oracle的存储过程一样，维护不方便，比较这个脚本语言会的人太少了
# 如果确实有需要，在考虑运维的情况下可以使用它来实现原子性等操作，慎用
# Max execution time of a Lua script in milliseconds.
#
# If the maximum execution time is reached Redis will log that a script is
# still in execution after the maximum allowed time and will start to
# reply to queries with an error.
#
# When a long running script exceeds the maximum execution time only the
# SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be
# used to stop a script that did not yet called write commands. The second
# is the only way to shut down the server in the case a write command was
# already issued by the script but the user doesn't want to wait for the natural
# termination of the script.
#
# Set it to 0 or a negative value for unlimited execution without warnings.
# 如果设置为0或者负值，表示不限制执行时间
lua-time-limit 5000

################################ REDIS CLUSTER 集群 ###############################
# 在看下面的内容之前建议先去百度一下redis hash slots,以及集群的架构图
#
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# WARNING EXPERIMENTAL: Redis Cluster is considered to be stable code, however
# in order to mark it as "mature" we need to wait for a non trivial percentage
# of users to deploy it in production.
# 虽然Redis Cluster被认为是稳定的，但是依然需要大量的用户在生产环境中使用它。。。这段注释应该从redis.conf中删除了，全世界已经有知名的大企业使用了Redis Cluster
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#
# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
# 将"cluster-enabled"设置为yes,redis instance才能成为集群的一部分，但集群要真正开始工作，还需要将
# 所有的slots分配给cluster node
#
# cluster-enabled yes

# Every cluster node has a cluster configuration file. This file is not
# intended to be edited by hand. It is created and updated by Redis nodes.
# Every Redis Cluster node requires a different cluster configuration file.
# Make sure that instances running in the same system do not have
# overlapping cluster configuration file names.
# 每个cluster node有自己的cluster configuration file，且该配置文件不能手工编辑，而是自动创建和更新的
# cluster configuration file不能重名
#
# cluster-config-file nodes-6379.conf

# Cluster node timeout is the amount of milliseconds a node must be unreachable
# for it to be considered in failure state.
# Most other internal time limits are multiple of the node timeout.
# 集群节点在"cluster-node-timeout"规定的超时时间内，如果不可达，则被认为是失败状态
# 注意：集群内的大多数其他内部时间限制是"cluster-node-timeout"的倍数
#
# cluster-node-timeout 15000

# A replica of a failing master will avoid to start a failover if its data
# looks too old.
# 如果一个掉线主节点的从节点数据太老了，是不允许参与故障转移的
#
# There is no simple way for a replica to actually have an exact measure of
# its "data age", so the following two checks are performed:
# 没得撒子简单的办法可以一下计算出数据的年龄，因此Redis提供下面的两点来校验数据年龄，以决定集群节点是否参与故障转移过程：
#
# 1) If there are multiple replicas able to failover, they exchange messages
#    in order to try to give an advantage to the replica with the best
#    replication offset (more data from the master processed).
#    Replicas will try to get their rank by offset, and apply to the start
#    of the failover a delay proportional to their rank.
#    根据从节点的偏移量(主从复制-复制挤压缓冲区里面的偏移量，这个偏移量会跟着命令发给从节点，并保存下来)谁是最新的，并且根据偏移量排序，根据这个排序结果将从节点作为候选主节点
#
# 2) Every single replica computes the time of the last interaction with
#    its master. This can be the last ping or command received (if the master
#    is still in the "connected" state), or the time that elapsed since the
#    disconnection with the master (if the replication link is currently down).
#    If the last interaction is too old, the replica will not try to failover
#    at all.
#    每个从节点都会计算它与主节点最后一次交互时间，比如最后一次ping时间、最后一次接收命令时间、与主节点断开连接过去的时长
#    如果最后一次交互时间太长，那么这个从节点也不会参与故障转移过程
#
# The point "2" can be tuned by user. Specifically a replica will not perform
# the failover if, since the last interaction with the master, the time
# elapsed is greater than:
# 前面讲到的第2点有一个计算公式来衡量"最后一次交互时间"是否太长
#
#   (node-timeout * replica-validity-factor) + repl-ping-replica-period
#
# So for example if node-timeout is 30 seconds, and the replica-validity-factor
# is 10, and assuming a default repl-ping-replica-period of 10 seconds, the
# replica will not try to failover if it was not able to talk with the master
# for longer than 310 seconds.
# 假设"cluster-node-timeout"是30S，"replica-validity-factor"是10，"repl-ping-replica-period"是10S
# 如果"最后一次交互"时间超过"30*10+10=310"就被认为太长，而不能参与故障转移
#
# A large replica-validity-factor may allow replicas with too old data to failover
# a master, while a too small value may prevent the cluster from being able to
# elect a replica at all.
# "replica-validity-factor"太大，从节点数据可能会太久，如果太小可能选举不成功，集群不可用，所以要根据实际情况设置
#
# For maximum availability, it is possible to set the replica-validity-factor
# to a value of 0, which means, that replicas will always try to failover the
# master regardless of the last time they interacted with the master.
# (However they'll always try to apply a delay proportional to their
# offset rank).
# 如果为了保证最大的可用性，可以将"cluster-replica-validity-factor"设置为0。此时所有的从节点考虑最后一次交互时间的大小，总是会参与故障转移过程
#
# Zero is the only value able to guarantee that when all the partitions heal
# the cluster will always be able to continue.
# "cluster-replica-validity-factor"的"0"是唯一可以让集群总是可用的选项值
#
# cluster-replica-validity-factor 10

# Cluster replicas are able to migrate to orphaned masters, that are masters
# that are left without working replicas. This improves the cluster ability
# to resist to failures as otherwise an orphaned master can't be failed over
# in case of failure if it has no working replicas.
# 再不看下面的配置项功能时，有可能集群从节点会变成一个孤立的从节点，针对这种情况，如果它再发生故障，因为没有备选的从节点，所以故障转移动作没法完成。
#
# Replicas migrate to orphaned masters only if there are still at least a
# given number of other working replicas for their old master. This number
# is the "migration barrier". A migration barrier of 1 means that a replica
# will migrate only if there is at least 1 other working replica for its master
# and so forth. It usually reflects the number of replicas you want for every
# master in your cluster.
# 为了避免上面的情况发生，Redis Cluster默认配置要求一个主节点至少有两个从节点，一旦主节点挂了被新选举出来的主节点至少有一个从节点在工作。"cluster-migration-barrier"可以指定该值的大小，默认值是"1"
#
# Default is 1 (replicas migrate only if their masters remain with at least
# one replica). To disable migration just set it to a very large value.
# A value of 0 can be set but is useful only for debugging and dangerous
# in production.
# 默认值是1，如果要想禁止"migration",可以将"cluster-migration-barrier"设置为一个超大的值
# 可以为了调试或者你想让自己的系统存在高风险的运行，可以设置为0。。。no zuo no die
#
# cluster-migration-barrier 1

# By default Redis Cluster nodes stop accepting queries if they detect there
# is at least an hash slot uncovered (no available node is serving it).
# This way if the cluster is partially down (for example a range of hash slots
# are no longer covered) all the cluster becomes, eventually, unavailable.
# It automatically returns available as soon as all the slots are covered again.
# Redis Cluster默认情况下如果有一个hash slot没有被分配(用一个Cluster Node接收它)，那么整个集群是不可用的
# 在这种模式下，一旦出现网络分区(一段hash slots 就变成未分配)，整个集群就不可用了，直到所有hash slots被分配，集群会自动变得可用
#
# However sometimes you want the subset of the cluster which is working,
# to continue to accept queries for the part of the key space that is still
# covered. In order to do so, just set the cluster-require-full-coverage
# option to no.
# 也许有时你想即使出现hash slots unconverd，而集群的部分节点仍然是可用的，可以将"cluster-require-full-coverage"设置为no
#
# cluster-require-full-coverage yes

# This option, when set to yes, prevents replicas from trying to failover its
# master during master failures. However the master can still perform a
# manual failover, if forced to do so.
# 如果将"cluster-replica-no-failover"设置为yes，那么该集群从节点不会参与自动故障转移过程，但是可以手动强制执行故障转移
#
# This is useful in different scenarios, especially in the case of multiple
# data center operations, where we want one side to never be promoted if not
# in the case of a total DC failure.
# 在不同场景可能非常有用，比如有多个数据中心，而我们又不希望整个集群中的某一个数据中心的从节点被提升为主节点
#
# cluster-replica-no-failover no

# In order to setup your cluster make sure to read the documentation
# available at http://redis.io web site.

########################## CLUSTER DOCKER/NAT support  ########################

# In certain deployments, Redis Cluster nodes address discovery fails, because
# addresses are NAT-ted or because ports are forwarded (the typical case is
# Docker and other containers).
#
# In order to make Redis Cluster working in such environments, a static
# configuration where each node knows its public address is needed. The
# following two options are used for this scope, and are:
#
# * cluster-announce-ip
# * cluster-announce-port
# * cluster-announce-bus-port
#
# Each instruct the node about its address, client port, and cluster message
# bus port. The information is then published in the header of the bus packets
# so that other nodes will be able to correctly map the address of the node
# publishing the information.
#
# If the above options are not used, the normal Redis Cluster auto-detection
# will be used instead.
#
# Note that when remapped, the bus port may not be at the fixed offset of
# clients port + 10000, so you can specify any port and bus-port depending
# on how they get remapped. If the bus-port is not set, a fixed offset of
# 10000 will be used as usually.
#
# Example:
#
# cluster-announce-ip 10.1.1.5
# cluster-announce-port 6379
# cluster-announce-bus-port 6380

################################## SLOW LOG 慢日志 ###################################

# The Redis Slow Log is a system to log queries that exceeded a specified
# execution time. The execution time does not include the I/O operations
# like talking with the client, sending the reply and so forth,
# but just the time needed to actually execute the command (this is the only
# stage of command execution where the thread is blocked and can not serve
# other requests in the meantime).
# 记录Redis执行耗时超过指定值的"查询命令"，整个"耗时"仅仅是执行命令的耗时(在这段时间内，因为线程被阻塞，其他命令会被阻塞)，不包括与客户端网络IO所耗时间或者发数据给客户端的耗时
#
# You can configure the slow log with two parameters: one tells Redis
# what is the execution time, in microseconds, to exceed in order for the
# command to get logged, and the other parameter is the length of the
# slow log. When a new command is logged the oldest one is removed from the
# queue of logged commands.
# 可以使用"slowlog-log-slower-than"指定耗时的阈值(单位是微妙)，一旦执行超过这个时间就会记录日志到缓冲区
# 可以使用"slowlog-max-len 128"指定队列长度，如果超过队列，最老的元素会被覆盖
#
# The following time is expressed in microseconds, so 1000000 is equivalent
# to one second. Note that a negative number disables the slow log, while
# a value of zero forces the logging of every command.
# 单位是微妙，不能设置为负值，如果设置为0，那么所有的查询命令都会记录到队列中
# 
slowlog-log-slower-than 10000

# There is no limit to this length. Just be aware that it will consume memory.
# 最大值没有限制，我们只需要考虑内存是否足够大
# You can reclaim memory used by the slow log with SLOWLOG RESET.
# 可以使用slowlog reset回收已使用的内存
slowlog-max-len 128

################################ LATENCY MONITOR ##############################

# The Redis latency monitoring subsystem samples different operations
# at runtime in order to collect data related to possible sources of
# latency of a Redis instance.
# 延迟监控子系统通过采集运行时的不同操作去收集造成Redis实例延迟的相关可能来源
# 
#
# Via the LATENCY command this information is available to the user that can
# print graphs and obtain reports.
# 可以通过latency命令获得可用信息的图表，比如latency docter xxx/latency graph等
#
# The system only logs operations that were performed in a time equal or
# greater than the amount of milliseconds specified via the
# latency-monitor-threshold configuration directive. When its value is set
# to zero, the latency monitor is turned off.
# 该监控子系统只会记录那些耗时>="latency-monitor-threshold"指定的值对应的操作，如果设置为0，表示关闭延时监控
#
# By default latency monitoring is disabled since it is mostly not needed
# if you don't have latency issues, and collecting data has a performance
# impact, that while very small, can be measured under big load. Latency
# monitoring can easily be enabled at runtime using the command
# "CONFIG SET latency-monitor-threshold <milliseconds>" if needed.
# Redis默认是关闭了延迟监控的，因为绝大多数时间是用不着的，因为开启它有一定的性能损失，除非你的服务发生了延时而开启监控
# 当Redis是运行着的时候，可以通过config set latency-monitor-threshold xxx轻松开启监控
latency-monitor-threshold 0

############################# EVENT NOTIFICATION ##############################
# 下面的条件说明很多看上去挺复杂的，其实很简单：就是多个字符代表的意思组合到一起而已
# Redis can notify Pub/Sub clients about events happening in the key space.
# This feature is documented at http://redis.io/topics/notifications
# Redis可以将关于"键空间(简单理解为Hash表中的键值对)"发生的事件以通知的形式发送给Pub/Sub客户端
# 更详细的请参考Redis的官方文档：http://redis.io/topics/notifications
#
# For instance if keyspace events notification is enabled, and a client
# performs a DEL operation on key "foo" stored in the Database 0, two
# messages will be published via Pub/Sub:
# 如果通过配置开启了键空间和键时间的通知，如果通过客户端在第0号database上执行一个DEL foo操作，那么会
# 发布两条消息
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# It is possible to select the events that Redis will notify among a set
# of classes. Every class is identified by a single character:
# 我们可以通过组合下面的分类将事件通知发给客户端
#  "K"和"E"代表两大类，无论怎么组合，必须有其中一个，可以两个同时选择，K代表Keyspace事件，E代表Keyevent事件
#  K以为着一个或多个数据类型的所有符合规则事件都会生成通知
#  E以为着一个或多个数据类型的某一个命令的时间会生成通知
#  如果看到这里还没明白，建议去百度一下，推荐一个：http://redisdoc.com/topic/notification.html#id1
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#
#  一般的命令，比如DEL SET EXPIRE RENAME等等，感觉像是所有会产生改变的命令都符合条件      
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#
#  下面的$ l s h z 分别代表大家都知道5种数据类型
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#
#  x 代表过期事件  e 代表内存使用超过maxmemory时KEY被淘汰的事件
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#
#  A 是一个别名，代表了"g$lshzxe"的组合，可以增强阅读性
#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
#
#  The "notify-keyspace-events" takes as argument a string that is composed
#  of zero or multiple characters. The empty string means that notifications
#  are disabled.
#  可以给"notify-keyspace-events"设置0或者多个字符，如果设置为空字符串，则表示关闭此功能
#
#  Example: to enable list and generic events, from the point of view of the
#           event name, use:
#
#  notify-keyspace-events Elg
#
#  Example 2: to get the stream of the expired keys subscribing to channel
#             name __keyevent@0__:expired use:
#
#  notify-keyspace-events Ex
#
#  By default all notifications are disabled because most users don't need
#  this feature and the feature has some overhead. Note that if you don't
#  specify at least one of K or E, no events will be delivered.
#  因为开启此功能是有一定开销的，会影响性能，而且大多数用户不需要此功能，所以默认是关闭了此功能的，不会有事件通知被发送
notify-keyspace-events ""

############################### ADVANCED CONFIG 高级配置 ###############################
# 下面的配置需要对Redis的原理，特别5中数据类型的底层数据结构有比较清楚的了解才能看得懂，总的来说就是根据自己的键-值选择5中数据类型在某些条件下使用何种数据结构来存放数据。
# 最常见的高效数据结构就是ziplist、intset,但是他们通常只有元素(条目)较小且元素(条目)较小时才适合
# 要学习这部分内容可以看看redis设计与实现和Redis资深历险两本书，前一本书将原理很多，且深度足够，但是Redis的版本有点太老了，后一步本书可以在原理上对前一本书进行补充，且Redis版本很新，已经到5了。而且它还将了很多实战的知识。
#
# Hashes are encoded using a memory efficient data structure when they have a
# small number of entries, and the biggest entry does not exceed a given
# threshold. These thresholds can be configured using the following directives.
#
# hash数据类型：如果条目数小于512，且条目大小不超过64字节，则使用ziplist作为hash数据类型的底层数据结构
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

# 新版本的Redis针对list数据类型的底层数据结构做了优化采用的是"链表+ziplist"，其思想有点像Java HashMap的"数组+链表/红黑树"
# Lists are also encoded in a special way to save a lot of space.
# The number of entries allowed per internal list node can be specified
# as a fixed maximum size or a maximum number of elements.
# 可以通过"list-max-ziplist-size"设置链表中ziplist的条目数量，其值可以是条目数量，也可以最大字节数
# For a fixed maximum size, use -5 through -1, meaning:
# 下面是5个可能取值，建议使用-1 和 -2，其他选项不推荐使用，除非有特殊需求
# -5: max size: 64 Kb  <-- not recommended for normal workloads
# -4: max size: 32 Kb  <-- not recommended
# -3: max size: 16 Kb  <-- probably not recommended
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
#
# Positive numbers mean store up to _exactly_ that number of elements
# per list node.
# 上面的负值就是单个链表节点所包含的条目数
#
# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
# but if your use case is unique, adjust the settings as necessary.
# 取值为-1 -2 发挥的性能是最好的
list-max-ziplist-size -2

# Lists may also be compressed.
# Compress depth is the number of quicklist ziplist nodes from *each* side of
# the list to *exclude* from compression.  The head and tail of the list
# are always uncompressed for fast push/pop operations.  Settings are:
# 0: disable all list compression
#    表示不压缩任何节点
# 1: depth 1 means "don't start compressing until after 1 node into the list,
#    going from either the head or tail"
#    So: [head]->node->node->...->node->[tail]
#    [head], [tail] will always be uncompressed; inner nodes will compress.
#    表示除链表的头尾以外，其他链表节点都压缩
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#    2 here means: don't compress head or head->next or tail->prev or tail,
#    but compress all nodes between them.
#    依次类推，即前两个和后两个以外的都压缩
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
#    依次类推
# etc.
# 默认压缩深度为0，也就是说不压缩。。。无论如何设置头尾是不会压缩的，比如当list被当做队列使用时，如果压缩了，还需要解压，降低了性能。
list-compress-depth 0

# Sets have a special encoding in just one case: when a set is composed
# of just strings that happen to be integers in radix 10 in the range
# of 64 bit signed integers.
# 当集合(set)存放的值都是64位的无符号10进制整数时，且条目数小于512时会采用intset作为集合的底层数据结构
# The following configuration setting sets the limit in the size of the
# set in order to use this special memory saving encoding.
set-max-intset-entries 512

# 和hash数据类型类似，如果条目数小于128，且条目大小<64会使用ziplist作为有序集合的底层数据结构
# Similarly to hashes and lists, sorted sets are also specially encoded in
# order to save a lot of space. This encoding is only used when the length and
# elements of a sorted set are below the following limits:
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

# HyperLogLog sparse representation bytes limit. The limit includes the
# 16 bytes header. When an HyperLogLog using the sparse representation crosses
# this limit, it is converted into the dense representation.
#
# A value greater than 16000 is totally useless, since at that point the
# dense representation is more memory efficient.
#
# The suggested value is ~ 3000 in order to have the benefits of
# the space efficient encoding without slowing down too much PFADD,
# which is O(N) with the sparse encoding. The value can be raised to
# ~ 10000 when CPU is not a concern, but space is, and the data set is
# composed of many HyperLogLogs with cardinality in the 0 - 15000 range.
# 这个是Redis高级功能，可以用这种数据结构统计网站的UV，能够去重，其准确度接近真实值
# 简单点说：当去重后统计出来的值小于"hll-sparse-max-bytes"指定的值时，Redis会使用稀疏矩阵来存放，一个Key占用的空间比稠密矩阵小，如果统计出来的值大于"hll-sparse-max-bytes"指定的值，那么使用稠密矩阵，此时一个Key占用的空间是12KB
# "hll-sparse-max-bytes"默认为3000，如果设置为16000以上完全是无用的，因为此时稠密矩阵效果更好
hll-sparse-max-bytes 3000

# Streams macro node max size / items. The stream data structure is a radix
# tree of big nodes that encode multiple items inside. Using this configuration
# it is possible to configure how big a single node can be in bytes, and the
# maximum number of items it may contain before switching to a new node when
# appending new stream entries. If any of the following settings are set to
# zero, the limit is ignored, so for instance it is possible to set just a
# max entires limit by setting max-bytes to 0 and max-entries to the desired
# value.
# 设置Stream的单个节点最大字节数和最多能有多少个条目，如果任何一个条件满足就会新增加一个节点用以保存新的数据
# 如果将任何一个配置项设置为0，表示不限制
stream-node-max-bytes 4096
stream-node-max-entries 100

# Redis数据库存放键值对数据结构是一个类型为字典长度为2的数组,假设这个数组名称为"ht"，在rehash的时候就是将其中一个字典(ht[0])中的所有数据搬到另一个字典(ht[1])中,而且rehash是惰性的(因为redis要高效的响应查询或者写，不可能去一次完成rehash操作，不像Java的HashMap)，当方式key时或者CPU比较空闲时会触发，因此也被称之为"渐进式hash"
# Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in
# order to help rehashing the main Redis hash table (the one mapping top-level
# keys to values). The hash table implementation Redis uses (see dict.c)
# performs a lazy rehashing: the more operation you run into a hash table
# that is rehashing, the more rehashing "steps" are performed, so if the
# server is idle the rehashing is never complete and some more memory is used
# by the hash table.
#
# The default is to use this millisecond 10 times every second in order to
# actively rehash the main dictionaries, freeing memory when possible.
# 默认是使用1秒钟的10毫秒进行rehash，在适当的时候回收内存
#
# If unsure:
# use "activerehashing no" if you have hard latency requirements and it is
# not a good thing in your environment that Redis can reply from time to time
# to queries with 2 milliseconds delay.
# 如果系统有严格的延时要求，在2毫秒内不断的查询出结果，可以将"activerehashing"设置no
# 但是这对你的系统并不是一个好事情，因此不建议这样设置，所以保持不动吧
#
# use "activerehashing yes" if you don't have such hard requirements but
# want to free memory asap when possible.
# 如果没有非常严格的要求，建议将"activerehashing"设置为yes，这样可以让内存尽可能快的释放
activerehashing yes

# The client output buffer limits can be used to force disconnection of clients
# that are not reading data from the server fast enough for some reason (a
# common reason is that a Pub/Sub client can't consume messages as fast as the
# publisher can produce them).
# 可以通过设置客户端输出缓冲区大小将待接收数据超过缓冲区大小的客户端断开
# 通常使用pub/sub的时候，客户端没有及时消费而导致超过缓冲区大小
#
# The limit can be set differently for the three different classes of clients:
# 提供三种客户端的设置，分别是普通的、主从复制的、pub/sub的客户端，我们可以分别对这三种客户端的输出缓冲区设置大小
#
# normal -> normal clients including MONITOR clients
# replica  -> replica clients
# pubsub -> clients subscribed to at least one pubsub channel or pattern
#
# The syntax of every client-output-buffer-limit directive is the following:
# 下面是三种客户端缓冲区大小设置的语法
# 
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
#
# A client is immediately disconnected once the hard limit is reached, or if
# the soft limit is reached and remains reached for the specified number of
# seconds (continuously).
# 如果客户端输出缓冲区的大小达到了"hard limit"，服务器会立即断开连接
# 如果客户端输出缓冲区的大小达到了"soft limit"，且持续时间达到了"soft seconds"，服务器会立即断开连接
#
# So for instance if the hard limit is 32 megabytes and the soft limit is
# 16 megabytes / 10 seconds, the client will get disconnected immediately
# if the size of the output buffers reach 32 megabytes, but will also get
# disconnected if the client reaches 16 megabytes and continuously overcomes
# the limit for 10 seconds.
# 这上面是一个举例，省略。。。。
#
# By default normal clients are not limited because they don't receive data
# without asking (in a push way), but just after a request, so only
# asynchronous clients may create a scenario where data is requested faster
# than it can read.
# 默认情况下普通的client不限制，因为它们都是发起请求后等待接收数据，并不像异步的客户端(比如主从复制客户端和PUB/SUB)会造成数据的挤压，挤压的原因就是客户端处理速度跟不上数据产生的速度
#
# Instead there is a default limit for pubsub and replica clients, since
# subscribers and replicas receive data in a push fashion.
#
# Both the hard or the soft limit can be disabled by setting them to zero.
# hard or soft limit 都可以通过设置为0而禁止掉
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# Client query buffers accumulate new commands. They are limited to a fixed
# amount by default in order to avoid that a protocol desynchronization (for
# instance due to a bug in the client) will lead to unbound memory usage in
# the query buffer. However you can configure it here if you have very special
# needs, such us huge multi/exec requests or alike.
# 客户端查询缓冲区会累加新命令，默认情况下，缓冲区大小是一个固定值以避免协议同步失效(如客户端的bug)导致查询缓冲区出现未绑定的内存(即客户端都已经不存在了，但是它发过来的命令还在缓冲区当中)
# 如果有巨大的multi/exec请求，则可以修改这个值以满足我们的特殊需求
# 
# client-query-buffer-limit 1gb

# In the Redis protocol, bulk requests, that are, elements representing single
# strings, are normally limited ot 512 mb. However you can change this limit
# here.
# 如果一个大容量请求(即客户端单次发送过来的字符串)被限制为512MB，我们也可以通过修改"proto-max-bulk-len"值
# 不过我可能一辈子也不会用到
# proto-max-bulk-len 512mb

# Redis calls an internal function to perform many background tasks, like
# closing connections of clients in timeout, purging expired keys that are
# never requested, and so forth.
#
# Not all tasks are performed with the same frequency, but Redis checks for
# tasks to perform according to the specified "hz" value.
#
# By default "hz" is set to 10. Raising the value will use more CPU when
# Redis is idle, but at the same time will make Redis more responsive when
# there are many keys expiring at the same time, and timeouts may be
# handled with more precision.
#
# The range is between 1 and 500, however a value over 100 is usually not
# a good idea. Most users should use the default of 10 and raise this up to
# 100 only in environments where very low latency is required.
# 简单点说：Redis有后台任务，通过设置"hz"可提高或者降低检查这些任务是否应该执行的频率，值越大消耗的CPU越多，反之越少
# 值可以设置在1到500之间，通常不建议将该值设置得比100大，一般都使用10这个默认值，除非我们的系统有非常严格的延时要求，才会将"hz"设置得等于或者超过100
hz 10

# Normally it is useful to have an HZ value which is proportional to the
# number of clients connected. This is useful in order, for instance, to
# avoid too many clients are processed for each background task invocation
# in order to avoid latency spikes.
#
# Since the default HZ value by default is conservatively set to 10, Redis
# offers, and enables by default, the ability to use an adaptive HZ value
# which will temporary raise when there are many connected clients.
#
# When dynamic HZ is enabled, the actual configured HZ will be used as
# as a baseline, but multiples of the configured HZ value will be actually
# used as needed once more clients are connected. In this way an idle
# instance will use very little CPU time while a busy instance will be
# more responsive.
# 英文有时还真的描述很啰唆，还是中文编码更高效。。。吐槽一下英文
# 前面的"ht"配置项是固定值，当连接客户端非常多时，如果"ht"还是10，则可能会导致延迟比较高，因此Redis搞了一个
# "dynamic-hz"配置项，当设置为yes时，可以基于"ht"配置值动态的调整使用的"ht"值，比如连接的客户端很多事，动态将ht调高，可以减少延迟。而当连接客户端比较少，又可以动态降低"ht"，这样消耗的CPU会很少
# 默认值是yes，这个根本不需要我们自己去动，有了它我们也不需要去动"ht"配置
dynamic-hz yes

# When a child rewrites the AOF file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
# 当子进程在重写AOF文件时，如果将"aof-rewrite-incremental-fsync"设置为yes，那么一旦生成32M数据才会调用一次OS的fsync函数，这样可以降低出现访问峰值时系统的延迟。因为可以减少fsync调用次数和IO请求
aof-rewrite-incremental-fsync yes

# When redis saves RDB file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
# 和"aof-rewrite-incremental-fsync"一个意思，只不过是用在生成RDB文件时用。
# 如果持久化采用的混合方式，即AOF文件是由"RDB部分+AOF部分"组成的话，我想"aof-rewrite-incremental-fsync"和"rdb-save-incremental-fsync"都会使用到
rdb-save-incremental-fsync yes

# Redis LFU eviction (see maxmemory setting) can be tuned. However it is a good
# idea to start with the default settings and only change them after investigating
# how to improve the performances and how the keys LFU change over time, which
# is possible to inspect via the OBJECT FREQ command.
#
# There are two tunable parameters in the Redis LFU implementation: the
# counter logarithm factor and the counter decay time. It is important to
# understand what the two parameters mean before changing them.
#
# Redis的LFU实现有两个可调整的参数：计数器对数因子(couter logarithm factor)和计数器衰退时间(counter decay time)
# 一定要充分理解这两个参数之后才能去修改，如果不懂就不要去瞎搞了，如果非要修改，一定要使用"OBJECT FREQ"命令充分调查并知道如何提升性能的情况下才能进行
# The LFU counter is just 8 bits per key, it's maximum value is 255, so Redis
# uses a probabilistic increment with logarithmic behavior. Given the value
# of the old counter, when a key is accessed, the counter is incremented in
# this way:
# 在介绍maxmemory的时候提到了两个参数作用的原理，这里就不赘述了。
#
# 1. A random number R between 0 and 1 is extracted.
# 2. A probability P is calculated as 1/(old_value*lfu_log_factor+1).
# 3. The counter is incremented only if R < P.
#
# The default lfu-log-factor is 10. This is a table of how the frequency
# counter changes with a different number of accesses with different
# logarithmic factors:
# "lfu-log-factor"的默认值=10,下表是不同对数因子下计数器的改变频率:
#
# +--------+------------+------------+------------+------------+------------+
# | factor | 100 hits   | 1000 hits  | 100K hits  | 1M hits    | 10M hits   |
# +--------+------------+------------+------------+------------+------------+
# | 0      | 104        | 255        | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 1      | 18         | 49         | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 10     | 10         | 18         | 142        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 100    | 8          | 11         | 49         | 143        | 255        |
# +--------+------------+------------+------------+------------+------------+
#
# NOTE: The above table was obtained by running the following commands:
# 上面的表格可以通过下面的命令得到：
#
#   redis-benchmark -n 1000000 incr foo
#   redis-cli object freq foo
#
# NOTE 2: The counter initial value is 5 in order to give new objects a chance
# to accumulate hits.
# 默认counter的初始值是5，为了让新的对象有机会累加它的命中率
#
# The counter decay time is the time, in minutes, that must elapse in order
# for the key counter to be divided by two (or decremented if it has a value
# less <= 10).
# 计数器衰减时间是key计数器除以2(如果值小于<=10，则递减)所必须经过的时间，单位为分钟。
#
# The default value for the lfu-decay-time is 1. A Special value of 0 means to
# decay the counter every time it happens to be scanned.
# "lfu-decay-time" 的默认值为 1，0 表示每次都对计数器进行衰减
#
# lfu-log-factor 10
# lfu-decay-time 1

########################### ACTIVE DEFRAGMENTATION #######################
########################### 在线碎片整理 #######################
#
# WARNING THIS FEATURE IS EXPERIMENTAL. However it was stress tested
# even in production and manually tested by multiple engineers for some
# time.
# 这还只是一个实验功能，就像Redis Cluster一样，其实已经有很多人在使用了
# What is active defragmentation?
# -------------------------------
#
# Active (online) defragmentation allows a Redis server to compact the
# spaces left between small allocations and deallocations of data in memory,
# thus allowing to reclaim back memory.
#
# Fragmentation is a natural process that happens with every allocator (but
# less so with Jemalloc, fortunately) and certain workloads. Normally a server
# restart is needed in order to lower the fragmentation, or at least to flush
# away all the data and create it again. However thanks to this feature
# implemented by Oran Agra for Redis 4.0 this process can happen at runtime
# in an "hot" way, while the server is running.
# 活动碎片整理允许Redis服务器压缩内存中由于申请和释放数据块导致的碎片，从而回收内存，就好像window的磁盘整理一样
# 碎片是每次申请内存（幸运的是Jemalloc出现碎片的几率小很多）的时候会自然发生的
# 通常来说，为了降低碎片化程度需要重启服务，或者清除所有的数据然后重新创建。 得益于Oran Agra在Redis 4.0实现的这个特性，进程可以在服务运行时以"热"方式完成
#
# Basically when the fragmentation is over a certain level (see the
# configuration options below) Redis will start to create new copies of the
# values in contiguous memory regions by exploiting certain specific Jemalloc
# features (in order to understand if an allocation is causing fragmentation
# and to allocate it in a better place), and at the same time, will release the
# old copies of the data. This process, repeated incrementally for all the keys
# will cause the fragmentation to drop back to normal values.
# 通常来说当碎片化达到一定程度（查看下面的配置）Redis 会使用Jemalloc创建连续的内存空间，并在此内存空间对现有的值进行拷贝，拷贝完成后会释放掉旧的数据。
# 这个过程会对所有的导致碎片化的key以增量的形式进行，Redis处处使用渐进式的，真实辛苦设计者了
#
# Important things to understand:
# 要重点理解的三点：
#
# 1. This feature is disabled by default, and only works if you compiled Redis
#    to use the copy of Jemalloc we ship with the source code of Redis.
#    This is the default with Linux builds.
#    默认情况下，该功能是关闭的，并且只有在编译Redis时使用了代码中的Jemalloc才生效(这是 Linux 下的默认行为)
# 2. You never need to enable this feature if you don't have fragmentation
#    issues.
#    如果没有碎片问题，我们永远也不需要启用该功能
#
# 3. Once you experience fragmentation, you can enable this feature when
#    needed with the command "CONFIG SET activedefrag yes".
#    可以通过命令"CONFIG SET activefrag yes"来启用并试验
#
# The configuration parameters are able to fine tune the behavior of the
# defragmentation process. If you are not sure about what they mean it is
# a good idea to leave the defaults untouched.
# 相关的配置参数可以很好的调整碎片整理过程，如果你不知道这些选项的作用最好使用默认值。

# Enabled active defragmentation
# 开启在线整理
# activedefrag yes

# Minimum amount of fragmentation waste to start active defrag
# 有多少碎片时开始整理
# active-defrag-ignore-bytes 100mb

# Minimum percentage of fragmentation to start active defrag
# 有多少比例的碎片时开始整理
# active-defrag-threshold-lower 10

# Maximum percentage of fragmentation at which we use maximum effort
# 有多少比例的碎片时开始进行整理
# active-defrag-threshold-upper 100

# Minimal effort for defrag in CPU percentage
# 进行碎片整理时使用多少比例的CPU时间
# active-defrag-cycle-min 5

# Maximal effort for defrag in CPU percentage
# 进行整理时使用多少CPU时间
# active-defrag-cycle-max 75

# Maximum number of set/hash/zset/list fields that will be processed from
# the main dictionary scan
# 进行主字典扫描时处理的 set/hash/zset/list 字段的最大数量(就是说在进行主字典扫描时 set/hash/zset/list 的长度小于这个值才会处理，大于这个值的会放在一个列表中延迟处理)
# 因为如果某一个key过大，一次性处理完会非常耗时的
# active-defrag-max-scan-fields 1000
```

