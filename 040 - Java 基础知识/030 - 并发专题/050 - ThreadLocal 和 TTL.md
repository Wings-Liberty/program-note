# 能干什么

- 给线程设置**一个**局部变量（线程级别的变量）

- 不同线程中的**线程的局部变量**互不影响

![[../../020 - 附件文件夹/Pasted image 20230328234950.png|575]]

# 怎么用

## 常用的方法

除构造方法外，`ThreadLocal` 对外暴露的方法只有下面这几个

除构造方法外，

| 方法头                   | 描述                            |
| ------------------------ | ------------------------------- |
| public ThreadLocal()     |ThreadLocal 只有一个空参构造方法|
| public void set(T value) |set 一个线程级别的变量|
| public T get()           | 获取**当前线程**的变量          |
| public void remove()     | 移除**当前线程**的变量          |

```java
class User {

    private String name;

    private ThreadLocal<String> t = new ThreadLocal<>();

    public User() {
    }

    public String getName() {
//        return name;
        return t.get();
    }

    public void setName(String name) {
        this.name = name;
        t.set(name);
    }

}

public class Test {

    public static void main(String[] args) {
        User user = new User();
        for (int i = 0; i < 100; i++) {
            new Thread(
                    ()->{
                        user.setName(Thread.currentThread().getName() + "的name");
                        try {
                            Thread.sleep(3);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println(Thread.currentThread().getName() + "获取到" + user.getName());
                    },
            i+ ""
            ).start();
        }
    }

}
```


# 和 synchronized 的区别

`synchronized` 牺牲时间换空间

`ThreadLocal` 牺牲空间换时间


同一时间只能有一个线程执行 `synchronized` 中的代码，当此线程执行完毕后其他线程才能执行 `synchronized` 中的代码

`ThreadLocal` 使得每一个线程都有一片独立的空间存储变量，不同线程可同时访问自己的变量，因为这些线程级别的变量在不同线程中互不影响


# ThreadLocal 的内部结构

`Thread` 中有两个成员变量

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
```


`ThreadLocal.ThreadLocalMap` 是 `ThreadLocal` 的静态内部类

- 每个 `Thread` 中都有一个 map（ `ThreadLocalMap` ）
- `ThreadLocalMap` 将变量都放到 `private Entry[] table;` 中
- `ThreadLocalMap` 中 key 是 `ThreadLocal`的对象，value是线程存的值
- `ThreadLocalMap` 中的数据由 `ThreadLocal` 对象操作


## set 方法

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    // 获取成员对象的成员变量threadLocals
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

// 获取到t的成员变量
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

// 为线程创建成员变量
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```


## get 方法

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

protected T initialValue() {
    return null;
}
```


## remove

```java
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```


## ThreadLocalMap

- 成员变量

  ```java
  private Entry[] table;
  ```

- `Entry` 类

![[../../020 - 附件文件夹/Pasted image 20230328235127.png|250]]

ps：`WeakReference` 弱引用

# 弱引用和内存泄漏

有人说 `ThreadLocal` 在使用时会出现内存泄漏的问题

接下来在讲解完内存泄漏和弱引用的相关概念后聊聊 `ThreadLocal` 是不是会导致内存泄漏


## 内存泄露

- Memory overflow：内存溢出。不能申请到内存来使用
- Memory leak：内存泄漏。程序中动态分配的堆内存由于某种原因导致未释放或无法释放内存，造成系统内存的浪费，也会导致内存溢出


## 弱引用相关概念

java 中由四种引用。强，软，弱，虚引用

**强引用**：就是我们最常见的普通对象的引用，只要还有奇纳国引用指向一个对象，就能表明对象还“活着”，垃圾回收器就不会回收这种对象

**弱引用**：垃圾回收器一旦发现只具有弱引用的对象，不管当亲啊内存空间足够与否，都会回收它的内存


## 问题分析

在使用完 `ThreadLocal` 并且此方法栈弹出后，`ThreadLocal` 的对象会保存在某个线程的 `ThreadLocalMap` 对象的 `Entry[] table` 的 key 中

如果 `ThreadLocal` 是强引用，那么它会因为存在于 `Entry` 的 key 中，如果线程没死，存放此对象的 `Entry` 没被删，那么这个对象将一直存在

如果 `ThreadLocal` 是弱引用，在方法栈弹出后，此对象将会因为没有强引用的指向而被回收，但是 `Entry` 中的 value 还存在，`Entry` 不会被自动清除而导致内存泄漏

![[../../020 - 附件文件夹/Pasted image 20230328235303.png|775]]

# Entry[]中的哈希冲突

先上一个 `ThreadLocalMap` 构造方法

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```

其中有关计算 hash 值的代码为

```java
int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
```

它涉及到了哪些方法？

```java
private final int threadLocalHashCode = nextHashCode();

private static int nextHashCode() {
    // 根据方法名的翻译：获取并增加
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}

// AtomicInteger是一个提供原子操作的Integer类，通过线程安全的方式操纵加减，适合高并发情况下使用。此类源代码涉及到了Unsafe类（有兴趣的可以探究一下）
private static AtomicInteger nextHashCode = new AtomicInteger();

// 此变量的作用是，在计算hash值时，令hash值能尽可能均匀分布，减少hash冲突的概率。此变量的取值和黄金分割有关
private static final int HASH_INCREMENT = 0x61c88647;
```

再看看 set 方法

```java
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    // 计算hash值
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}

private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
```


# TTL

#待补充 