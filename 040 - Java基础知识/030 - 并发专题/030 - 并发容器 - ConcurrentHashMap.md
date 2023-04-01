#还没有复习 

`ConcurrentHashMap`的`put`和`get`操作[参考](https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/)





# 一、jdk1.8容器初始化

## 1. 源码分析

> 在`jdk8`的`ConcurrentHashMap`中一共有5个构造方法，这四个构造方法中都没有对内部的数组做初始化，只是对—些变量的初始值做了处理
>
> `jdk8`的`ConcurrentHashMap`的数组初始化是在第一次添加元素时完成



```java
// 没有维护任何变量的操作，如果调用该方法，数组长度默认是16
public ConcurrentHashMap() {
}
```

```java
// 传递进来一个初始容量，ConcurrentHashMap会基于这个值计算一个比这个值大的2的幂次方数作为初始容量
public ConcurrentHashMap(int initialcapacity) {
	if (initialCapacity < 0)
		throw new IllegalArgumentException();
	int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>>1))?
		MAXINAUM_CAPACITY :
		tableSizeFor(initialcapacity + (initialCapacity >1) + 1));
	this.sizeCtl = cap;
}
```

> 注意，调用这个方法（tableSizeFor），得到的初始容量和我们之前讲的`HashMap`以及jdk7的`ConcurrentHashMap`不同，即使你传递的是一个2的幂次方数，该方法计算出来的初始容量依然是比这个值大的2的幂次方数。如果是jdk7的`ConcurrentHashMap`，容量计算就是参数对最近的2的幂次方数向上取整

```java
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins
        initialCapacity = concurrencyLevel;   // as estimated threads
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
        MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap;
}
```

```java
//基于一个Map集合,构建一个ConcurrentHashMap//初始容量为16
public ConcurrentHashMap(Map<? .extends K，? extends V> m){
	this.sizeCtl = DEFAULT_CAPACITY;
	putAll(m);
}
```



## 2. sizectl含义解释

> 注意:以上这些构造方法中，都涉及到一个变量sizeCt1，这个变量是一个非常重要的变量，而且具有非常丰富的含义，它的值不同，对应的含义也不一样，这里我们先对这个变量不同的值的含义做一下说明，后续源码分析过程中，进—步解释
>
> `sizectl`为0，代表数组未初始化，且数组的初始容量为16
>
> `sizectl`为正数，如果数组未初始化，那么其记录的是数组的初始容量，如果数组已经初始化，那么其记录的是数组的扩容阈值（数组的初始容量*0.75，0.75是默认的负载因子的值）
>
> `sizectl`为-1，表示数组正在进行初始化
>
> `sizectl`小于0，并且不是-1，表示数组正在扩容，-(1+n)，表示此时有n个线程正在共同完成数组的扩容操作



# 二、jdk1.8添加的安全机制



## 1. 源码分析



###  1.1. 添加元素put/putVal方法

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
```

```java
final V putVal(K key, v value, boolean onlyIfAbsent){
	//如果有空值或者空键,直接抛异常
	if (key == null ll value == null) throw new NullPointerException();/基于key计算hash值，并进行一定的扰动
	// 基于key算的hash值，并进行一定的扰动
    // 这个值一定是正数，方便后面添加元素时判断节点的类型（和HashMap一样，有链表节点和树节点两种节点）
	int hash = spread(key .hashCode());
	//记录菜个桶上元素的个数，如杲超过8个,会转成红雪树int binCount = 0;
	for (Node<K,V>[0]tab = table; ;){
		Node<K,V> f; int n, i,fh;
		//如果数组还未初始化,先对数组进行初始化
        if (tab m= nu)l [l(n = tab. length) -= e)
			tab = initTable();
		//如果hash计算得到的桶位置没有元素，利用cas将元素添加
		else if ((f = tabAt(tab， i - (n -1)& hash)) -= null){
			// cas+自旋(和外侧的for构成自旋循环)，保证元察添加安全
            if ( casTabAt(tab, i, null, new Node<K,V>(hash,key, value, null)))
				break; // no lock when adding to empt, bin
		}
    	// 如果hash计算得到的桶位置元素的hash值为MOVED，证明正在扩容，那么协助扩容
        else if (( fh = f.hash) == MOVED)
        	tab = helpTransfer(tab,f);
		else {
    		// hash计算的桶位置元素不为空，且当前没有处于扩容操作,也行元素
            V oldVal = null;
    		// 对当前桶进行加锁,保证线程安全,执行元素添加操作
            synchronized(f){
	    		if (tabAt(tab,i) == f) (
		    	// 普通链表节点
				if (fh >= 0) {
                    binCount = 1;
                    for (Node<K,V> e = f;; ++binCount) {
                        K ek;
                        if (e.hash == hash &&
                            ((ek = e.key) == key ||
                             (ek != null && key.equals(ek)))) {
                            oldVal = e.val;
                            if (!onlyIfAbsent)
                                e.val = value;
                            break;
                        }
                        Node<K,V> pred = e;
                        if ((e = e.next) == null) {
                            pred.next = new Node<K,V>(hash, key, value, null);
                            break;
                        }
                    }
                }
                // 树节点 
                else if (f instanceof TreeBin) {
                    Node<K,V> p;
                    binCount = 2;
                    if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                        oldVal = p.val;
                        if (!onlyIfAbsent)
                            p.val = value;
                    }
				}
			}
		}
        if (binCount != 0) {
        	if (binCount >= TREEIFY_THRESHOLD) // 如果链表中节点数大于8，可以考虑树化
            	treeifyBin(tab, i); // 如果map中总节点数符合要求，可以考虑树化
            if (oldVal != null)
				return oldVal;
			break;
			}
		}
	}
	// 节点数+1，维护集合的长度
	addCount(1L, binCount);
	return null;
}
```

> 通过以上源码，我们可以看到，当需要添加元素时，会针对当前节点（桶）加锁。只有当前的桶上锁，不影响新元素添加到其他的桶里
> 加锁操作，这样一方面保证元素添加时，多线程的安全，同时对某个桶位加锁不会影响其他桶位的操作，进一步提升多线程的并发效率

	abstract class	interface



### 1.2. 数组初始化,initTable方法
```java
private final Node<K,V>[]initTable() {
    Node<K,V>[] tab; int sc;
    // cas+自旋。保证线程安全,对数组进行初始化操作
    while 《(tab = table) == null ll tab.length -- 0) {
    	//如果sizect1的值(-1)小于8，说明此时正在初始化,让出cpu
        if ((sc =- sizeCtl)<0)
    		Thread .yield(); // lost initialization race; just spin
        //cas修改sizeCt1的值为-1，修改成功，进行数组初始化,失败，继续自旋
        else if (U.compareAndSwapInt(this,SIZECTL， sc，-1)){
            try {
                if ((tab = table) == null ll tab.length -- 0){
                    // sizeCtl为0，取默认长度16，否则去sizeCtl的值
                    int n = (se >0)? sc : DEFAULT_CAPACITY;Suppresslarnings("unchecked")
                    //基于初始长度,构建数组对象
                    Node<K,V>[]nt - (Node<K,V>[])new Node< ? ,2>[n];table = tab - nt;
                    //计算扩容阈值,并赋值给scsc = n - (n >>2);
                }	finally {
                    //将扩容阈值，赋值给sizeCtl
                    sizeCtl = sc;
                }
                break;
            }
        }
    return tab;
}
```

 

### 1.3. 维护集合大小

`baseCount`保存集合的大小

插入元素时，维护集合的大小分两步

1. 尝试使用cas修改`baseCount`。如果失败，走第二步
2. 尝试向一个数组中某个位置添加此次插入的元素数量
3. 如果还是加失败，尝试自旋执行1和2
4. 最终的集合大小等于`baseCount`+数组中所有元素的总和

![[../../020 - 附件文件夹/Pasted image 20230401233031.png|675]]

```java
// 尝试计算集合大小。baseSum+数组中所有元素
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

