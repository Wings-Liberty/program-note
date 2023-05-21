#正在复习

# CompletableFuture 能实现什么功能

有一个这样的需求。有一个方法里需要完成 5 个任务，需要按照以下顺序完成

![[../../020 - 附件文件夹/Pasted image 20230419225752.png|300]]

这个过程有两个地方不是写一段顺序代码就能实现的

- task1 和 task2 只要有一个完成了就能走下一步，task3 和 task4 都完成了才能走下一步
- task1 和 task2 能并行执行，task3 和 task 4 能并行执行


# 异步回调是什么

异步回调是干啥的？为什么会有异步回调？新功能来源于新需求，因为有以下需求所以就有了这个功能

一个方法里有一些费时行为，但需要方法的响应时间不要太长，同时容忍可以不用立刻知道费时行为的响应结果

于是就可以用多线程，为费时行为创建一个 `Runnable` 后丢给线程池

但又希望可以这样，先让线程池做一件事，然后主线程做自己的事，等主线程做完后找线程池要任务的执行结果，于是有了 `Callable` 和 `Future` 接口及其实现类

但这还不够，我想这样，同时创建多个任务丢给线程池，然后等这些任务都返回结果，或有任意一个返回结果后，主线程再继续向下走。有这个需求是因为这种场景很常见

比如一个 “新增一个策略” 的接口，用户填了 10 个数据，程序需要检查 10 个数据是否合法时，我如果能用 10 个线程并行执行检查数据是否合法，接口响应速度就会缩短好几倍

比如一个 “创建订单” 的接口，用户一个订单里有 10 种商品，每种商品的购买数量还不一样，程序用 10 个线程分别查询每种商品的库存是否都够，在10个线程都返回结果后，我再用 10 个线程查询商品的价格，这 10 个线程都返回结果后，我再让主线程计算总价

> 小学你一定在数学书的某个章节里见过这个问题
>
> 问题：烧水需要 10 分钟，扫地需要 5 分钟，写作业需要 6 分钟，问你完成这三件事需要几分钟


**也就是说我希望方法的执行过程可以是一个工作流**，而不是传统的串行方式执行。我根据每个任务的依赖关系，在工作流中设置哪些任务可以并行执行，哪些任务必须等其他任务完成后才能执行

**因为有了并行的部分，所以工作流能比传统串行执行方法更快地完成任务。**然后 JDK 就有了 `CompletableFuture` 帮助我们 “绘制” 每个任务之间的依赖关系


# 怎么用 CompletableFuture 的 API 实现工作流

以这张图为模板，实现各种并行计算

![[../../020 - 附件文件夹/Pasted image 20230419225752.png|300]]

`CompletableFuture` 用了类似门面模式，需要用户创建一些 `CompletableFuture` 对象，每个对象就相当于上图的一个节点

- 当需要这个节点执行任务时，在创建 `CompletableFuture` 的时候就把任务作为一个参数传进去
- 当需要指定这个节点和其他节点的关系时，就把 `CompletableFuture` 对象作为参数传进入

为了实现上述流程，上述每一个节点应该有这些信息

- 它和其他节点的关系
- 需要执行的任务
- 获取上一个任务的返回值（任务入参）
- 这个任务的返回值是什么（任务返回结果）
- 如果执行任务时发生了异常，这个节点应该返回什么给下一个节点（任务异常的返回结果）

举几个例子

## 如果执行异常，怎么捕获异常并返回结果

```java
@Test  
public void testExceptionFunc() throws InterruptedException {  
    CompletableFuture<Integer> cf = CompletableFuture.supplyAsync(() -> {  
        System.out.println("0 不能作为除数，所以会抛异常");
        return 1 / 0;  
    });  
    cf.exceptionally(e -> {  
        System.out.println("处理异常，返回默认值 0，原因: " + e.getLocalizedMessage());  
        return 0;  
    });  
    TimeUnit.SECONDS.sleep(2);  
}
```

## 设置两个串行执行的任务

用一个 `CompletableFuture` 对象的方法链式调用就能创建另一个 `CompletableFuture` 对象，这两个任务有串行执行的关系

```java
@Test  
public void testSerial() throws InterruptedException {  
    CompletableFuture<Integer> cf1 = CompletableFuture.supplyAsync(() -> {  
        System.out.println("完成任务1");  
        return 1;  
    });
    CompletableFuture<Integer> cf2 = cf1.thenApplyAsync(num -> {  
        System.out.println("完成任务2");  
        return num + 1;  
    });
    TimeUnit.SECONDS.sleep(2);  
}
```

## 并行执行的任务

独立创建两个 `CompletableFuture` 对象，里面的两个任务就会并行执行

如果这样创建了两个任务后，还想实现

- task1 和 task2 其中一个执行完就执行下一步
- task3 和 task4 都执行完后再执行下一步

![[../../020 - 附件文件夹/Pasted image 20230419225752.png|300]]

这就需要 `CompletableFuture` 提供的方法把多个对象合并成一个

并行执行两个任务，只要有一个任务完成就能结束

```java
@Test  
public void parallelAndAnyOf() throws InterruptedException {  
    AtomicInteger a = new AtomicInteger(0);  
    CompletableFuture<Integer> cf1 = CompletableFuture.supplyAsync(() -> {  
        a.incrementAndGet();  
        System.out.println("完成 1 号任务");  
        return 1;  
    });  
    CompletableFuture<Integer> cf2 = CompletableFuture.supplyAsync(() -> {  
        a.incrementAndGet();  
        System.out.println("完成 2 号任务");  
        return 2;  
    });  
    // 合并两个任务  
    CompletableFuture<Object> merge = CompletableFuture.anyOf(cf1, cf2);  
    // cf1 和 cf2 只要有一个任务完成了就执行下一步。但 merge 节点后的节点都只会执行一次
    merge.thenAccept(code -> System.out.println("完成了 " + a.get() + " 个任务，并且先完成的任务编号是: " + code));  
    TimeUnit.SECONDS.sleep(2);  
}
```

并行完成两个任务，都完成后再输出

```java
@Test  
public void parallelAndAllOf() throws InterruptedException {  
    AtomicInteger a = new AtomicInteger(0);  
    CompletableFuture<Integer> cf1 = CompletableFuture.supplyAsync(() -> {  
        a.incrementAndGet();  
        System.out.println("完成 1 号任务");  
        return 1;  
    });  
    CompletableFuture<Integer> cf2 = CompletableFuture.supplyAsync(() -> {  
        a.incrementAndGet();  
        System.out.println("完成 2 号任务");  
        return 2;  
    });
    // 合并两个任务  
    CompletableFuture<Void> merge = CompletableFuture.allOf(cf1, cf2);  
    // cf1 和 cf2 都执行完才会执行 merge 。但 cf1 先完成后会执行下面的输出，但 cf2 仍会执行，不过不再执行 merge 的输出  
    merge.thenAccept(code -> System.out.println("完成了所有任务" + code));  
    TimeUnit.SECONDS.sleep(2);  
}
```

## 更多创建 CompletableFuture 的方法

`CompletableFuture` 对象里的任务以一个函数接口实例作为参数，它表示需要执行的任务内容

这个函数式接口有 3 种

- 无入参，无返回值
- 无入参，有返回值
- 有入参，有返回值

参考资料有以下

> - [廖雪峰的教程](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581182447650)用很简单的例子讲明白了 `CompletableFuture` 是什么，怎么用和常用的 3，4 个方法（完全足够了，其他API都是这几个的变体和组合）。但是很多 API 都没介绍
> - [更多 API 的介绍](https://zhuanlan.zhihu.com/p/344431341)。文章下面还提供了一些优秀的参考资料
>   - [简单介绍它是什么](https://blog.csdn.net/u011726984/article/details/79320004)
>   - [Future 的实现机制 - 讲解实现原理](https://zhuanlan.zhihu.com/p/54459770)
>   - [外文](callicoder.com/java-8-completablefuture-tutorial/)
> - [更详细的API](https://www.jianshu.com/p/558b090ae4bb)：其实其他 API 都是常用的 3，4 个 API 的组合变体，而且还难以理解
> - [美团技术分享](https://tech.meituan.com/2022/05/12/principles-and-practices-of-completablefuture.html)


> 这里的异步回调机制能实现异步回调和Future的组合

核心类 `CompletableFuture`   此类提供了很多静态方法处理 `Future`

示例代码

```java
/**
 * 异步回调
 */
public class AsyncDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
//        CompletableFuture<Void> runAsync = CompletableFuture.runAsync(() -> {
//            System.out.println("hello");
//        });

        CompletableFuture<Integer> asyncFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println("异步回调");
            int i= 3/0;
            return 200;
        });

        System.out.println("任务执行的返回值 : " + asyncFuture.whenComplete((res, throwable) -> {
            System.out.println("res : " + res);
            System.out.println("throwable : " + throwable);
        }).exceptionally(throwable -> {
            System.out.println("exceptionally抛出的异常 throwable" + throwable);
            return 100;
        }).get());
    }
}
```

# 最旧的笔记

回调的方式实现异步编程

Java 的一些框架， 比如 Netty， 自己扩展了 Java 的 `Future` 接口， 提供了 `addListener`等多个扩展方法； Google guava 也提供了通用的扩展 Future

`Future` 以及相关使用方法提供了异步执行任务的能力， 但是对于结果的获取却是很不方便， 只能通过阻塞或者轮询的方式得到任务的结果。 阻塞的方式显然和我们的异步编程的初衷相违背， 轮询的方式又会耗费无谓的 CPU 资源


在 Java 8 中, 新增加了一个包含 50 个方法左右的类: CompletableFuture， 提供了非常强大的 Future 的扩展功能， 可以帮助我们简化异步编程的复杂性， 提供了函数式编程的能力， 可以通过回调的方式处理计算结果， 并且提供了转换和组合 CompletableFuture 的方法

CompletableFuture 类实现了 Future 接口， 所以还是可以像以前一样通过 `get` 方法阻塞或者轮询的方式获得结果， **但是这种方式不推荐使用**
 
CompletableFuture 和 FutureTask 同属于 Future 接口的实现类， 都可以获取线程的执行结果

![../91 - 静态资源/Pasted image 20220918225907.png](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220918225907.png)


CompletableFuture 的主要使用方式如下

1. 创建一个对象
2. 计算完成时回调方法
3. handle 方法发
4. 线程串行化方法
5. 多个任务组合，都完成后再向下执行
6. 多个任务组合，完成一个就能向下执行
7. 多任务组合

异步回调的本质就是用线程池 + 同步器 实现

## 创建异步回调的方法

![Pasted image 20220918230954](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220918230954.png)

runXxxx 都是没有返回结果的， supplyXxx 都是可以获取返回结果的，可以传入自定义的线程池， 否则就用默认的线程池


## 回调方法获取计算的返回值

![Pasted image 20220918231958](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220918231958.png)

- whenComplete 可以处理正常和异常的计算结果
- exceptionally 处理异常情况


whenComplete 和 whenCompleteAsync 的区别：  

- whenComplete： 是执行当前任务的线程执行继续执行 whenComplete 的任务
- whenCompleteAsync： 是执行把 whenCompleteAsync 这个任务继续提交给线程池  
来进行执行


## 修改结果返回值

handle 和 complete 都能修改任务的执行结果


## 串行化执行的方法调用

当前置任务都完成后再执行这些 thenXxx 方法，这些方法会依赖前置任务的返回值

![Pasted image 20220918233105](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220918233105.png)

## 两个任务都完成后就运行指定方法合并结果


![[../../020 - 附件文件夹/07、异步&线程池.pdf]]