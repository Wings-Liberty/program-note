#还没有复习 

大纲：

- 概述
- 语法
- 实战


本笔记参考[这里](https://juejin.cn/post/6844904013683507207)

# 概述

ognl，全称 Object Graphic Navigation Language(对象图导航语言)，根据约定的一些简单的规则，组装一个利于阅读、可执行的表达式语句

如下面是一个典型的表达式

```java
"name".toCharArray()[0].numericValue.toString()
```

即便完全不懂 ognl，单纯的以 java 的基础知识就可以看懂，而这就是 ognl 的魅力所在 （学习一点点东西，就可以马上入手）

java 中，一切都是对象，所以 ognl 表达式必然是着手于某一个对象的

通常在 ognl 中，可以将待执行目标对象划分为三类

-   简单对象：(如基本数据类型，String)
-   非简单对象：(非简单对象，实例访问)
-   静态对象：(静态类)

# 基础语法


## 访问对象

在 ognl 中，待执行目标对象划分为三类

-   简单对象：(如基本数据类型，String)
-   非简单对象：(非简单对象，实例对象)
-   静态对象：(静态类)

**ognl 中也有 `null` 值**

在 ognl 的语法中，上面三种 case，根据不同的开头来标记

### 访问静态类对象

简单来说就是我想访问静态类的某个方法（或者静态类的成员）就要先访问类对象，再访问静态方法

```java
@java.lang.Math
```

语法规则是以 `@` 开头，后面接上完整的类名

一个实例访问静态方法的 case 如下，相当于 java 代码中直接调用 `Math.max(10, 20)`

```less
@java.lang.Math@max(10, 20)
```


### 访问非简单对象

ognl 的非简单对象是 ognl上下文中的对象，不是当前 Java 程序的堆内存里的对象

ognl 访问一个对象就是用对象名

访问一个普通对象的成员的 case 如下

```java
#demo
```

语法规则是以 `#` 开头，后面为对象名（说明，这个对象需要在 Ognl 的上下文中，且可以根据对象名可以唯一定位）

在 ognl 里对象都是全局级别的

### 简单对象

即基本类型的对象访问，不加任何前缀，直接使用即可，如下

```java
// 字符串的长度
"name".length()

// 数字计算
1+2

// boolean
true
```


## 方法调用


ognl 把方法分为静态方法，非简单对象方法和简单对象方法

```java
// 非基本对象的方法访问，# 开头，对象与方法之间用.连接
#obj.method( 参数 )
// 静态对象的方法访问，@ 开头，对象与方法之间用 @ 连接
@xxx@method( 参数 ) 
// 基本对象的方法访问，和非基本对象方法方式一致
"name".length()
```


## 访问对象的成员属性

访问目标对象的成员，规则如下

```java
// 非基本对象的成员访问，#开头，对象与成员之间用.连接
#obj.field

// 静态对象的成员访问，@开头，对象与成员之间用@连接
@xxx@field

// 基本对象的成员访问，和非基本对象成员方式一致
"name".hash
```


## 创建集合

ognl 针对常用的集合进行了特殊的支持

### 创建 List

- 通过 `{}` 创建列表
- 通过 `[]` 来访问对象下标的元素

下面表示创建一个列表，有三个元素: 1，2，3；获取列表中下标为 2 的元素

```java
{1, 2, 3}[2]
```



### 创建 Arrays

数组，可以结合 new 来使用

```java
new int[] {1,2,3}

new int[5] // 默认每个元素的值都是 0
```

array 自带一些属性，比如 `array.length`


> [!warning] ognl 中如果下标越界，也会抛下标越界异常


### 创建 Map

- `#{k:v, k:v}` 方式来创建 map
- 通过 `[key]` 访问元素

下面的语句，表示创建一个 map，并获取其中 key 为 name 的元素

```java
#{ "name" : "一灰灰Blog", "age" :  18}["name"]
```


> [!NOTE] 也能指定 Map 的实现类
> ```java
> #@java.util.LinkedHashMap@{ "foo" : "foo value", "bar" : "bar value" }
> ```



## 表达式语句

前面是一些简单的，基本的成员访问，方法调用，除此之外还存在更牛逼的用法，支持表达式的执行

### 成员赋值

```java
#demo.name = "一灰灰blog"
```

### 表达式计算

```java
500 + 20 - 30 * 3
```

### 三目运算符

```java
"name".length() % 2 == 0 ? "偶数长度" : "奇数长度"
```

### 集合支持

针对集合的遍历做了一些简化，方便调用

```java
// in 语句，判断列表中是否包含。也支持 not in 关键字
"name" in {"name", "hello"}

// 遍历集合，获取元素里对象的指定属性（map 函数）
list.{name}

// 遍历集合，把元素全转为 string（map 函数）
objects.{ #this instanceof String ? #this : #this.toString()}

// 遍历集合，获取所有的偶数（filter 函数）
{1,2,3,4,5,6}.{? #this % 2 == 0}

// 遍历集合，获取第一个满足条件的元素（filter 函数）
{1,2,3,4,5,6}.{^ #this % 2 == 0}

// 遍历集合，获取最后一个满足条件的元素（filter 函数）
{1,2,3,4,5,6}.{$ #this % 2 == 0}
```

另外，每种集合都有一些内置的属性方便访问

- Map，List，Set 的实现类都有 size，isEmpty 属性
- List 有 iterator 属性
- Map 有 keys（Set 类型对象）属性，values 属性
- Set 有 iterator 属性
- Iterator 有 next 和 hasNext 属性
- Enumeration 有 next，hasNext

### 对象创建

可以直接通过 new 来创建一个对象，当我们需要执行的目标方法的参数为非基本类型时，可能会非常好用

```javascript
// new + 完整的类名
new java.lang.String("hello world")
```

### 逗号运算符 - 链式语句

什么是链式语句呢？

有点类似设计模式中的 Builder 模式，我要执行一串的操作，最后获取目标

> 定义规则如下，圆括号包裹起来，中间用逗号分隔，依次执行，最后一个为需要返回的目标

```scss
(step1, step2,..., result)
```

结合上面的对象创建，可以实现非常强大的功能

```java
package git.hui;
class User {
  public String name;
  public Integer age;
}
```

直接创建一个可用的 User 对象，下面执行完毕之后，直接获取一个属性被初始化后的 User 对象

```java
(#user=new git.hui.User(), #user.name="一灰灰Blog", #user.age=18, #user)
```


这种逗号运算符用法在 Java 没有

如果调用有两个参数的方法，Java 和 ognl 会这样用

```java
obj.method1(param1, param2)
```

链式语句中，小括号里的最后一个参数是整个括号内的返回值。比如我想让第二个参数做一些事后再作为参数传给 `method1`

```java
obj.method1(param1, (param2.doSomething(), param2));
```

### this 对象

OGNL 会把当前对象存储在 `this` 变量中，表达式求值的每个点运算符后，可以像任何其他变量一样使用 `this`

例如，如果一个对象的方法返回值大于100，则返回两倍，否则返回20倍

```java
listeners.size().(#this > 100? 2*#this : 20+#this)
```

### lambda 表达式

这个有点高端了，首先是定义 lambda 表达式，然后借助前面的链式方式调用，下面是一个阶乘的 case

```kotlin
#fact = :[#this<=1? 1 : #this*#fact(#this-1)], #fact(3)
```

## 根对象

ognl 上下文实际上是一个 Map 结构，所有的对象都被放在这里

ognl 还内置了一个 root 对象，如果表达式没有指定被操作的对象，默认就是从 root 里获取

比如访问 `#name` ，其实就是在访问 root 的 name 属性。方法也是

```java
headline.parent.(ensureLoaded(), name)
```

这个方法中访问了 root 对象的 headline.parent 属性，然后又访问了 parent 的一个属性，这个属性的名字是 `(ensureLoaded(), name)` 的返回值

这个链式语句会执行 root 对象的 `ensureLoaded()` 方法，然后返回 root 对象的 name 属性

root 对象有一些内置的方法和属性，比如

- `ensureLoaded()`，用来确保对象被完整加载到内存里

## 对象类型转换为布尔类型的规则

对象被转化为布尔值时

- 0 被转为 `false`，非 0 是 `true`
- 非空字符是 `true`，其他都是 `false`
- 非空对象都是 `true`，其他都是 `false`

# 使用技巧

## 调用静态变量的方法

静态变量会指向一个对象，通过访问静态变量调用方法时，调用 `@` 访问静态对象，调用 `.` 访问静态对象的方法，`@` 使用来访问静态对象和静态方法的，不要混淆

比如

```java
@java.lang.System@out.println("hello")
```

`out` 是静态对象，但是 `println` 是实例方法

## ognl 表达式返回多个值

比如想让一个 ognl 表达式返回多个值帮助定位问题

用逗号运算符和数组方式，让最后一个逗号后的返回值作为 ognl 的返回值，并构造一个 List 把多个返回值放进去实现 ognl 返回多个值

```sh
$ ognl '#value1=@System@getProperty("java.home"), #value2=@System@getProperty("java.runtime.name"), {#value1, #value2}'
@ArrayList[
    @String[/opt/java/8.0.181-zulu/jre],
    @String[OpenJDK Runtime Environment],
]
```