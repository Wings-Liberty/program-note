
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

**创建带有任务的对象**
| 静态方法名                                                 | 说明             |
|:---------------------------------------------------------- |:---------------- |
|CompletableFuture\<Void\> runAsync(Runnable runnable)|无入参，无返回值|
|CompletableFuture\<U\> supplyAsync(Supplier\<U\> supplier)| 无入参，有返回值 |

**组织多个对象之间的执行关系**


| 静态方法名                                                       | 说明                                                                                                |
|:---------------------------------------------------------------- |:--------------------------------------------------------------------------------------------------- |
| CompletableFuture\<Void\> allOf(CompletableFuture\<?\>... cfs)   | 创建一个汇聚节点，当 cfs 里所有任务都执行完后，汇聚节点才会放行，并继续执行下一步                   |
| CompletableFuture\<Object\> anyOf(CompletableFuture\<?\>... cfs) | 创建一个汇聚节点，当 cfs 里任意一个任务执行完后，汇聚节点就会放行，并继续执行下一步，但只会放行一次 |



**链式调用，设置下一步要执行的任务内容**
| 成员方法                                                                    | 说明                   |
|:--------------------------------------------------------------------------- |:---------------------- |
| CompletableFuture\<Void\> thenRun(Runnable action)                          | 无入参，无返回值       |
| CompletableFuture\<Void\> thenAcceptAsync(Consumer\<? super T\> action)     | 无入参，有返回值       |
| CompletableFuture\<U\> thenApplyAsync(Function\<? super T,? extends U\> fn) | 有入参，有返回值       |
| CompletableFuture\<T\> exceptionally(Function\<Throwable, ? extends T\> fn) | 接收异常对象，有返回值 | 

`CompletableFuture` 还提供了其他很多方法，但本质上都是上述方法的排列组合，或者说是封装

> [!tip] `CompletableFuture` 还提供了预设返回结果的 API
> - `boolean complete(T value)`，`CompletableFuture<U> completedFuture(U value)` 可以预设返回结果，如果还没计算完就调用 `get` 方法，就返回预先设置的值
> - 也可以用 `boolean completeExceptionally(Throwable ex)` 预先设置一个异常，如果没计算完就 `get` 结果，就抛出预设的异常

虽然是上述方法的排列组合，不过下面这些方法也确实用使用价值


| 成员方法                                                                                                                        | 说明                                               |
|:------------------------------------------------------------------------------------------------------------------------------- |:-------------------------------------------------- |
| CompletableFuture\<U\> handleAsync(BiFunction\<? super T, Throwable, ? extends U\> fn)                                          | 有入参，有返回值，且入参有上一个步骤留下的异常对象 |
|CompletableFuture\<V\> thenCombineAsync(CompletionStage\<? extends U\> other, BiFunction\<? super T,? super U,? extends V\> fn)| 等 this 和 other 都执行完后就执行 fn               |
|CompletableFuture\<T\> whenCompleteAsync(BiConsumer\<? super T, ? super Throwable\> action)| 等 this 执行完后执行 action，action 有入参，无返回值                                                   |


>[!tip] 参考资料
> - [廖雪峰的教程](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581182447650)用很简单的例子讲明白了 `CompletableFuture` 是什么，怎么用和常用的 3，4 个方法（完全足够了，其他API都是这几个的变体和组合）。但是很多 API 都没介绍
> - [更多 API 的介绍](https://zhuanlan.zhihu.com/p/344431341)。文章下面还提供了一些优秀的参考资料
>   - [简单介绍它是什么](https://blog.csdn.net/u011726984/article/details/79320004)
>   - [Future 的实现机制 - 讲解实现原理](https://zhuanlan.zhihu.com/p/54459770)
>   - [外文](callicoder.com/java-8-completablefuture-tutorial/)
> - [更详细的API](https://www.jianshu.com/p/558b090ae4bb)：其实其他 API 都是常用的 3，4 个 API 的组合变体，而且还难以理解
> - [美团技术分享 - CompletableFuture 实现原理](https://tech.meituan.com/2022/05/12/principles-and-practices-of-completablefuture.html)

