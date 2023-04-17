#还没有复习 

> [!note] Java IO 的三种读写方式
> 这里说的不是 BIO，NIO，AIO。而是在说操作系统上读写数据的方式
> -   传统 IO，字节流，由 java.io 包提供
> -   缓冲 IO，FileChannel，通过 ByteBuffer 实现更快的 IO
> -   零拷贝 IO，mmap，sendfile
 

传统 IO 指普通的输入输出流

FileChannel 和传统 IO 相比，并没有减少读写文件时的数据拷贝次数或 CPU 状态切换次数，但能通过缓存加快速度

关于零拷贝的理论知识见[[../05 - 计算机基础/03 - 操作系统/零拷贝|零拷贝]]


Java 的 java.nio 包实现了 mmap 和 sendfile 的 API，其底层是用 JNI 调用了 mmap 和 sendfile 系统调用

# mmap

```java
RandomAccessFile raf = new RandomAccessFile(File, "rw");

FileChannel channel = raf.getChannel();

MappedByteBuffer buff = channel.map(FileChannel.MapMode.READ_WRITE, startAddr, SIZE);

buff.put((byte)255);
```


> [!warning] mmap 的使用误区
> 如果经常需要把 MappedByteBuffer 中的数据读到 Java 的多个变量里，那么用  mmap 和不用都是一样的，因为在内核空间的很多数据又被读到了在用户空间的 Java 程序的变量里


# sendfile

```java
RandomAccessFile raf = new RandomAccessFile(File, "rw");

FileChannel channel = raf.getChannel();

fileChannel.transferTo(position, count, socketChannel);
```



> [!NOTE] Java 的 mmap 和 sendfile 都由 FileChannel 提供



> [!quote]+ 参考
> 
> [Java文件映射[mmap]全接触](https://site.douban.com/161134/widget/articles/8506170/article/18487141/)
> 
> [重新认识 Java 中的内存映射（mmap）](https://cloud.tencent.com/developer/article/1902272)
> 
> [Java网络编程与NIO详解8：浅析mmap和Direct Buffer](https://juejin.cn/post/6844903993102041102)

# 不能用 mmap 代替数据库

mmap 只是一个系统调用函数，只能完成基本的工作，不提供日志，事务，回滚等功能。

所以不能用 mmap 代替数据库。但可以考虑用 mmap 实现以文件为主的持久化方案，比如 Kafka，RocketMQ 都用到了 MappedByteBuffer

> RocketMQ 的持久化文件最大，且仅能为 2G
> 
> 因为 mmap 最大只能支持 2G 文件的映射
> 
> 原因是 mmap 函数用一个 int 变量保存内存地址，int 只能实现 2G 内存空间的寻值


# RandomAccessFile

文件被 RandomAccessFile 打开后，程序就能像控制 byte 数组一样读写文件

RandomAccessFile 维护一个指针，指向将要操作的字节的地址。调用读写 API 时就从这个位置开始

```java
RandomAccessFile file = new RandomAccessFile(file_name, "rw");

file.seek(file.length()); // 重新设置文件指针

file.write(content); // 在文件末尾追加内容

```



> [!warning] 选择性使用 RandomAccessFile
> RandomAccessFile 用的是传统 IO 方式，根据实际情况，使用零拷贝方式访问文件数据可能会更好