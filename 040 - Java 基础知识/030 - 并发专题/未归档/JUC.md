#还没有复习 

# 多线程

> 线程：intel的一个处理器同一时间可运行一个线程。不过会频繁切换线程，所以宏观上来看，一个处理器可同时处理多个线程
>
> 并发：多个请求请求同一份资源
>
> 并行：一个进程（多个线程）同时干好几件事



## 创建并运行线程

两个重要的类`Thread`，`Runnable`

1. 创建线程前的准备

   创建一个类，实现`Runnable`接口中的`run()`方法

2. 创建并运行线程

   调用`Thread`的构造方法，传递一个`Runnable`实现类的实例化对象或一个`lambda表达式`作为参数，创建一个`Thread`的对象。执行它的`start()`方法

这样就会创建一个线程，并调用`Thread`的`run()`方法（此方法调用了传入的`Runnable`对象的`run()`方法）



`Thread`的`start()`方法做了什么？

1. 开启一个新的线程
2. 调用`Thread`的`run()`方法

如果直接在main方法中调用`Runnable`实现类的对象的`run()`方法，这个方法会在main线程中运行，而不是开启一个新的线程

一个`Thread`对象的`start()`方法只能执行一次，再次执行时会抛异常



 ## 线程的礼让

`Thread`的`yield()`方法

作用：令此线程对象暂时让出部分`CPU`的资源，起到的效果是让其他线程执行代码的概率增大



## 线程的join

`Thread`的`join()`方法

使用场景：A线程中调用了B线程的`join()`方法

作用：在A线程执行到`b.join()`后，A线程进入阻塞状态，会执行完B线程再执行A线程中其余部分。而不是AB线程所有代码并行执行



## 线程的优先级

`Thread`的`setPriority(int newPriority)`方法

传递一个int类型的数字，范围1~10。数字越大，优先级越高，执行此线程的概率也高

参数越界会抛异常



## 线程的分类

- 用户线程
- 守护线程

jvm的垃圾回收机制运行的gc线程就是守护线程



## 线程的生命周期

- 新建。创建以恶搞`Thread`或其子类对象即为新建线程
- 就绪。新建好线程后，`Thread`对象执行`start()`方法并等待`CPU`分配时间片，此时线程处于就绪状态。线程已具备运行条件，只是没有分配到`CPU`的资源
- 运行。线程获得到`CPU`资源，进入运行状态
- 阻塞。线程被挂起 ，暂时让出`CPU`资源并临时中止执行，此时线程处于阻塞状态
- 死亡。线程已完成工作或线程被主动强行终止或线程因异常导致结束

详细状态见`java.lang.Thread.State`

![[../../../020 - 附件文件夹/Pasted image 20230401233440.png|700]]


## 线程的同步

关键字`synchronized`

- 同步代码块
- 同步方法

```java
/**
 * 同步代码块
 * 这个object名字叫“同步监视器”，它也可以是this，类的字节码对象——XXX.class
 * 将可能会出现线程不安全的变量放在这里，如果此对象正在被某个线程使用，则其他线程执行到此代码块时进入阻塞，直到对象的锁被解开
**/
synchronized(object){
    // ......
}
```

```java
/**
 * 同步方法
 * 非静态同步方法的同步监视器是this，静态同步方法的同步监视器是类本身（类的字节码对象——XXX.class）
 * 此方法被一个线程执行时，其他方法执行此方法前会进入阻塞，直到此方法被执行完后其他线程才能使用（前提是这些线程使用的是同一个target）
 * 这种锁锁的范围大，影响性能
**/
public synchronized void test(){
    // ......
}
```



线程同步的出现涉及到了同步块的范围，锁的范围越大，效率越低。所的范围越小，可能并发情况下就会出问题

那么，锁的范围和业务逻辑就有很大的修改空间



**单例模式下线程的案例**

```java
public class Test {

    private Test test = null;

    private Test() {
    }

    public Test getInstance() {
        if (test == null) {
            synchronized (this) {
                if (test == null) {
                    test = new Test();
                }
            }
        }
        return test;
    }
}
```





## 死锁问题

> 死锁：不同的线程分别占用对方所需要的同步资源不放弃，都等待对方放弃自己所需要的同步资源，就形成了线程的死锁
>
> 出现死锁后，不会出现异常和提示，所有线程都处于阻塞状态，无法继续





## Lock锁


> lock锁也是线程同步的一种实现
>
> synchronized和lock锁的区别。前者是关键字，后者是接口/类。前者效率低且存在死锁问题，后者需要手动加锁开锁，但可定制化程度高，灵活



```java
public static void main(String[] args) {
    ReentrantLock lock = new ReentrantLock();
    try {
        // 上锁
        lock.lock();
		// 业务逻辑......
    }finally {
        // 解锁
        lock.unlock();
    }
}
```



注：lock对象必须使用的是同一个，多个线程使用同一个锁才有意义。如果多个线程使用的不是同一个锁，那是锁不上资源的。那么锁的声明应该放在类的成员变量中而不是方法中



`Lock`出现的比`synchronized`晚，所以jre源码中使用`synchronized`居多。但是`Lock`更好用

建议使用优先级`Lock` > `synchronized`同步代码块 > `synchronized`同步方法



## TimeUnit

> 线程sleep的工具类，能更方便设置各种单位的时间。底层用的还是Thread.sleep()
>

```java
// 睡5秒
TimeUnit.SECONDS.sleep(5);
// 睡5小时
TimeUnit.HOURS.sleep(5);
// ... 
```



## 线程之间的通信



### 通信方式

`synchronized`线程间通信方式涉及到的三个方法

`wait()`，	`notify()`，`notifyAll()`



`wait()`方法会释放同步监视器，`sleep()`方法不会释放同步监视器

`notify`随机唤醒一个线程，将其转为就绪状态

这三个方法必须在`synchronized`中使用，且这三个方法的调用者必须和同步监视器一致（一般这三个方法的调用者是this）

使用`Lock`的话，线程的通信有其他方式



`Lock`线程间通信方式涉及到`Condition`类的三个方法



`await()`，`signal()`，`signalAll()`

```java
/*
 * 多线程下实现number变量y0->1
 * 即使多个线程对number执行increment，number的值也不会大于1
*/

private int number = 0;
// 锁
private Lock lock = new ReentrantLock();
// 用于线程间通信的类
private Condition condition  = lock.newCondition();

public  void increment() {

    lock.lock();
    try {
        // 判断。下面会讲此种判断方式（虚假唤醒）
        while(number!=0) {
            // 阻塞
            condition.await();
        }
        // 业务
        ++number;
        System.out.println(Thread.currentThread().getName()+" \t "+number);
        // 唤醒
        condition.signalAll();
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }

}
```

`Condition`就像是信号枪

`condition.await()`当前线程会被阻塞（发出阻塞信号）

`condition.signal()`**唤醒被此对象使用`condition.await()`阻塞的线程**

这里不能把`Condition`比作`Lock`的钥匙，因为它并没有做解锁和上锁，它做的是阻塞线程和唤醒线程



### 虚假唤醒

```java
// 下面的代码是消费者消费产品。如果产品数量=0，那么消费者等待产品数量>0时再执行num--
if ( num == 0 ){
    this.wait();
}
num--;
```

会存在这样的问题

场景：num=0后，多个消费者进入判断并执行`wait()`。生产者生产商品后`num=1`并唤醒所有消费者。

结果：多个消费者执行`num--`。导致num为负数

由于虚假的唤醒，导致消费者做出了错误的判断 / 没做判断就执行余下业务



解决方案：使用`while`替换`if`

```java
// 循环判断，消费者被唤醒后会重新判断num的值是否为0
while ( num == 0 ){
    this.wait();
}
num--;
```



## 新增创建线程的方式

> 创建线程的传统方式有以下两种
>
> 1. 创建一个Thread类或子类的对象，重写其run方法
> 2. 传递一个Runnable实现类的对象
>
> JDK5.0新增方式，实现Callable接口；使用线程池创建线程



### 实现Callable接口

`Callable`接口比`Runnable`强大

`Callable`接口的`call`方法有泛型返回值，且抛异常

```java
V call() throws Exception;
```



使用`Callable`接口时需要结合`Future`接口和其实现类`FutureTask`

下面是一个简单的示例

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    // 构造器中使用lambda表达式实现Callable的call()方法
    FutureTask futureTask = new FutureTask(() -> "hello word");

    // 必须开启线程才能执行上述的call方法
    new Thread(futureTask).start();

    // 接收call方法的返回值 
    Object res = futureTask.get();
    System.out.println("res : " + res);

}
```



`FutureTask`字面翻译：“未来的任务”

简单聊一下创建线程时传递`Runnable`和`FutureTask`

区别：前者的`run`方法没有返回值，后者的`Callable`的`call`方法有返回值（使用`FutureTask`的`get()`方法即可获取返回值）

那么，重点在于`FutureTask`的`Thread`执行`get()`方法。在main中执行费时的业务放到`FutureTask`中，等到需要返回值的时候在主线程中`get()`即可。如果线程未执行完毕，`get()`方法会阻塞主线程，直至`FutureTask`线程获取到返回结果



一个`FutureTask`的对象中的`Callable`的`call()`方法只能使用一次

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {

    FutureTask<String> f = new FutureTask<>(()->{
        System.out.println("hello");
        return UUID.randomUUID().toString();
    });

    new Thread(f, "A").start();
    System.out.println(f.get());

    new Thread(f, "B").start();
    System.out.println(f.get());

}
```



运行结果

```
hello
77494b00-e4c6-4b0f-82f9-2abea10495c1
77494b00-e4c6-4b0f-82f9-2abea10495c1
```

可见只输出了一次"hello"。`f.get()`获取到的值也一样



### 线程池创建线程

> 在开发环境中常使用线程池创建线程

为避免频繁创建线程，销毁线程。在线程池中创建多个线程，当需要使用线程时直接使用线程池中的线程，用完线程后将线程返还线程池

#### 能干啥

管理线程。比如，设置线程池中最少/多线程数，设置线程闲置多久后自动销毁......



#### 怎么用

`Executor`级别很高的接口。地位和集合中的`Collection`一样

`ExecutorService`上个类的子接口。地位和集合中的`List`一样

上述两个接口是用于执行线程任务的接口

`Executors`工具类。创建线程池的工厂（创建上述接口的实现类的工厂）

`ThreadPoolExecutor`线程池实现类

![[../../../020 - 附件文件夹/Pasted image 20230401233513.png|300]]

```java
public static void main(String[] args) throws Exception {
	// 创建有10个线程的线程池
    ExecutorService service = Executors.newFixedThreadPool(10);
    
	// execute方法参数为Runnable接口的实现类
    service.execute(()-> System.out.println("run"));

    // submit适用于Callable接口
    Future<String> future = service.submit(() -> "call");
    String res = future.get();
    System.out.println(res);
    
    // 关闭线程池，释放线程资源
    service.shutdown();
}
```



**在哪设置线程池中的常量？**

`ThreadPoolExecutor`有七个重要的参数，有哪些重要参数见`ThreadPoolExecutor`的需要七个参数的构造方法

将获取到的`service`强换为`ThreadPoolExecutor`后可使用各种set方法

![[../../../020 - 附件文件夹/Pasted image 20230401233534.png|500]]

有 有关于线程池的框架，所以这里说的就是框架实现的基本原理



#### 详细点

不同类型的线程池

分三类：线程数固定的，只有一个线程的，可扩容的。三种线程池

| 创建线程池                          | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| Executors.newFixedThreadPool(int)   | 执行长期任务性能好，创建一个线程池，<br/>一池有N个固定的线程，有固定线程数的线程 |
| Executors.newSingleThreadExecutor() | 一个任务一个任务的执行，一池一线程（不建议用）               |
| Executors.newCachedThreadPool()     | 执行很多短期异步任务，线程池根据需要创建新线程，<br/>但在先前构建的线程可用时将重用它们。可扩容，遇强则强 |



#### ThreadPoolExecutor线程池

底层和阻塞队列`BlockingQueue`有关系。创建线程池的方法中需要阻塞队列的对象，不同种类的线程池使用的阻塞队列不同

有多个构造方法，但最后都会调用一个有7个参数的构造方法



**原理**

线程池中有一个阻塞队列，如果所有线程都在执行任务，新来的任务将进入阻塞队列中等待线程获取

![[../../../020 - 附件文件夹/Pasted image 20230401233547.png|500]]

==线程池的使用规范见《阿里巴巴java开发规约手册》==

实际应用中不使用`Executors`的方法创建线程池。而使自定义的方法创建`ThreadPoolExecutor`类型的线程池

因为`Executors`拆功能键线程池的方法都有很大的缺陷。详情见阿里巴巴手册



代码示例（使用`ThreadPoolExecutor`的构造方法创建线程池，而不是使用`Executors`创建）

```java
// 最后一个参数是阻塞队列的拒绝策略，下面会讲
ExecutorService threadPool = new ThreadPoolExecutor(
    2,
    5,
    2L,
    TimeUnit.SECONDS,
    new ArrayBlockingQueue<Runnable>(3),
    Executors.defaultThreadFactory(),
    //new ThreadPoolExecutor.AbortPolicy()
    //new ThreadPoolExecutor.CallerRunsPolicy()
    //new ThreadPoolExecutor.DiscardOldestPolicy()
    new ThreadPoolExecutor.DiscardOldestPolicy()
);
```



#### 线程池中阻塞队列的拒绝策略

| 策略名                                   | 说明                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| ThreadPoolExecutor.AbortPolicy()         | （默认策略）直接抛出RejectedExecutionException异常阻止系统正常运行 |
| ThreadPoolExecutor.CallerRunsPolicy()    | “调用者运行”一种调节机制，该策略既不会抛弃任务，也不会抛出异常<br/>而是将某些任务回退到调用者，**由调用者执行任务** |
| ThreadPoolExecutor.DiscardOldestPolicy() | 抛弃队列中等待最久的任务，然后把当前任务加人队列中<br/>尝试再次提交当前任务 |
| ThreadPoolExecutor.DiscardOldestPolicy() | 该策略默默地丢弃无法处理的任务，不予任何处理也不抛出异常<br/>如果允许任务丢失，这是最好的一种策略 |





# JUC进阶



## 常用集合/容器的线程安全问题

先上一张集合的继承&实现关系图

![[../../../020 - 附件文件夹/Pasted image 20230401233606.png]]

### ArrayList

此容器线程不安全。会抛`java.util.ConcurrentModificationException` 译：并发修改异常

- 单线程下。对一个list对象循环 执行`add()`和打印list操作。没问题
- 多线程。多个线程执行`list.add()`和打印操作。
  - 线程数少的话，不报错，但是打印结果出错
  - 线程数多的话，会抛异常，数据打印结果也会出错



解决方法

- 使用`Vector`类
- 使用`Collections`
- 写时复制



### Vector（不使用）

![Image](file://D:\workspace\Typora-workspace\imgs\Image.bmp?lastModify=1680363376)

`Vector`类的`add()`等方法使用了`synchronized`。是线程安全的，但是锁的范围太大了，不建议用



### Collections（不使用）

```java
// 使用此方法，传入线程不安全的容器，返回线程安全的容器
List list = Collections.synchronizedList(new ArrayList<>());
```

除此以外，还有获取其他线程安全的容器的方法

![[../../../020 - 附件文件夹/Pasted image 20230401233640.png|500]]

### CopyOnWriteArrayList

底层用的是`Lock`

```java
List<String> list = new CopyOnWriteArrayList<>();
```

下面简单介绍一下`CopyOnWriteArrayList`

读写分离的思想

`CopyOnWriteArrayList`中有两个list，一个用来读，一个用来写。执行写操作的list是线程安全的，执行完写操作后，将此llist复制到用来读的list中

这两个list的关系就像是，已经发布的软件和开发人员手中==正在更新/未发布==的软件

`CopyOnWriteArrayList`的`add()`源代码还是比较容易阅读的，可以看一下加深理解



除此以外还有`CopyOnWriteArraySet`（set的高并发容器），`CopyOnWriteMap`（不使用此类做map的高并发容器）

map的高并发容器是`ConcurrentHashMap`

`HashTable`是线程安全的



## 辅助类

问题：下面这些方法都是线程安全的吗？



### CountDownLatch

> 直译：倒计时闩锁。设置一个时间，执行方法可进行倒计时。下面有简单示例代码助于理解
>
> 可用于多个线程间执行顺序的调度。例如：希望A线程在B，C，D线程都执行完后再执行

- 创建倒计时对象并设置倒计时，`CountDownLatch countDownLatch = new CountDownLatch(6);`

* 每执行一次`countDownLatch.countDown();`，其成员变量`count`-1
* `countDownLatch.await();`在`count`值为0之前，此方法将阻塞所在线程，`count`为0后当前线程方可向下运行



示例代码（班长会在所有同学离开后再离开）

```java
public static void main(String[] args) throws InterruptedException {
    // 设置倒计时时间
    CountDownLatch countDownLatch = new CountDownLatch(6);

    for (int i = 1; i <= 6; i++) //6个上自习的同学，各自离开教室的时间不一致
    {
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " 号同学离开教室");
            // 倒计时-1
            countDownLatch.countDown();
        }, String.valueOf(i)).start();
    }
    // 如果倒计时不为0，阻塞；为0，执行下面的业务
    countDownLatch.await();
    System.out.println(Thread.currentThread().getName() + " 班长关门走人，main线程是班长");

}
```



### CyclicBarrier

>字面意思是可循环（Cyclic）使用的屏障（Barrier）
>
>CyclicBarrier 设立了一个屏障（终点），当所有线程都执行到屏障（终点）面前就会执行预先实现好的run方法

它与`CountDownLatch`相反。`CountDownLatch`是倒计时，`CyclicBarrier`是正计时

- 创建对象`CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {...});`，第一个参数为`parties`
- 线程执行`cyclicBarrier.await();`表示此线程到达终点，并阻塞线程，等待其他线程到达终点
- 其他线程到达终点后，唤醒线程继续向下执行，并执行cyclicBarrier的“终点方法”

代码示例（集齐七龙珠后召唤神龙）

```java
public static void main(String[] args) {

    // 正计时完成后，执行此处的Runnable的接口实现的run()方法
    CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
        System.out.println("*****集齐7颗龙珠就可以召唤神龙");
    });

    for (int i = 1; i <= 7; i++) {
        new Thread(() -> {
            try {
                System.out.println(Thread.currentThread().getName() + "\t 星龙珠被收集 ");
                cyclicBarrier.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, String.valueOf(i)).start();
    }

}
```



### Semaphore

> 翻译：信号

- 创建对象`Semaphore semaphore = new Semaphore(3);`，声明此处有3个资源
- 发出信号请求占用资源`semaphore.acquire();`，如果有空闲的资源，则占用一个资源；如果没有闲置的资源可被占用，那么此线程阻塞，直到有空闲的资源再占领
- 发出信号释放资源`semaphore.release();`，会有一个资源被释放



 * acquire（获取） 当一个线程调用acquire操作时，它要么通过成功获取信号量（信号量减1），
 *       要么一直等下去，直到有线程释放信号量，或超时。
 * release（释放）实际上会将信号量的值加1，然后唤醒等待的线程。
 * 信号量主要用于两个目的，一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制。

案例代码（三个车位—资源，6个车抢车位—6个线程）

```java
public static void main(String[] args) {
    Semaphore semaphore = new Semaphore(3);//模拟3个停车位

    for (int i = 1; i <= 6; i++) //模拟6部汽车
    {
        new Thread(() -> {
            try {
                // 占用车位
                semaphore.acquire();
                System.out.println(Thread.currentThread().getName() + "\t 抢到了车位");
                TimeUnit.SECONDS.sleep(new Random().nextInt(5));
                System.out.println(Thread.currentThread().getName() + "\t------- 离开");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                // 释放车位
                semaphore.release();
            }
        }, String.valueOf(i)).start();
    }

}
```





## ReentrantReadWriteLock

> 读写锁。此类实现了ReadWriteLock接口

使用`ReentrantLock`的`lock()`时，只能有一个线程能执行余下的业务，直到执行`unlock()`时其他线程才可自行上锁占用资源

根据读写分离思想，写的时候只能一个人操作资源，读的时候可以多个人一起读

那么就有了：读读可共享；读写不可共享

即：

一个线程写的时候，其他线程不能使用资源

有线程进行读操作，且没有线程执行写操作时是可以有多个线程同时读的



**简单实现**

```java
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock rwLock = new ReentrantReadWriteLock();

    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
            map.put(key, value);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public Object get(String key) {
        rwLock.readLock().lock();
        Object result = null;
        try {
			result = map.get(key);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            rwLock.readLock().unlock();
        }
        return result;
    }
}

public class Test {

    public static void main(String[] args) {
        MyCache myCache = new MyCache();

        // 5个线程插入数据
        for (int i = 1; i <= 5; i++) {
            final int num = i;
            new Thread(() -> {
                myCache.put(num + "", num + "");
            }, String.valueOf(i)).start();
        }

        // 5个线程读数据
        for (int i = 1; i <= 5; i++) {
            final int num = i;
            new Thread(() -> {
                myCache.get(num + "");
            }, String.valueOf(i)).start();
        }

    }
}
```

下面说说读锁的用处（因为一开始可能会认为读锁没有存在的必要）

- 加上读锁

  A线程正在写，还没写完，B线程来读数据。B线程上的读锁看见写锁被锁上了，B线程阻塞，等到A线程释放写锁后进入就绪状态

- 去掉读锁

  A线程正在写，还没写完，B线程就来读数据。导致表面上执行了A写数据，B却没有读到数据





## BlockingQueue

> 直译：阻塞队列。是个接口。有多个实现类

它和一般队列的区别在于它的方法种类多，返回值和返回方式不同

![[../../../020 - 附件文件夹/sadfagvde.bmp|700]]

| 处理方式 | 详细处理方式                                                                                                                                                                              |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 抛出异常 | 当阻塞队列满时，再往队列里add插入元素会抛IllegalStateException:Queue full<br>当阻塞队列空时，再往队列里remove移除元素会抛NoSuchElementException                                           |
| 特殊值   |插入方法，成功ture失败false<br>移除方法，成功返回出队列的元素，队列里没有就返回null|
| 一直阻塞 | 当阻塞队列满时，生产者线程继续往队列里put元素，队列会一直阻塞生产者线程直到put数据or响应中断退出<br/>当阻塞队列空时，消费者线程试图从队列里take元素，队列会一直阻塞消费者线程直到队列可用 |
| 超时退出 | 当阻塞队列满时，队列会阻塞生产者线程一定时间，超过限时后生产者线程会退出                                                                                                                  |

有多个实现类

![[../../../020 - 附件文件夹/Pasted image 20230401234001.png]]


