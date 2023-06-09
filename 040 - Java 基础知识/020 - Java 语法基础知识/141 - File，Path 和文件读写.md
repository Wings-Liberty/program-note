
# File 类

`File` 对象既可以表示文件，也可以表示目录

特别要注意的是，构造一个 `File` 对象，即使传入的文件或目录不存在，代码也不会出错，因为构造一个 `File` 对象，并不会导致任何磁盘操作。只有当我们调用 `File` 对象的某些方法的时候，才真正进行磁盘操作

用构造器创建 `File` 对象需要传入文件的相对或绝对路径

```java
File file = new File(path);
```

> [!NOTE] 相对路径
> 用相对路径创建 File 对象时，使用的相对路径默认是 JVM 运行时的根目录
> 
> 具体来说是 JVM 运行时的 `user.dir` 变量的值

对 File 对象的操作可分为：访问 file **物理文件**的元数据，访问 file **物理文件**的内容，管理**文件对象**

![[../../020 - 附件文件夹/Drawing 2022-06-09 13.54.13.excalidraw|700]]

但首先需要**检查 file 物理文件是否存在** `file.exists()`，或直接调用 `file.createNewFile()` 进行如果不存在就创建新文件


> [!NOTE] 文件和目录的区别
> - 用 File 读取文件指读内容，读目录指获取目录中的 File 对象
> - 用 File 创建文件和创建目录的 api 是不一样的


# File 对象的文件路径

文件路径可以是绝对路径或相对路径，但定位文件地址本质上一定是绝对路径

文件路径 = 根目录 + 路径 + 文件分隔符

Linux 的根目录为 '/'，Windows 的根目录为 '盘符号://'


File 的文件路径分为 3 种：相对路径 `getPath`，绝对路径 `getAbsolutePath`，规范路径 `getCanonicalPath`


> [!NOTE] 规范路径是没有带多余的 '.' 或 '..' 的绝对路径
> 比如："/dir1/./dir2/../fil1.txt" 是绝对路径，"/dir1/file1.txt" 是规范路径



# Path 类

定义：`Path` 对象表示一个目录名序列，用于操作文件路径

这个类能很方便地处理路径字符串，但它包含的路径是抽象的名字序列，**路径可以不必对应某个实际存在的文件**

其行为包含：创建 Path 对象，规范化，获取相对、绝对路径。

```java
Path absolute = Paths.get("/home", "harry"); // 得到绝对路径 /home/harry
Path relative = Paths.get("myprog","conf","user.properties"); // 得到相对路径 myprog/conf/user.properties 
```

还包含了解析相对路径和绝对路径的方法

```java
/**
 * 如果 q 是绝对路径，返回值就是 q 对象
 * 如果 q 是相对路径，返回值就是 (p + 文件分割符 + q) 的Path对象
**/
p.resolve(q);
```

```java
// 如果 p 是 /opt/myapp/work  返回 /opt/myapp/temp 的 Path 对象
// 1. 如果 p 没有父路径或 other（方法参数）是绝对路径，方法就返回 other 的 Path 对象
// 2. 如果 other 是空，就返回 p 的父路径。如果它不仅是空，父路径也是空，返回空路径  空——""
Path tempPath = p.resolveSibling("temp"); // /opt/myapp/temp
```

```java
// 例子。如果 p 是"/home/cay"，r 是 "/home/fred/myprog"
Path res = p.relativize(r); // "../fred/myprog"  结果是在 p 的路径下如何访问到 r 的路径
```

```java

Path getParent(); // 获取父路径的 Path 对象。没有父路径就返回 null

Path getFileName(); // 获取只包含文件名的 Path 对象（去掉原路径中的所有前缀，只剩下最后一个文件分隔符后的字符串）

Path getRoot(); // 获取调用者的根路径。如果调用者没有根路径就返回 null。如果调用者是相对路径，默认是没有根路径的

Path toAbsolutePath(); // 获取调用者的绝对路径 Path 对象。如果调用者已经是绝对路径返回原对象，否则尝试获取绝对路径，如果获取不到就抛错

Path normalize(); // 规范化

```

还有 `startWith`，`endWith` 等方法（顾名思义）


# File 和 Path 的双向获取

```java
// File 转 Path
file.toPath();
```

```java
// Path 转 File
path.toFile();
```


# Paths 工具类

仅提供 `get` 方法，用户创建 Path 对象

```java
public static Path get(String first, String... more);
```


# Files 工具类

`Files` 提供了快速读写，复制，删除，移动，检查文件是否存在，获取文件元数据等 API


> [!NOTE] Files 不适用于读写大文件
> `Files` 提供的读写方法，受内存限制，只能读写小文件，例如配置文件等，不可一次读入几个G的大文件。读写大型文件仍然要使用文件流，每次只读写一部分文件内容

