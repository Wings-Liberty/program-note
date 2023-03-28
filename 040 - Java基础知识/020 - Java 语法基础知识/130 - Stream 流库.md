# Stream 流式计算

`java.util.stream`

> `Stream` 的迭代速度与 `for` 循环遍历相比
>
> `Stream` 串行速度 慢于 `for` 速度
>
> `Stream` 并行速度 快于 `for` 和 `Stream` 串行速度
>
> 但是单核情况下 `Stream` 并行速度要慢于前两者


## 流是什么

java 中常用的集合有 `List`，`Set`，`Map`

集合重点在于数据，将数据放进集合中，对其进行增删改查

流的重点在于计算。流式计算将集合中的数据的存放形式转换为流，对流中的数据进行筛选，过滤，增强，对流中的数据更细粒度的操作


**流的特点**

1. 流并不存储其元素。这些元素可能存储在底层的集合中，或者是按需生成的
2. 流的操作不会修改其数据源。例如，fi1ter 方法不会从新的流中移除元素，而是会生成一个新的流，其中不包含被过滤掉的元素
3. 流的操作是尽可能惰性执行的。这意味着直至需要其结果时，操作才会执行。因此，我们甚至可以操作无限流


**流的动作分3种**

- 创建流的动作（`stream` 或 `parallelStream` 创建流）
- 流的中间操作。将初始流转换为其他流（`filter`，`map` 等操作流）
- 流的终止操作。执行完并返回结果后将关闭流，此流不可再使用（`count` 等返回计算结果并关闭流）


## 创建流的方法

- 流和数组或集合的互相转换
- 创建无限流
- 其他方式

**将集合或数组转换为流**

```java
// 将 Collection 的对象转为 Stream 对象
Stream<E> stream();

// 将数组对象转为 Stream 对象。of - 静态工厂方法实现的 “聚合函数”（把多个元素合并为 1 个对象）
static<T> Stream<T> of(T... values)

// 将数组对象转为Stream对象
Arrays.stream(xxx[] array);
```

**将流转换为集合或数组**

```java
// 将 Stream 对象转为指定的Collection的子类/实现类
<R, A> R collect(Collector<? super T, A, R> Collector);

// 将 Stream 中的元素放到数组中，并返回 Object 类型数组对象
Object[] toArray();

// 将Stream中的元素放到泛型数组中，并返回指定类型的数组对象
<A> A[] toArray(IntFunction<A[]> generator);
```

**创建无限流的两种方式**

```java
// 无限循环执行 Supplier 的函数式接口方法，将返回值放到流中
public static<T> Stream<T> generate(Supplier<T> s);

// 无限打印 Echo
Stream<String> stream = Stream.generate(()->"Echo");
stream.forEach(System.out::println);
```

```java
// 无限执行 UnaryOperator 的函数式接口方法（此接口实现了 Function，所以此方法就是 apply 方法）
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f);

// seed 初始值是0，方法是每次让 seed+1 并返回此值。执行结果是无限打印从0开始并递增的数字
Stream<BigInteger> stream = Stream.iterate(BigInteger.ZERO, e -> e.add(BigInteger.ONE));
stream.forEach(System.out::println);
```


**其他方式**

除上述创建流的方式外还有大量创建流的方式

- `Stream<String> lines = Files.lines(path);` 获取以文件所有行为元素的 `Stream` 流对象
- `Stream<String> words = Pattern.compile("\\PL+").splitAsStream(contents);` 将字符串分割为单词并放入流中


## 流的使用方式

获取到流对象后，对流对象的操作有两种：中间操作和终端操作

- 中间操作：方法关闭原来的 Stream，并返回另一个 Stream 供后续调用
- 终端操作：方法返回一个执行结果，可以是集合，也可以是指定类型

```java
/**
 * 过滤。使用 Predicate 接口
 * 将流中不满足 Predicate 的判断的数据过滤掉，返回保存由剩余数据的Stream流对象
 */
Stream<T> filter(Predicate<? super T> predicate);
```

```java
/**
 * 映射，使用 Function 接口
 * 对流中所有元素执行一遍 Function 的 apply() 方法并将方法返回值放进流，返回此流
 */
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

```java
/**
 * “打平”。处理 元素是流的流（流中的数据是很多流/流中的元素是流）
 * ps：flat 译：打平
 */
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);

// 举例：将一个List<String>中的所有字符串执行流操作split为字符串数组并返回流。将所有流中的字符全放到一个流或集合中
// res: [...["o", "u", "r"], ["b"，"o", "a", "t"] ...]
Stream<String> result = words.stream().map(w ->letters(w));
// res: [... "o", "u", "r", "b", "o", "a", "t" ...]
Stream<Stream<String>> flatResult = words.stream().flatMap(w->letters(w));
```

关于 `flatMap` 的实用方式，上面的例子不好，可以参考[这个例子](https://blog.csdn.net/dengjili/article/details/90557392)

```java
/**
 * 排序
 * 将流中的数据排序，并将此流返回。可使用自定义的排序方法
 */
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
```

```java
// 流的调试工具。因为流操作不好调试，所以Stream流提供了这个函数
// 每当从流中获取一个元素时都会执行peek参数列表中的Consumer函数式接口方法
Stream<T> peek(Consumer<? super T> action);
```


**抽取子流和连接流**

```java
/**
 * 将指定数量的数据放进新流中，并返回此流
 * 可以和 stream.generate 一起用
 * 如：Stream.generate(Math::random).limit(100); 创建含有100个随机数的流
 */
Stream<T> limit(long maxSize);

// 将指定数量的数据删除，并返回此流
Stream<T> skip(long n);

// 连接流
static <T> Stream<T> concat(Stream<? extends T>a, Stream<? extends T>b);
```

还有其他方法，比如去重，排序，计数，结果归并，匹配流


**结果归并**

结果归并 reduce 函数属于终端调用，返回一个统计结果对象，min，max 其实就是用 reduce 对流中的数据做了统计最后只返回一个统计结果

**匹配流**

- 是否匹配任一元素：anyMatch
- 是否匹配所有元素：allMatch
- 是否未匹配所有元素：noneMatch
- 获取任一元素：findAny

**代码示例**

```java
// 筛选出年龄不小于 24 的人，并输出
public static void main(String[] args) {
    User user1 = new User("1", "zhangsan", 24);
    User user2 = new User("2", "lisi", 15);
    User user3 = new User("3", "wangwu", 26);
    User user4 = new User("4", "zhaoliu", 14);
    User user5 = new User("5", "tianqi", 28);

    List<User> list = Arrays.asList(user1, user2, user3, user4, user5);

    list.stream()
        .filter(u -> u.getAge() >= 24)
        .forEach(System.out::println);

}
```


注：`stream` 对象在调用一个方法后流会**被关闭**，方法也会**返回一个新的流对象**，新的流对象是开启状态的。

只有开启状态的流对象才能被使用。使用已经关闭了的流会报 `IllegalStateException: stream has already been operated upon or closed` 异常


## 并行流 - 不建议轻易使用

集合获取并行流的方式是 `collection.parallelStream();`

普通流转并行流的方式是 `stream.parallel();`

使用并行流就能进行并行计算（多线程计算），多线程下就可能出现线程不安全

## 终结操作

返回值是非流值的流方法。例如 `count`，`max`，`min`。很多返回值都是 `Optional` 类型或 `boolean` 类型

# Optional

> `Optional<T>` 对象是一种包装器对象，要么包装了类型 T 的对象，要么没有包装任何对象。对于第一种情况，我们称这种值为存在的。`Optional<T>`类型被当作一种更安全的方式，用来替代类型`T`的引用，这种引用要么引用某个对象，要么为`null`。但是，它只有在正确使用的情况下才会更安全

## Optional 是什么

`Optiona<T>` 是一个包装类，我们使用 `Optional<T>` 对象间接的操作原对象。

此类有两个成员变量

```java
// 被Optional包装的对象
private final T value;

// 一个 value 为 null 的 Optional 对象，作为一个工具。当需要返回以 vaule 为空的 Optional 对象时会使用到此变量
private static final Optional<?> EMPTY = new Optional<>();
```


## 创建 Optional 对象

创建 `Optional` 对象的方式有两种

- 流操作的返回值是 `Optional` 对象
- 调用 `Optional` 的静态工厂方法

`Optional` 对象的两个构造函数都是 `private` 的，所以我们只能通过，`of` 和  `ofNullable` 两个静态方法去构造 `Optional` 对象。

```java
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);
}

private Optional(T value) {
    this.value = Objects.requireNonNull(value);
}

public static <T> Optional<T> ofNullable(T value) {
    return value == null ? empty() : of(value);
}
```

`of` 方法调用的构造器存在抛空指针异常，所以显然 `ofNullable` 方法更安全

```java
private Optional(T value) {
    this.value = Objects.requireNonNull(value);
}

public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}

public static<T> Optional<T> empty() {
    @SuppressWarnings("unchecked")
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}
```


## 常用 API

接下来聊聊 `Optional` 的常用方法

`ifPresent` 判空

`get` 获取包装的对象

`map`，`flatMap` 将 value 作为参数执行函数行接口的方法

`orElse`，`orElseGet`，`orElseThrow` 判空并返回指定对象

`filter` 将 value 作为参数执行断言型接口的方法


`ifPresent`

```java
// 判空方法
public boolean isPresent() {
    return value != null;
}

public void ifPresent(Consumer<? super T> consumer) {
    if (value != null)
        consumer.accept(value);
}
```

```java
// 举例
// 创建一个 Optional 对象
Optional<User> optional = Optional.ofNullable(user);
System.out.println(optional.ifPresent()); // 返回user是否为空
optional.ifPresent(u->System.out.println(u.getName()));
```


`get`

```java
// 返回包装的对象，但会抛异常，不建议使用
public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```


`map`，`flatMap`

```java
// 对value执行函数型接口方法
// 通常参数mapper不为空，所以不需要担心此方法会抛出异常
public <U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Optional.ofNullable(mapper.apply(value));
    }
}

// 此方法和上述方法的不同点在于方法的参数和返回值不同，不过区别不大
public <U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Objects.requireNonNull(mapper.apply(value));
    }
}

// 举例
Optional<String> nameOptional = Optional.ofNullable(u).map(u->u.getName);
```


使用 map + orElse / orElseGet / orElseThrow 来完成大部分的判空链式操作

`orElse`，`orElseGet`，`orElseThrow`

```java
// value 为空，返回 other
public T orElse(T other) {
    return value != null ? value : other;
}

// vaule 为空，返回供给型接口方法的返回值
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}

// value为空，抛指定的异常
 public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
     if (value != null) {
         return value;
     } else {
         throw exceptionSupplier.get();
     }
 }

// 举例
User u = Optional.ofNullable(u).orElse(new User());
User u = Optional.ofNullable(u).orElseGet(()->new User());
User u = Optional.ofNullable(u).orElseThrow(()->new NullPointerException());
```


`filter`

```java
// 对 value 执行断言型接口方法
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent())
        return this;
    else
        return predicate.test(value) ? this : empty();
}

// 举例
Optional<User> result = Optional.ofNullable(user).filter(u->!"zhangsan".equals(u.getName()));
```


## 使用场景

- 级联调用。比如 `obj.getA().getB().getC();` 一连串调用， 易产生 NPE
- 给实体类属性赋值。比如 `a.setField(b.getField);` 防止 b 的属性值是 `null`

# 流转映射

之前说的是流和集合之间的转换

```java
// Collectors.toMap 需要两个 Function，一个为map提供k，一个为map提供v
// 如果需要v就是它本身可以写e->e（Function.identity()实现了它）
Map<Integer, Person> idToPerson = people.collect(Collectors.toMap(Person::getId, Function.identity());
                                                 
// 如果有重复的v，流执行时会报错。可以通过为toMap传第三个参数（BinaryOperator）解决
// BinaryOperator是函数式接口，它继承了BiFunction
public static <T, K, U>
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper,
                                BinaryOperator<U> mergeFunction)
```

ps：`toMap` 的第四个参数是 `Supplier`，可以用构造器引用指定映射的具体类型

流转 `ConcurrentHashMap` 映射的方法 `Collectors.toConcurrentMap`


## 群组和分区

```java
Collector.groupingBy
    
// k是Collector.groupingBy方法中传的Function的方法返回值；v是流中元素组成List
Map<String,List<Locale>> countryTolocales = locales.collect(Collectors.groupingBy(Locale::getCountry));
```

`groupingBy` 的第一个函数是分类函数

**补充**

- 当分类函数是断言函数（即返回 `boolean` 值的函数）时，流的元素可以分区为两个列表∶该函数返回 true 的元素和其他的元素。在这种情况下，使用`partitioningBy`比使用 `groupingBy` 要更高效
- `groupingByConcurrent` 返回的 `Map` 类型是 `ConcurrentHashMap`


## 下游收集器

`groupingBy` 的第二个函数是下游收集器

```java
Map<String,List<Locale>> countryTolocales = locales.collect(Collectors.groupingBy(Locale::getCountry);
```

此语句的分类函数将分类的数据放到 `List` 中，因为 `groupingBy` 默认使用 `List` 作为下游收集器。可以通过传递一个 `Collector` 作为下游收集器

举个例子

```java
// 将分类后的数据放到一个Set里
Map<String, Set<Locale>> countryToLocaleSet = locales.collect(Collectors.groupingBy(Locale::getCountry, Collectors.toSet()));

// counting 会产生收集到的元素的个数。所以这里的下游收集器就是一个Long类型的变量
Map<String, Long>countryToLocaleCounts =locales.collect(Collectors.groupingBy(Locale::getCountry, Collectors.counting()));

// 下游收集器是一个Integer类型的变量。变量值为Collectors.summingInt中指定的函数的返回值的累加和
Map<String, Integer> stateToCityPopulation = cities.collect(Collectors.groupingBy(City::getState, Collectors.summingInt(City::getPopulation));
                                                            
// ps: maxBy和minBy会接受一个比较器，并产生下游元素中的最大值和最小值
```

`mapping` 方法会产生将函数应用到下游结果上的收集器，并将函数值传递给另一个收集器

```java
Map<String, Optional<String>> stateTolongestCityName = cities.collet(
	groupingBy(City::getState, // 根据State分类
		mapping(City::getName, // 再次指定一个分类函数和下游收集器
                maxBy(Comparator.comparing(String::length)))));
```

[一些使用建议](https://lw900925.github.io/java/java8-optional.html)

https://cloud.tencent.com/developer/article/1596865

## 约简操作

> `reduce` 方法是一种用于从流中计算某个值的通用机制，其最简单的形式将接受一个二元函数，并从前两个元素开始持续应用它。

```java
// 计算values中所有值的累加和
List<Integer> values =...;
Optional<Integer> sum = values.stream().reduce(x, y)->x+y));
```


# 基本类型流和并行流

基本类型流里能直接包含基本类型数据，而不是对象

如：`IntStream`，`LongStream`

创建基本类型流方式如下

```java
IntStream stream= IntStream.of(1, 1, 2, 3, 5);
stream= Arrays.stream(values, from, to); // values is an int[] array

Stream<String> words =.,.;
IntStream lengths = words.mapToInt(String::length));
```

将基本类型流转换为对象流

```java
// boxed方法 基本类型流->对象流
Stream<Integer> integers = IntStream.range(0,100).boxed();
```


并行流是多线程计算，默认情况下并行计算和顺序计算的结果是相同的。

但是有些操作中，如果并行计算可以打乱原有元素的顺序的话，效率会有所提高。例如：`distinct`，`limit`

`.parallelStream.unordered()` 可以让并行流放弃维护原有元素的顺序，从而提高一些操作的速度


# 其他

https://blog.csdn.net/qq_43570650/article/details/121687883?spm=1001.2014.3001.5501