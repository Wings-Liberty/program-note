#还没有复习 

# volatile介绍

> [参考](https://blog.csdn.net/m0_37163942/article/details/89879271)

关键字`volatile`（译：易变的）

`volatile`是java中轻量级的同步机制

`synchronized`是重量级锁



**能干什么**

- 内存可见性（要留意复合类操作问题）
- 禁止指令重排序



# 内存可见性

```java
public class TestVolatile {
    boolean status = false;
 
    /**
     * 状态切换为true
     */
    public void changeStatus(){
        status = true;
    }
 
    /**
     * 若状态为true，则running。
     */
    public void run(){
        if(status){
            System.out.println("running....");
        }
    }
}
```

问：线程A操作`TestVolatile`的对象执行`changeStatus`，线程B操作同一个对象执行`run`方法就一定能成功打印"running...."吗？

答：不一定

解释：不一定成功的原因和java的内存模型有关系

# JMM（java内存模型）

内存分两种，线程的本地内存和主内存。不同线程只能使用自己的本地内存

![[../../020 - 附件文件夹/Pasted image 20230401232337.png|500]]

线程A持有的`TestVolatile`的对象中的成员变量的值都存放在线程A的本地内存中

线程B持有同一对象，对象的成员变量的值在线程B的本地内存中。

A线程的本地内存对于B线程来说是不可见的。

同时主内存中也有对象的所有数据。

问题就出现了

线程A中对象的数据一旦修改，新的数据将同步到主内存中，但是线程B不知道。

解决方法：使用`synchronized`或`lock`

但是现在知道了`JMM`，就可以用更轻量级的锁`volatile`



被`volatile`修饰的变量会出现以下效果

1. 当写一个volatile变量时，JMM会把该线程对应的本地内存中的变量强制刷新到主内存中去
2. 这个写会操作会导致其他线程中的缓存无效



# 复合类操作

`volatile`可以解决一些问题，但是并不能代替`synchronized`。

```java
public class Counter {
    public static volatile int num = 0;
    //使用CountDownLatch来等待计算线程执行完
    static CountDownLatch countDownLatch = new CountDownLatch(30);
    public static void main(String []args) throws InterruptedException {
        //开启30个线程进行累加操作
        for(int i=0;i<30;i++){
            new Thread(){
                public void run(){
                    for(int j=0;j<10000;j++){
                        num++;//自加操作
                    }
                    countDownLatch.countDown();
                }
            }.start();
        }
        //等待计算线程执行完
        countDownLatch.await();
        System.out.println(num);
    }
}
```

问题：上述程序输出结果不是30000，且每次执行结果都不同

原因：`num++`是`num = num +1`的简写，执行了读取num，计算num+1，将num+1赋值给num。

复合操作，在多线程环境下，有可能线程A将num读取到本地内存中，此时其他线程可能已经将num增大了很多，线程A依然对过期的num进行自加，重新写到主存中，最终导致了num的结果不合预期，而是小于30000

解决方法：使用类似`AtomicInteger`的原子操作类





# AtomicInteger

```java
public class Counter {
　　//使用原子操作类
    public static AtomicInteger num = new AtomicInteger(0);
    //使用CountDownLatch来等待计算线程执行完
    static CountDownLatch countDownLatch = new CountDownLatch(30);
    public static void main(String []args) throws InterruptedException {
        //开启30个线程进行累加操作
        for(int i=0;i<30;i++){
            new Thread(){
                public void run(){
                    for(int j=0;j<10000;j++){
                        num.incrementAndGet();//原子性的num++,通过循环CAS方式
                    }
                    countDownLatch.countDown();
                }
            }.start();
        }
        //等待计算线程执行完
        countDownLatch.await();
        System.out.println(num);
    }
}
```

使用了原子操作类将不会出现复合操作带来的问题



# 禁止指令重排序

volatile还有一个特性：禁止指令重排序优化。

重排序是指编译器和处理器为了优化程序性能而对指令序列进行排序的一种手段。但是重排序也需要遵守一定规则：

1. 重排序操作不会对存在数据依赖关系的操作进行重排序

   比如：a=1;b=a; 这个指令序列，由于第二个操作依赖于第一个操作，所以在编译时和处理器运行时这两个操作不会被重排序

2. 重排序是为了优化性能，但是不管怎么重排序，单线程下程序的执行结果不能被改变

   比如：a=1;b=2;c=a+b这三个操作，第一步（a=1)和第二步(b=2)由于不存在数据依赖关系，所以可能会发生重排序，但是c=a+b这个操作是不会被重排序的，因为需要保证最终的结果一定是c=a+b=3

   重排序在单线程模式下是一定会保证最终结果的正确性，但是在多线程环境下，问题就出来了，来开个例子，我们对第一个TestVolatile的例子稍稍改进，再增加个共享变量a

```java
public class TestVolatile {

    int a = 1;

    boolean status = false;

    // 状态切换为true
    public void changeStatus(){
        a = 2; // 1
        status = true; // 2
    }

    // 若状态为true，则running。
    public void run(){
        if(status){ // 3
            int b = a+1; // 4
            System.out.println(b);
        }
    }
}
```

 

假设线程A执行changeStatus后，线程B执行run，我们能保证在4处，b一定等于3么？

　　**答案依然是无法保证！**也有可能b仍然为2。上面我们提到过，为了提供程序并行度，编译器和处理器可能会对指令进行重排序，而上例中的1和2由于不存在数据依赖关系，则有可能会被重排序，先执行status=true再执行a=2。而此时线程B会顺利到达4处，而线程A中a=2这个操作还未被执行，所以b=a+1的结果也有可能依然等于2。

　　使用`volatile`关键字修饰共享变量便可以禁止这种重排序。**若用volatile修饰共享变量，在编译时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序**

　　volatile禁止指令重排序也有一些规则，简单列举一下：

1. 第二个操作是`voaltile`写时，无论第一个操作是什么，都不能进行重排序
2. 第一个操作是`volatile`读时，不管第二个操作是什么，都不能进行重排序
3. 第一个操作是vol`atile写时，第二个操作是volatile读时，不能进行重排序

# 总结

`volatile`是一种轻量级的同步机制，它主要有两个特性：

- 保证共享变量对所有线程的可见性

- 禁止指令重排序优化。同时需要注意的是，`volatile`对于单个的共享变量的读/写具有原子性，但是像`num++`这种复合操作，`volatile`无法保证其原子性，文中提出了解决方案，使用并发包中的原子操作类，通过循环CAS地方式来保证`num++`操作的原子性。

  