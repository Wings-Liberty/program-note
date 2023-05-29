
## 它可以跳到文件的任意位置读写数据

`RandomAccessFile` 是 `java.io` 体系中功能最丰富的文件内容访问类。可以读写文件内容。但是和传统 IO 流不同的是，`RandomAccessFile` 可以自由访问文件的任意位置，可以直接跳到文件的任意位置来读写数据

`RandomAccessFile` 维护一个指针，它指向了当前读写的位置，就像在文本编辑器里的光标一样。它提供了两个控制指针的方法


| 方法                    | 含义                                                                   |
|:----------------------- |:---------------------------------------------------------------------- |
| `long getFilePointer()` | 获取指针在文件里表示的偏移量，单位是字节，偏移量指向下一个被读写的字节 |
| `void seek(long pos)`   |让指针指向指定的偏移量|

## 它提供了丰富的 API 读写数据

创建 `RandomAccessFile` 对象需要传入文件地址或对象，还需要指定打开文件的方式

|模式|解释|
|:--|:--|
|r|只读。如果调用写 API 就会报错|
|rw|读写。如果文件不存在就自动创建文件|
|rwd|读写。相对于 rw 模式，还要求同步写文件内容数据到底层设备|
|rws|读写。相对于 rw 模式，还要求同步写文件内容数据和元数据到底层设备|

然后提供了 `read()` 和 `write()` 方法，可以像 InputStream / OutputStream 一样读写数

还提供了 `readXxx()` 和 `writeXxx()` 方法，便于读写结构化数据。除此以外就没有其他杂七杂八的方法了


## 从任意位置读数据

默认文件指针的偏移量是 0，可以用 `seek()` 移动指针，只要指针不小于 0 就不会报错（指针的便宜量超过文件字节数后不会报错，只是读不出数据了而已）

```java
@Test  
public void testReadByte() throws IOException {  
    RandomAccessFile raf = new RandomAccessFile("src/readme.txt", "r");  
    System.out.println("file pointer at " + raf.getFilePointer());  
    System.out.println("file byte count is " + raf.length());  
    raf.seek(5L);
    byte[] data = new byte[1024];  
    int hasRead = 0;  
    System.out.println(raf.readLine());  
    while ((hasRead = raf.read(data)) != -1) {  
        System.out.println(new String(data, 0, hasRead, StandardCharsets.UTF_8));  
    }  
}
```

还提供了读各种基本类型，字符，一整行字符串的 API

![[../../020 - 附件文件夹/Pasted image 20230526232756.png|425]]

## 在文件里写数据

**在文件末尾追加数据**很简单，让偏移量跳到文件末尾，然后调用各种 `write` 方法

```java
RandomAccessFile raf = new RandomAccessFile("src/readme.txt", "rw");
raf.seek(raf.length());  
raf.writeUTF("This is append text.\n");
```

**在文件任意位置写数据**不是在任意位置插入数据，在已经存在数据的地方写数据会覆盖原有数据，然后指针向后移动

如果像在指定位置插入数据，就需要像在 `ArrayList` 的中间位置插入数据一样，需要手动先把指定位置之后的数据放到其他地方，然后把新数据放到末尾，再把搬出来的数据放到新数据后

```java
// 目的：在指定位置 (pos) 后插入数据
raf.seek(pos);
// ------下面代码将插入点后的内容读入临时文件中保存-----
byte[] bbuf = new byte[64];
// 用于保存实际读取的字节数据
int hasRead = 0;
// 用循环读取插入点后的数据
while ((hasRead = raf.read(bbuf)) != -1) {
  //将读取的内容写入临时文件
  fileOutputStream.write(bbuf, 0, hasRead);
}
// -----下面代码用于插入内容 -----
// 把文件记录指针重新定位到pos位置
raf.seek(pos);
// 追加需要插入的内容
raf.write(insertContent.getBytes());
// 追加临时文件中的内容
while ((hasRead = fileInputStream.read(bbuf)) != -1) {
  //将读取的内容写入临时文件
  raf.write(bbuf, 0, hasRead);
}
```

写数据的 API 

> [!quote] 参考
> - [[../../021 - 离线网页备份文件夹/1.RandomAccessFile特点 - 萌哥-爱学习 - 博客园 (2023_5_26 22_40_34).html|RandomAccessFile类使用详解]]
