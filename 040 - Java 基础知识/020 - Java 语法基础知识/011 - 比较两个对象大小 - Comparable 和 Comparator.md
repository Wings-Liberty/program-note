
# 比较两个对象大小是什么意思

对有序集合里的元素排序时，需要指定一个排序规则：就是用来判断两个对象进行比较时谁大谁小的规则。举几个例子

- `int` 类型的自然排序规则是自然数的比较大小
- 字符串的默认排序规则是对字符串编码后的数字比较规则

但是两个对象大小一样不代表 `equals` 相关方法也会返回 `true`


# 怎么比较两个对象的大小

当需要比较两个对象的大小时，通常用 JDK 提供的这两种方式

- `Comparable` 接口提供了 `int compareTo(T o)` 方法，根据返回值
- `Comparator` 接口提供了 `int compare(T o1, T o2)` 方法

她们规定根据返回值判断两个对象的大小。如果把调用 `Comparable` 接口 `compareTo` 方法的对象和 `Comparator` 接口 `compare` 方法传入的第一个参数视为参与比较的第一个对象，另一个视为第二个对象，那么返回值的含义就是

|返回值|含义|
|:--|:--|
|0|两个对象大小相等|
|大于 0|第一个对象 > 第二个对象 |
|小于 0|第一个对象 < 第二个对象|

如果只关心两个对象谁大对小，只需要关心返回值是大于 0 还是小于 0

如果返回值不等于 0，它的值具体是几，以及返回值是 1 或者是 n 的含义是什么均和具体实现有关


# Comparable 是什么

如果字面意思，实现这个接口的类表示根据这个类创建的多个之间可以比较大小

一般不会直接用对象调用 `.compareTo` 方法，而是把对象放到集合里，然后让集合里的元素根据 `.compareTo` 方法比较大小再令其变成有序集合

```java
Integer t1 = 1;  
Integer t2 = 2;  
// 输出：-1
System.out.println(t1.compareTo(t2));
```

# Comparator 是什么

它是比较器，为它创建一个实现类就相当于为一个类指定了一个比较大小的规则。举个简单的例子，为 `Integer` 创建一个比较器对象 `c1`，就能用 `c1.compare()` 比较两个 `Integer` 对象的大小

不过 `Comparator` 本质上还是调用 `Comparable` 的 `compareTo` 方法进行的比较大小

如果想用人的名字长度作为比较规则对 `List<User>` 里的元素排序，要么让 `User` 这个类实现 `Comparable` 接口，令其具有比较大小的能力；要么创建 `Comparator` 的实现类，定义 `User` 对象比较大小的规则

通俗点说：`Comparable` 是为了让对象本身具有比较大小的能力，`Comparator` 是创建一个裁判，让这个裁判判断两个对象谁打谁小，但这个对象自己可能没有比较大小的能力

# 怎么创建和使用 Comparator

为了快速创建比较器，`Comparator` 提供了一些静态方法直接创建简单的比较器

![[../../020 - 附件文件夹/Pasted image 20230521121724.png]]

比如：

```java
// 让 users 里的元素根据 User 的 ID 排序，因为 ID 是 Integer，所以默认用 Integer 的 compareTo 方法
Arrays.sort(users, Comparator.comparing(User::getId));

// 让 users 里的元素现根据 User 的 name 排序，再根据 ID 排序。并且自定义 name 的排序规则是根据 name 的长度排序
Arrays.sort(users, Comparator.comparing(User::getName, Comparator.comparingInt(String::length)));

```

```java
// 自然排序指直接调用指定类的 compare 方法
Arrays.sort(users, Comparator.naturalOrder());
// 逆序就是对 compare 结果取反后排序
Arrays.sort(users, Comparator.reverseOrder());
```

创建好比较器后还能调用成员方法继续定制它，设计更复杂的比较规则

比如创建一个根据名字长度排序的比较器，还可以继续设置如果名字长度相等再比较另一个字段

```java
Arrays.sort(users, Comparator.comparing(User::getName, Comparator.comparingInt(String::length)).thenComparing(User::getId));
```

如果想做做逆序排序，调用成员方法 `reverse`，这个方法是成员方法，不是静态方法，注意和 `reverseOrder` 的不同

