#还没有复习 

> 环境：jdk8
>
> 涉及知识：数据结构&算法：红黑树（HashMap用到了红黑树，但是此篇博客重在讲HashMap的put方法的执行流程，对于红黑树的细节此篇不介绍）



# HashMap简介

	HashMap基于哈希表的Map接口实现，以key-value形式存储数据
	HashMap它是线程不安全的。
	它的key、value都可以为null，但是key只能有一个为null。
	此外，HashMap中的映射不是有序的。



`HashMap`的数据结构：数组 + 链表 + 红黑树



# HashMap的数据结构

在JDK1.8 之后 HashMap 由 **数组+链表 +红黑树**数据结构组成的。

![[../../020 - 附件文件夹/Pasted image 20230401232701.png|500]]


```java
// 存放Node的数组
transient Node<K,V>[] table;

// Node是存放键值对的节点，也是链表的节点。Node是HaahMap的内部类
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;	// 键
    V value;		// 值
    Node<K,V> next; // 链表中此节点的下一个节点

    // 为节省篇幅，不展示成员方法
}
```

```java
// 红黑树的节点，LinkedHashMap.Entry继承了HashMap.Node，所以TreeNode也是Node，
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // 父
    TreeNode<K,V> left;    // 左
    TreeNode<K,V> right;   // 右
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;           // 判断颜色
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
    // 返回根节点
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
}
```



# HashMap存储数据的粗略过程



在上述的数据结构的基础上，下文讲围绕`put`方法展开对`HashMap`的分析



## 插入一对键值对



```java
public class Demo {
    public static void main(String[] args) {
        HashMap<String, Integer> map = new HashMap<>();
        map.put("tom", 53);
    }
}
```



1. 创建`HashMap`对象，默认`table`数组的大小是16
2. 根据`map.put("tom", 53);`中传递的key（这里的键是"tom"），执行方法计算key的hash值，最后讲hash值的大小控制在0~15（因为数组大小是16），以此hash值作为下标，将 "tom", 53 封装到`Node`对象并保存在table数组中

当使用`HashMap`的`get`方法时，计算`get`方法传来的参数的hash值（hash大小为0~n-1，n为当前table数组的大小），如果table数组中tablep[hash]中能找到key相同的数据，就返回结果。



上述流程中，只考虑了向新建的`HashMap`对象中put一个数据。熟悉了简单的流程后，下面开始讲述详细的流程

1. put一对键值对，结果key的hash值和之前的键值对中key的hash相同，怎么办？
2. `HashMap`不是用到了红黑树吗？红黑树在哪里？
3. table数组的大小只有16，那么它是怎么自动扩容的？



## 插入多对键值对



~~~java
public class Demo {
    public static void main(String[] args) {
        HashMap<String, Integer> map = new HashMap<>();
        map.put("刘德华", 53);
        map.put("柳岩", 35);
        map.put("张学友", 55);
        map.put("郭富城", 52);
        map.put("黎明", 51);
        map.put("林青霞", 55);
        map.put("刘德华", 50);
    }
}
~~~

![[../../020 - 附件文件夹/Pasted image 20230401232724.png|500]]

# HashMap继承关系

HashMap继承关系如下图所示：



说明：

- `Cloneable` 空接口，表示可以克隆。 创建并返回HashMap对象的一个副本。
- `Serializable` 序列化接口。属于标记性接口。HashMap对象可以被序列化和反序列化。
- `AbstractMap` 父类提供了Map实现接口。以最大限度地减少实现此接口所需的工作。

上述继承关系我们发现一个很奇怪的现象， 就是HashMap已经继承了`AbstractMap`而`AbstractMap`类实现了Map接口，那为什么`HashMap`还要在实现`Map`接口呢？同样在`ArrayList`中`LinkedList`中都是这种结构。 



答：据 java 集合框架的创始人Josh Bloch描述，这样的写法是一个失误。在java集合框架中，类似这样的写法很多，最开始写java集合框架的时候，他认为这样写，在某些地方可能是有价值的，直到他意识到错了。显然的，JDK的维护者，后来不认为这个小小的失误值得去修改，所以就这样存在下来了。



# HashMap的成员成员和成员方法



table数组中的`Node`对象可能会指向下一个`Node`对象，也就是说table[i] 中可以保存多个`Node`对象

因此，table数组的每个元素都是一个 桶(bucket)



## 成员变量

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L;    
    // table数组默认的初始容量是16。   数组大小必须是2的次幂，如果创建对象时自定义的大小不是2的次幂，HashMap也会将其转换为2的次幂
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    // table数组最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认的负载因子(强烈建议不要修改)  如果table数组中有 75%（负载因子）的下标被占用，就扩容。负载因子是map是否需要扩容的参考指标
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时，桶中的链表会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 当桶(bucket)上的结点数小于这个值时桶中的树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小 （用于解决是扩容还是树形化的选择）
    // 当Map里面的数量超过这个值时，表中的桶才能进行树形化，否则桶元素太多时会扩容，而不是树形化。为了避免进行扩容，还是树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD  （此变量默认值为8）
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table;
    // 存放具体元素的集合
    transient Set<map.entry<k,v>> entrySet;
    // 元素的个数，不等于数组的长度/桶的数量。
    transient int size;
    // 记录扩容和更改map结构次数的计数器
    transient int modCount;   
    // 临界值（也称阈值） 当节点数量超过临界值(容量*负载因子)时，会进行桶的扩容
    int threshold;
    // 加载因子（也称负载因子），默认负载因子就是一个存默认值的变量，实际使用的负载因子是这个变量
    final float loadFactor;
}
```



### DEFAULT_INITIAL_CAPACITY初始大小

集合的初始化容量( **必须是二的n次幂** )

~~~java
// 默认的初始容量是16 -- 1<<4相当于1*2的4次方---1*16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
~~~

问题： 为什么必须是2的n次幂？如果输入值不是2的幂比如10会怎么样？

HashMap构造方法还可以指定集合的初始化容量大小：

~~~java
HashMap(int initialCapacity) // 构造一个带指定初始容量和默认加载因子 (0.75) 的空 HashMap。
~~~



`tableSizeFor(int cap)`方法将传入的cap变成2的次幂

~~~java
// 创建HashMap集合的对象，指定数组长度是10，不是2的幂
HashMap hashMap = new HashMap(10);
public HashMap(int initialCapacity) {//initialCapacity=10
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
public HashMap(int initialCapacity, float loadFactor) {//initialCapacity=10
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    // 阈值 = 容量*负载因子，但是这里直接将计算出的容量赋给了阈值。其原因到构造方法中再说
    this.threshold = tableSizeFor(initialCapacity);
}

// 把输入值变成2的次幂
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
~~~



## 构造方法

 HashMap 中重要的构造方法，它们分别如下： 

1、构造一个空的 `HashMap` ，默认初始容量（16）和默认负载因子（0.75）。 

~~~java
public HashMap() {
   this.loadFactor = DEFAULT_LOAD_FACTOR; // 将默认的加载因子0.75赋值给loadFactor，并没有创建数组（第一次调用put时再创建数组）
}
~~~

2、 构造一个具有指定的初始容量和默认负载因子（0.75） `HashMap`。 

~~~java
// 指定table数组大小的构造函数
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR); //重载
}
~~~

3、传入另一个Map

```java
// 包含另一个“Map”的构造函数
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false); // 下面会分析到这个方法
}
```

4、 构造一个具有指定的初始容量和负载因子的 `HashMap`。 

### HashMap(initialCapacity,loadFactor)

~~~java
public HashMap(int initialCapacity, float loadFactor) {
    // 判断初始化容量initialCapacity是否小于0
    if (initialCapacity < 0)
        // 如果小于0，则抛出非法的参数异常IllegalArgumentException
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    // 判断初始化容量initialCapacity是否大于集合的最大容量MAXIMUM_CAPACITY——2的30次幂
    if (initialCapacity > MAXIMUM_CAPACITY)
        // 如果超过MAXIMUM_CAPACITY，会将MAXIMUM_CAPACITY赋值给initialCapacity
        initialCapacity = MAXIMUM_CAPACITY;
    // 判断负载因子loadFactor是否小于等于0或者是否是一个非数值
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        // 如果满足上述其中之一，则抛出非法的参数异常IllegalArgumentException
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    
    // 将指定的加载因子赋值给HashMap成员变量的负载因子loadFactor
    this.loadFactor = loadFactor;
    /*
    		tableSizeFor(initialCapacity) 判断指定的初始化容量是否是2的n次幂，如果不是那么会变为比指定初始化容量大的最小的2的n次幂。这点上述已经讲解过。
    		但是注意，在tableSizeFor方法体内部将计算后的数据返回给调用这里了，并且直接赋值给threshold边界值了。有些人会觉得这里是一个bug,应该这样书写：
    		this.threshold = tableSizeFor(initialCapacity) * this.loadFactor;
    		这样才符合threshold的意思（当HashMap的size到达threshold这个阈值时会扩容）。
			但是，请注意，在jdk8以后的构造方法中，并没有对table这个成员变量进行初始化，table的初始化被推迟到了put方法中，在put方法中会对threshold重新计算，put方法的具体实现我们下面会进行讲解
    	*/
    this.threshold = tableSizeFor(initialCapacity);
}

~~~





## 成员方法

#### 计算索引思路：

由key到hashCode：key有个key.hashCode()方法

由hashCode()得到hash：利用hash()函数计算出hash

hash值得到坐标：下标 i = `(length - 1) & hash`;

#### hash()

这个哈希方法首先计算出key的hashCode赋值给h,然后与h无符号右移16位后的二进制进行按位异或得到最后的hash值。计算过程如下所示：

```java
static final int hash(Object key) {
    int h;
    /*  
    	1. 如果key等于null：null也是有哈希值的，返回的是0.
     	2. 如果key不等于null：首先计算出key的hashCode赋值给h,然后与h【无符号右移16位】后的二进制进行【按位异或】得到最后的hash值
    */
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

从上面可以得知HashMap是支持Key为空的，而HashTable是直接用Key来获取HashCode所以key为空会抛异常。



在putVal函数中使用到了上述hash函数计算的哈希值：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    ...
    // 判断tble[i] 是否为null
	if ((p = tab[i = (n - 1) & hash]) == null)
    ...
  }
```



#### put()

```java
public V put(K key, V value) {
    // final V putVal()方法缺省权限修饰符，对外不可见，用户无法调用，只能通过put()
    return putVal(hash(key), key, value, false, true);
}
```

#### putVal()

主要参数：

- hash: key的hash值
- key: 原始Key
- value: 要存放的值
- onlyIfAbsent: true代表只插入新值，不修改
- evict: 如果为false表示table为创建状态

putVal()方法流程：




```java
//JDK8
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
}
//JDK8
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    /*
    	4）执行完n = (tab = resize()).length，数组tab每个空间都是null
    */
   	// 如果table数组为空，或数组长度为0。那么为table数组扩容，并将新的table数组赋值给tab，数组大小赋值给n
    if ((tab = table) == null || (n = tab.length) == 0)// Node数组为空的话就resize
        n = (tab = resize()).length;
    
    // 将table[i]赋值给p，并判断tbale[i]是否为空（i = (n - 1) & hash）
    if ((p = tab[i = (n - 1) & hash]) == null)//该索引上没有Node
        // table[i]为空，直接插入键值对
        tab[i] = newNode(hash, key, value, null);//还不return，一会做些记录后再return
    else {
        // tab[i]!=null，表示这个位置已经有值了（哈希冲突，两个key的hash最终计算出的下标相同）
        
        Node<K,V> e; K k; // e标识要修改的那个结点，先查出来那个结点（不存在就创建），最后再改
        /*
        	比较桶中第一个元素(数组中的结点)的hash值和key是否相等
        	1）p.hash == hash ：p.hash表示原来存在数据的hash值,hash表示后添加数据的hash值,比较两个hash值是否相等。不等就直接跳过if
             2）(k = p.key) == key ：p.key获取原来数据的key赋值给k,key表示后添加数据的key,比较两个key的地址值是否相等
             3）key != null && key.equals(k)：能够执行到这里说明两个key的地址值不相等,那么先判断后添加的key是否等于null,如果不等于null再调用equals方法判断两个key的内容是否相等
             //把首节点与后面的for分离是为了拿到
        */
        // 比较桶中第一个元素(数组中的结点)的hash值和key的hash值是否相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k)))){ 
                e = p; // 将旧的元素整体对象赋值给e，用e来记录
    	}else if (p instanceof TreeNode)// key不同；判断p是否为红黑树结点
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {// 不是红黑树，说明是链表节点
            // 放链表中
            for (int binCount = 0; ; ++binCount) {
                // 如果p没有下一个节点了，将新键值对插入到p的下一个节点
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 如果当前桶中链表的节点数达到TREEIFY_THRESHOLD（树形化阈值），会尝试将链表树形化
                    if (binCount >= TREEIFY_THRESHOLD - 1) 
                        //treeifyBin()里不一定会把链表转成红黑树。如果table数组长度<64，会扩容；如果数组长度>=，会属性话
                        treeifyBin(tab, hash);
                    break;
                }
                
                /*
                	执行到这里说明(e = p.next)!=null，且不是最后一个元素。继续判断链表中结点的key值与插入的元素的key值是否相等
                */
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 新键值对和当前table[i]中的某个节点的key相等，跳出循环
                    break;
                /*
                	说明新添加的元素和当前节点不相等，继续查找下一个节点。
                	e 的值为 p.next组合，所以这里将e赋值给p
                */
                p = e;
            }
        }
        /*
        	表示在桶中找到key值、hash值与插入元素相等的结点
        	也就是说通过上面的操作找到了重复的键，所以这里就是把该键的值变为新的值，并返回旧值
        	这里完成了put方法的修改功能
        */
        if (e != null) { // e==null代表遍历到最后一个节点后还没找到key相同的节点。如果e!=null代表在table[i]中找到了key相同的节点
            // 此时e是table[i]中key值和新键值对的key相同的节点
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false（默认）或者旧值为null
            if (!onlyIfAbsent || oldValue == null)//这里说明旧值可以为null
                //e.value 表示旧值  value表示新值 
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    ++modCount; // 添加记录次数+1
    // 判断实际大小是否大于threshold阈值，如果超过则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
} 
```





####  resize()扩容

~~~java
// JDK8
final Node<K,V>[] resize() {
    //得到当前数组
    Node<K,V>[] oldTab = table;
    
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    
    int oldThr = threshold;
    // HashMap构造函数中有这么一句:this.threshold = tableSizeFor(initialCapacity);
    // 即第一次扩容(包括0->16)前阈值==容量
    int newCap, newThr = 0; // 新的容量和阈值
    // 如果老的数组长度大于0，开始计算扩容后的大小
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你哈希碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        /*
        	没超过最大值，就扩充为原来的2倍
        	1. (newCap = oldCap << 1) < MAXIMUM_CAPACITY 扩大到2倍之后容量要小于最大容量
        	2. oldCap >= DEFAULT_INITIAL_CAPACITY 原数组长度大于等于数组初始化长度16
        */
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 阈值扩大一倍
            newThr = oldThr << 1;
    }
    else if (oldThr > 0)
        newCap = oldThr;
    else {// oldCap==0 && oldThr==0,直接使用默认值16与0.75
        newCap = DEFAULT_INITIAL_CAPACITY; // 16
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); // 重新计算阈值16*0.75
    }
    // 计算新的resize最大上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //新的阈值 默认原来是12 乘以2之后变为24
    threshold = newThr;
    
    // 创建新的table数组，并为旧table数组中所有节点的key重新计算hash值
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];//创建空Node数组
    table = newTab;
    //判断旧数组是否等于空
    if (oldTab != null) { // 原来有值则复制到新Node数组里
        // 把每个bucket都移动到新的buckets中
        // 遍历旧的哈希表的每个桶，重新计算桶里元素的新位置
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                // 原来的数据赋值为null 便于GC回收
                oldTab[j] = null;
                // 如果数组当前位置只有一个元素
                if (e.next == null)
                    // 没有下一个引用，说明不是链表，当前桶上只有一个键值对，直接插入
                    newTab[e.hash & (newCap - 1)] = e;
                // 判断是否是红黑树
                else if (e instanceof TreeNode)
                    // 说明是红黑树来处理冲突的，则调用相关方法把树分开
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // 采用链表处理冲突
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 通过上述讲解的原理来计算节点的新位置
                    do {
                        // 原索引 //记录下个结点
                        next = e.next;
                     	// 这里来判断如果等于true e这个节点在resize之后不需要移动位置
                        if ((e.hash & oldCap) == 0) {// hash最高位为1
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;//更新尾节点
                        }
                        // 原索引+oldCap
                        else {//hash最高位为1
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到索引为j的bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到索引为j+oldCap的bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
~~~



##### resize时线程安全问题

为什么线程不安全：https://blog.csdn.net/loveliness_peri/article/details/81092360	





# 遍历HashMap的几种方法

- 遍历kv键值对

  `map.keySet()`获取key的set集合

  `map.values()`获取value的集合

- 使用迭代器

  `map.entrySet()`获取节点的set集合，节点中有key和value。使用节点的`.getKey()`获取key，`.getValue()`获取value

- for/while循环使用get方法（效率低，阿里手册不建议使用）

  `map.keySet()`获取key的set集合，使用循环执行`.get()`方法获取

- 流式编程，使用foreach

```java
Set<String> keys = map.keySet();
for (String key : keys) {
    System.out.print(key+"  ");
}

Collection<String> values = map.values();
for (String value : values) {
    System.out.print(value+"  ");
}

Set<java.util.Map.Entry<String, String>> entrys = map.entrySet();
for (java.util.Map.Entry<String, String> entry : entrys) {
    System.out.println(entry.getKey() + "--" + entry.getValue());
}
```

