#还没有复习 

#  The Linux Commend Line

[Linux指令分类列表](https://www.runoob.com/linux/linux-command-manual.html)

- date
- cal
- file
- less
- ln
- type
- which
- apropos
- info
- alias
- cat
- sort
- uniq
- '>'   '>>'   '1 | 2 > '  '&>'
- wc 文本行数，词数，字节数计数器
- head
- tail
- tee   从标准输入中读数据，输出到目标文件中
- df
- free



- $((expression))      $((1+1))
- {}     `touch test{1,2,3}.txt`创建三个文件   `echo {Z..A}`   ` echo Number_{1..5}`
- printenv
- env
- $(commend) 或\`commend\`           `ls -l $(which cp)` √            `ls -l $which cp`×            
- 单引号和双引号
- history
- !n     n是用history查询出来的命令的行数
- id
- chmod   
- umask   掩码
- su    sudo  su输入的是目标id的密码，sudo输入的是本用户的密码，且使用sudo的用户必须提前在/etc/sudoers中配置
- chown
- chgrp
- passwd
- adduser    useradd    groupadd



只列举部分快捷键

- crtl  + a   光标移至行首
- crtl  + e   光标移至行尾
- crtl  + l    等于clear
- crtl  + k    剪切  到行尾
- crtl  + u    剪切  到行首
- crtl  + y    粘贴
- ctrl + r     增量搜索历史执行过的命令  （回车：直接执行搜索到的命令，ctrl + j：将搜到的命令放到命令提示符后但不执行）
- ctrl + c     中断信号
- ctrl + z    终端停止信号
- alt  + u    光标到行尾字母全转大写
- alt  + l      光标到行尾字母全转小写



- ps
- top   htop
- &
- jobs      展示执行的& 后台命令
- bg       把前台程序移至后台
- fg        指定进程的任务号进程返回到前台，任务号由jpbs命令查询
- kill
- killall
- shutdown
- source



可选命令

- xlogo
- vmstat
- xload
- tload
- 都是进程监控工具



vim

- 快捷键
- 查找指令   /
- 替换指令    `:%s/old/new/g`
- 跨文件复制   部分复制和全文件复制

命令提示符

- `echo $PS1`



• mount –挂载一个文件系统

• umount –卸载一个文件系统

• fsck –检查和修复一个文件系统

• fdisk –分区表控制器

• mkfs –创建文件系统

• fdformat –格式化一张软盘

• dd —把面向块的数据直接写入设备

• genisoimage (mkisofs) –创建一个 ISO 9660 的映像文件

• wodim (cdrecord) –把数据写入光存储媒介

• md5sum –计算 MD5 检验码





网络系统

• ping - 发送 ICMP ECHO_REQUEST 数据包到网络主机

• traceroute - 打印到一 台网络主机的路由数据包

• netstat - 打印网络连接，路由表，接口统计数据，伪装连接，和多路广播成员

• ftp - 因特网文件传输程序

• wget - 非交互式网络下载器

• ssh - OpenSSH SSH 客户端（远程登录程序）

- scp - 安全复制 （可双向复制-本机复制到目标或目标复制到本机）
- sftp - ftp的安全的替代品（ftp使用明码传输文件，不安全，sftp使用ssh传输且加密）
- ps：**rsync**



查找文件

• locate –在文件名数据库中  通过名字来查找文件（数据库由定时任务定时执行updatedb命令更新数据库）

• find –在一个目录层次结构中搜索文件（使用方法可以简单也可以非常复杂）

我们也将看一个经常与文件搜索命令一起使用的命令，它用来处理搜索到的文件列表：

• xargs –从标准输入生成和执行命令行

另外，我们将介绍两个命令以便在我们探索的过程中协助我们：

• touch –更改文件时间

• stat –显示文件或文件系统状态



文件压缩程序：

• gzip –压缩或者展开文件    压缩文件拓展名  .gz

• bzip2 –块排序文件压缩器     压缩文件拓展名    .bz2

归档程序：

• tar –磁带打包工具

• zip –打包和压缩文件

还有文件同步程序：

• rsync –同步远端文件和目录


# End


# Linux

> linux的命令基本结构为 `命令体` + 若干个（`选项` + `参数`）

## 目录树结构

![[../020 - 附件文件夹/Pasted image 20230402140304.png|500]]

## 预热

```shell
# 查询当前登录的用户
$ who am i

# 查询某个用户用的shell是什么
$ grep root /etc/passwd
```

除了bash以外的shell还有ksh、tcsh、cshsh、dash等

```shell
# 获取当前时间
$ date

# 显示当前所在目录
$ pwd

# 列举当前目录的文件和目录 
# - l 长列表 a 显示隐藏文件 t 按时间排序 h 以K/M为单位显示文件大小（默认以字节为单位）
$ ls
```

一般来说 - 后面加上行为参数 -- 后面加上一个完整的单词作为参数

一条命令 = 命令 + [ -可选参数 ] + [ 选项 ]

```shell
# 这条命令意思为不要列出名字为Desktop的文件或文件夹，=前后不要有空格
$ ls --hide=Desktop

# 查看当前用户信息
# 这里表示用户是root gid指用户所属的主组 groups指用户所在的组，可能有多个
$ id
uid=0(root) gid=0(root) groups=0(root)

# 获取当前会话信息
$ who -uH

# 获取环境变量的位置，大小写敏感
$ echo $PATH

# 查看之前输入的命令历史列表 后面直接加空格和数字表示站址指定数量的列表
$ history

# 查询某个命令的所在位置
$ type bash
# 列出ls命令的别名和文件位置 / 这是谁的别名
$ type -a ls
$ type ll
ll 是 `ls -alF' 的别名

# 对于不在PATH中的命令，可以使用locate命令查询
$ locate chage

# 管道命令 这里 sort 指将列表按首字母进行排序 less 将列表更人性化的展示出来
$ cat /etc/passwd | sort | less

# 用分号将不同的命令隔开，达到一行命令行中有好几条命令的效果
$ date ; troff -me verlargedocument | lpr

# 后台命令。在命令末尾加上&符号，命令的执行会在后台执行，而不会占用当前窗口。然后返回一个进程号，可通过kill [进程号] 杀死进程
$ java -jar imakerlab-bbs.jar &
```

扩展命令 $(command) 或 `command`

例：$ vi $(find /home | grep xyzzy)  find /home 并打印该目录下所有不包括xyzzy字符串的文件和目录。最后用vi命令打开所有文件进行编辑（每次打开一个文件）

```shell
# 扩展算术表达式。在shell中执行计算，使用$[xx] 或 $(xx)
$ echo  "i am $[1+1] years old."

# 拓展变量 shell语法里可以定义变量并在需要的时候取出来。使用$+变量名，shell有自己的初始变量，如下,$BASH取出BASH变量
$ echo $BASH

# 查看所有环境变量
$ env
```

常见的环境变量

![[../020 - 附件文件夹/Pasted image 20230402140326.png|575]]

```shell
# 创建和使用别名，给pwd赋别名。下次使用pwd命令时直接使用p即可
$ alias p='pwd'
```

linux的配置文件位置

![[../020 - 附件文件夹/Pasted image 20230402140427.png|625]]

图中前两个是root用户才能修改的

```shell
# 提示符，默认由 '用户名@主机名 当前工作目录 $' 组成。如：
root@root:/home$
```


## 开始学


## 文件移动

![[../020 - 附件文件夹/Pasted image 20230402140443.png|500]]

### chmod

 > 修改文件权限

```shell
# 修改文件权限 r=4，w=2，x=1 -=0  文件的三段权限字母分别表示拥有者，组群，其他用户对改文件的操作权限
$ chmod +/- rwx 文件路径
$ chmod 700 文件路径

# - R 递归修改权限，对指定目录及其下的所有子文件和子文件夹统一进行权限修改
$ chmod 700 文件夹路径

# 元字符的使用
# * 表示任意个数字符 ?表示一个字符 [...]表示匹配括号之间的任何一个字符，可以用连字符表示数字或字母的范围
$ ls -l a*
$ ls -l [1-5]
$ ls -l [a-f]n*

# - p 创建文件夹及其子目录 
$ mkdir -p test/a/b/c
```

### >

> 文件的重定向

![[../020 - 附件文件夹/Pasted image 20230402140456.png]]

```shell
# 把指定文件以邮件形式发送到root用户
$ mail root < ~/.bashrc

# 把指定内容添加到指定文件里 >> 的内容会先自动换行再追加内容
$ echo asdfasdfsdfd  > 1.txt
$ echo asdfasdfsdfd  >> 1.txt
```

### {}

> 使用括号扩展字符

```shell
# 使用大括号，可同时操作括号里的所有文件和文件夹，大括号外的字符表示公共字符
$ touch txt{1,2,3,4}
$ ll
-rw-rw-r--  1 changxu changxu        0 2月  25 19:56 txt1
-rw-rw-r--  1 changxu changxu        0 2月  25 19:56 txt2
-rw-rw-r--  1 changxu changxu        0 2月  25 19:56 txt3
-rw-rw-r--  1 changxu changxu        0 2月  25 19:56 txt4
并不是创建了一堆.txt文件 ....mmp
```

### ls

> 列出文件和目录

```shell
# 列举当前目录的文件和目录 添加目录作为命令的选项可ls指定文件夹
# - l 长列表 a 显示隐藏文件 t 按时间排序 h 以K/M为单位显示文件大小（默认以字节为单位）
# - S 按文件大小排序，默认从大到小
$ ls
$ ls -l /etc
```

创建新文件的文件权限的默认值在umask中，这个命令暂时不学习

### chown

> 更改文件所有权

普通用户无法修改，需要是root用户

```shell
# 在root用户下或sudo命令 修改指定文件的拥有者，不过组里其他成员对文件的权限不变
$ chown username 目标文件所在路径

# 这样可连同群一起改变
$ chown username:group 目标文件所在路径
$ chown root:root 

# - R 递归修改指定文件夹及其子文件和子文件的拥有者和组
```

### mv,cp,rm

> 移动，复制，删除文件

```shell
# 将指定文件移动到指定路径中，默认如果指定路径下已存在同名文件，新文件会覆盖原文件
$ mv 指定文件 指定路径

# - i 如果有同名文件，会询问是否覆盖旧文件 r 递归操作
$ mv -i 指定文件 指定路径
mv：是否覆盖'f/1.txt'？

# cp 同理 注：如果指定的目录是文件，那么cp和mv会起到给文件重命名的作用

# rm 的-i也会在执行递归操作时给予提示
# rm - f 强制删除 r 递归删除
$ rm -rf /* # 快乐操作
```

## 使用文本文件

### vi基操

- 命令模式——Esc
- 插入模式——i



- locate——根据名称查找命令
- find——根据不同属性查找文件
- grep——在文本文件内部搜索包含搜索文本的行

```shell
# 使用locate命令前可能需要先安装
$ sudo apt install mlocate
```

### locate

```shell
# 这是一个使用例子，权限不同的人搜索的区域会有不同之处
changxu@ubuntu:~/Desktop$ locate 1.txt
/home/changxu/.mozilla/firefox/m8zlodj8.default-release/pkcs11.txt
/home/changxu/.pki/nssdb/pkcs11.txt
/home/changxu/.thunderbird/nf0dn0vd.default-release/pkcs11.txt
/home/changxu/Desktop/1.txt
/home/changxu/Desktop/f/1.txt
/usr/lib/firmware/brcm/brcmfmac4330-sdio.Prowise-PT301.txt
/usr/lib/firmware/brcm/brcmfmac43430-sdio.Hampoo-D2D3_Vi8A1.txt
/usr/share/doc/git/RelNotes/1.5.0.1.txt
/usr/share/doc/git/RelNotes/1.5.1.1.txt
/usr/share/doc/git/RelNotes/1.5.1.txt
/usr/share/doc/git/RelNotes/1.5.2.1.txt
/usr/share/doc/git/RelNotes/1.5.3.1.txt
/usr/share/doc/git/RelNotes/1.5.4.1.txt
...

# - i 忽略大小写
# locate会搜索 文件名或文件内容有 指定字符串的文件
# find 会搜索 文件名中有 指定字符串的文件
```

### find

```shell
# 查找当前文件夹下的所有子文件和文件夹，并列出来
$ find

# 查找指定路径下的子文件和子文件夹
$ find /etc/

# -ls 像ls一样把搜索结果和其他信息
$ find /etc/ -ls

# - name（大小写敏感） 和 iname(大小写不敏感) 选项后加文件名，根据文件名进行搜索，默认目录名不被搜索，支持文件名用*和?进行模糊搜索
$ find /etc/ -name passwd

# - size +/- n 搜索指定大小的文件，这里举例搜索指定文件夹下大于10M的文件
$ find /etc/ -size +10M

# find还能根据用户查找文件，根据日期查找文件，根据权限查找文件，使用'not'和'or'查找文件
# find的模糊搜索需要字符串在引号中，如 $ find / 'java*'
```

### grep

```shell
# grep默认区分大小写搜索
# grep 要搜索的内容 指定文件    不能是文件夹，搜索结果是含有指定搜索内容的文本
$ grep 'asd' ./1.txt

# - r 递归搜索 可以指定文件夹为搜索区域  搜索结果是含有指定搜索内容的文件所在路径 
# - i 不区分大小写搜索 v 查找不包含指定内容的结果
$ grep 'asd' 
```



## 管理运行中的进程

> ps, top, kill, jobs  用htop代替top

```shell
# 显示用户名和进程信息
$ ps u

# ux 当前用户所有进程 aux 所有用户的进程
$ ps aux | less

# - e o 输出指定内容
$ ps -eo pid,user,uid,group,gid,vsz | less
```

```shell
# 杀进程
$ kill 进程ID
```

跳过kill的可选参数，killall，nice，renice，cgroups



## Shell脚本

> [看这里](file:///D:/下载/python全套教程、手册/shell/运维和shell.html)



### 问题

- 单引号和双引号的区别（环境变量不能在单引号中，例如输出‘$PWD’会直接输出字符串'$PWD'，而不是其代表的值）



### Shell脚本能干啥

在反复执行某些命令时，可以提前将shell命令以纯文本形式存到文件里

以便以后执行

### 注释

```shell
# 注释以 # 开头
```

### 变量

- 本地变量
- 环境变量

```shell
# 设置一个本地变量，等号两边没有空格。此指令也可修改变量的值
$ VAR=value

# 将本地变量变为环境变量，或设置环境变量
$ export VAR
$ export VAR=value

# 删除本地变量或环境变量
$ unset VAR

# 声明静态变量，此变量不能被unset
$ readonly VAR=value

# 获取变量中的值，但是不会自动打印，这里需要使用echo辅助
$ echo $VAR
```

声明变量的规则：

1. 等号两边不能有空格。默认value都是字符串
2. 如果value需要有空格，可使用双引号或单引号把value括起来
3. 使用上述方式创建的变量，在服务器重启或关闭此次会话后会消失。想定义持久化的变量需要在`/`

### 命令替换

```shell
# 变量的值用``或$()包住，特指值是命令
$ DATE=`date`
$ echo $DATE
# 输出当前时间
Sun Mar 1 22:03:23 CST 2020
```

### 算数替换

```shell
# $(())包住的变量将作为整数进行计算，如果不包住，默认是字符串
$ VAR=3
$ echo $(($VAR+1))
```

### 转义字符

```shell
$ echo \$VAR
# 下面是输出结果
$VAR
```

### 条件测试

内容有点多，暂时不记录

...


## 管理系统

### 用户root权限

```shell
# 修改 /etc/sudoers 文件
```


### 获取和管理软件

```shell
# 更新软件包
$ apt update
# 更新已安装的软件包
$ apt upgrade
# 一条命令即可更新并下载最新软件包
$ sudo apt update && sudo apt -y upgrade
```

跳过其他部分（DEB，RMP，yum等）

### 获取用户账户

#### useradd

添加用户

```shell
# 具体选项看help，以下例子，- c 描述用户全名和用户登录名 m 创建用户的home目录
$ useradd -c 'changxu' cx -m
# 添加密码
$ passwd username
```

**查找用户**

```shell
# 可进入 /etc/passwd 或 /etc/shadow 查看用户
$ vi /etc/passwd
```

```shell
# 如下输出
# 用户名:密码:用户ID:主组ID:全名:家目录:默认使用的shell
root:x:0:0:root:/root:/bin/bash
...
cx:x:1001:1001:cx:/home/cx:/bin/sh
changxu:x:1002:1002:changxu:/home/changxu:/bin/sh
```


**查看用户所属的组**

```shell
$ vi /etc/group

root:x:0:
daemon:x:1:
bin:x:2:
...
```

#### usermod

和useradd差不多，具体使用见--help

#### userdel

```shell
# - r 删除用户同时删除家目录，其他选项键--help
$ userdel -r username
```

不加-r至善出用户在/etc/passwd等文件下该用户的记录，加-r删除家目录

但是不会删除其他与用户有关的文件

因此要删干净一个用户及其相关的所有文件，需要在删除用户前，找到与该用户所有相关文件

```shell
# 以下是根据用户名或用户ID搜索相关文件的命令，使用那个都行
$ find / -user username -ls
$ find / -uid userid -ls

# 这是搜索没有分配给任何用户的文件
$ find / -nouser -ls
```

**newgrp** 创建临时组，试了试不好用

#### groupadd

```shell
# 创建group01组，创建groupID为1111的group02组
$ groupadd group01
$ groupadd -g 1111 group02
```

#### groupmod

见--help

**跳过ACL（好像是管理企业中linux用户的）**


### 开放端口

```shell
# 检查指定端口是否开放，如果有返回信息代表开放，没有返回信息表示没有开放
$ lsof -i:8080

# 查看所有已开放的接口
$ netstat -nultp
```


## 管理磁盘和文件系统

跳过


## 服务器管理

### 启动ssh服务

1. 更新并下载软件包

```shell
$ apt update
$ pat upgrade
```

2. 安装ssh

```shell
$ apt install openssh-server
```

3. 查看ssh服务是否启动

```shell
$ ps -e | grep ssh
# 如果ssh服务没有启动，执行下启动命令
$ service ssh start
```

4. 查看ip地址

```shell
# 如果不能执行ifconfig命令就安装net-tools
$ apt install net-tools
$ ifconfig
```


## 启动和停止服务


- /sbin/iptables -I INPUT -p tcp --dport 6379 -j ACCEPT  开放指定端口的命令
- 找文件，设置开机就启动程序的脚本
- shell
- vim常用操作，删除整行，撤销到上一步，上下翻页，查找指定字符串，跳到指定行数
- 安装redis的时候，使用了maker   maker install  好像是用来生成命令的（额，希望我下次看的时候还记得是啥意思）
- 软连接，硬链接
- 通道 | 
- curl命令实现用命令行发送http请求

## 查端口，进程，服务的命令

> 这里包括查询指定端口号，进程，解压缩等命令

**ps** 查进程

```shell
# - e 显示所有进程 f 显示父进程的进程号
$ ps -ef

UID        PID  PPID  C STIME TTY          TIME CMD
root     26250     1  0 Mar22 ?        00:00:00 nginx: master process ./nginx
nobody   26334 26250  0 Mar22 ?        00:00:00 nginx: worker process
root     28720     1  0 Mar23 ?        00:02:51 /usr/lib/jvm/java-8-openjdk-amd64/bin/java -Dproc_namenode -Djava.net.preferI
root     28832     1  0 Mar23 ?        00:03:00 /usr/lib/jvm/java-8-openjdk-amd64/bin/java -Dproc_datanode -Djava.net.preferI
root     29105 20364  0 Mar19 pts/2    00:05:58 java -jar blog-img-upload.jar
```

**netstat** 查端口  输出结果附带服务名和进程号

```shell
# - t 显示tcp端口 u 显示udp端口 l显示套接字 p显示程序名称和进程标识符
$ netstat -tulpn | grep redis

Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name 
tcp        0      0 0.0.0.0:2181            0.0.0.0:*               LISTEN      4696/java           
tcp        0      0 0.0.0.0:9864            0.0.0.0:*               LISTEN      28832/java          
tcp6       0      0 [::]:mysql              [::]:*                  LISTEN      513/mysqld          
```

能显示程序名称，显示程序的进程号和端口号，显示地址（只能本机访问还是全网都能访问）


**** 解压缩

- *.tar 用 tar -xvf 解压
- *.gz 用 gzip -d或者gunzip 解压
- *.tar.gz和*.tgz 用 tar -xzf 解压
- *.bz2 用 bzip2 -d或者用bunzip2 解压
- *.tar.bz2用tar -xjf 解压
- *.Z 用 uncompress 解压
- *.tar.Z 用tar -xZf 解压
- *.rar 用 unrar e解压
- *.zip 用 unzip 解压


**df** 磁盘使用情况

```shell
# - h 修改显示文件大小的单位（友好的显示）
$ df -h
```


### telnet远程登录

```shell
# 
$ telnet 192.168.0.5 8080
```


## Screen命令

> 此命令用于创建一个新的shell，并绑定一个id和name。就像是将一个shell放进java中的map中一样。可通过id或name来get到shell

**安装**

```shell
$ apt install screen
```

**创建作业**

```shell
$ screen -S [Name]
```

**挂起screen**

`ctrl+a+d`

**查看screen列表**

```shell
$ screen -ls
```

**返回到指定的screen**

```shell
$ screen -r [Name/ID]
```

**删除指定screen**

```shell
$ screen -XS [Name/ID] quit
```


## 服务管理

`sysv-rc-conf`

设置开机自启服务的命令，`Ubuntu`中使用此命令替代了`chkconfig`

[参考](https://blog.csdn.net/u013554213/article/details/86584705)


## Tumx命令


## 新建ubuntu后要做的事

新建的ubuntu20系统，root用户的密码不是root

### 修改root用户的密码

1. sudo passwd
2. 输入当前用户密码
3. 输入要设置的root用户的密码
4. 二次确认root用户的密码

### 修改apt的下载源

1. 修改`/etc/apt/sources.list`（可实现复制一份以作备份）
2. 删除`sources.list`中所有内容，添加国内的镜像源
3. 忽略所有警告，输入`apt update`和`apt upgrade`

### 开启 ssh服务

1. 下载ssh`apt install openssh-server`

   实操时，报异常。缺少依赖软件包`openssh-client`和`openssh-sftp-server`

2. 安装缺少的依赖软件包

   `apt install openssh-client`。结果显示`openssh-client`已存在，所以命令没有生效。原因：所需要的和已有的软件包版本不一致
   
3. 安装指定版本的`openssh-client`。`apt install openssh-client=xxxxxx`

4. 安装`openssh-server`。`apt install openssh-server`

查看ssh服务是否已开启

````shell
$ ps -e | grep
````

有sshd的结果，表示ssh已开启

若ssh没有开启，就手动开启

```shell
$ /etc/init.d/ssh start
```

ssh的常用命令`servivce ssh status/restart/stop`


### 设置root用户开启远程登录

1. /etc/ssh/sshd_config，找到PermitRootLogin，将后面的字符串修改为yes
2. 重启ssh服务


## 脚本

群起脚本

xcall

xsync