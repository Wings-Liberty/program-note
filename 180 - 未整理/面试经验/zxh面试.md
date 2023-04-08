# javase知识面试题

**Java语言是编译型还是解释型语言？**

**答：**Java的执行经历了编译和解释的过程，是一种**先编译，后解释**执行的语言，不可以单纯归到编译性或者解释性语言的类别中。

## 1.基础知识

### 1.java常量和变量之间的区别

常量：

![image-20201125214926169](C:\Users\33066\AppData\Roaming\Typora\typora-user-images\image-20201125214926169.png)



变量：

![image-20201125214954730](C:\Users\33066\AppData\Roaming\Typora\typora-user-images\image-20201125214954730.png)



### 2.Integer与int的区别

int是基本数据类型，Integer是java为int提供的封装类。

int的默认值为0，而Integer的默认值为null；

int则无法表达出未赋值的情况，例如，要想表达出没有参加考试和考试成绩为0的区别，则只能使用Integer。在JSP开发中，Integer的默认为null，所以用el表达式在文本框中显示时，值为空白字符串，而int默认的默认值为0，所以用el表达式在文本框中显示时，结果为0，也就是说int不适合作为web层的表单数据的类型。

另外，Integer提供了多个与整数相关的操作方法，Integer中还定义了表示整数的最大值和最小值的常量。



### 3.== 和 equals 的区别是什么？

==：是直接比较的两个对象的堆内存地址，如果相等，则说明这两个引用实际是指向同一个对象地址的。

equals：比较的是引用数据类型，比较的是引用数据类型，不同类型有不同的equals方法，根据不同的数据类型调用不同的equals方法。

==是判断两个人是不是住在同一个地址，而equals是判断同一个地址里住的人是不是同一个

```java
@Test
public void test() throws SQLException {
    int a01 = 1;
    int b01 = 1;
    System.out.println(a01==b01); //true

    Integer a02 = 127;
    Integer b02 = 127;
    System.out.println(a02.equals(b02)); //true
    System.out.println(a02 == b02);  //true

    Integer a03 = 128;
    Integer b03 = 128;
    System.out.println(a02.equals(b02)); //true
    System.out.println(a03 == b03);  //false

    Integer a04=new Integer(127);
    Integer b04=new Integer(127);
    System.out.println(a04 == b04); //false
    System.out.println(a04.equals(b04)); //true

    String a05 = "a";
    String b05 = "a";
    System.out.println(a05 == b05); //true
    System.out.println(a05.equals(b05)); //true

    String a06 = "a";
    String b06 = new String("a");
    System.out.println(a06 == b06);  //false
    System.out.println(a06.equals(b06));   //true
    
     String arr = new String("abc");
     String arr01  = new String("abc");
     System.out.println(arr.equals(arr01));  //true
    
}
```

equals源码：

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```



#### 为什么 == 的时候Integer为128不相等？

Integer源码分析：

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```



### 4.两个对象的 hashCode()相同，则 equals()也一定为 true，对吗？

不对，两个对象的 hashCode()相同，equals()不一定 true。

```java
String str1 = "通话";
String str2 = "重地";
System.out.println(String.format("str1：%d | str2：%d",  str1.hashCode(),str2.hashCode()));
System.out.println(str1.equals(str2));
```

```java
public int hashCode() {
    int h = this.hash;
    if (h == 0 && this.value.length > 0) {
        this.hash = h = this.isLatin1() ? StringLatin1.hashCode(this.value) : StringUTF16.hashCode(this.value);
    }
    return h;
}
```

```java
public static int hashCode(byte[] value) {
    int h = 0;
    int length = value.length >> 1;

    for(int i = 0; i < length; ++i) {
        h = 31 * h + getChar(value, i);
    }
    return h;
}
```

运行结果

> str1：1179395 | str2：1179395
>
> false

代码解读：很显然“通话”和“重地”的 hashCode() 相同，然而 equals() 则为 false，因为在散列表中，hashCode()相等即两个键值对的哈希值相等，然而哈希值相等，并不一定能得出键值对相等。



#### 4.1 hashcode是什么？

​		答：就是在hash表中对应的位置

​		hashcode是通过hash函数得来的，通俗的说，就是通过某一种算法得到的

​		通过对象的内存地址（也就是物理地址）转换成一个整数，然后该整数通过hash函数的算法得到hashcode

#### String.hashcode源码：

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

这里有个数字 **31 ，**为什么选择31作为乘积因子，而且没有用一个常量来声明？主要原因有两个：

　　①、31是一个不大不小的质数，是作为 hashCode 乘子的优选	质数之一。

　　②、31可以被 JVM 优化，`31 * i = (i << 5) - i。因为移位运算比乘法运行更快更省性能。`



### 5.如何理解HashCode的作用：

从Object角度看，JVM每new一个Object，它都会将这个Object丢到一个Hash表中去，这样的话，下次做Object的比较或者取这个对象的时候（读取过程），它会根据对象的HashCode再从Hash表中取这个对象。这样做的目的是提高取对象的效率。若HashCode相同再去调用equal。



##### 3.1.1加法hash

 所谓的加法Hash就是把输入元素一个一个的加起来构成最后的结果。

 这里的prime是任意的质数，看得出，结果的值域为[0,prime-1]。

```java
static int additiveHash(String key, int prime){
  	int hash, i;
  	for (hash = key.length(), i = 0; i < key.length(); i++)
   	hash += key.charAt(i);
  	return (hash % prime);
 }
```



#### 5.1 hashCode()与 equals()

**1）对于HashCode的理解？**

①java8默认是通过和当前线程有关的一个随机数+三个确定值

②String.hashCode底层：①根据String.value每一个字符计算哈希吗

​											 ②有个常数31*hash+ASCII码，合适的素数为了减少碰撞

③tostring方法也与hashcode方法有关，不重写时返回的是hashcode的16进制表达形式

**2)为什么重写 equals 时必须重写 hashCode 方法？**

①两个对象相等，hashcode 一定相同。

②两个对象相等， equals 方法都返回 true。

③但是，两个对有相同的 hashcode 值，它们也不一定是相等的 。

注意：hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

**3）为什么两个对象有相同的 hashcode 值，它们也不一定是相等的？**

因为 `hashCode()` 使用杂凑算法，越糟糕的杂凑算法越容易碰撞。

我们刚刚也提到了 `HashSet`,如果 `HashSet` 在对比的时候，同样的 hashcode 有多个对象，它会使用 `equals()` 来判断是否真的相同。也就是说 `hashcode` 只是用来缩小查找成本。



### 6.java创建对象的几种方式

1.用new语句创建对象，这是最常用的创建对象的方式。

2.运用反射手段，调用Java.lang.Class或者java.lang.reflect.Constructor类的newInstance()实例方法。

3.调用对象的clone()方法。

4.运用反序列化手段，调用java.io.ObjectInputStream对象的readObject()方法.



利用反射创建对象：

```java
class Per{
    public String name;
    public int age;
    public Per(){
    }

    public Per(String name){
    }

    public String toString() {
        return "对象！！！";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
public class Demo04 {
    public static void main(String[] args) throws Exception{

        Class<Per> c = Per.class;

        Constructor<Per> con = c.getDeclaredConstructor();

        Per p = con.newInstance();

        /**
         * 从JVM的角度看，我们使用关键字new创建一个类的时候，
         * 这个类可以没有被加载。但是使用newInstance()方法的时候，
         * 就必须保证：
         * 1、这个 类已经加载；
         * 2、这个类已经连接了。而完成上面两个步骤的正是Class的静态方法forName()所完成的，
         * 这个静态方法调用了启动类加载器，即加载 java API的那个加载器。
         */
        p.setName("zhangsan");
        System.out.println(p.getName());
    }
}
```



clone方法创建对象：

```java
Person p = new Person(23, "zhang");
Person p1 = (Person) p.clone();

System.out.println(p);
System.out.println(p1);
```



序列化的方法创建对象：

```java
package org.westos.Demo;

import java.io.*;

public class Demo2 {
    public static void main(String[] args) throws IOException {
    ObjectOutputStream obji = new ObjectOutputStream(new FileOutputStream("Object1.txt"));
    Teacher teacher = new Teacher();
    teacher.setName("张三");
    teacher.setAge(20);
    obji.writeObject(teacher); 
	}
}
```





```java
package org.westos.Demo;

import java.io.Serializable;

public class Teacher implements Serializable {
    private  String name;
    public Teacher() {
    }

    public Teacher(String name, int age) {
        this.name = name;
        this.age = age;
    }

    private  int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}	
```



###  7.2*8的最高效率算法

使用位运算来实现效率最高。位运算符是对操作数以二进制比特位为单位进行操作和运算，操作数和结果都是整型  数。

对于位运算符“<<”,  是将一个数左移n位，就相当于乘以了2的n次方，那么，一个数乘以8只要将其左移3位即可，位运算cpu直接支持的，效率最高。所以，2乘以8等于几的最效率的方法是2 << 3



### 8.ﬁnal和abstract关键字的作用

abstract可以用来修饰类和方法，不能用来修饰属性和构造方法；

使用abstract修饰的类是抽象类，需要被继承，使用abstract修饰的方法是抽象方法，需要子类被重写。



ﬁnal可以用来修饰类、方法和属性，不能修饰构造方法。

使用ﬁnal修饰的类不能被继承，使用ﬁnal修饰的方法不能被重写，使用ﬁnal修饰的变量的值不能被修改，所以就成了常量。



特别注意：ﬁnal修饰基本类型变量，其值不能改变，由原来的变量变为常量；

但是ﬁnal修饰引用类型变量，栈内存中的引用不能改变，但是所指向的堆内存中的对象的属性值仍旧可以改变。



### 9. final 在 java 中有什么作用？

- final 修饰的类叫最终类，该类不能被继承。

- final 修饰的方法不能被重写。

- final 修饰的变量叫常量，*常量必须初始化*，初始化之后值就不能被修改。




### 10.ﬁnal、ﬁnally、ﬁnalize的区别

**ﬁnal**修饰符(关键字)如果一个类被声明为ﬁnal，意味着它不能再派生出新的子类，不能作为父类被继承例如：String   类、Math类等。将变量或方法声明为ﬁnal，可以保证它们在使用中不被改变。被声明为ﬁnal的变量必须在声明时给定初值，而在以后的引用中只能读取，不可修改。被声明为ﬁnal的方法也同样只能使用，不能重写，但是能够重载。使用ﬁnal修饰的对象，对象的引用地址不能变，但是对象的值可以变!

**ﬁnally**在异常处理时提供 ﬁnally 块来执行任何清除操作。如果有ﬁnally的话，则不管是否发生异常，ﬁnally语句都会被执行。一般情况下，都把关闭物理连接(IO流、数据库连接、Socket连接)等相关操作，放入到此代码块中.。

**ﬁnalize**方法名。Java 技术允许使用 ﬁnalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要清理工作。ﬁnalize() 方法是在垃圾收集器删除对象之前被调用的。它是在 Object  类中定义的f，因此所有的类都继承了它。子类覆盖 ﬁnalize() 方法以整理系统资源或者执行其他清理工作。 一般情况下，此方法由JVM调用，程序员不要去调用!



### 11.写出java.lang.Object类的六个常用方法

Object类常用的**六种**方法是**toString()**、**equals()**、**hashCode()**、**wait()**、**notify()**、**notifyAll()**。

(1) public boolean **equals**(java.lang.Object)比较对象的地址值是否相等，如果子类重写，则比较对象的内容是否相等；

(2) public native int **hashCode()** 获取哈希码

(3) public java.lang.String **toString()** 把数据转变成字符串

(4) public ﬁnal native java.lang.Class **getClass()** 获取类结构信息

(5) protected void **ﬁnalize()** throws java.lang.Throwable垃圾回收前执行的方法

(6)protected native Object **clone()** throws java.lang.CloneNotSupportedException 克 隆

(7)public ﬁnal void  **wait()** throws java.lang.InterruptedException多线程中等待功能

(8)public ﬁnal native void **notify()** 多线程中唤醒功能

(9)public ﬁnal native void **notifyAll()** 多线程中唤醒所有等待线程的功能



### 12. java 中的 Math.round(-1.5) 等于多少？

等于 -1，因为在数轴上取值时，中间值（0.5）向右取整，所以正 0.5 是往上取整，负 0.5 是直接舍弃。



### 13. String 属于基础的数据类型吗？

String 不属于基础类型

基础类型有 8 种：byte、boolean、char、short、int、float、long、double，

而 String 属于对象。~	



### 14. java 中操作字符串都有哪些类？它们之间有什么区别？

操作字符串的类有：String、StringBuffer、StringBuilder。

 String 声明的是不可变的对象，每次操作都会生成新的 String 对象，然后将指针指向新的 String 对象

 StringBuffer、StringBuilder 可以在原有对象的基础上进行操作，所以在经常改变字符串内容的情况下最好不要使用 String。

StringBuffer 和 StringBuilder 最大的区别在于

- String不可变线程安全，StringBuffer对方法加了同步锁， 是线程安全的，而 StringBuilder 是非线程安全的
- 但 StringBuilder 的性能却高于 StringBuffer
- String底层：private **final** char value[]（不可变字符数组）。Java9之后改用（不可变byte数组）
- String每次会产生新的对象，StringBuffer和StringBuilder不会。
- 在单线程环境下推荐使用 StringBuilder，多线程环境下推荐使用 StringBuffer。
- StringBuffer和数组的区别：二者都可以看成是一个容器装数据。但是，StringBuffer的数据最终是一个字符串数据，而数组可以放置任意类型的同一种数据。

  

### 15.double和long线程安全么

Java虚拟机规范定义的许多规则中的一条：所有对基本类型的操作，除了某些对long类型和double类型的操作之外，都是原子级的。目前的JVM（java虚拟机）都是将**32位**作为原子操作，并非64位。当线程把主存中的 long/double类型的值读到线程内存中时，可能是**两次32位值的写操作**，显而易见，如果几个线程同时操作，那么就可能会出现高低2个32位值出错的情况发生。要在线程间共享long与double字段是，必须在synchronized中操作，或是声明为volatile。



### 16. 如何将字符串反转？

```java
package com.wsheng.aggregator.algorithm.string;

import java.util.Stack;

/**
 * 8 种字符串反转的方法, 其实可以是9种方法，第9种是使用StringBuffer和StringBuilder中实现的方法
 * @author Josh Wang(Sheng)
 * @email  swang6@ebay.com
 */
public class StringReverse {

    /**
     * 二分递归地将后面的字符和前面的字符连接起来。
     * @param s
     * @return
     */
    public static String reverse1(String s) {
        int length = s.length();
        if (length <= 1)
            return s;
        String left = s.substring(0, length / 2);
        String right = s.substring(length / 2, length);
        return reverse1(right) + reverse1(left);
    }

    /**
     * 取得当前字符并和之前的字符append起来
     * @param s
     * @return
     */
    public static String reverse2(String s) {
        int length = s.length();
        String reverse = "";
        for (int i=0; i<length; i++)
            reverse = s.charAt(i) + reverse;
        return reverse;
    }

    /**
     * 将字符从后往前的append起来
     * @param s
     * @return
     */
    public static String reverse3(String s) {
        char[] array = s.toCharArray();
        String reverse = "";
        for (int i = array.length - 1; i >= 0; i--) {
            reverse += array[i];
        }
        return reverse;
    }

    /**
     * 和StringBuffer()一样，都用了Java自实现的方法，使用位移来实现
     * @param s
     * @return
     */
    public static String reverse4(String s) {
        return new StringBuilder(s).reverse().toString();
    }

    /**
     * 和StringBuilder()一样，都用了Java自实现的方法，使用位移来实现
     * @param s
     * @return
     */
    public static String reverse5(String s) {
        return new StringBuffer(s).reverse().toString();
    }

    /**
     * 二分交换，将后面的字符和前面对应的那个字符交换
     * @param s
     * @return
     */
    public static String reverse6(String s) {
        char[] array = s.toCharArray();
        int end = s.length() - 1;
        int halfLength = end / 2;
        for (int i = 0; i <= halfLength; i++) {
            char temp = array[i];
            array[i] = array[end-i];
            array[end-i] = temp;
        }
        return new String(array);
    }

    /**
     * 原理是使用异或交换字符串
     * a=a^b;
     * b=b^a;
     * a=b^a;
     *
     * @param s
     * @return
     */
    public static String reverse7(String s) {
        char[] array = s.toCharArray();
	
        int begin = 0;
        int end = s.length() - 1;

        while (begin < end) {
            array[begin] = (char) (array[begin] ^ array[end]);
            array[end] = (char) (array[end] ^ array[begin]);
            array[begin] = (char) (array[end] ^ array[begin]);
            begin++;
            end--;
        }
        return new String(array);
    }
    

    /**
     * 基于栈先进后出的原理
     *
     * @param s
     * @return
     */
    public static String reverse8(String s) {
        char[] array = s.toCharArray();
        Stack<Character> stack = new Stack<Character>();
        for (int i = 0; i < array.length; i++)
            stack.push(array[i]);

        String reverse = "";
        for (int i = 0; i < array.length; i++)
            reverse += stack.pop();
        return reverse;
    }

    public static void main(String[] args) {
        System.out.println(reverse1("Wang Sheng"));
        System.out.println(reverse2("Wang Sheng"));
        System.out.println(reverse3("Wang Sheng"));
        System.out.println(reverse4("Wang Sheng"));
        System.out.println(reverse5("Wang Sheng"));
        System.out.println(reverse6("Wang Sheng"));
        System.out.println(reverse7("Wang Sheng"));
        System.out.println(reverse8("Wang Sheng"));
    }
}
```



### 18. String 类的常用方法都有那些？

indexOf()：返回指定字符的索引。
charAt()：返回指定索引处的字符。
replace()：字符串替换。
trim()：去除字符串两端空白。
split()：分割字符串，返回一个分割后的字符串数组。
getBytes()：返回字符串的 byte 类型数组。
length()：返回字符串长度。
toLowerCase()：将字符串转成小写字母。
toUpperCase()：将字符串转成大写字符。
substring()：截取字符串。
equals()：字符串比较



### 19.String底层为什么要用final?

一.被final修饰的类,不可以被继承,所以不会别其它类改变,这样会更加的安全.

二.string是共享在常量池中的,String str="abc", char data[] = {'a', 'b', 'c'};是等价的,他们都放在了字符串常量池中.

​	

### 20.抽象类必须要有抽象方法吗？

不需要，抽象类不一定非要有抽象方法。

示例代码：

```java
abstract class Cat {
    public static void sayHi() {
        System.out.println("hi~");
    }
}
```



### 21.普通类和抽象类有哪些区别？

- 普通类不能包含抽象方法，抽象类可以包含抽象方法。
- 抽象类不能直接实例化，普通类可以直接实例化。



### 22.抽象类能使用 final 修饰吗？

不能，

定义抽象类就是让其他类继承的，如果定义为 final 该类就不能被继承，这样彼此就会产生矛盾，所以 final 不能修饰抽象



### 23.接口和抽象类有什么区别？

实现：

抽象类的子类使用 extends 来继承；接口必须使用 implements 来实现接口。
构造函数：抽象类可以有构造函数；接口不能有。
main 方法：抽象类可以有 main 方法，并且我们能运行它；接口不能有 main 方法。
实现数量：类可以实现很多个接口；但是只能继承一个抽象类。
访问修饰符：接口中的方法默认使用 public 修饰；抽象类中的方法可以是任意访问修饰符。



### 24. &和&&的区别？

虽然二者都要求运算符左右两端的布尔值都是true整个表达式的值才是true。

&&之所以称为短路运算是因为，如果&&左边的表达式的值是false，右边的表达式会被直接短路掉，不会进行运算。很多时候我们可能都需要用&&而不是&，例如在验证用户登录时判定用户名不是null而且不是空字符串，应当写为：username != null &&!username.equals(“”)，二者的顺序不能交换，更不能用&运算符，因为第一个条件如果不成立，根本不能进行字符串的equals比较，否则会产生NullPointerException异常。注意：逻辑或运算符（|）和短路或运算符（||）的差别也是如此。



### 25. switch 是否能作用在byte 上，是否能作用在long 上，是否能作用在String上？

在 switch（ expr1）中， expr1 只能是一个整数表达式或者枚举常量（更大字体），整数表达式可以是 int
基本类型或 Integer 包装类型，由于， byte,short,char 都可以隐含转换为 int，所以，这些类型以及这些类型
的包装类型也是可以的。显然， long 和 String 类型都不符合 switch 的语法规定，并且不能被隐式转换成 int
类型，所以，它们不能作用于 swtich 语句中。 另外由于 JDK1.7 中引入新特性，所以 swtich 语句可以接收
一个 String 类型的值， String 可以作用在 swtich 上。

基本数据类型除了long都是可以的，在jdk1.8之后String也是可以的





### 26、构造器（constructor）是否可被重写（override）？

![img](https://images0.cnblogs.com/blog2015/790669/201508/070003299403370.png)

答：构造器不能被继承，因此不能被重写，但可以被重载。

一个类会有一个默认的构造函数,这个构造函数没有内容也没有返回值,一般都回略去不写.这种情况下,编译器在编译的时候会默认加上一个无参且方法体为空的构造函数.但是,如果类的构造函数被重写了,如上例,Person类已经有了一个有参数有方法体的构造函数,这时编译器就不会再给它默认加上一个无参且方法体为空的构造函数.可以理解为无参的构造函数被覆盖了.这种情况称为没有默认构造函数.

而在函数的继承里,子类必须调用父类的构造函数。但是,子类只能继承父类的**默认**构造函数,如果父类没有默认的构造函数,那子类不能从父类继承默认构造函数.这时子类必须使用super来实现对父类的非默认构造函数的调用.

在创建对象时，先调用父类默认构造函数对对象进行初始化，然后调用子类自身自己定义的构造函数。

### 27. Java 重载与重写是什么？有什么区别？

答：

重载（Overload）是让类以统一的方式处理不同类型数据的一种手段，实质表现就是多个具有不同的参数个数或者类型的同名函数（返回值类型可随意，不能以返回类型作为重载函数的区分标准）同时存在于同一个类中，是一个类中多态性的一种表现（调用方法时通过传递不同参数个数和参数类型来决定具体使用哪个方法的多态性）。

重写（Override）是父类与子类之间的多态性，实质是对父类的函数进行重新定义，如果在子类中定义某方法与其父类有相同的名称和参数则该方法被重写，不过子类函数的访问修饰权限不能小于父类的；若子类中的方法与父类中的某一方法具有相同的方法名、返回类型和参数表，则新方法将覆盖原有的方法，如需父类中原有的方法则可使用 super 关键字



### 28.什么是 Java 序列化？什么情况下需要序列化？

其实序列化就是用来通信的，java对象序列化后可以很方便的存储或者网络中传输

​			序列化：把Java对象转化为字节序列

​			反序列化：把字节序列恢复为原先的Java对象



以下情况需要使用 Java 序列化：

- 想把的内存中的对象状态保存到一个文件中或者数据库中时候；
- 想用套接字在网络上传送对象的时候；
- 想通过RMI（远程方法调用）传输对象的时候。



序列化：

```java
public static void serialize(  ) throws IOException {

        Student student = new Student();
        student.setName("CodeSheep");
        student.setAge( 18 );
        student.setScore( 1000 );

        ObjectOutputStream objectOutputStream =
        new ObjectOutputStream( new FileOutputStream( new File("student.txt") ) );
        objectOutputStream.writeObject( student );
        objectOutputStream.close();

        System.out.println("序列化成功！已经生成student.txt文件");
        System.out.println("==============================================");
}
```

反序列化：

```java
public static void deserialize(  ) throws IOException, ClassNotFoundException {
        ObjectInputStream objectInputStream =
        new ObjectInputStream( new FileInputStream( new File("student.txt") ) );
        Student student = (Student) objectInputStream.readObject();
        objectInputStream.close();

        System.out.println("反序列化结果为：");
        System.out.println( student );
}
```

运行结果：

```
序列化成功！已经生成student.txt文件
==============================================
反序列化结果为：
Student:
name = CodeSheep
age = 18
score = 1000
```



问题一  如果使用ObjectOutPutStream方式序列化类里面一定要有serialVersionUID，否则旧数据会反序列化失败，

问题二 一旦序列化保存到磁盘操作后，就不要修改类名了，苟泽数据反序列化就会失败



### 29.Serializable接口有何用？

上面在定义`Student`类时，实现了一个`Serializable`接口，然而当我们点进`Serializable`接口内部查看，发现它**竟然是一个空接口**，并没有包含任何方法！

![img](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzrIJpQyicWIiagQBV75jzuSkicttzt9DsOcQgUcovoPYngBIhA9LbGnwYqKCHiastcIguGCApgicYtETCg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

试想，如果上面在定义`Student`类时忘了加`implements Serializable`时会发生什么呢？

实验结果是：此时的程序运行**会报错**，并抛出`NotSerializableException`异常：

![img](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzrIJpQyicWIiagQBV75jzuSkicD8cHzLepjkd6eAkNgA1TNQHX3523zC5HmFjT7T34we96n6BDQcD2JA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们按照错误提示，由源码一直跟到`ObjectOutputStream`的`writeObject0()`方法底层一看，才恍然大悟：

![img](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzrIJpQyicWIiagQBV75jzuSkicTxvcicQjC22DpAOXqgE0NmoU9QaXibhrQnoR3gBGvrOQ3F1WgribBSJZQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果一个对象既不是**字符串**、**数组**、**枚举**，而且也没有实现`Serializable`接口的话，在序列化时就会抛出`NotSerializableException`异常！

哦，我明白了！

原来`Serializable`接口也仅仅只是做一个标记用！！！

它告诉代码只要是实现了`Serializable`接口的类都是可以被序列化的！然而真正的序列化动作不需要靠它完成。



### 30. forward 和 redirect 的区别？

forward 是转发 和 redirect 是重定向：

- 地址栏 url 显示：foward url 不会发生改变，redirect url 会发生改变；

- 数据共享：forward 可以共享 request 里的数据，redirect 不能共享；

- 效率：forward 比 redirect 效率高。

  

### 31. 什么是反射？

反射是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；

对于任意一个对象，都能够调用它的任意一个方法和属性；

这种动态获取的信息以及动态调用对象的方法的功能称为 Java 语言的反射机制。



### 32.动态代理是什么？有哪些应用？

动态代理是运行时动态生成代理类。

动态代理的应用有 spring aop、hibernate 数据查询、测试框架的后端 mock、rpc，Java注解对象获取等。



### 33.面向对象三大特性-封装继承多态

**答：**面向对象是一种思想，可以将复杂问题简单化，让我们从执行者变为了指挥者。

面向对象的三大特性为：**封装，继承与多态。**

- **封装：**将事物封装成一个类，**减少耦合，隐藏细节**。保留特定的接口与外界联系，当接口内部发生改变时，不会影响外部调用方。
- **继承：**从一个已知的类中**派生出一个新的类**，新类可以拥有已知类的行为和属性，并且可以**通过覆盖/重写来增强**已知类的能力。
- **多态：**多态的本质就是一个程序中存在多个同名的不同方法，主要通过三种方式来实现：
  - 通过子类对父类的**覆盖**来实现
  - 通过在一个类中对方法的**重载**来实现
  - 通过将**子类对象作为父类对象**使用来实现

**解析:**

​		这算是一个相当基础的问题，面向对象的思想以及其**三大特性**我们均需要有较好的理解。接下来，我们对三大特性进行一个详细的阐述与解析吧。



#### 33.1关于封装

封装主要是为了增加程序的可读性，解耦合并且隐藏部分实现细节。让我们来看下边的案例，看看该如何实现封装。

**案例：**

```java
package com.pak1;

public class Test {
    public static void main(String[] args) {
        Student student = new Student();
        student.name = "小明";
        student.age = 16;
        student.printStudentAge();

        Student student2 = new Student();
        student2.name = "小白";
        student2.age = 120;
        student2.printStudentAge();
     }
}

class Student {
    String name;
    int age;

    public void printStudentAge() {
        System.out.println(name + "同学的年龄：" + age);
    }
}
```

**程序输出如下：**

![图片说明](https://uploadfiles.nowcoder.com/images/20191109/5459305_1573290170415_4A47A0DB6E60853DEDFCFDF08A5CA249)

我们看到小白同学的年龄120，（假设）不符合业务逻辑需要，所以我们需要做一些内部逻辑的处理。所以需要进行代码封装，将内部逻辑进行一个隐藏。

**封装之后的代码如下：**

```java
package com.pak1;

public class Test {
    public static void main(String[] args) {
        Student student = new Student();
        student.setName("小明");
        student.setAge(16);
        student.printStudentAge();

        Student student2 = new Student();
        student.setName("小白");
        student.setAge(120);
        student2.printStudentAge();
     }
}

class Student {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age < 0 || age > 60)
            throw new RuntimeException("年龄设置不合法");
        this.age = age;
    }

    public void printStudentAge() {
        System.out.println(name + "同学的年龄：" + age);
    }
}
```



**程序输出结果如下：**

![图片说明](https://uploadfiles.nowcoder.com/images/20191109/5459305_1573290440346_FB5C81ED3A220004B71069645F112867)

通过将Student这个类的name和age属性私有化，**只有通过公共的get/set方法才能进行访问**，在get/set方法中我们可以对**内部逻辑进行封装处理**，外部的调用方不必关心我们的处理逻辑。



#### 33.2继承：

我们需要注意Java中不支持多继承，即一个类只可以有一个父类存在。另外Java中的构造函数是不可以继承的，如果构造函数被private修饰，那么就是不明确的构造函数，该类是不可以被其它类继承的，具体原因我们可以先来看下**Java中类的初始化顺序**：

- 初始化**父类**中的**静态成员变量和静态代码块**
- 初始化**子类**中的**静态成员变量和静态代码块**
- 初始化**父类**中的普通成员变量和代码块，再**执行父类的构造方法**
- 初始化**子类**中的普通成员变量和代码块，再**执行子类的构造方法**



如果父类构造函数是**私有（private）**的，则初始化子类的时候不可以被执行，所以解释了为什么该类不可以被继承，也就是说其不允许有子类存在。我们知道，子类是由其父类派生产生的，**那么子类有哪些特点呢？**

- 子类**拥有**父类非private的属性和方法
- 子类可以添加自己的方法和属性，即**对父类进行扩展**
- 子类可以重新定义父类的方法，即**方法的覆盖/重写**

既然子类可以通过**方法的覆盖/重写**以及方法的**重载**来重新定义父类的方法，那么我们来看下什么是方法的覆盖/重写吧。（其实这也是一个**高频的面试热身题目**）



#### 33.3覆盖（@Override）：

**覆盖也叫重写**，是指**子类和父类**之间方法的一种关系，比如说父类拥有方法A，子类扩展了方法A并且添加了丰富的功能。那么我们就说子类覆盖或者重写了方法A，也就是说子类中的方法与父类中继承的方法有完全相同的返回值类型、方法名、参数个数以及参数类型。

**Demo展示如下：**

```java
package niuke;

public class OverrideTest {
    public static void main(String[] args) {
        new Son().say();
    }
}
class Parent {
    public void say(){
        System.out.println("我是父类中的say方法");
    }
}
class Son extends Parent {
    @Override
    public void say(){
        System.out.println("我是子类中的say方法，我覆盖了父类的方法");
    }
}
```

我们可以看到，子类本身继承了父类的say方法，但是其想重新定义该方法的逻辑，所以就进行了覆盖。



#### 33.4多态：

通过方法的覆盖和重载可以实现多态，上边我们介绍了何为方法的覆盖，这里我们先来介绍下**何为方法的重载：**



### 34.重载：

**重载是指在一个类中（包括父类）存在多个同名的不同方法**，这些方法的**参数个数，顺序以及类型不同**均可以构成方法的重载。如果仅仅是修饰符、返回值、抛出的异常不同，那么这是2个相同的方法。

**Demo展示如下：**

```java
package niuke;

public class OverLoadTest {

    public void method1(String name, int age){
        System.out.println("");
    }
    // 两个方法的参数顺序不同，可以构成方法的重载
    public void method1(int age, String name){
        System.out.println("");
    }
    //---------------------------------------------
    public void method2(String name){
        System.out.println("");
    }
    // 两个方法的参数类型不同，可以构成方法的重载
    public void method2(int age){
        System.out.println("");
    }

    //---------------------------------------------
    public void method3(String name){
        System.out.println("");
    }
    // 两个方法的参数个数不同，可以构成方法的重载
    public void method3(int age, int num){
        System.out.println("");
    }
}
```



在本题目中，我们较为详细的解释了面向对象的**三大特性封装，继承与多态**。另外对常见面试考察点覆盖和重载也做出了区别与解释。由于重写（覆盖）与重载在名字上容易混淆，所以**重写（覆盖）与重载的区别是面试中的高频考点**，希望大家可以从原理上来理解与记忆。



### 35.如果只有方法返回值不同，可以构成重载吗？

**答：**不可以。因为我们调用某个方法，有时候并**不关心其返回值**，这个时候编译器根据方法名和参数无法确定我们调用的是哪个方法。

举例：如果我们分别定义了如下的两个方法：

```java
public String Test(String userName){ 
}
public void Test(String userName){
}
```

在调用的时候，直接 Test(“XiaoMing”)； 那么就会存在歧义。

我们再来看看如何通过**将子类对象作为父类对象使用**来实现多态。

把不同的子类对象都当作父类对象来看，可以屏蔽不同子类对象之间的差异，写出通用的代码，做出通用的编程，以适应需求的不断变化。这样操作之后，父类的对象就可以根据当前赋值给它的子类对象的特性以不同的方式运作。

对象的引用型变量具有多态性，因为**一个引用型变量可以指向不同形式的对象**，即：子类的对象作为父类的对象来使用。在这里涉及到了向上转型和向下转型，我们分别介绍如下：

**向上转型：**
子类对象转为父类，父类可以是接口。
公式：Father f = new Son（）; Father是父类或接口，Son是子类。

**向下转型：**
父类对象转为子类。公式：Son s = (Son) f;

在向上转型的时候我们可以直接转，但是在向下转型的时候我们必须强制类型转换。并且，如案例中所述，**该父类必须实际指向了一个子类对象才可强制类型向下转型**，即其是以这种方式Father f = new Son（）创建的父类对象。若以Father f = new Father（）这种方式创建的父类对象，那么不可以转换向下转换为子类的Son对象，运行会报错，因为其本质还是一个Father对象。





### 36.关于static的使用：（修饰类、方法、变量、静态块）

**1.** static修饰的类只能为内部类，普通类无法用static关键字修饰。static修饰的内部类相当于一个普通的类，访问方式为（new 外部类名.内部类的方法() ）。如下所示：

```java
public class OuterClass {
    public static class InnerClass{
        InnerClass(){
            System.out.println("============= 我是一个内部类'InnerClass' =============");
        }
    }
}


public class TestStaticClass {
    public static void main(String[] args) {
        // 不需要new一个InnerClass
        new OuterClass.InnerClass();
    }
}
```

**2.**static修饰静态方法的访问方式为 （类名.方法名）

**3.**静态变量和实例变量的区别

 static修饰的变量为静态变量，静态变量在内存中只有一份存储空间，静态变量不属于某个实例对象，被一个类中的所有对象所共享，属于类，所以也叫类变量，可以直接通过类名来引用。

实例变量属于某个固定对象所私有，实例变量的使用必须先创建一个类的实例，然后通过这个实例来引用使用。

```java
public class Person {
    int age = 10;
    static  int age1 =11;
}
public class Test {    
    public static void main(String[] args) {        
        Person person = new Person();        
        person.age=11;        
        person.age1 = 12;               
        Person person1 = new Person();                
        System.out.println(person.age);        
        System.out.println(person.age1);       
        System.out.println(person1.age);        
        System.out.println(person1.age1);    
    }
}
```





## 2.异常

![img](https://img-blog.csdn.net/20180920165502957?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21pY2hhZWxnbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 1.什么是异常？

异常是发生在程序执行过程中阻碍程序正常执行的错误事件。比如：用户输入错误数据、硬件故障、网络阻塞等都会导致出现异常。 只要在Java语句执行中产生了异常，一个异常对象就会被创建，JRE就会试图寻找异常处理程序来处理异常。如果有合适的异常处理程序，异常对象就会被异常处理程序接管，否则，将引发运行环境异常，JRE终止程序执行。 Java异常处理框架只能处理运行时错误，编译错误不在其考虑范围之内。



### 2.**java 异常有哪几种，特点是什么？**

Throwable 是所有异常的父类，

它有两个直接子类:

Exception 又被继续划分为被检查的异常（checked exception）和运行时的异常（runtime exception即不受检查的异常）

Error 表示系统错误，通常不能预期和恢复（譬如 JVM 崩溃、内存不足等）； 恢复不是不可能但很困难的情况下的一种严重问题。比如说内存溢出。不可能指望程序能处理这样的情况。



常见的五个编译时异常:

SQLException 、

IOexception 、

 FileNotFoundException 、

 ClassNotFoundException 



运行时异常：

NullPointerException - 空指针引用异常
ClassCastException - 类型强制转换异常。
IllegalArgumentException - 传递非法参数异常。
ArithmeticException - 算术运算异常
ArrayStoreException - 向数组中存放与声明类型不兼容对象异常
IndexOutOfBoundsException - 下标越界异常
NegativeArraySizeException - 创建一个大小为负数的数组错误异常
NumberFormatException - 数字格式异常
SecurityException - 安全异常
UnsupportedOperationException - 不支持的操作异常



### **3.java 中被检查的异常和不受检查的异常有什么区别？**

检查型异常和非检查型异常的主要区别在于其处理方式。

检查型异常都需要使用try,catch 和finally 关键字在编译器进行处理，否则会出现编译器报错。

对于非检查型异常则不需要这样做。Java中所有继承 Exception 的类的异常都是检查型异常，所有继承RuntimeException 的异常都被称为 非检查型异常。



### 4. Java 中什么是异常链？

异常链是指在进行一个异常处理时抛出了另外一个异常，由此产生了一个异常链条，

大多用于将受检查异常（checked exception）封装成为非受检查异常（unchecked exception)或者RuntimeException。特别注意如果你因为一个异常而决定抛出另一个新的异常时一定要包含原有的异常，这样处理程序才可以通过 getCause() 和 initCause() 方法来访问异常最终的根源



### 5.try{}里有一个return语句，那么紧跟在这个try后的finally{}里的代码会不会被执行，什么时候被执行，在return前还是后?

会执行，在方法返回调用者前执行。

在finally中改变返回值的做法是不好的，因为如果存在finally代码块，try中的return语句不会立马返回调用者，而是记录下返回值待finally代码块执行完毕之后再向调用者返回其值，然后如果在finally中修改了返回值，就会返回修改后的值

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(test());  //1,3,2

    }
    private static int test() {
        int temp = 1;
        try {
            System.out.println(temp);
            return ++temp;
        } catch (Exce
                 
                 ption e) {
            System.out.println(temp);
            return ++temp;
        } finally {
            ++temp;
            System.out.println(temp);
        }
    }
}
/*
*	执行顺序为：
*	输出try里面的初始temp：1；
*	temp=2；
*	保存return里面temp的值：2；
*	执行finally的语句temp：3，输出temp：3；
*	返回try中的return语句，返回存在里面的temp的值：2；
*	输出temp：2。
*/
```



### 6.Throw和Throws的区别

Throw：

1. 作用在方法内，表示抛出具体异常，由方法体内的语句处理。
2. 具体向外抛出的动作，所以它抛出的是一个异常实体类。若执行了Throw一定是抛出了某种异常。

Throws：

1. 作用在方法的声明上，表示如果抛出异常，则由该方法的调用者来进行异常处理。
2. 主要的声明这个方法会抛出会抛出某种类型的异常，让它的使用者知道捕获异常的类型。
3. 出现异常是一种可能性，但不一定会发生异常。



### 7.常见的异常

NullPointException：空指针异常，对象是null时会抛出，在调用传入对象时尽量判断是否为null，Jdk8里面可以用Optional对象来避免

IndexOutOfBoundsException：数组下标越界，数组的下标超过了最大值时会抛出，在迭代循环时检查下标是否越界

NumberFormatException：数字类型转化异常，将非数字类型转成数字类型，将类型转化的代码catch住

ClassCastException：类型转换异常，发生在强转时，将不同类型转成同一类型，尽量少用强转，或用instanceof（判断继承中子类的实例是否是父类的实现）做类型判断，或多用泛型

FileNotFoundException：找不到指定文件，文件路径错误或文件不存在，可能用了绝对路径检查文件是否存在，路径是否写错，多用相对路径

ClassNotFoundException：在classpath中找不到引用的类缺乏引用当前类的jar或没有设置classpath或jar损坏-，找到jar并放入classpath中或检查jar是否损坏

OutOfMemoryError：内存溢出异常，产生对象太多，内存不够->不要在循环体重创建大量对象，或对象及时回收，增大初始化堆：-Xms 增加最大值：-Xmx

NoClassDefFoundError：找不到相应的类错误，缺乏当前引用类的jar或jar版本不对->找到jar并放入classpath中或找到合适的版本

ConcurrentModificationException：并发修改异常，在集合迭代时修改里面的元素->在迭代时不要修改集合或用并发集合做遍历（如：ConcurrentHashMap）

NoSuchMethodError：类里找不到相应的方法，一般是jar版本不对，当前引用的jar版本中没有这个方法->检查jar版本是否正确

UnsupportedClassVersionError:版本不支持错误，编译class的jdk和运行时候的jdk版本不一致或比较高->将低版本换成高版本

StackOverflowError：栈溢出错误，一般是函数的死循环，或递归调用无法退出->检查死循环的代码，或让递归有退出值，或加大栈初始化参数



### 8. 编译时异常和运行时异常的区别

最简单的说法：

javac出来的异常就是编译时异常，就是说把源代码编译成字节码（class）文件时报的异常，一般如果用Eclispe,你敲完代码保存的时候就是编译的时候。Java出来的异常就是运行时异常

Java异常可分为3种：

　　(1)编译时异常:Java.lang.Exception

　　(2)运行期异常:Java.lang.RuntimeException

　　(3)错误:Java.lang.Error

编译时异常： 程序正确，但因为外在的环境条件不满足引发。例如：用户错误及I/O问题----程序试图打开一个并不存在的远程Socket端口。这不是程序本身的逻辑错误，而很可能是远程机器名字错误(用户拼写错误)。对商用软件系统，程序开发者必须考虑并处理这个问题。Java编译器强制要求处理这类异常，如果不捕获这类异常，程序将不能被编译。

运行期异常： 这意味着程序存在bug，如数组越界，0被除，入参不满足规范.....这类异常需要更改程序来避免，Java编译器强制要求处理这类异常。

错误： 一般很少见，也很难通过程序解决。它可能源于程序的bug，但一般更可能源于环境问题，如内存耗尽。错误在程序中无须处理，而有运行环境处理。



### 9.如何自定义异常

继承Exception是检查性异常，继承RuntimeException是非检查性异常，

一般要复写两个构造方法，用throw抛出新异常

如果同时有很多异常抛出，那可能就是异常链，就是一个异常引发另一个异常，另一个异常引发更多异常，一般我们会找它的原始异常来解决问题，一般会在开头或结尾，异常可通过initCause串起来，可以通过自定义异常





## 3.文件File，io流

### 1常见方法：

Files.exists()：检测文件路径是否存在。
Files.createFile()：创建文件。
Files.createDirectory()：创建文件夹。
Files.delete()：删除一个文件或目录。
Files.copy()：复制文件。
Files.move()：移动文件。
Files.size()：查看文件个数。
Files.read()：读取文件。
Files.write()：写入文件。

```
通过将给定路径来创建一个新File实例。
new File(String pathname);   
```

```   
根据parent路径名字符串和child路径名创建一个新File实例。parent是指上级目录的路径，完整的路径为parent+child.
new File(String parent, String child);      
```

```
根据parent抽象路径名和child路径名创建一个新File实例。 parent是指上级目录的路径，完整的路径为parent.getPath()+child.
说明：如果指定的路径不存在（没有这个文件或是文件夹），不会抛异常，这时file.exists()返回false。
new File(File parent, String child);
```

创建：

```
//在指定位置创建一个空文件，成功就返回true，如果已存在就不创建然后返回false
createNewFile() 
```

```
//在指定位置创建目录，这只会创建最后一级目录，如果上级目录不存在就抛异常。
mkdir()
```

```
//在指定位置创建目录，这会创建路径中所有不存在的目录。
mkdirs()
```

```
//重命名文件或文件夹，也可以操作非空的文件夹，文件不同时相当于文件的剪切,剪切时候不能操作非空的文件夹。移动/重命名成功则返回true，失败则返回false。
renameTo(File dest)
```

删除：

```
//删除文件或一个空文件夹，如果是文件夹且不为空，则不能删除，成功返回true，失败返回false。
delete() 
```

```
//在虚拟机终止时，请求删除此抽象路径名表示的文件或目录，保证程序异常时创建的临时文件也可以被删除
deleteOnExit()  
```

判断：

```
//文件或文件夹是否存在。
exists()
```

```
//是否是一个文件，如果不存在，则始终为false。
isFile()
```

```
//是否是一个目录，如果不存在，则始终为false。
isDirectory()
```

```
//是否是一个隐藏的文件或是否是隐藏的目录。
isHidden()
```

```
//测试此抽象路径名是否为绝对路径名。
isAbsolute()
```

获取：

```
//获取文件或文件夹的名称，不包含上级路径。
getName()
```

```
//返回绝对路径，可以是相对路径，但是目录要指定
getPath()
```

```
//获取文件的绝对路径，与文件是否存在没关系
getAbsolutePath()
```

```
// 获取文件的大小（字节数），如果文件不存在则返回0L，如果是文件夹也返回0L。
length()
```

```
//返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回null。
getParent()
```

```
//获取最后一次被修改的时间。
lastModified()
```

```
//列出所有的根目录（Window中就是所有系统的盘符）
staic File[] listRoots()
```

```
//返回目录下的文件或者目录名，包含隐藏文件。对于文件这样操作会返回null。
list() 
```

```
//返回指定当前目录中符合过滤条件的子文件或子目录。对于文件这样操作会返回null。
list(FilenameFilter filter) 
```

```
//返回目录下的文件或者目录对象（File类实例），包含隐藏文件。对于文件这样操作会返回null。
listFiles()
```

```
//返回指定当前目录中符合过滤条件的子文件或子目录。对于文件这样操作会返回null
listFiles(FilenameFilter filter)
```



题目：

1.在指定的路径下新建一个 .txt 文件 “test.txt”，利用程序在文件中写入如下内容：

```java
private static void work1() throws IOException {
        //要求：
        //创建文件
        File file = new File("test.txt");
        file.createNewFile();
        //写入数据
        FileWriter fwriter = new FileWriter(file);
        fwriter.write("Java是一种可以撰写跨平台应用软件的面向对象的程序设计语言.....");
        fwriter.close();
}
```



2.利用程序读取 test.txt 文件的内容, 并在控制台打印

```java
 private static void work2() throws IOException {
        /**
         * 2.利用程序读取 test.txt 文件的内容, 并在控制台打印
         */
        FileReader reader = new FileReader("test.txt");
        //采用数组来缓冲
        char[] c =new char[100];
        int len ;
        while ((len=reader.read(c))!=-1){
            //遍历数组
            for (int i=0;i<len;i++) {
                System.out.print(c[i]);
            }
		}
 }
```



3.利用程序复制 test.txt 为 test1.txt

```java
private static void work3() throws IOException {
         /*
        3.利用程序复制 test.txt 为 test1.txt
        因为文件类型是文本文档，所以选择字符流
         */
        File file = new File("test.txt");
        FileReader fr = new FileReader(file);
        FileWriter fw = new FileWriter("test1.txt");
        int len;
        char[] chars = new char[10];
        while((len = fr.read(chars)) != -1){
            fw.write(chars, 0, len);
        }
        fr.close();
        fw.close();
}
```



4.列出当前目录下全部java文件的名称

```java
 private static void work4() {
        /**
         * 4.列出当前目录下全部java文件的名称
         * 考点：
         *    1.File的list/listFile方法
         *    2.过滤器FilenameFilter/Filefilter
         */
        File file = new File("D:\\AAtemp");
        String[] filenames = file.list(new FilenameFilter() {
            @Override
            //accept会对文件夹的每一个子文件夹进行检测
            public boolean accept(File dir, String name) {
                return name.endsWith(".java");
            }
        });
        for (String f :filenames)
            System.out.println(f);
}
```



5.列出workspace目录下.class的名字及其大小，并删除其中的一个？

```java
 private static void work5() {
        //列出workspace目录下.class的名字及其大小，并删除其中的一个？
        File file = new File("C:\\Test");
        File file2 = new File("C:\\Test\\WiFi_Log.txt");
        File[] file1 = file.listFiles(new FilenameFilter() {
            @Override
            public boolean accept(File dir, String name) {
                return name.endsWith(".txt");
            }
        });
        file2.delete();
        for (File l : file1) {
            System.out.println(l);
        }
    }
```



## 4.工厂设计模式

### 1.几个常用的设计模式:

**创建型**

- 工厂模式与抽象工厂模式 （Factory Pattern）（Abstract Factory Pattern）

- 单例模式 （Singleton Pattern）
- 建造者模式 （Builder Pattern）
- 原型模式 （Prototype Pattern）

**结构型**

- 适配器模式 （Adapter Pattern）
- 装饰器模式 （Decorator Pattern）
- 桥接模式 （Bridge Pattern）
- 外观模式 （Facade Pattern）
- 代理模式 （Proxy Pattern）
- 过滤器模式 （Filter、Criteria Pattern）
- 组合模式 （Composite Pattern）
- 享元模式 （Flyweight Pattern）

**行为型**

- 责任链模式（Chain of Responsibility Pattern）

- 观察者模式（Observer Pattern）

- 模板模式（Template Pattern）

- 命令模式（Command Pattern）

- 解释器模式（Interpreter Pattern）

- 迭代器模式（Iterator Pattern）

- 中介者模式（Mediator Pattern）

- 策略模式（Strategy Pattern）

- 状态模式（State Pattern）

- 备忘录模式（Memento Pattern）

- 空对象模式（Null Object Pattern）  

  

 **工厂模式：**工厂类可以根据条件生成不同的子类实例，这些子类有一个公共的抽象父类并且实现了相同的方法，但是这些方法针对不同的数据进行了不同的操作（多态方法）。当得到子类的实例后，开发人员可以调用基类中的方法而不必考虑到底返回的是哪一个子类的实例。

**代理模式：**给一个对象提供一个代理对象，并由代理对象控制原对象的引用。

**适配器模式：**把一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起使用的类能够一起工作。

**单例模式：**一个类只有一个实例，即一个类只有一个对象实例。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170928225241215-295252070.png)



### 2.简单工厂和抽象工厂有什么区别？

- 简单工厂：用来生产同一等级结构中的任意产品，对于增加新的产品，无能为力。
- 工厂方法：用来生产同一等级结构中的固定产品，支持增加任意产品。
- 抽象工厂：用来生产不同产品族的全部产品，对于增加新的产品，无能为力；支持增加产品族。

### 3.单例模式的特点：

​		从系统启动到终止，整个过程只会产生一个实例。



### 4.手写两个单例模式

饿汉式

```java
//饿汉式：类加载的时候就实例化，并且创建单例对象。
class Singleton {
     private static Singleton instance=new Singleton();
     private Singleton(){}
     static Singleton getInstance() {
     	return instance;
     }
}
```

懒汉式

```java
//懒汉式：默认不会实例化，什么时候用什么时候new。
class Singleton {
     private static Singleton instance=null;
     private Singleton(){}
     static Singleton getInstance() {
     	if(instance==null)
             instance=new Singleton();
     	return instance;
     }
}
```



###  5.懒汉式和饿汉式区别：

实例化方面：懒汉式默认不会实例化，外部什么时候调用什么时候new。饿汉式在类加载的时候就实例化，并且创建单例对象。
线程安全方面：

饿汉式线程安全 (在线程还没出现之前就已经实例化了，因此饿汉式线程一定是安全的)。懒汉式线程不安全( 因为懒汉式加载是在使用时才会去new 实例的，那么你去new的时候是一个动态的过程，是放到方法中实现的，比如：public static synchronized Lazy getInstance(){  if(lazy==null){ lazy=new Lazy(); } 如果这个时候有多个线程访问这个实例，这个时候实例还不存在，还在new，就会进入到方法中，有多少线程就会new出多少个实例。一个方法只能return一个实例，那最终return出哪个呢？是不是会覆盖很多new的实例？这种情况当然也可以解决，那就是加同步锁，避免这种情况发生) 。
执行效率上：饿汉式没有加任何的锁，因此执行效率比较高。懒汉式一般使用都会加同步锁，效率比饿汉式差。

性能上：饿汉式在类加载的时候就初始化，不管你是否使用，它都实例化了，所以会占据空间，浪费内存。懒汉式什么时候需要什么时候实例化，相对来说不浪费内存。

------



## 5.集合面试题

### **1.集合的基本概念：**

​	 1.一个存储对象的容器
​	 2.集合只能存放对象
​	 3.集合存放的都是多个不同类型的对象
​	 4.对象本身还是存放在堆内存中7

- **List集合的使用：**
  1.List:有序的，可重复，有下标
  2.ArrayList：数组结构，带下标，长度可变，查询速度快，增删速度较慢
  3.LinkedList:链表结构，查询速度较慢，增删速度快

- **map集合的使用：**
  1.Map 集合一一对应（键值对）
  2.基于数组和链表进行存储数据
  3.HashMap 类按哈希算法来存取键对象
  4.TreeMap是基于红黑树实现的，适用于按自然顺序遍历key。

- **Set集合的使用：**
  1.Set集合 无序，不重复
  2.无序性（没下标），不能用for循环去遍历，可以用迭代器
  3.HashSet：增加删除时效率高
  4.LinkedHashSet：查看检索时，效率比较高

**一些不太常用的Map集合：TreeMap，HashTable**

> 题目：已知数组存放一批QQ号码，QQ号码最。
>
> 长为11位，最短为5位String[] strs =
> {"12345","67891","12347809933","98765432102","67891","12347809933"}。
> 将该数组里面的所有qq号都存放在LinkedList中，将list中重复元素删除，将list中所有元素分别用迭代器打印出来

```java
public static void main(String[] args) {
	//定义数组存放qq号码
	String[] strs = {"12345","67891","12347809933","98765432102","67891","12347809933"};
	
	//定义LinkedList
	LinkedList<String> lis = new LinkedList<String>();
	
	//c.增强for循环遍历数组存到集合
	for(String value : strs){
		lis.add(value);
	}
	
	//d.依次拿到每一个与这一个后的元素进行比较，如果相同则移除
	for(int i = 0; i < lis.size();i++)
		for(int j = i + 1;j < lis.size();j++)
			if(lis.get(i).equals(lis.get(j)))
				lis.remove(j);
	//e.用迭代器遍历集合
	for(Iterator<String> it = lis.iterator();it.hasNext();){
		String value = it.next();
		System.out.println(value + "\t");
}
```



### 2.常见的集合有哪些?

答：Map接口和Collection接口是所有集合框架的父接口

- Collection接口的子接口包括：Set接口和List接口

- Map接口的实现类主要有：HashMap、TreeMap、Hashtable、ConcurrentHashMap以及Properties等

- Set接口的实现类主要有：HashSet、TreeSet、LinkedHashSet等

  - List接口的实现类主要有：ArrayList、LinkedList、Stack以及Vector等

  

### 3. HashMap 和 Hashtable 有什么区别？

- hashMap去掉了HashTable 的contains方法，但是加上了containsValue（）和containsKey（）方法。
- hashTable同步的，而HashMap是非同步的，效率上比hashTable要高。
- hashMap允许空键值，而hashTable不允许。
- HashMap没有考虑同步，是线程不安全的；Hashtable使用了synchronized关键字，是线程安全的；
- HashMap继承自AbstractMap类；而Hashtable继承自Dictionary类；



### 7. List、Map、Set三个接口存取元素时，各有什么特点？

- List以特定索引来存取元素，可以有重复元素。
- Set不能存放重复元素（用对象的equals()方法来区分元素是否重复）。
- Map保存键值对（key-value pair）映射，映射关系可以是一对一或多对一



### 8. List、Set、Map 之间的区别是什么？

![image-20200617133142870](C:\Users\33066\AppData\Roaming\Typora\typora-user-images\image-20200617133142870.png)



### 9.遍历集合的方法

#### List：

**遍历List方法一：普通for循环**

```java
for(int i=0;i<list.size();i++){//list为集合的对象名
    String temp = (String)list.get(i);
    System.out.println(temp);
}
```

**遍历List方法二：增强for循环(使用泛型!)**

```java
for (String temp : list) {
	System.out.println(temp);
}
```

**遍历List方法三：使用Iterator迭代器(1)**

```java
for(Iterator iter= list.iterator();iter.hasNext();){
    String temp = (String)iter.next();
    System.out.println(temp);
}
```

**遍历List方法四：使用Iterator迭代器(2)**

```java
Iterator iter =list.iterator();
while(iter.hasNext()){
    Object  obj =  iter.next();
    iter.remove();//如果要遍历时，删除集合中的元素，建议使用这种方式！
    System.out.println(obj);
}
```

#### Set：

**遍历Set方法一：增强for循环**

```java
for(String temp:set){
	System.out.println(temp);
}
```

**遍历Set方法二：使用Iterator迭代器**

```java
for(Iterator iter = set.iterator();iter.hasNext();){
    String temp = (String)iter.next();
    System.out.println(temp);
}
```

#### Map:

**遍历Map方法一：根据key获取value**

```java
Map<Integer, Man> maps = new HashMap<Integer, Man>();
Set<Integer>  keySet =  maps.keySet();
for(Integer id : keySet){
	System.out.println(maps.get(id).name);
}
```

**遍历Map方法二：使用entrySet**

```java
Set<Entry<Integer, Man>>  ss = maps.entrySet();
for (Iterator iterator = ss.iterator(); iterator.hasNext();) {
    Entry e = (Entry) iterator.next(); 
    System.out.println(e.getKey()+"--"+e.getValue());
}
```



### 10.ConcurrentHashMap是怎么实现线程安全的。

在并发环境下，可能会形成**环状链表**（扩容时可能造成），导致get操作时，cpu空转，并发危险

 jdk 1.7  首先将数据分为一段一段存储，然后给每段数据配一把锁，当一个线程占用访问其中一个段数据时，其他数据段能被其他数据访问。

 ConcurrentHashMap的大部分操作和HashMap是相同的，例如初始化，扩容和链表向红黑树的转变等。但是，在ConcurrentHashMap中，利用一个CAS算法实现无锁化的修改值的操作，他可以大大降低锁代理的性能消耗。

数据结构和HashMap类似。采用数组+链表/红黑树。使用CAS和synchronized保证并发。synchronized只锁住当前链表或红黑树的首节点，这样是要hash不冲突，就不会产生并发冲突。

这个算法的基本思想就是不断地去比较当前内存中的变量值与你指定的一个变量值是否相等，如果相等，则接受你指定的修改的值，否则拒绝你的操作。因为当前线程中的值已经不是最新的值，你的修改很可能会覆盖掉其他线程修改的结果。这一点与乐观锁，SVN的思想是比较类似的。

put方法的源码：

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    // 计算hash值
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable(); // table是在首次插入元素的时候初始化，lazy
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, // 如果这个位置没有值，直接放进去，由CAS保证线程安全，不需要加锁
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {  // 节点上锁，这里的节点可以理解为hash值相同组成的链表的头节点，锁的粒度为头节点。
                if (tabAt(tab, i) == f) {
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
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```





### 11.hashmap集合

#### 11.1HashMap底层数据结构

- **JDK1.7及之前：数组+链表**
- **JDK1.8：数组+链表+红黑树**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS85ZDkyZGRkYS1lZmRiLTRmZjctYTlmYi00MTFjMTY5MzNkYmMucG5n?x-oss-process=image/format,png)

#### 11.2HashMap的存取实现

存储

```java
public V put(K key, V value) {
    // HashMap允许存放null键和null值。
    // 当key为null时，调用putForNullKey方法，将value放置在数组第一个位置。 
    if (key == null)
        return putForNullKey(value);
    // 根据key的keyCode重新计算hash值。
    int hash = hash(key.hashCode());
    // 搜索指定hash值在对应table中的索引。
    int i = indexFor(hash, table.length);
    // 如果 i 索引处的 Entry 不为 null，通过循环不断遍历 e 元素的下一个元素。
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    // 如果i索引处的Entry为null，表明此处还没有Entry。
    modCount++;
    // 将key、value添加到i索引处。
    addEntry(hash, key, value, i);
    return null;
}
```

​		从上面的源代码中可以看出：当我们往HashMap中put元素的时候，先根据key的hashCode重新计算hash值，根据hash值得到这个元素在数组中的位置（即下标）， 如果数组该位置上已经存放有其他元素了，那么在这个位置上的元素将以链表的形式存放，新加入的放在链头，最先加入的放在链尾。如果数组该位置上没有元素，就直接将该元素放到此数组中的该位置上。

​		addEntry(hash, key, value, i)方法根据计算出的hash值，将key-value对放在数组table的i索引处。addEntry 是 HashMap 提供的一个包访问权限的方法，代码如下：

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    // 获取指定 bucketIndex 索引处的 Entry 
    Entry<K,V> e = table[bucketIndex];
    // 将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry 
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    // 如果 Map 中的 key-value 对的数量超过了极限
    if (size++ >= threshold)
        // 把 table 对象的长度扩充到原来的2倍。
        resize(2 * table.length);
}
```

读取：

```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
```



**简单地说，HashMap 在底层将 key-value 当成一个整体进行处理，这个整体就是一个 Entry 对象。HashMap 底层采用一个 Entry[] 数组来保存所有的 key-value 对，当需要存储一个 Entry 对象时，会根据hash算法来决定其在数组中的存储位置，在根据equals方法决定其在该数组位置上的链表中的存储位置；当需要取出一个Entry时，也会根据hash算法找到其在数组中的存储位置，再根据equals方法从该位置上的链表中取出该Entry。**



#### 11.3HashMap的resize（rehash）

当HashMap中的元素越来越多的时候，hash冲突的几率也就越来越高，因为数组的长度是固定的。所以为了提高查询的效率，就要对HashMap的数组进行扩容，数组扩容这个操作也会出现在ArrayList中，这是一个常用的操作，而在HashMap数组扩容之后，最消耗性能的点就出现了：原数组中的数据必须重新计算其在新数组中的位置，并放进去，这就是resize。

那么HashMap什么时候进行扩容呢？当HashMap中的元素个数超过数组大小*loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，这是一个折中的取值。也就是说，默认情况下，数组大小为16，那么当HashMap中元素个数超过16*0.75=12的时候，就把数组的大小扩展为 2*16=32，即扩大一倍，然后重新计算每个元素在数组中的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数能够有效的提高HashMap的性能。



#### 11.4HashMap的数据插入原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020031712385760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poZW5nd2FuZ3p3,size_16,color_FFFFFF,t_70)

1. 判断数组是否为空，为空进行初始化;

2. 不为空，计算 k 的 hash 值，通过`(n - 1) & hash`计算应当存放在数组中的下标 index;

3. 查看 table[index] 是否存在数据，没有数据就构造一个Node节点存放在 table[index] 中；

4. 存在数据，说明发生了hash冲突(存在二个节点key的hash值一样), 继续判断key是否相等，相等，用新的value替换原数据(onlyIfAbsent为false)；

5. 如果不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；(如果当前节点是树型节点证明当前已经是红黑树了)

6. 如果不是树型节点，创建普通Node加入链表中；判断链表长度是否大于 8并且数组长度大于64， 大于的话链表转换为红黑树；

7. 插入完成之后判断当前节点数是否大于阈值，如果大于开始扩容为原数组的二倍。

   

#### 11.5 HashMap怎么设定初始容量大小

默认大小是16，负载因子是0.75， 如果自己传入初始大小k，初始化大小为 大于k的 2的整数次方，例如如果传10，大小为16。（补充说明:实现代码如下）

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```



#### 11.6HashMap的哈希函数怎么设计的

hash函数是先拿到 key 的hashcode，是一个32位的int值，然后让hashcode的高16位和低16位进行异或操作。

![image-20201126150158092](C:\Users\33066\AppData\Roaming\Typora\typora-user-images\image-20201126150158092.png)

为什么这么设计

这个也叫扰动函数，这么设计有二点原因：

1. 一定要尽可能降低hash碰撞，越分散越好；
2. 算法一定要尽可能高效，因为这是高频操作, 因此采用位运算；



#### 11.7 jdk1.8的优化：

1. 数组+链表改成了数组+链表或红黑树；
2. 链表的插入方式从头插法改成了尾插法，简单说就是插入时，如果数组位置上已经有元素，1.7将新元素放到数组中，原始节点作为新节点的后继节点，1.8遍历链表，将元素放置到链表的最后；
3. 扩容的时候1.7需要对原数组中的元素进行重新hash定位在新数组的位置，1.8采用更简单的判断逻辑，位置不变或索引+旧容量大小；
4. 在插入时，1.7先判断是否需要扩容，再插入，1.8先进行插入，插入完成再判断是否需要扩容；



#### 11.8为什么要做这些优化呢

1. 防止发生hash冲突，链表长度过长，将时间复杂度由`O(n)`降为`O(logn)`;

2. 因为1.7头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环；

   A线程在插入节点B，B线程也在插入，遇到容量不够开始扩容，重新hash，放置元素，采用头插法，后遍历到的B节点放入了头部，这样形成了环，如下图所示：

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020031302480988.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poZW5nd2FuZ3p3,size_16,color_FFFFFF,t_70)



#### 11.9 扩容的时候为什么jdk1.8 不用重新hash就可以直接定位原节点在新数据的位置呢?

这是由于扩容是扩大为原数组大小的2倍，用于计算数组位置的掩码仅仅只是高位多了一个1，怎么理解呢？

扩容前长度为16，用于计算(n-1) & hash 的二进制n-1为0000 1111，扩容为32后的二进制就高位多了1，为0001 1111。

因为是& 运算，1和任何数 & 都是它本身，那就分二种情况，如下图：原数据hashcode高位第4位为0和高位为1的情况；

第四位高位为0，重新hash数值不变，第四位为1，重新hash数值比原来大16（旧数组的容量）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200313030841557.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poZW5nd2FuZ3p3,size_16,color_FFFFFF,t_70)



#### 11.10 那HashMap是线程安全的吗？

在多线程环境下，

1.7 会产生死循环、数据丢失、数据覆盖的问题，

1.8 中会有数据覆盖的问题，以1.8为例，当A线程判断index位置为空后正好挂起，B线程开始往index位置的写入节点数据，这时A线程恢复现场，执行赋值操作，就把A线程的数据给覆盖了；还有++size这个地方也会造成多线程同时扩容等问题。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)  //多线程执行到这里
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode) // 这里很重要
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold) // 多个线程走到这，可能重复resize()
        resize();
    afterNodeInsertion(evict);
    return null;
}
```



#### 11.11那你平常怎么解决这个线程不安全的问题？

Java中有HashTable、Collections.synchronizedMap、以及ConcurrentHashMap可以实现线程安全的Map。

HashTable是直接在操作方法上加synchronized关键字，锁住整个数组，粒度比较大，

Collections.synchronizedMap是使用Collections集合工具的内部类，通过传入Map封装出一个SynchronizedMap对象，内部定义了一个对象锁，方法内通过对象锁实现；

ConcurrentHashMap使用分段锁，降低了锁粒度，让并发度大大提高。



#### 11.12HashMap内部节点是有序的吗？

是无序的，根据hash值随机插入

LinkedHashMap 和 TreeMap是有序的



#### 11.13LinkedHashMap怎么实现有序的？

LinkedHashMap内部维护了一个单链表，有头尾节点，同时LinkedHashMap节点Entry内部除了继承HashMap的Node属性，还有before 和 after用于标识前置节点和后置节点。可以实现按插入的顺序或访问顺序排序。

```java
    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;
    //链接新加入的p节点到链表后端
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
//LinkedHashMap的节点类
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```



TreeMap怎么实现有序的？

TreeMap是按照Key的自然顺序或者Comprator的顺序进行排序，内部是通过红黑树来实现。所以要么key所属的类实现Comparable接口，或者自定义一个实现了Comparator接口的比较器，传给TreeMap用于key的比较。



### 12  List集合：

#### 12.1 分类

1.Vector：线程安全（底层是一个数组，方法加了synchronized，默认容量10）

2.LinkedList（双向链表，线程不安全，Node节点）

3.ArrayList（数组，线程不安全，put时扩容为10，增删时调用System.arraycopy()，极消耗资源）



#### 12.2 如何得到线程安全的List

1.使用Vector

2.重写类似AarrayList但是线程安全的集合

3.使用Collections的synchronizedList（）方法

4.可以使用java.util.concurrent包下的**CopyOnWriteArrayList**



#### 12.3.为什么不推荐使用Vector

1.因为vector是线程安全的，所以效率低
2.Vector空间满了之后，扩容是一倍，而ArrayList仅仅是一半
3.Vector分配内存的时候需要连续的存储空间，如果数据太多，容易分配内存失败
4.只能在尾部进行插入和删除操作，效率低



#### 12.4什么是CopyOnWriteArrayList 

1.CopyOnWriteArrayList 读取是完全不用加锁的，并且更厉害的是：

**写入也不会阻塞读取操作，只有写入和写入之间需要进行同步等待**，读操作的性能得到大幅度提升。

2.类的所有可变操作（add，set等等）都是通过**创建底层数组的新副本**来实现的。当 List 需要被修改的时候，修改副本，然后将原来指向内存的指针指向新的内存，回收原来的内存。

3.add和remove的时候会**加锁和解锁**。	

缺点：

1.底层数组，插入和删除效率不高；写的时候需要复制，占内存，浪费空间，容易触发GC

2.只能保证数据最终一致性，不能保证数据实时一致性。



#### 12.5CopyOnWriteArrayList和Collections.synchronizedList区别

1.CopyOnWriteArrayList的写操作性能较差，而多线程的读操作性能较好。

2.Collections.synchronizedList的写操作性能好，而读操作因为是采用了synchronized关键字的方式性能不好。



#### 12.6 ArrayList扩容机制

1.添加元素时，首先判断是否大于默认容量10

2.如果小于默认容量，直接在原来基础上+1，元素添加完毕

3.如果大于默认容量，则需扩容，核心是grow（）方法

​	3.1扩容之前，创建一个新数组，将旧数组复制到新数组

​	3.2通过位运算，newCapacity为扩容为旧容量的1.5倍（如果超过int的最大值，会变成一个负数）

​	3.3如果newCapacity-minCapacity<0，则newCapacity = minCapacity（如果新的数组容量**newCapacity**小于传入的参数要求的最小容量**minCapacity，**那么新的数组容量以传入的容量参数为准。）

​		  如果newCapacity-MAX_ARRAY_SIZE > 0)，则newCapacity = hugeCapacity(minCapacity);

​		  再判断(minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;





------

## 6.线程面试题

***并行和并发***

> 并行：在同一个时刻，有多个指令在单个CPU同时执行 
> 并发：在同一个时刻，有多个指令在单个CPU交替执行

***进程和线程***

进程：正在运行的软件(就是操作系统中正在运行的一个应用程序)

> 独立性：进程是一个能独立运行的基本单位，同时也是系统分配资源和调度的独立单位 	
> 动态性：进程的实质是程序的一次执行过程，进程是动态产生的，动态消亡的
> 并发性：任何进程都可以同其他进程一起并发执行（CPU在多个进程之间进行一个动态的切换）

*线程：是进程中的单个顺序控制流，是一条执行路径*（就是应用程序中做的事情）

> 		单线程：一个线程如果只有一条执行路径，则称为单线程程序 	
> 		多线程：一个进程如果有多条执行路径，则成为多线程程序



### 1.线程和进程的区别？

一个程序下至少有一个进程，一个进程下至少有一个线程，一个进程下也可以有多个线程来增加程序的执行速度。



### 2.编写多线程程序有几种实现方式？

一种是继承Thread类；

另一种是实现Runnable接口。

还有一种是callable

两种方式都要通过重写run()方法来定义线程的行为，推荐使用后者，因为Java中的继承是单继承，一个类有一个父类，如果继承了Thread类就无法再继承其他类了，显然使用Runnable接口更为灵活。

runnable 没有返回值，callable 可以拿到有返回值，callable 可以看作是 runnable 的补充。

创建一个线程池：

```java
public static void main(String[] args) {
    ExecutorService threadpool = Executors.newCachedThreadPool();
    threadpool.execute(()->{
        for (int i = 0; i < 20; i++) {
            System.out.println(Thread.currentThread().getName()+":"+i);
        }
    });
    threadpool.shutdown();
}
```



### 3. synchronized关键字的用法？

synchronized关键字可以将对象或者方法标记为同步，

以实现对对象和方法的互斥访问，可以用synchronized(对象) { … }定义同步代码块，或者在声明方法时将synchronized作为方法的修饰符。



### 4.多线程中 synchronized 锁升级的原理是什么？

synchronized 锁升级原理：在锁对象的对象头里面有一个 threadid 字段，在第一次访问的时候 threadid 为空，

jvm 让其持有偏向锁，并将 threadid 设置为其线程 id，再次进入的时候会先判断 threadid 是否与其线程 id 一致，如果一致则可以直接使用此对象，如果不一致，则升级偏向锁为轻量级锁，通过自旋循环一定次数来获取锁，执行一定次数之后，如果还没有正常获取到要使用的对象，此时就会把锁从轻量级升级为重量级锁，此过程就构成了 synchronized 锁的升级。

锁的升级的目的：在 Java 6 之后优化 synchronized 的实现方式，使用了偏向锁升级为轻量级锁再升级到重量级锁的方式，从而减低了锁带来的性能消耗。



### 5.什么是死锁？

当线程 A 持有独占锁a，并尝试去获取独占锁 b 的同时，线程 B 持有独占锁 b，并尝试获取独占锁 a 的情况下，就会发生 AB 两个线程由于互相持有对方需要的锁，而发生的阻塞现象，我们称为死锁。



#### 5.1 死锁的四个必要条件

互斥条件：一个资源每次只能被一个进程使用，即在一段时间内某 资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。

请求与保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源 已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。

不可剥夺条件:进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能 由获得该资源的进程自己来释放（只能是主动释放)。

循环等待条件: 若干进程间形成首尾相接循环等待资源的关系

*这四个条件是死锁的必要条件，只要系统发生死锁，这些条件必然成立，而只要上述条件之一不满足，就不会发生死锁。*



#### 5.2. 死锁避免

- 加锁顺序（线程按照一定的顺序加锁）
- 加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁）
- 死锁检测



### 6) Thread 类中的start() 和 run() 方法有什么区别？

这个问题经常被问到，但还是能从此区分出面试者对Java线程模型的理解程度。start()方法被用来启动新创建的线程，而且start()内部调用了run()方法，JDK 1.8源码中start方法的注释这样写到：Causes this thread to begin execution; the Java Virtual Machine calls the <code>run</code> method of this thread.这和直接调用run()方法的效果不一样。**当你调用run()方法的时候，只会是在原来的线程中调用，没有新的线程启动，start()方法才会启动新线程**，JDK 1.8源码中注释这样写： The result is that two threads are running concurrently: the current thread (which returns from the call to the <code>start</code> method) and the other thread (which executes its <code>run</code> method)。



### 6.在 Java 程序中怎么保证多线程的运行安全？

- 方法一：使用安全类，比如 Java. util. concurrent 下的类。
- 方法二：使用自动锁 synchronized。
- 方法三：使用手动锁 Lock。



### 7.sleep() 和 wait() 有什么区别？

- 类的不同：sleep() 来自 Thread，wait() 来自 Object。

- 释放锁：sleep() 不释放锁；wait() 释放锁。

- 用法不同：sleep() 时间到会自动恢复；wait() 可以使用 notify()/notifyAll()直接唤醒。

- sleep()方法（休眠）是线程类（Thread）的静态方法，调用此方法会让当前线程暂停执行指定的时间，将执行机会让给其他线程，但是对象的锁依然保持，因此休眠时间结束后会自动恢复。

- wait()是Object类的方法，调用对象的wait()方法导致当前线程放弃对象的锁（线程暂停执行），进入对象的等待池，只有调用对象的notify()方法（或notifyAll()方法）时才能唤醒等待池中的线程进入等锁池（lock pool），如果线程重新获得对象的锁就可以进入就绪状态。

  

### 8.线程的sleep()方法和yield()方法有什么区别？

① sleep()方法给其他线程运行机会时不考虑线程的优先级，因此会给低优先级的线程以运行的机会；yield()方法只会给相同优先级或更高优先级的线程以运行的机会；
② 线程执行sleep()方法后转入阻塞（blocked）状态，而执行yield()方法后转入就绪（ready）状态；
③ sleep()方法声明抛出InterruptedException，而yield()方法没有声明任何异常；

④ sleep()方法比yield()方法（跟操作系统CPU调度相关）具有更好的可移植性。



### 9.synchronized和lock的区别

1.作用位置不同：

synchronized可以给方法、代码块加锁

lock只能给代码块加锁

2.锁的获取锁和释放机制不同

synchronized无需手动获取和释放锁，发生异常会自动解锁，不会出现死锁

lock需要自己加锁或释放锁，如lock()和unlock()，如果忘记使用unlock（），则会出现死锁

补充：synchronized修饰成员方法时，默认的锁对象，就是当前对象

​			synchronized修饰静态方法时，默认的锁对象，当前的class对象，比如User.class

​			synchronized修饰代码块时，可以自己来设置锁对象，比如synchronized（this）



### 10.synchronized和volatile区别

1.作用位置不同

synchronized修饰方法和代码块

volatile修饰变量

2.作用不同

synchronized可以保证变量修改的可见性和原子性，可能会造成线程阻塞

volatile仅能实现变量修改的可见性，无法保证原子性，不会造成阻塞



### 11.阻塞和等待的区别

阻塞：线程和其他线程抢占锁，没抢到，处于阻塞状态

等待：进入同步代码块了，调用wait（）等待



### 12.线程同步和互斥的区别

1. 互斥是指某一资源同时只允许一个访问者对其进行访问，具有唯一性和排它性。但互斥无法限制访问者对资源的访问顺序，即访问是无序的。
2. 同步是指在互斥的基础上（大多数情况），通过其它机制实现访问者对资源的有序访问。
3. 同步其实已经实现了互斥，所以同步是一种更为复杂的互斥。
4. 互斥是一种特殊的同步。

所谓互斥，就是不同线程通过竞争进入临界区（共享的数据和硬件资源），为了防止访问冲突，在有限的时间内只允许其中之一独占性的使用共享资源。如不允许同时写

同步关系则是多个线程彼此合作，通过一定的逻辑关系来共同完成一个任务。一般来说，同步关系中往往包含互斥，同时对临界区的资源会按照某种逻辑顺序进行访问。如先生产后使用

总的来说，两者的区别就是：
**互斥是通过竞争对资源的独占使用，彼此之间不需要知道对方的存在，执行顺序是一个乱序。**
**同步是协调多个相互关联线程合作完成任务，彼此之间知道对方存在，执行顺序往往是有序的。**

lock与unlock方法，替换synchronized，这就是互斥锁的体现。消费者生产者模式就是同步锁的体现。



### 13.线程生命周期

5状态模型：新建、就绪、**运行（sleep或wait）阻塞**、终止

6状态模型：新建、运行、**阻塞、无限等待、有限等待**、终止

1.当进入synchronized同步代码块或同步方法时，且目运获取到锁，线程就进入blocked状态，直到锁释放，重新进入runnable状态

2.当线程调用wait（）或join时，线程都会进入waiting状态，当调用notify或notifyAll时，或者join的线程执行结束后，会进入runnable状态

3.当线程调用sleep（time）或wait（time)，进入timed waiting状态

当休眠时间结束后，或者调用notify或notifyAll会重新进入runnable状态

4.程序执行结束，线程进入terminated



### 14.ThreadLocal是什么？

这个玩意有什么用处，或者说为什么要有这么一个东东？先解释一下，

在并发编程的时候，成员变量如果不做任何处理其实是线程不安全的，各个线程都在操作同一个变量，显然是不行的，并且我们也知道volatile这个关键字也是不能保证线程安全的。

那么在有一种情况之下，我们需要满足这样一个条件：变量是同一个，但是每个线程都使用同一个初始值，也就是使用同一个变量的一个新的副本。这种情况之下ThreadLocal就非常使用，

比如说DAO的数据库连接，我们知道DAO是单例的，那么他的属性Connection就不是一个线程安全的变量。而我们每个线程都需要使用他，并且各自使用各自的。这种情况，ThreadLocal就比较好的解决了这个问题。

```java
public final class ConnectionUtil {

    private ConnectionUtil() {}

    private static final ThreadLocal<Connection> conn = new ThreadLocal<>();

    public static Connection getConn() {
        Connection con = conn.get();
        if (con == null) {
            try {
                Class.forName("com.mysql.jdbc.Driver");
                con = DriverManager.getConnection("url", "userName", "password");
                conn.set(con);
            } catch (ClassNotFoundException | SQLException e) {
                // ...
            }
        }
        return con;
    }

}
```



**总结：可以看到每个线程内部都有一个threadLocals的成员变量，类型为ThreadLocalMap, 默认为null。只有当前线程第一次调用ThreadLocal的set或者get方法时才会创建它们，其中key为我们定义的ThreadLocal变量的this引用，value则是我们set方法设置的值，由于是存在线程中的，所以如果当前线程一直不消亡，那么这些变量就会一直存在，所以可能会造成内存泄露最终导致内存溢出，因此建议使用完毕后调用remove方法删除对应的变量。**



官方解释：

```text
jdk1.8
/**
 * This class provides thread-local variables.  These variables differ from
 * their normal counterparts in that each thread that accesses one (via its
 * {@code get} or {@code set} method) has its own, independently initialized
 * copy of the variable.  {@code ThreadLocal} instances are typically private
 * static fields in classes that wish to associate state with a thread (e.g.,
 * a user ID or Transaction ID).
 * 
 * <p>For example, the class below generates unique identifiers local to each
 * thread.
 * A thread's id is assigned the first time it invokes {@code ThreadId.get()}
 * and remains unchanged on subsequent calls.
 */  
```

​		大概的意思是这个类提供了线程本地变量，也就是你如果创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的一个本地副本。当多个线程操作这个变量时，实际操作的是自己本地内存里面的变量，从而避免了线程安全问题。

```java
	/**
     * Creates a thread local variable.
     * @see #withInitial(java.util.function.Supplier)
     */
    public ThreadLocal() {
    }
```

我们可以看到ThreadLocal的构造函数并没有做任何操作，仅仅只是定义了一个ThreadLocal变量，其实每个线程的本地变量并不是存放在ThreadLocal实例里面，而是存放在调用线程的threadLocals变量里，所以我们可以看到Thread类中有这两个变量的声明，初始值都是null。

```java
	/* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    	ThreadLocal.ThreadLocalMap threadLocals = null;
    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

​		通过上面的定义可以知道threadLocals 是一个ThreadLocalMap 类型的变量，类似于我们的Hashmap。那我们就来 接着看看ThreadLocal类是如何和Thread产生联系的，下面简单分析下ThreadLocal的set、get以及remove的方法

```java
	public T get() {
        Thread t = Thread.currentThread();
        
        //第一次执行的时候，返回null
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            //不为null时获取map中的值
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        //为null时，初始化
        return setInitialValue();
    }
  	ThreadLocalMap getMap(Thread t) {
        //获取线程的threadLocals变量
        return t.threadLocals;
    } 
	private T setInitialValue() {
       //初始化为null值
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
    void createMap(Thread t, T firstValue) {
        //创建一个ThreadLocalMap变量，并插入第一次的值
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
   protected T initialValue() {
        return null;
    }


	public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    /**
     * Removes the current thread's value for this thread-local
     * variable.  If this thread-local variable is subsequently
     * {@linkplain #get read} by the current thread, its value will be
     * reinitialized by invoking its {@link #initialValue} method,
     * unless its value is {@linkplain #set set} by the current thread
     * in the interim.  This may result in multiple invocations of the
     * {@code initialValue} method in the current thread.
     *
     * @since 1.5
     */
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```



### 15.线程池的处理流程？

任务提交，如果线程池数量还没到核心数的话启动线程来接这个任务，如果线程数已经到了核心数则任务入队。如果队列满了则看是否超过最大核心数，如果超过最大核心数则看拒绝策略如果定义的，不超过则启动新线程来处理这个任务。



### 16.这样的线程池有什么不好的地方？

适合cpu密集型把，IO密集型的还是需要改造下，让更多的线程工作起来，而不是优先排队。



### 17.线程的实现

三种方式的对比：
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20191206200147328.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNjYxODQx,size_16,color_FFFFFF,t_70)

--------------------------------------------------------------------------------------------------------------------------------------



#### **方式1.继承Thread类** 	

> 1> 定义一个类MyThread继承Thread类 	
> 2>  在MyThread类中从写run()方法 	
> 3> 创建MyThread类的对象 	
> 4> 启动线程

```java
/**
 * @program: javase
 * @description: 继承Thread类实现多线程
 * @Author: 小白白
 * @create: 2019/12/06 - 12:54
 **/
public class MyThread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 20; i++){
            System.out.println("线程开启了" + i);
        }
    }
}
```

```java
/**
 * @program: javase
 * @description: 继承Thread类的测试类
 * @Author: 小白白
 * @create: 2019/12/06 - 12:56
 **/
public class MyThread_text {
    public static void main(String[] args) {
        //创建一个线程对象
        MyThread t1 = new MyThread();
        //创建一个线程对象
        MyThread t2 = new MyThread();
        //开启一个线程
        t1.start();
        //开启第二条线程
        t2.start();
    }
}
```

**运行结果为两个线程交替运行，交替去争抢CPU的资源**

思考：

> 为什么要重写run（）方法？
> ----------因为run()用来封装被线程执行的代码

> run（）方法和start（）方法的区别？
> ---------run：封装线程执行的代码，直接调用，相当于普通方法的调用，并没有开启线程
> ---------start：启动线程，然后由JVM调用此线程的run()方法

start方法的底层源码。-----------native表示调取本地方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191206135424412.png)



#### **方式2.实现Runnable接口**

> 1.定义一个类MyRunnable实现Runnable接口
> 2.在MyRunnable类中重写run()方法
> 3.创建Runnable类的对象
> 4.创建Thread类的对象，把MyRunnbable独享作为构造方法的参数
> 5.启动线程
> package com.zxh.Thread.Demo;

```java
/**
 * @program: javase
 * @description: 实现Runnable接口
 * @Author: 小白白
 * @create: 2019/12/06 - 14:01
 **/
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        //线程启动后执行的代码
        for (int i = 0; i < 100;i++){
            System.out.println("第二种方法实现多线程" + i);
        }
    }
}
```

```java
/**
 * @program: javase
 * @description: 实现Runnable接口的测试类
 * @Author: 小白白
 * @create: 2019/12/06 - 14:03
 **/
public class MyRunnable_test {
    public static void main(String[] args) {
        //创建了一个参数的对象
        MyRunnable myRunnableTest = new MyRunnable();
        //创建了一个线程对象，并把参数传递给这个线程
        //在线程启动之后，执行的就是参数里面的run方法
        Thread t= new Thread(myRunnableTest);
        //开启线程
        t.start();
    }
}
```


#### **方式3.通过Callable接口进行实现**

> 1.定义一个类MyCallable实现Callable接口
> 2.在MyCallable中重写call（）方法
> 3.创建MyCallable类的对象
> 4.创建Future的是实现类FutureTask对象，把MyCallable对象作为构造方法的参数
> 5.创建Thread类的对象，把FutureTask对象作为构造方法的参数
> 6.启动线程

```java
/**
 * @program: javase
 * @description: 实现Callable接口
 * @Author: 小白白
 * @create: 2019/12/06 - 18:59
 **/
public class MyCallable implements Callable<Object> {

    //在MyCallable中重写call（）方法
    @Override
    public Object call() throws Exception {
        for (int i = 0; i < 100; i++) {
            System.out.println("跟女孩表白" + i);
        }
        //返回值表示线程运行完毕之后的结果
        return "答应";
    }
}
```

```java
import java.util.concurrent.*;

/**
 * @program: javase
 * @description: 实现Callable接口的测试类
 * @Author: 小白白
 * @create: 2019/12/06 - 19:03
 **/
public class MyCallable_text {
    public static void main(String[] args) {
        //创建MyCallable类的对象
        MyCallable mc = new MyCallable();
        //创建Future的是实现类FutureTask对象，把MyCallable对象作为构造方法的参数
        FutureTask<Object> ft = new FutureTask<Object>(mc);
        //创建Thread类的对象，把FutureTask对象作为构造方法的参数
        Thread t1 = new Thread(ft);
        //启动线程
        t1.start();
    }
}
```





# 并发编程：

## 前言

## 线程池相关问题

### 问题一：Java 中的线程池是如何实现的？

创建线程池：

```java
import java.util.concurrent.*; 

public class App { 
    public static void main(String[] args) throws Exception { 
        ExecutorService executorService = new ThreadPoolExecutor(1, 1, 60L, TimeUnit.SECONDS, 
                new ArrayBlockingQueue<>(10)); 

        executorService.execute(new Runnable() { 
            @Override 
            public void run() { 
                System.out.println("abcdefg"); 
            } 
        }); 
     
        executorService.shutdown(); 
    } 
}
```



### 问题二：创建线程池的几个核心构造参数？

1. corePoolSize 线程池核心线程大小
2. maximumPoolSize 线程池最大线程数量
3. keepAliveTime 空闲线程存活时间
4. unit 空闲线程存活时间单位
5. workQueue 工作队列
   1. ①ArrayBlockingQueue   数组的有界阻塞队列
   2. ②LinkedBlockingQuene  基于链表的无界阻塞队列
   3. ③SynchronousQuene    一个不缓存任务的阻塞队列
   4. ④PriorityBlockingQueue   具有优先级的无界阻塞队列
6. threadFactory 线程工厂
7. handler 拒绝策略



### 问题三：线程池中的线程是怎么创建的？是一开始就随着线程池的启动创建好的吗？

### 问题四：既然提到可以通过配置不同参数创建出不同的线程池，那么 Java 中默认实现好的线程池又有哪些呢？请比较它们的异同。

### 问题五：如何在 Java 线程池中提交线程？



## 线程安全和线程同步器相关问题

### 问题一：java如何实现多线程之间的通讯和协作？

### 问题二：什么叫线程安全？servlet是线程安全吗?

### 问题三：同步有几种实现方法？

1. 有synchronized关键字修饰的方法
2.  即有synchronized关键字修饰的语句块。，会自动被加上内置锁
3. **使用特殊域变量(volatile)实现线程同步**
4. 使用重入锁实现线程同步



### 问题四：volatile有什么用？能否用一句话说明下volatile的应用场景？

​		Java内存模型的主要目标就是**定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的细节**。

`volatile`是通过编译器在生成字节码时，在指令序列中添加“**内存屏障**”来禁止指令重排序的。



### 问题五：请说明下java的内存模型及其工作流程。

### 问题六：为什么代码会重排序？

### 问题七：分析下JUC 中倒数计数器 CountDownLatch 的使用与原理？

### 问题八：CountDownLatch 与线程的 Join 方法区别是什么？

### 问题九：讲讲对JUC 中回环屏障 CyclicBarrier 的使用？

### 问题十：CyclicBarrier内部的实现与 CountDownLatch 有何不同？

### 问题十一：Semaphore 的内部实现是怎样的？

### 问题十二：简单对比同步器实现，谈谈你的看法？

### 问题十三：并发组件CopyOnWriteArrayList 是如何通过写时拷贝实现并发安全的 List？



## 内存模型相关问题

问题一：什么是 Java 的内存模型，Java 中各个线程是怎么彼此看到对方的变量的？

问题二：请谈谈 volatile 有什么特点，为什么它能保证变量对所有线程的可见性？

问题三：既然 volatile 能够保证线程间的变量可见性，是不是就意味着基于 volatile 变量的运算就是并发安全的？

问题四：请对比下 volatile 对比 Synchronized 的异同。

问题五：请谈谈 ThreadLocal 是怎么解决并发安全的？

问题六：很多人都说要慎用 ThreadLocal，谈谈你的理解，使用 ThreadLocal 需要注意些什么？



## 锁相关问题

问题一：什么是可重入锁、乐观锁、悲观锁、公平锁、非公平锁、独占锁、共享锁？

问题二：当一个线程进入某个对象的一个synchronized的实例方法后，其它线程是否可进入此对象的其它方法？

问题三：synchronized和java.util.concurrent.locks.Lock的异同？

问题四：什么是锁消除和锁粗化？

问题五：乐观锁和悲观锁的理解及如何实现，有哪些实现方式？

问题六：如何实现乐观锁（CAS）？如何避免ABA                                                                                                                                                                 

问题七：读写锁可以用于什么应用场景？

问题八：什么是可重入性，为什么说 Synchronized 是可重入锁？

问题九：什么时候应该使用可重入锁？

问题十：什么场景下可以使用volatile替换synchronized？



## 并发框架和并发队列相关问题

问题一：SynchronizedMap和ConcurrentHashMap有什么区别？

问题二：CopyOnWriteArrayList可以用于什么应用场景？

问题三：如何让一段程序并发的执行，并最终汇总结果？

问题四：任务非常多的时候，使用什么阻塞队列能获取最好的吞吐量？

问题五：如何使用阻塞队列实现一个生产者和消费者模型？

问题六：多读少写的场景应该使用哪个并发容器，为什么使用它？

问题七：谈下对基于链表的非阻塞无界队列 ConcurrentLinkedQueue 原理的理解？

问题八：ConcurrentLinkedQueue 内部是如何使用 CAS 非阻塞算法来保证多线程下入队出队操作的线程安全？

问题九：基于链表的阻塞队列 LinkedBlockingQueue 原理。

问题十：阻塞队列LinkedBlockingQueue 内部是如何使用两个独占锁 ReentrantLock 以及对应的条件变量保证多线程先入队出队操作的线程安全？

问题十一：为什么不使用一把锁，使用两把为何能提高并发度？

问题十二：基于数组的阻塞队列 ArrayBlockingQueue 原理。

问题十三：ArrayBlockingQueue 内部如何基于一把独占锁以及对应的两个条件变量实现出入队操作的线程安全？

问题十四：谈谈对无界优先级队列 PriorityBlockingQueue 原理？

问题十五：PriorityBlockingQueue 内部使用堆算法保证每次出队都是优先级最高的元素，元素入队时候是如何建堆的，元素出队后如何调整堆的平衡的？



## CountDownLatch相关问题

问题一：介绍一下 CountDownLatch 工作原理？

问题二：CountDownLatch 和 CyclicBarrier 的区别？

问题三：CountDownLatch 的使用场景?

问题四：CountDownLatch 类中主要的方法?



Java提供了种类丰富的锁，每种锁因其特性的不同，在适当的场景下能够展现出非常高的效率。本文旨在对锁相关源码（本文中的源码来自JDK 8）、使用场景进行举例，为读者介绍主流锁的知识点，以及不同的锁的适用场景。

Java中往往是按照是否含有某一特性zx来定义锁，我们通过特性将锁进行分组归类，再使用对比的方式进行介绍，帮助大家更快捷的理解相关知识。下面给出本文内容的总体分类目录： ![图片1](https://img-blog.csdnimg.cn/20181226102235229)

### 1. 乐观锁 VS 悲观锁

乐观锁与悲观锁是一种广义上的概念，体现了看待线程同步的不同角度。在Java和数据库中都有此概念对应的实际应用。

先说概念。对于同一个数据的并发操作，悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。Java中，synchronized关键字和Lock的实现类都是悲观锁。

而乐观锁认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）。

乐观锁在Java中是通过使用无锁编程来实现，最常采用的是CAS算法，Java原子类中的递增操作就通过CAS自旋实现的。 ![图片1](https://img-blog.csdnimg.cn/20181226102235257)

根据从上面的概念描述我们可以发现：

- 悲观锁适合写操作多的场景，先加锁可以保证写操作时数据正确。
- 乐观锁适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升。

光说概念有些抽象，我们来看下乐观锁和悲观锁的调用方式示例：

```java
// ------------------------- 悲观锁的调用方式 -------------------------
// synchronized
public synchronized void testMethod() {
	// 操作同步资源
}



// ReentrantLock
private ReentrantLock lock = new ReentrantLock(); // 需要保证多个线程使用的是同一个锁
public void modifyPublicResources() {
	lock.lock();
	// 操作同步资源
	lock.unlock();
}


// ------------------------- 乐观锁的调用方式 -------------------------
private AtomicInteger atomicInteger = new AtomicInteger();  // 需要保证多个线程使用的是同一个AtomicInteger
atomicInteger.incrementAndGet(); //执行自增1
```

通过调用方式示例，我们可以发现悲观锁基本都是在显式的锁定之后再操作同步资源，而乐观锁则直接去操作同步资源。那么，为何乐观锁能够做到不锁定同步资源也可以正确的实现线程同步呢？我们通过介绍乐观锁的主要实现方式 “CAS” 的技术原理来为大家解惑。

CAS全称 Compare And Swap（比较与交换），是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。java.util.concurrent包中的原子类就是通过CAS来实现了乐观锁。

CAS算法涉及到三个操作数：

- 需要读写的内存值 V。
- 进行比较的值 A。
- 要写入的新值 B。

当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作。一般情况下，“更新”是一个不断重试的操作。

之前提到java.util.concurrent包中的原子类，就是通过CAS来实现了乐观锁，那么我们进入原子类AtomicInteger的源码，看一下AtomicInteger的定义： ![图片2](https://img-blog.csdnimg.cn/20181226102235294)

根据定义我们可以看出各属性的作用：

- unsafe： 获取并操作内存的数据。
- valueOffset： 存储value在AtomicInteger中的偏移量。
- value： 存储AtomicInteger的int值，该属性需要借助volatile关键字保证其在线程间是可见的。

接下来，我们查看AtomicInteger的自增函数incrementAndGet()的源码时，发现自增函数底层调用的是unsafe.getAndAddInt()。但是由于JDK本身只有Unsafe.class，只通过class文件中的参数名，并不能很好的了解方法的作用，所以我们通过OpenJDK 8 来查看Unsafe的源码：

```java
// ------------------------- JDK 8 -------------------------
// AtomicInteger 自增方法
public final int incrementAndGet() {
  return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

// Unsafe.class
public final int getAndAddInt(Object var1, long var2, int var4) {
  int var5;
  do {
      var5 = this.getIntVolatile(var1, var2);
  } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  		return var5;
}



// ------------------------- OpenJDK 8 -------------------------
// Unsafe.java
public final int getAndAddInt(Object o, long offset, int delta) {
   int v;
   do {
       v = getIntVolatile(o, offset);
   } while (!compareAndSwapInt(o, offset, v, v + delta));
   return v;
}
```

根据OpenJDK 8的源码我们可以看出，getAndAddInt()循环获取给定对象o中的偏移量处的值v，然后判断内存值是否等于v。如果相等则将内存值设置为 v + delta，否则返回false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。整个“比较+更新”操作封装在compareAndSwapInt()中，在JNI里是借助于一个CPU指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。

后续JDK通过CPU的cmpxchg指令，去比较寄存器中的 A 和 内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后通过Java代码中的while循环再次调用cmpxchg指令进行重试，直到设置成功为止。

CAS虽然很高效，但是它也存在三大问题，这里也简单说一下：

1.**ABA问题**。CAS需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是A，后来变成了B，然后又变成了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。

JDK从1.5开始提供了AtomicStampedReference类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。

2.**循环时间长开销大**。CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。

3.**只能保证一个共享变量的原子操作**。对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。

Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

### 2. 自旋锁 VS 适应性自旋锁

在介绍自旋锁前，我们需要介绍一些前提知识来帮助大家明白自旋锁的概念。

阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态转换需要耗费处理器时间。如果同步代码块中的内容过于简单，状态转换消耗的时间有可能比用户代码执行的时间还要长。

在许多场景中，同步资源的锁定时间很短，为了这一小段时间去切换线程，线程挂起和恢复现场的花费可能会让系统得不偿失。如果物理机器有多个处理器，能够让两个或以上的线程同时并行执行，我们就可以让后面那个请求锁的线程不放弃CPU的执行时间，看看持有锁的线程是否很快就会释放锁。

而为了让当前线程“稍等一下”，我们需让当前线程进行自旋，如果在自旋完成后前面锁定同步资源的线程已经释放了锁，那么当前线程就可以不必阻塞而是直接获取同步资源，从而避免切换线程的开销。这就是自旋锁。 ![图片3](https://img-blog.csdnimg.cn/20181226102235315) 自旋锁本身是有缺点的，它不能代替阻塞。自旋等待虽然避免了线程切换的开销，但它要占用处理器时间。如果锁被占用的时间很短，自旋等待的效果就会非常好。反之，如果锁被占用的时间很长，那么自旋的线程只会白浪费处理器资源。所以，自旋等待的时间必须要有一定的限度，如果自旋超过了限定次数（默认是10次，可以使用-XX:PreBlockSpin来更改）没有成功获得锁，就应当挂起线程。

自旋锁的实现原理同样也是CAS，AtomicInteger中调用unsafe进行自增操作的源码中的do-while循环就是一个自旋操作，如果修改数值失败则通过循环来执行自旋，直至修改成功。 ![图片4](https://img-blog.csdnimg.cn/20181226102235341) 自旋锁在JDK1.4.2中引入，使用-XX:+UseSpinning来开启。JDK 6中变为默认开启，并且引入了自适应的自旋锁（适应性自旋锁）。

自适应意味着自旋的时间（次数）不再固定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。如果对于某个锁，自旋很少成功获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。

在自旋锁中 另有三种常见的锁形式:TicketLock、CLHlock和MCSlock，本文中仅做名词介绍，不做深入讲解，感兴趣的同学可以自行查阅相关资料。

### 3. 无锁 VS 偏向锁 VS 轻量级锁 VS 重量级锁

这四种锁是指锁的状态，专门针对synchronized的。在介绍这四种锁状态之前还需要介绍一些额外的知识。

首先为什么Synchronized能实现线程同步？

在回答这个问题之前我们需要了解两个重要的概念：“Java对象头”、“Monitor”。

Java对象头

synchronized是悲观锁，在操作同步资源之前需要给同步资源先加锁，这把锁就是存在Java对象头里的，而Java对象头又是什么呢？

我们以Hotspot虚拟机为例，Hotspot的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）。

**Mark Word**：默认存储对象的HashCode，分代年龄和锁标志位信息。这些信息都是与对象自身定义无关的数据，所以Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据。它会根据对象的状态复用自己的存储空间，也就是说在运行期间Mark Word里存储的数据会随着锁标志位的变化而变化。

**Klass Point**：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

Monitor

Monitor可以理解为一个同步工具或一种同步机制，通常被描述为一个对象。每一个Java对象就有一把看不见的锁，称为内部锁或者Monitor锁。

Monitor是线程私有的数据结构，每一个线程都有一个可用monitor record列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个monitor关联，同时monitor中有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。

现在话题回到synchronized，synchronized通过Monitor来实现线程同步，Monitor是依赖于底层的操作系统的Mutex Lock（互斥锁）来实现的线程同步。

如同我们在自旋锁中提到的“阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态转换需要耗费处理器时间。如果同步代码块中的内容过于简单，状态转换消耗的时间有可能比用户代码执行的时间还要长”。这种方式就是synchronized最初实现同步的方式，这就是JDK 6之前synchronized效率低的原因。这种依赖于操作系统Mutex Lock所实现的锁我们称之为“重量级锁”，JDK 6中为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”。

所以目前锁一共有4种状态，级别从低到高依次是：无锁、偏向锁、轻量级锁和重量级锁。锁状态只能升级不能降级。

通过上面的介绍，我们对synchronized的加锁机制以及相关知识有了一个了解，那么下面我们给出四种锁状态对应的的Mark Word内容，然后再分别讲解四种锁状态的思路以及特点：

| 锁状态   | 存储内容                                                | 存储内容 |
| :------- | :------------------------------------------------------ | :------- |
| 无锁     | 对象的hashCode、对象分代年龄、是否是偏向锁（0）         | 01       |
| 偏向锁   | 偏向线程ID、偏向时间戳、对象分代年龄、是否是偏向锁（1） | 01       |
| 轻量级锁 | 指向栈中锁记录的指针                                    | 00       |
| 重量级锁 | 指向互斥量（重量级锁）的指针                            | 10       |

**无锁**

无锁没有对资源进行锁定，所有的线程都能访问并修改同一个资源，但同时只有一个线程能修改成功。

无锁的特点就是修改操作在循环内进行，线程会不断的尝试修改共享资源。如果没有冲突就修改成功并退出，否则就会继续循环尝试。如果有多个线程修改同一个值，必定会有一个线程能修改成功，而其他修改失败的线程会不断重试直到修改成功。上面我们介绍的CAS原理及应用即是无锁的实现。无锁无法全面代替有锁，但无锁在某些场合下的性能是非常高的。

**偏向锁**

偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁，降低获取锁的代价。

在大多数情况下，锁总是由同一线程多次获得，不存在多线程竞争，所以出现了偏向锁。其目标就是在只有一个线程执行同步代码块时能够提高性能。

当一个线程访问同步代码块并获取锁时，会在Mark Word里存储锁偏向的线程ID。在线程进入和退出同步块时不再通过CAS操作来加锁和解锁，而是检测Mark Word里是否存储着指向当前线程的偏向锁。引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次CAS原子指令，而偏向锁只需要在置换ThreadID的时候依赖一次CAS原子指令即可。

偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动释放偏向锁。偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，判断锁对象是否处于被锁定状态。撤销偏向锁后恢复到无锁（标志位为“01”）或轻量级锁（标志位为“00”）的状态。

偏向锁在JDK 6及以后的JVM里是默认启用的。可以通过JVM参数关闭偏向锁：-XX:-UseBiasedLocking=false，关闭之后程序默认会进入轻量级锁状态。

**轻量级锁**

是指当锁是偏向锁的时候，被另外的线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，从而提高性能。

在代码进入同步块的时候，如果同步对象锁状态为无锁状态（锁标志位为“01”状态，是否为偏向锁为“0”），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝，然后拷贝对象头中的Mark Word复制到锁记录中。

拷贝成功后，虚拟机将使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock Record里的owner指针指向对象的Mark Word。

如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象Mark Word的锁标志位设置为“00”，表示此对象处于轻量级锁定状态。

如果轻量级锁的更新操作失败了，虚拟机首先会检查对象的Mark Word是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行，否则说明多个线程竞争锁。

若当前只有一个等待线程，则该线程通过自旋进行等待。但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁升级为重量级锁。

**重量级锁**

升级为重量级锁时，锁标志的状态值变为“10”，此时Mark Word中存储的是指向重量级锁的指针，此时等待锁的线程都会进入阻塞状态。

整体的锁状态升级流程如下： ![图片5](https://img-blog.csdnimg.cn/20181226102235367)

综上，偏向锁通过对比Mark Word解决加锁问题，避免执行CAS操作。而轻量级锁是通过用CAS操作和自旋来解决加锁问题，避免线程阻塞和唤醒而影响性能。重量级锁是将除了拥有锁的线程以外的线程都阻塞。

### 4. 公平锁 VS 非公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁。公平锁的优点是等待锁的线程不会饿死。缺点是整体吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大。

非公平锁是多个线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾等待。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁，所以非公平锁有可能出现后申请锁的线程先获取锁的场景。非公平锁的优点是可以减少唤起线程的开销，整体的吞吐效率高，因为线程有几率不阻塞直接获得锁，CPU不必唤醒所有线程。缺点是处于等待队列中的线程可能会饿死，或者等很久才会获得锁。

直接用语言描述可能有点抽象，这里作者用从别处看到的一个例子来讲述一下公平锁和非公平锁。 ![图片6](https://img-blog.csdnimg.cn/20181226102235448) 如上图所示，假设有一口水井，有管理员看守，管理员有一把锁，只有拿到锁的人才能够打水，打完水要把锁还给管理员。每个过来打水的人都要管理员的允许并拿到锁之后才能去打水，如果前面有人正在打水，那么这个想要打水的人就必须排队。管理员会查看下一个要去打水的人是不是队伍里排最前面的人，如果是的话，才会给你锁让你去打水；如果你不是排第一的人，就必须去队尾排队，这就是公平锁。

但是对于非公平锁，管理员对打水的人没有要求。即使等待队伍里有排队等待的人，但如果在上一个人刚打完水把锁还给管理员而且管理员还没有允许等待队伍里下一个人去打水时，刚好来了一个插队的人，这个插队的人是可以直接从管理员那里拿到锁去打水，不需要排队，原本排队等待的人只能继续等待。如下图所示： ![图片7](https://img-blog.csdnimg.cn/20181226102235483)

接下来我们通过ReentrantLock的源码来讲解公平锁和非公平锁。 ![图片8](https://img-blog.csdnimg.cn/20181226102235512) 根据代码可知，ReentrantLock里面有一个内部类Sync，Sync继承AQS（AbstractQueuedSynchronizer），添加锁和释放锁的大部分操作实际上都是在Sync中实现的。它有公平锁FairSync和非公平锁NonfairSync两个子类。ReentrantLock默认使用非公平锁，也可以通过构造器来显示的指定使用公平锁。

下面我们来看一下公平锁与非公平锁的加锁方法的源码: ![图片9](https://img-blog.csdnimg.cn/20181226102235543) 通过上图中的源代码对比，我们可以明显的看出公平锁与非公平锁的lock()方法唯一的区别就在于公平锁在获取同步状态时多了一个限制条件：hasQueuedPredecessors()。 ![图片10](https://img-blog.csdnimg.cn/20181226102235589) 再进入hasQueuedPredecessors()，可以看到该方法主要做一件事情：主要是判断当前线程是否位于同步队列中的第一个。如果是则返回true，否则返回false。

综上，公平锁就是通过同步队列来实现多个线程按照申请锁的顺序来获取锁，从而实现公平的特性。非公平锁加锁时不考虑排队等待问题，直接尝试获取锁，所以存在后申请却先获得锁的情况。

### 5. 可重入锁 VS 非可重入锁

可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。Java中ReentrantLock和synchronized都是可重入锁，可重入锁的一个优点是可一定程度避免死锁。下面用示例代码来进行分析：

```java
public class Widget {
    public synchronized void doSomething() {
        System.out.println("方法1执行...");
        doOthers();
    }
    public synchronized void doOthers() {
        System.out.println("方法2执行...");
    }
}
```

在上面的代码中，类中的两个方法都是被内置锁synchronized修饰的，doSomething()方法中调用doOthers()方法。因为内置锁是可重入的，所以同一个线程在调用doOthers()时可以直接获得当前对象的锁，进入doOthers()进行操作。

如果是一个不可重入锁，那么当前线程在调用doOthers()之前需要将执行doSomething()时获取当前对象的锁释放掉，实际上该对象锁已被当前线程所持有，且无法释放。所以此时会出现死锁。

而为什么可重入锁就可以在嵌套调用时可以自动获得锁呢？我们通过图示和源码来分别解析一下。

还是打水的例子，有多个人在排队打水，此时管理员允许锁和同一个人的多个水桶绑定。这个人用多个水桶打水时，第一个水桶和锁绑定并打完水之后，第二个水桶也可以直接和锁绑定并开始打水，所有的水桶都打完水之后打水人才会将锁还给管理员。这个人的所有打水流程都能够成功执行，后续等待的人也能够打到水。这就是可重入锁。 ![图片11](https://img-blog.csdnimg.cn/20181226102235607) 但如果是非可重入锁的话，此时管理员只允许锁和同一个人的一个水桶绑定。第一个水桶和锁绑定打完水之后并不会释放锁，导致第二个水桶不能和锁绑定也无法打水。当前线程出现死锁，整个等待队列中的所有线程都无法被唤醒。 ![图片12](https://img-blog.csdnimg.cn/20181226102235698) 之前我们说过ReentrantLock和synchronized都是重入锁，那么我们通过重入锁ReentrantLock以及非可重入锁NonReentrantLock的源码来对比分析一下为什么非可重入锁在重复调用同步资源时会出现死锁。

首先ReentrantLock和NonReentrantLock都继承父类AQS，其父类AQS中维护了一个同步状态status来计数重入次数，status初始值为0。

当线程尝试获取锁时，可重入锁先尝试获取并更新status值，如果status == 0表示没有其他线程在执行同步代码，则把status置为1，当前线程开始执行。如果status != 0，则判断当前线程是否是获取到这个锁的线程，如果是的话执行status+1，且当前线程可以再次获取锁。而非可重入锁是直接去获取并尝试更新当前status的值，如果status != 0的话会导致其获取锁失败，当前线程阻塞。

释放锁时，可重入锁同样先获取当前status的值，在当前线程是持有锁的线程的前提下。如果status-1 == 0，则表示当前线程所有重复获取锁的操作都已经执行完毕，然后该线程才会真正释放锁。而非可重入锁则是在确定当前线程是持有锁的线程之后，直接将status置为0，将锁释放。 ![图片13](https://img-blog.csdnimg.cn/20181226102235724)

### 6. 独享锁 VS 共享锁

独享锁和共享锁同样是一种概念。我们先介绍一下具体的概念，然后通过ReentrantLock和ReentrantReadWriteLock的源码来介绍独享锁和共享锁。

独享锁也叫排他锁，是指该锁一次只能被一个线程所持有。如果线程T对数据A加上排它锁后，则其他线程不能再对A加任何类型的锁。获得排它锁的线程即能读数据又能修改数据。JDK中的synchronized和JUC中Lock的实现类就是互斥锁。

共享锁是指该锁可被多个线程所持有。如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排它锁。获得共享锁的线程只能读数据，不能修改数据。

独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。

下图为ReentrantReadWriteLock的部分源码： ![图片14](https://img-blog.csdnimg.cn/20181226102235772) 我们看到ReentrantReadWriteLock有两把锁：ReadLock和WriteLock，由词知意，一个读锁一个写锁，合称“读写锁”。再进一步观察可以发现ReadLock和WriteLock是靠内部类Sync实现的锁。Sync是AQS的一个子类，这种结构在CountDownLatch、ReentrantLock、Semaphore里面也都存在。

在ReentrantReadWriteLock里面，读锁和写锁的锁主体都是Sync，但读锁和写锁的加锁方式不一样。读锁是共享锁，写锁是独享锁。读锁的共享锁可保证并发读非常高效，而读写、写读、写写的过程互斥，因为读锁和写锁是分离的。所以ReentrantReadWriteLock的并发性相比一般的互斥锁有了很大提升。

那读锁和写锁的具体加锁方式有什么区别呢？在了解源码之前我们需要回顾一下其他知识。 在最开始提及AQS的时候我们也提到了state字段（int类型，32位），该字段用来描述有多少线程获持有锁。

在独享锁中这个值通常是0或者1（如果是重入锁的话state值就是重入的次数），在共享锁中state就是持有锁的数量。但是在ReentrantReadWriteLock中有读、写两把锁，所以需要在一个整型变量state上分别描述读锁和写锁的数量（或者也可以叫状态）。于是将state变量“按位切割”切分成了两个部分，高16位表示读锁状态（读锁个数），低16位表示写锁状态（写锁个数）。如下图所示： ![图片15](https://img-blog.csdnimg.cn/20181226102235821)

了解了概念之后我们再来看代码，先看写锁的加锁源码：

```java
protected final boolean tryAcquire(int acquires) {
	Thread current = Thread.currentThread();
	int c = getState(); // 取到当前锁的个数
	int w = exclusiveCount(c); // 取写锁的个数w
	if (c != 0) { // 如果已经有线程持有了锁(c!=0)
    // (Note: if c != 0 and w == 0 then shared count != 0)
		if (w == 0 || current != getExclusiveOwnerThread()) // 如果写线程数（w）为0（换言之存在读锁） 或者持有锁的线程不是当前线程就返回失败
			return false;
	if (w + exclusiveCount(acquires) > MAX_COUNT)   
    // 如果写入锁的数量大于最大数（65535，2的16次方-1）就抛出一个Error。
    throw new Error("Maximum lock count exceeded");
	// Reentrant acquire
    setState(c + acquires);
    return true;
  }
  if (writerShouldBlock() || !compareAndSetState(c, c + acquires)) 
  // 如果当且写线程数为0，并且当前线程需要阻塞那么就返回失败；或者如果通过CAS增加写线程数失败也返回失败。
	return false;
	setExclusiveOwnerThread(current); // 如果c=0，w=0或者c>0，w>0（重入），则设置当前线程或锁的拥有者
	return true;
}
```

- 这段代码首先取到当前锁的个数c，然后再通过c来获取写锁的个数w。因为写锁是低16位，所以取低16位的最大值与当前的c做与运算（ int w = exclusiveCount(c); ），高16位和0与运算后是0，剩下的就是低位运算的值，同时也是持有写锁的线程数目。
- 在取到写锁线程的数目后，首先判断是否已经有线程持有了锁。如果已经有线程持有了锁(c!=0)，则查看当前写锁线程的数目，如果写线程数为0（即此时存在读锁）或者持有锁的线程不是当前线程就返回失败（涉及到公平锁和非公平锁的实现）。
- 如果写入锁的数量大于最大数（65535，2的16次方-1）就抛出一个Error。
- 如果当且写线程数为0（那么读线程也应该为0，因为上面已经处理c!=0的情况），并且当前线程需要阻塞那么就返回失败；如果通过CAS增加写线程数失败也返回失败。
- 如果c=0,w=0或者c>0,w>0（重入），则设置当前线程或锁的拥有者，返回成功！

tryAcquire()除了重入条件（当前线程为获取了写锁的线程）之外，增加了一个读锁是否存在的判断。如果存在读锁，则写锁不能被获取，原因在于：必须确保写锁的操作对读锁可见，如果允许读锁在已被获取的情况下对写锁的获取，那么正在运行的其他读线程就无法感知到当前写线程的操作。

因此，只有等待其他读线程都释放了读锁，写锁才能被当前线程获取，而写锁一旦被获取，则其他读写线程的后续访问均被阻塞。写锁的释放与ReentrantLock的释放过程基本类似，每次释放均减少写状态，当写状态为0时表示写锁已被释放，然后等待的读写线程才能够继续访问读写锁，同时前次写线程的修改对后续的读写线程可见。

接着是读锁的代码：

```java
protected final int tryAcquireShared(int unused) {

    Thread current = Thread.currentThread();
    int c = getState();

    if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)
        return -1;                                   
        // 如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状

    int r = sharedCount(c);
    if (!readerShouldBlock() && r < MAX_COUNT &&compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

可以看到在tryAcquireShared(int unused)方法中，如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状态。如果当前线程获取了写锁或者写锁未被获取，则当前线程（线程安全，依靠CAS保证）增加读状态，成功获取读锁。读锁的每次释放（线程安全的，可能有多个读线程同时释放读锁）均减少读状态，减少的值是“1<<16”。所以读写锁才能实现读读的过程共享，而读写、写读、写写的过程互斥。

此时，我们再回头看一下互斥锁ReentrantLock中公平锁和非公平锁的加锁源码： ![图片16](https://img-blog.csdnimg.cn/20181226102235845) 我们发现在ReentrantLock虽然有公平锁和非公平锁两种，但是它们添加的都是独享锁。根据源码所示，当某一个线程调用lock方法获取锁时，如果同步资源没有被其他线程锁住，那么当前线程在使用CAS更新state成功后就会成功抢占该资源。而如果公共资源被占用且不是被当前线程占用，那么就会加锁失败。所以可以确定ReentrantLock无论读操作还是写操作，添加的锁都是都是独享锁。

​	

------

# jvm面试题



### 1.Java中JVM,JRE和JDK的区别

**答：**三者的基本概念可以概括如下：

- **JDK（Java Development Kit）**是一个开发工具包，是Java开发环境的核心组件，并且提供编译、调试和运行一个Java程序所需要的所有工具，可执行文件和二进制文件，是一个平台特定的软件
- **JRE（Java Runtime Environment）**是指Java运行时环境，是JVM的实现，提供了运行Java程序的平台。JRE包含了JVM，但是不包含Java编译器/调试器之类的开发工具
- **JVM（Java Virtual Machine）**是指Java虚拟机，当我们运行一个程序时，JVM负责将字节码转换为特定机器代码，JVM提供了内存管理/垃圾回收和安全机制等

**区别与联系：**

- JDK是开发工具包，用来开发Java程序，而JRE是Java的运行时环境
- JDK和JRE中都包含了JVM
- JVM是Java编程的核心，独立于硬件和操作系统，具有平台无关性，而这也是Java程序可以一次编写，多处执行的原因



### 2. 说一下堆栈的区别？

- 功能方面：堆是用来存放对象的，栈是用来执行程序的。
- 共享性：堆是线程共享的，栈是线程私有的。
- 空间大小：堆大小远远大于栈。



### 3. 描述一下JVM加载class文件的原理机制？

答：JVM中类的装载是由类加载器（ClassLoader）和它的子类来实现的，Java中的类加载器是一个重要的Java运行时系统组件，它负责在运行时查找和装入类文件中的类。类的加载是指把类的.class文件中的数据读入到内存中，通常是创建一个字节数组读入.class文件



### 4. Java 内存泄漏和内存溢出

内存泄漏：

理论上Java因为有垃圾回收机制（GC）不会存在内存泄露问题（这也是Java被广泛使用于服务器端编程的一个重要原因）；然而在实际开发中，可能会存在无用但可达的对象，这些对象不能被GC回收，因此也会导致内存泄露的发生。

例如hibernate的Session（一级缓存）中的对象属于持久态，垃圾回收器是不会回收这些对象的，然而这些对象中可能存在无用的垃圾对象，如果不及时关闭（close）或清空（flush）一级缓存就可能导致内存泄露



内存溢出：

答：指程序申请内存时，没有足够的内存供申请者使用，或者说，给了你一块存储int类型数据的存储空间，但是你却存储long类型的数据，那么结果就是内存不够用，此时就会报错OOM,即所谓的内存溢出。

### 5. 什么是虚拟机？

一台虚拟的计算机。是一款软件，用来执行一系列虚拟计算机指令。

例：

Visual Box，VMware属于系统虚拟机，完全是对物理计算机的仿真，提供了可运行完整操作系统的软件平台

java虚拟机是程序虚拟机，专门为执行单个计算机程序而设计，在java虚拟机中执行的指令我们成为java字节码指令

#### 5.1 什么是jvm虚拟机？

- JVM 是 java虚拟机，是用来执行java字节码(二进制的形式)的虚拟计算机
- JVM 是运行在操作系统之上的，与硬件没有任何关系

#### 5.2 jvm虚拟机的作用：

答： java虚拟机就是二进制字节码的运行环境，负责装载字节码到其内部，解释/编译为对应平台上的机器指令执行。每一条java指令，java虚拟机规范中都有详细定义，如怎么取操作数，怎么处理操作数，结果放在哪里

#### 5.3 jvm虚拟机特点：

- 一次编译，到处运行
- 自动内存管理
- 自动垃圾回收功能



### 6.  简述Java的垃圾回收机制

传统的C/C++语言，需要程序员负责回收已经分配内存。

**显式回收垃圾回收的缺点：**

1） 程序忘记及时回收，从而导致内存泄露，降低系统性能。

2） 程序错误回收程序核心类库的内存，导致系统崩溃。

Java语言不需要程序员直接控制内存回收，是由JRE在后台自动回收不再使用的内存，称为垃圾回收机制，简称GC；

 1）可以提高编程效率。

2） 保护程序的完整性。

3） 其开销影响性能。Java虚拟机必须跟踪程序中有用的对象，确定哪些是无用的。

**垃圾回收机制的特点**

1） 垃圾回收机制回收JVM堆内存里的对象空间,不负责回收栈内存数据。

2） 对其他物理连接，比如数据库连接、输入流输出流、Socket连接无能为力。

3） 垃圾回收发生具有不可预知性，程序无法精确控制垃圾回收机制执行。

4） 可以将对象的引用变量设置为null，暗示垃圾回收机制可以回收该对象。 现在的JVM有多种垃圾回收实现算法，表现各异。



垃圾回收机制回收任何对象之前，总会先调用它的ﬁnalize方法（如果覆盖该方法，让一个新的引用变量重新引用该对象，则会重新激活对象）。

程序员可以通过System.gc()或者Runtime.getRuntime().gc()来通知系统进行垃圾回收，会有一些效果，但是系统是   否进行垃圾回收依然不确定。

永远不要主动调用某个对象的ﬁnalize方法，应该交给垃圾回收机制调用。



### 7.简述Java中的常量池

两种形态：**静态常量池**和**运行时常量池**。

静态常量池，就是 class文件中的常量池。

​		class文件中的常量池不仅仅包含字符串(数字)字面量，还包含类、方法的信息，占用class文件绝大部分空间。

​		这种常量池主要用于存放两大类常量：**字面量**(Literal)和**符号引用量**(Symbolic References)，字面量相当于Java语言层面常量的概念如文本字符串，声明为final的常量值等，符号引用则属于编译原理方面的概念，包括了如下三种类型的常量：

- 类和接口的全限定名
- 字段名称和描述符
- 方法名称和描述符

源文件：

```java
public class HelloWorld{
	public static void main(String args[]){
		System.out.println("hello world");

	}
}
```

生成的class文件：

![img](https://img-blog.csdn.net/20160331152504448?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



1>  **魔数**

魔数就是这个文件的前四个字节：ca fe ba be(漱壕).它的唯一作用是确定这个屋文件是否可以被JVM接受。很多文件存储标准中都使用魔术来进行身份识别。

2>  **版本号**

第5和第6个字节是次版本号，第7个和第8 个是主版本号。这里的第7和第8位是0034，即：0x0034。0x0034转为10进制是52。Java的版本是从45开始的然而从1.0 到1.1 是45.0到45.3, 之后就是1.2 对应46， 1.3 对应47 … 1.6 对应50,我这里是1.6.0_24对应的是52，就是0x0034;

3>  **常量池的入口**

由于常量池中的常量的数量不是固定的，所以常量池的入口需要放置一项u2类型的数据，代表常量池的容量计数值。这里的常量池容量计数值是从1开始的。如图常量池的容量：0x001d(29)。所以共有29个常量。

4>  **常量池**

常量池中主要存放两类常量：字面量和符号引用。字面量比较接近Java语言层面的常量概念。就是我们什么提到的常量。而符号引用则属于编译原理的方面的概念。包括以下三类常量：

   i> 类和接口的全限定名

  ii>字段的名称和描述符

  iii>方法的名称和描述符



运行时常量池，是jvm虚拟机在完成类装载操作后，将class文件中的常量池载入到内存中，并保存在**方法区**中，我们常说的常量池，就是指方法区中的运行时常量池。

运行时常量池包含：

- 类、接口、方法和类字段的表述信息

- 字符串常量池

- 被final所修饰的类变量

- 自动包装类Byte,Short,Integer,Long,Character在-128到127之间值。




### 8.JMM是什么？

JMM即为JAVA 内存模型（java memory model）。因为在不同的硬件生产商和不同的操作系统下，内存的访问逻辑有一定的差异，结果就是当你的代码在某个系统环境下运行良好，并且线程安全，但是换了个系统就出现各种问题。Java内存模型，就是为了屏蔽系统和硬件的差异，让一套代码在不同平台下能到达相同的访问结果。JMM从java 5开始的JSR-133发布后，已经成熟和完善起来。



Java内存模型规定和指引Java程序在不同的内存架构、CPU和操作系统间有确定性地行为。它在多线程的情况下尤其重要。Java内存模型对一个线程所做的变动能被其它线程可见提供了保证，它们之间是先行发生关系。这个关系定义了一些规则让程序员在并发编程时思路更清晰。比如，先行发生关系确保了：

- 线程内的代码能够按先后顺序执行，这被称为程序次序规则。
- 对于同一个锁，在时间上，一个解锁操作一定要发生后发生的另一个锁定操作之前，也叫做管程锁定规则。
- 前一个对volatile的写操作在后一个volatile的读操作之前，也叫volatile变量规则。
- 一个线程内的任何操作必需在这个线程的start()调用之后，也叫作线程启动规则。
- 一个线程的所有操作都会在线程终止之前，线程终止规则。
- -一个对象的终结操作必需在这个对象构造完成之后，也叫对象终结规则。
- 可传递性

#### 内存划分

　　JMM规定了内存主要划分为**主内存和工作内存两种**。此处的主内存和工作内存跟JVM内存划分（堆、栈、方法区）是在不同的层次上进行的，如果非要对应起来，主内存对应的是Java堆中的对象实例部分，**工作内存对应的是栈中的部分区域，从更底层的来说，主内存对应的是硬件的物理内存，工作内存对应的是寄存器和高速缓存。**

JVM在设计时候考虑到，如果JAVA线程每次读取和写入变量都直接操作主内存，对性能影响比较大，所以每条线程拥有各自的工作内存，工作内存中的变量是主内存中的一份拷贝，线程对变量的读取和写入，直接在工作内存中操作，而不能直接去操作主内存中的变量。但是这样就会出现一个问题，当一个线程修改了自己工作内存中变量，对其他线程是不可见的，会导致线程不安全的问题。因为JMM制定了一套标准来保证开发者在编写多线程程序的时候，能够控制什么时候内存会被同步给其他线程。

#### 内存交互操作

 　内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的

（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）

- - lock   （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
  - unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
  - read  （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
  - load   （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
  - use   （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
  - assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
  - store  （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
  - write 　（写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

　JMM对这八种指令的使用，制定了如下规则：

- - 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
  - 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
  - 不允许一个线程将没有assign的数据从工作内存同步回主内存
  - 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
  - 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
  - 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
  - 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
  - 对一个变量进行unlock操作之前，必须把此变量同步回主内存

　　JMM对这八种操作规则和对[**volatile的一些特殊规则]**就能确定哪里操作是线程安全，哪些操作是线程不安全的了。但是这些规则实在复杂，很难在实践中直接分析。所以一般我们也不会通过上述规则进行分析。更多的时候，使用java的happen-before规则来进行分析。



#### 模型特征

　　**原子性：**操作系统里面是不可分割的单元。被synchronized关键字或其他锁包裹起来的操作也可以认为是原子的。从一个线程观察另外一个线程的时候，看到的都是一个个原子性的操作。

　　**可见性：**每个工作的线程都有自己的工作内存，所以当某个线程修改完某个变量之后，在其他的线程里面，未必能观察到该变量已经被修改。volatile关键字要求被修改之后的变量要求立即更新到主内存，每次使用前从主内存处进行读取。因此volatile可以保证可见性。除了volatile以外，synchronized和final也能实现可见性。

synchronized保证unlock之前必须先把变量刷新回主内存。final修饰的字段在构造器中一旦完成初始化，并且构造器没有this逸出，那么其他线程就能看到final字段的值。

　　**有序性：**java如果在线程内部观察，会发现当前线程的一切操作都是有序的。如果在线程的外部来观察的话，会发现线程的所有操作都是无序的。因为JMM的工作内存和主内存之间存在延迟，而且java会对一些指令进行重新排序。volatile和synchronized可以保证程序的有序性，volatile和synchronized能保证指令不进行重排序。



#### Happen-Before（先行发生规则）

　　在常规的开发中，如果我们通过上述规则来**分析一个并发程序是否安全**，估计脑壳会很疼。因为更多时候，我们是分析一个并发程序是否安全，其实都依赖Happen-Before原则进行分析。Happen-Before被翻译成先行发生原则，意思就是当A操作先行发生于B操作，则在发生B操作的时候，操作A产生的影响能被B观察到，“影响”包括修改了内存中的共享变量的值、发送了消息、调用了方法等。

　　Happen-Before的规则有以下几条

- - 程序次序规则：在一个线程内，程序的执行规则跟程序的书写规则是一致的，从上往下执行。
  - 管程锁定规则：一个Unlock的操作肯定先于下一次Lock的操作。这里必须是同一个锁。同理我们可以认为在synchronized同步同一个锁的时候，锁内先行执行的代码，对后续同步该锁的线程来说是完全可见的。
  - volatile变量规则：对同一个volatile的变量，先行发生的写操作，肯定早于后续发生的读操作
  - 线程启动规则：Thread对象的start()方法先行发生于此线程的每一个动作
  - 线程中止规则：Thread对象的中止检测（如：Thread.join()，Thread.isAlive()等）操作，必定晚于线程中所有操作
  - 线程中断规则：对线程的 interruption() 调用，先于被调用的线程检测中断事件(Thread.interrupted())的发生
  - 对象中止规则：一个对象的初始化方法先于一个方法执行Finalizer()方法
  - 传递性：如果操作A先于操作B、操作B先于操作C,则操作A先于操作C



### 9.JVM内存模型图

①**堆**和**方法区（元空间）**线程共享

②一个线程开启一块**栈**空间，里面的每一个方法为**栈桢**

③栈桢：**局部变量表、操作数栈、动态链接、方法出口**

④**本地方法栈**：调用底层native方法（其他语言）实现

⑤**程序计数器：**线程切换时，记录当前线程执行位置，切换回来后继续执行

⑥**方法区（元空间）**：存储static、final常量、类信息、字符串常量

#### 9.1堆模型

①新生代占1/3，老年代占2/3

②对象出生在Eden元区，GC清理到S0，S0满GC清理给S1（S0和S1内存区域会交换 MinorGC）

③对象数字到达15，进入老年区

④老年代满产生FullGC，同时会有一个很长时间的STW，影响比较大

⑤JVM调优的目的是不让Old区满，出现OOM（Out Of Memory）或频繁的STW（Stop The World），尽量让对象在新生代就被销毁掉

### 10类加载机制

#### 10.1类的生命周期

1.加载：加载.class文件到内存

2.连接

​	2.1验证：验证字节码文件

​	2.2准备：给静态变量分配内存，赋默认值

​	2.3解析：类装载器装入类所引用的其他所有类

3.初始化：为类的静态变量赋真正的初始值

4.使用：

5.卸载：

#### 10.2类加载器

启动类加载器：启动加载JRE的核心类库

扩展类加载器：JRE扩展目录ext中jar类包

系统类加载器：ClassPath路径下的类包

用户自定义加载器：加载用户自定义路径下的类包

#### 10.3类加载机制

**全盘负责委托机制：**

​	当一个ClassLoader加载一个类的时候，除非显示的使用另一个ClassLoader，该类所依赖和引用的类也由这个	ClassLoader载入。

**双亲委派机制：**

​	向上委托父类先进行加载。

​	优点：①沙箱安全：比如自己写的String.class，类不会被加载，防止核心库被改。

​				②避免重复加载：当父类ClassLoader已加载，子类ClassLoader不会加载。

### 11GC算法（如何判断对象可以被回收）

#### 11.1引用记数法

引用+1	不引用-1	计数器为0表示该对象不会再被使用

缺点：若a,b相互引用，则GC无法回收

#### 11.2可达性分析法

GC Roots根节点：

1. 方法区中的静态属性（静态属性指向一个对象）
2. 方法区的中的常量（常量指向一个对象）
3. 虚拟机中的局部变量（变量指向一个对象）
4. 本地方法栈中JNI（native修饰的方法指向的对象）



向下搜索“引用链”：强引用、软引用、弱引用、虚引用

#### 11.3如何判断一个类是无用类

①该类的所有实例已经被回收，Java堆中不存在该类的任何实例

②加载该类的ClassLoader已经被回收

③该类对应的java.lang.class对象没有在任何地方被引用，无法在任何地方通过反射访问该类

### 12.垃圾回收算法

#### 12.1.标记-清除算法（Eden）

可用内存 可回收内存 存活对象

先标记可回收内存，然后统一清除。

1.效率问题：效率不高

2.空间问题：大量不连续碎片

#### 12.2复制算法（s0，s1）

多了一块**保留内存**

标记-清除完后，将存活对象放入保留内存，然后空出来的内存再做保留内存

挺高效率，但是浪费空间，适合小空间

#### 12.3标记-整理算法

先标记可回收，然后将存活对象向一段移动，然后清理

#### 12.4分代收集算法





### 13.垃圾收集器

Young：Serial、Par New、Parallel Scavenge

Old：CMS、Serial Old、Parallel Old

皆可：G1

#### 13.1Serial（串行）收集器

单线程：只会使用一条垃圾收集线程去完成垃圾收集工作

STW：它在进行垃圾收集工作时，必须暂停其他所有工作线程

新时代：复制算法	老年代：标记整理算法

#### 13.2Par New（多线程）收集器

除了使用多线程进行垃圾收集以外，其余行为和Serial行为一样。还是要停掉用户线程。

#### 13.3Parallel Scavenge（多线程）收集器（默认）

与Par New区别：

两者都是复制算法，都是并行处理。但是不同的是，paralel scavenge 可以设置

**最大gc停顿时间**（-XX:MaxGCPauseMills）和**gc时间占比**(-XX:GCTimeRatio)，

#### 13.4CMS收集器

并行：多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。

并发：指用户线程和垃圾收集线程同时执行（但不一定是并行，可能会交替执行），用户程序继续运作，垃圾收集器运行在另一CPU上。

**CMS（Concurrent Mark Sweep）：**目标是最短回收停顿时间。是标记-清除算法的实现。

①初始标记：（CMS Initial mark）：暂停所有的其他线程，并记录下直接与Root相连的对象，速度很快

②并发标记：（CMS concurrent mark）：同时开始GC和用户线程，用一个闭包结构去记录可达对象。**但是GC线程无法保证可达性分析法能追踪到所有的不断更新的引用。**

③重新标记：（CMS remark）：修正并发标记期间用户程序继续运行而产生变动的记录。停顿时间比初始标记长，但远比并发标记短。

④并发清除：（CMS concurrent sweep）：开启用户线程，同时GC线程对标记的区域清除。

**优点：**并发收集、停顿低

**缺点：**①对CPU资源敏感；②无法处理浮动垃圾；③回收-标记算法会产生大量空间碎片。

#### 13.5 G1收集器（推荐）

G1（Garbage First）：内存分块Eden、Survivor、Old、**Humongous**。发生的是**MixGc**。

**可预测的停顿：**让使用测明确指定一个长度M毫秒的时间判断作为停顿时间。

①初始标记：

②并发标记： 

③最终标记：

④筛选回收：在后台维护一个优先列表，根据每次允许的收集时间，优先选择回收价值最大的Region。



### 14.Java语言的平台无关性是如何实现的？

- JVM屏蔽了操作系统和底层硬件的差异
- Java面向JVM编程，先编译生成字节码文件，然后交给JVM解释成机器码执行
- 通过规定基本数据类型的取值范围和行为



------

# 数据库面试题

### mysql 为什么用b+树结构，而不用哈希什么的？

主要是b+树数据都存在叶子节点，然后非叶子节点就存了主键和指针，比较少，加载到内存中的数据更多，这样查找数据磁盘IO次数少，并且叶子节点还是有序的适合范围查询，而哈希的话对于等值查询来说很好，但是像范围查询啊就比较无力了。

```mysql
//创建数据库
CREATE DATABASE booksales；
//创建表格
CREATE TABLE `cb` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '主键',
);
//插入数据
insert  into `customer`(`cid`,`cname`,`sex`,`email`,`tel`,`address`) values ('zb01','Jack','男','jack@126.com','13100010001','上海市杨浦区国顺路288号');
//删除一行数据
delete  from book where bid = 'b123';
//修改一行数据
UPDATE book SET price = '九折' where press = '清华大学出版社出版';
```



```mysql
1.creat table student()：//在table中建立一个student的表

2.alter  table student add 年龄 number(3)：往表中插入一个列

3.alter table student drop column age：删除表中的age列

4.alter table student rename column 年龄 to age：修改年龄为age

5.alter table student modify age varchar2(20)：修改列名属性

6.rename student to stu：修改表名

7.drop table stu：删除表

8.instert into teachers values(1,'小小'):插入数据

9.update teachers set name = '托尼' where ids = 9：修改name为托尼
```



```mysql
1.select * from emp where deptno in(10,20) : 查询某一区间

2.select * from emp where ename like '%A%'：查询是否含有某个字符

3.select * from emp where ename like '__'：查询是否含有两个字符的名字

4.select * from emp where sal > 2000 or sal < 1000：只要有一个成立即可

5.select sal + nvl(comm,0) 总工资 from emp：查询相加运算

6.select rownum,E.* from emp e where rownum < 15：进行排序（伪列）

7.select * from emp order by sal desc：将工资进行降序排列(ASC是默认升序)
```



### 1. 数据库的三范式是什么？

- 第一范式：强调的是列的原子性，即数据库表的每一列都是不可分割的原子数据项。

- 第二范式：属性完全依赖于主键

- 第三范式：任何非主属性不依赖于其它非主属性。

  ​	数据不能存在传递关系，即每个属性都跟主键有直接关系而不是间接关系。从而建立冗余较小、结构合理的数据库。



### 2. 一张自增表里面总共有 7 条数据，删除了最后 2 条数据，重启 MySQL 数据库，又插入了一条数据，此时 id 是几？

- 表类型如果是 MyISAM ，那 id 就是 8。

- 表类型如果是 InnoDB，那 id 就是 6。

  InnoDB 表只会把自增主键的最大 id 记录在内存中，所以重启之后会导致最大 id 丢失。



### 3. MySQL 的内连接、左连接、右连接有什么区别？

内连接关键字：inner join；左连接：left join；右连接：right join。

内连接是把匹配的关联数据显示出来；

左连接是左边的表全部显示出来，右边的表显示出符合条件的数据；

右连接正好相反。



### 4.什么是事务？

1. 事务是数据库系统区别于其他一切文件系统的重要特性之一
2. 事务是一组具有原子性的SQL语句，或是一个独立的工作单元

定义:

**数据库事务是构成单一逻辑工作单元的操作集合**



### 5 事务的ACID特性以及实现原理概述

原子性(Atomicity):事务中的所有操作作为一个整体像原子一样不可分割，要么全部成功,要么全部失败。

一致性(Consistency):事务的执行结果必须使数据库从一个一致性状态到另一个一致性状态。一致性状态是指:1.系统的状态满足数据的完整性约束(主码,参照完整性,check约束等) 2.系统的状态反应数据库本应描述的现实世界的真实状态,比如转账前后两个账户的金额总和应该保持不变。

隔离性(Isolation):并发执行的事务不会相互影响,其对数据库的影响和它们串行执行时一样。比如多个用户同时往一个账户转账,最后账户的结果应该和他们按先后次序转账的结果一样。

持久性(Durability):事务一旦提交,其对数据库的更新就是持久的。任何事务或系统故障都不会导致数据丢失。

在事务的ACID特性中,C即一致性是事务的根本追求,而对数据一致性的破坏主要来自两个方面

- 1.事务的并发执行
- 2.事务故障或系统故障

数据库系统是通过并发控制技术和日志恢复技术来避免这种情况发生的。

并发控制技术保证了事务的隔离性,使数据库的一致性状态不会因为并发执行的操作被破坏。
日志恢复技术保证了事务的原子性,使一致性状态不会因事务或系统故障被破坏。同时使已提交的对数据库的修改不会因系统崩溃而丢失,保证了事务的持久性。



#### 5.1 常见的并发异常

- 脏写
  脏写是指事务回滚了其他事务对数据项的已提交修改,比如下面这种情况

在事务1对数据A的回滚,导致事务2对A的已提交修改也被回滚了。

- 丢失更新
  丢失更新是指事务覆盖了其他事务对数据的已提交修改,导致这些修改好像丢失了一样。

事务1和事务2读取A的值都为10,事务2先将A加上10并提交修改,之后事务2将A减少10并提交修改,A的值最后为,导致事务2对A的修改好像丢失了一样

- 脏读
  脏读是指一个事务读取了另一个事务未提交的数据

在事务1对A的处理过程中,事务2读取了A的值,但之后事务1回滚,导致事务2读取的A是未提交的脏数据。

- 不可重复读
  不可重复读是指一个事务对同一数据的读取结果前后不一致。脏读和不可重复读的区别在于:前者读取的是事务未提交的脏数据,后者读取的是事务已经提交的数据,只不过因为数据被其他事务修改过导致前后两次读取的结果不一样,比如下面这种情况

由于事务2对A的已提交修改,事务1前后两次读取的结果不一致。

- 幻读
  幻读是指事务读取某个范围的数据时，因为其他事务的操作导致前后两次读取的结果不一致。幻读和不可重复读的区别在于,不可重复读是针对确定的某一行数据而言,而幻读是针对不确定的多行数据。因而幻读通常出现在带有查询条件的范围查询中,比如下面这种情况:

事务1查询A<5的数据,由于事务2插入了一条A=4的数据,导致事务1两次查询得到的结果不一样



#### 5.2 事务的隔离级别

1. 事务具有隔离性,理论上来说事务之间的执行不应该相互产生影响,其对数据库的影响应该和它们串行执行时一样。
2. 然而完全的隔离性会导致系统并发性能很低,降低对资源的利用率,因而实际上对隔离性的要求会有所放宽,这也会一定程度造成对数据库一致性要求降低
3. SQL标准为事务定义了不同的隔离级别,从低到高依次是

- 读未提交(READ UNCOMMITTED)
- 读已提交(READ COMMITTED)
- 可重复读(REPEATABLE READ)
- 串行化(SERIALIZABLE)

事务的隔离级别越低,可能出现的并发异常越多,但是通常而言系统能提供的并发能力越强。

不同的隔离级别与可能的并发异常的对应情况如下表所示,有一点需要强调,这种对应关系只是理论上的,对于特定的数据库实现不一定准确,比如mysql的Innodb存储引擎通过Next-Key Locking技术在可重复读级别就消除了幻读的可能。



### 6.基于封锁的并发控制实现原理

核心思想:对于并发可能冲突的操作,比如读-写,写-读,写-写,通过锁使它们互斥执行。
锁通常分为共享锁和排他锁两种类型

- 1.共享锁(S):事务T对数据A加共享锁,其他事务只能对A加共享锁但不能加排他锁。
- 2.排他锁(X):事务T对数据A加排他锁,其他事务对A既不能加共享锁也不能加排他锁

基于锁的并发控制流程:

1. 事务根据自己对数据项进行的操作类型申请相应的锁(读申请共享锁,写申请排他锁)
2. 申请锁的请求被发送给锁管理器。锁管理器根据当前数据项是否已经有锁以及申请的和持有的锁是否冲突决定是否为该请求授予锁。
3. 若锁被授予,则申请锁的事务可以继续执行;若被拒绝,则申请锁的事务将进行等待,直到锁被其他事务释放。

可能出现的问题:

- 死锁:多个事务持有锁并互相循环等待其他事务的锁导致所有事务都无法继续执行。
- 饥饿:数据项A一直被加共享锁,导致事务一直无法获取A的排他锁。

对于可能发生冲突的并发操作,锁使它们由并行变为串行执行,是一种悲观的并发控制。



### 7.基于时间戳的并发控制

核心思想:对于并发可能冲突的操作,基于时间戳排序规则选定某事务继续执行,其他事务回滚。

系统会在每个事务开始时赋予其一个时间戳,这个时间戳可以是系统时钟也可以是一个不断累加的计数器值,当事务回滚时会为其赋予一个新的时间戳,先开始的事务时间戳小于后开始事务的时间戳。

每一个数据项Q有两个时间戳相关的字段:
W-timestamp(Q):成功执行write(Q)的所有事务的最大时间戳
R-timestamp(Q):成功执行read(Q)的所有事务的最大时间戳

时间戳排序规则如下:

1. 假设事务T发出read(Q),T的时间戳为TS
   a.若TS(T)<W-timestamp(Q),则T需要读入的Q已被覆盖。此
   read操作将被拒绝,T回滚。
   b.若TS(T)>=W-timestamp(Q),则执行read操作,同时把
   R-timestamp(Q)设置为TS(T)与R-timestamp(Q)中的最大值
2. 假设事务T发出write(Q)
   a.若TS(T)<R-timestamp(Q),write操作被拒绝,T回滚。
   b.若TS(T)<W-timestamp(Q),则write操作被拒绝,T回滚。
   c.其他情况:系统执行write操作,将W-timestamp(Q)设置
   为TS(T)。

基于时间戳排序和基于锁实现的本质一样:对于可能冲突的并发操作,以串行的方式取代并发执行,因而它也是一种悲观并发控制。它们的区别主要有两点:

- 基于锁是让冲突的事务进行等待,而基于时间戳排序是让冲突的事务回滚。
- 基于锁冲突事务的执行次序是根据它们申请锁的顺序,先申请的先执行;而基于时间戳排序是根据特定的时间戳排序规则。



### 8. MySQL 索引是怎么实现的？

索引是满足某种特定查找算法的数据结构，而这些数据结构会以某种方式指向数据，从而实现高效查找数据。

具体来说 MySQL 中的索引，不同的数据引擎实现有所不同，但目前主流的数据库引擎的索引都是 B+ 树实现的

B+ 树的搜索效率，可以到达二分法的性能，找到数据区域之后就找到了完整的数据结构了，所有索引的性能也是更好的。



### 9. Mysql四种常见数据库引擎

**InnoDB存储引擎**

事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键，上图也看到了，InnoDB是默认的MySQL引擎。

1、InnoDB给MySQL提供了具有提交、回滚和崩溃恢复能力的事物安全（ACID兼容）存储引擎。InnoDB锁定在行级并且也在SELECT语句中提供一个类似[**Oracle**](http://lib.csdn.net/base/oracle)的非锁定读。这些功能增加了多用户部署和性能。在SQL查询中，可以自由地将InnoDB类型的表和其他MySQL的表类型混合起来，甚至在同一个查询中也可以混合

2、InnoDB是为处理巨[**大数据**](http://lib.csdn.net/base/hadoop)量的最大性能设计。它的CPU效率可能是任何其他基于磁盘的关系型数据库引擎锁不能匹敌的

3、InnoDB存储引擎完全与MySQL服务器整合，InnoDB存储引擎为在主内存中缓存数据和索引而维持它自己的缓冲池。InnoDB将它的表和索引在一个逻辑表空间中，表空间可以包含数个文件（或原始磁盘文件）。这与MyISAM表不同，比如在MyISAM表中每个表被存放在分离的文件中。InnoDB表可以是任何尺寸，即使在文件尺寸被限制为2GB的[**操作系统**](http://lib.csdn.net/base/operatingsystem)上

4、InnoDB支持外键完整性约束，存储表中的数据时，每张表的存储都按主键顺序存放，如果没有显示在表定义时指定主键，InnoDB会为每一行生成一个6字节的ROWID，并以此作为主键

5、InnoDB被用在众多需要高性能的大型数据库站点上

InnoDB不创建目录，使用InnoDB时，MySQL将在MySQL数据目录下创建一个名为ibdata1的10MB大小的自动扩展数据文件，以及两个名为ib_logfile0和ib_logfile1的5MB大小的日志文件

**MyISAM存储引擎**

它是在Web、数据仓储和其他应用环境下最常使用的存储引擎之一。MyISAM拥有较高的插入、查询速度，但**不支持事物**。

**MEMORY存储引擎**

MEMORY存储引擎将表中的数据存储到内存中，未查询和引用其他表数据提供快速访问。



### 9. 说一下乐观锁和悲观锁？

- 乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在提交更新的时候会判断一下在此期间别人有没有去更新这个数据。
- 悲观锁：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻止，直到这个锁被释放。

数据库的乐观锁需要自己实现，在表里面添加一个 version 字段，每次修改成功值加 1，这样每次修改的时候先对比一下，自己拥有的 version 和数据库现在的 version 是否一致，如果不一致就不修改，这样就实现了乐观锁。



### 10. Redis 有哪些功能？

- 数据缓存功能
- 分布式锁的功能
- 支持数据持久化
- 支持事务
- 支持消息队列



### 11. 什么是缓存穿透？怎么解决？

缓存穿透：指查询一个一定不存在的数据，由于缓存是不命中时需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，造成缓存穿透。

解决方案：最简单粗暴的方法如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），我们就把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。



### 12. Redis 支持的数据类型有哪些？

Redis 支持的数据类型：string（字符串）、list（列表）、hash（字典）、set（集合）、zset（有序集合）。

#### 12.1这个string 底层是怎么样的？

是动态的，相对于c语言的string可以常数时间获取长度、能自动扩容并且惰性删除和二进制安全。

#### 12.2那redis 的hash扩容过程知道么？

字典有两个表，平时用一个，当扩容的时候在每次添加删除的时候顺带移一个槽去新的表，直到扩容完毕。在查找的时候先去老的找，找不到去新的找。



### 13. 怎么保证缓存和数据库数据的一致性？

- 合理设置缓存的过期时间。
- 新增、更改、删除数据库操作时同步更新 Redis，可以使用事物机制来保证数据的一致性。



### 14.redis集群是如何访问的？

客户端是先往集群中的一个节点发，如果命中直接返回，如果不命中，会返回moved指令，然后告知key所在的是哪个节点。



### 15.简述mysql主从复制原理?

(1) master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events）；
(2) slave将master的binary log events拷贝到它的中继日志(relay log)；
(3) slave重做中继日志中的事件，将改变反映它自己的数据。



### 16.数据库优化的几点：

1. 建立和优化使用索引

2. 减少子查询和联表查询

3. 主从分离

4. 用临时表代替大表插入



### 17. redis是什么？

Redis是一个开源的基于内存的，key-value数据结构的缓存数据库，支持数据持久化，m-s复制，常用数据类型有string set hash list,
最佳应用场景：适用于数据变化快且数据库大小可遇见（适合内存容量）的应用程序。
例如：股票价格、数据分析、实时数据搜集、实时通讯。
Redis只能使用单线程，性能受限于CPU性能，故单实例CPU最高才可能达到5-6wQPS每秒（取决于数据结构，数据大小以及服务器硬件性能，日常环境中QPS高峰大约在1-2w左右）



### 18.存储引擎

1.**MyISAM**只支持表级锁。**InnoDB**支持表级锁和行级锁。

2.MyISAM强调性能，不提供事务。InnoDB具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。

3.InnoDB支持外键。

4.InnoDB 支持MVCC。应对高并发事务, MVCC比单纯的加锁更高效;MVCC只在 READ COMMITTED 和 REPEATABLE READ 两个隔离级别下工作;MVCC可以使用 乐观(optimistic)锁 和 悲观(pessimistic)锁来实现;各数据库中MVCC实现并不统一。

补：**MEMORY存储引擎**

MEMORY存储引擎将表中的数据存储到内存中，为查询和引用其他表数据提供快速访问。



### 19.索引

- **MyISAM:** B+Tree叶节点的data域存放的是数据记录的地址。这被称为**“非聚簇索引”**。
- **InnoDB:** 其数据文件本身就是索引文件。相比MyISAM，索引文件和数据文件是分离的，其表数据文件本身就是按B+Tree组织的一个索引结构，树的叶节点data域保存了完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引。这被称为“**聚簇索引**（或聚集索引）”。而其余的索引都作为辅助索引，辅助索引的data域存储相应记录主键的值而不是地址，这也是和MyISAM不同的地方。**在根据主索引搜索时，直接找到key所在的节点即可取出数据；在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引。** **因此，在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。** PS：整理自《Java工程师修炼之道》



#### 18.1.索引类型

MySQL 的索引有两种分类方式：逻辑分类和物理分类。

 按照逻辑分类，索引可分为：

- **主键索引**：一张表只能有一个主键索引，**不允许重复**、**不允许为 NULL**；
- **唯一索引**：数据列**不允许重复**，允许为 NULL 值，一张表可有多个唯一索引，但是一个唯一索引只能包含一列，比如身份证号码、卡号等都可以作为唯一索引；
- **普通索引**：一张表可以创建多个普通索引，一个普通索引可以包含多个字段，允许数据重复，允许 NULL 值插入；
- **组合索引：**最左前缀法则
- **全文索引**：让搜索关键词更高效的一种索引。

按照物理分类，索引可分为：

- **聚集索引**：一般是表中的主键索引，如果表中没有显示指定主键，则会选择表中的第一个不允许为 NULL 的唯一索引，如果还是没有的话，就采用 Innodb 存储引擎为每行数据内置的 6 字节 ROWID 作为聚集索引。每张表只有一个聚集索引，因为聚集索引的键值的逻辑顺序决定了表中相应行的物理顺序。聚集索引在精确查找和范围查找方面有良好的性能表现（相比于普通索引和全表扫描），聚集索引就显得弥足珍贵，聚集索引选择还是要慎重的（一般不会让没有语义的自增 id 充当聚集索引）；
- **非聚集索引**：该索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同（非主键的那一列），一个表中可以拥有多个非聚集索引。



#### 18.2.MySQL索引的设计原则

1.选择唯一性索引

2.为经常需要排序、分组和联合操作的字段建立索引

3.为经常需要查询的字段建立索引

4.限制索引的数目

5.尽量使用数据量少的索引

6.数据量小的表最好不要使用索引

7.尽量使用前缀来索引

8.删除不再使用或者很少使用的索引



#### 18.3.什么情况下索引会失效？

①如果条件中有or，即使其中有条件带索引也不会使用(**这也是为什么尽量少用or的原因**)

②like查询是以%开头

③如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引。

④如果MySQL估计全表扫描比索引快，则不使用索引（比如非常小的表）.

⑤最佳左前缀法则，不能缺少索引项里面左边的那个字段。



### 20.事务

事务是逻辑上的一组操作，要么都执行，要么都不执行。

1. **原子性（Atomicity）：** 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；

2. **一致性（Consistency）：** 执行事务后，数据库从一个正确的状态变化到另一个正确的状态；

3. **隔离性（Isolation）：** 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；

4. **持久性（Durability）：** 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也可以依靠日志完成数据持久化。

   日志包括回滚日志（undo）和重做日志（redo），当我们通过事务修改数据时，首先会将数据库变化的信息记录到重做日志中，然后再对数据库中的数据进行修改。这样即使发生崩溃，还可以通过重做日志进行数据恢复。回滚日志（原来的数据），重做日志（原来->现在)。

   

### 21.并发事务4大问题

- **脏读（Dirty read）:** **一个事务读了另一个事务还没有提交的数据**，那么这个事务读到的这个数据是“脏数据”。
- **丢失修改（Lost to modify）:** 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，**第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失**，因此称为丢失修改。 例如：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=A-1，事务2也修改A=A-1，最终结果A=19，事务1的修改被丢失。
- **不可重复读（Unrepeatableread）:** 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，**由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样**。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
- **幻读（Phantom read）:** 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现**多了一些原本不存在的记****录**，就好像发生了幻觉一样，所以称为幻读。

**不可重复读和幻读区别：**

**不可重复读的重点是修改**比如多次读取一条记录发现其中某些列的值被修改，**幻读的重点在于新增或者删除**比如多次读取一条记录发现记录增多或减少了。



### 22.四大隔离级别

- **READ-UNCOMMITTED(读取未提交)：** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**。
- **READ-COMMITTED(读取已提交)：** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**。
- **REPEATABLE-READ(可重复读)：** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生**。
- **SERIALIZABLE(可串行化)：** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。

| 隔离级别         | 脏读 | 不可重复读 | 幻影读 |
| ---------------- | ---- | ---------- | ------ |
| READ-UNCOMMITTED | √    | √          | √      |
| READ-COMMITTED   | ×    | √          | √      |
| REPEATABLE-READ  | ×    | ×          | √      |
| SERIALIZABLE     | ×    | ×          | ×      |

这里需要注意的是：与 SQL 标准不同的地方在于 InnoDB 存储引擎在 **REPEATABLE-READ（可重读）** 事务隔离级别下使用的是Next-Key Lock 锁算法，因此可以避免幻读的产生，这与其他数据库系统(如 SQL Server) 是不同的。所以说InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）** 已经可以完全保证事务的隔离性要求，即达到了 SQL标准的 **SERIALIZABLE(可串行化)** 隔离级别。因为隔离级别越低，事务请求的锁越少，所以大部分数据库系统的隔离级别都是 **READ-COMMITTED(读取提交内容)** ，但是你要知道的是InnoDB 存储引擎默认使用 **REPEAaTABLE-READ（可重读）** 并不会有任何性能损失。

InnoDB 存储引擎在 **分布式事务** 的情况下一般会用到 **SERIALIZABLE(可串行化)** 隔离级别。



### 23.锁机制与InnoDB锁算法

#### 23.1.MyISAM和InnoDB存储引擎使用的锁：

- MyISAM采用表级锁(table-level locking)。
- InnoDB支持行级锁(row-level locking)和表级锁,默认为行级锁

#### 23.2.表级锁和行级锁对比：（按照锁的粒度分类）

- **表级锁：** MySQL中锁定 **粒度最大** 的一种锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。其锁定粒度最大，触发锁冲突的概率最高，并发度最低，MyISAM和 InnoDB引擎都支持表级锁。
- **行级锁：** MySQL中锁定 **粒度最小** 的一种锁，只针对当前操作的行进行加锁。 行级锁能大大减少数据库操作的冲突。其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，会出现死锁。

**InnoDB存储引擎的锁的算法有三种：**

- Record lock：单个行记录上的锁
- Gap lock：间隙锁，锁定一个范围，不包括记录本身
- Next-key lock：record+gap 锁定一个范围，包含记录本身

**相关知识点：**

1. innodb对于行的查询使用next-key lock
2. Next-locking keying为了解决Phantom Problem幻读问题
3. 当查询的索引含有唯一属性时，将next-key lock降级为record key
4. Gap锁设计的目的是为了阻止多个事务将记录插入到同一范围内，而这会导致幻读问题的产生
5. 有两种方式显式关闭gap锁：（除了外键约束和唯一性检查外，其余情况仅使用record lock） A. 将事务隔离级别设置为RC B. 将参数innodb_locks_unsafe_for_binlog设置为1



#### 23.3.共享锁（s）和排他锁（X）（按照是否可写分类）

- **共享锁（s）**

  **共享锁（Share Locks，简记为S）又被称为读锁**，其他用户可以并发读取数据，但任何事务都不能获取数据上的排他锁，直到已释放所有共享锁。

  共享锁(S锁)又称为读锁，若事务T对数据对象A加上S锁，则事务T只能读A；其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这就保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。

- **排他锁（X）：**

  **排它锁（(Exclusive lock,简记为X锁)）又称为写锁**，若事务T对数据对象A加上X锁，则只允许T读取和修改A，其它任何事务都不能再对A加任何类型的锁，直到T释放A上的锁。它防止任何其它事务获取资源上的锁，直到在事务的末尾将资源上的原始锁释放为止。在更新操作(INSERT、UPDATE 或 DELETE)过程中始终应用排它锁。

**两者之间的区别：**

1. 共享锁（S锁）：如果事务T对数据A加上共享锁后，则其他事务只能对A再加共享锁，不 能加排他锁。获取共享锁的事务只能读数据，不能修改数据。

2. 排他锁（X锁）：如果事务T对数据A加上排他锁后，则其他事务不能再对A加任任何类型的封锁。获取排他锁的事务既能读数据，又能修改数据。

   

#### 23.4.另外两个表级锁：IS和IX

意向锁的作用就是当一个事务在需要获取资源锁定的时候，如果遇到自己需要的资源已经被排他锁占用的时候，该事务可以需要锁定行的表上面添加一个合适的意向锁。如果自己需要一个共享锁，那么就在表上面添加一个意向共享锁。而如果自己需要的是某行（或者某些行）上面添加一个排他锁的话，则先在表上面添加一个意向排他锁。**意向共享锁可以同时并存多个，但是意向排他锁同时只能有一个存在。**

**InnoDB另外的两个表级锁：**

- **意向共享锁（IS）：** 表示事务准备给数据行记入共享锁，事务在一个数据行加共享锁前必须先取得该表的IS锁。
- **意向排他锁（IX）：** 表示事务准备给数据行加入排他锁，事务在一个数据行加排他锁前必须先取得该表的IX锁。

**注意：**

1. **这里的意向锁是表级锁，表示的是一种意向，仅仅表示事务正在读或写某一行记录，在真正加行锁时才会判断是否冲突。意向锁是InnoDB自动加的，不需要用户干预。**
2. **IX，IS是表级锁，不会和行级的X，S锁发生冲突，只会和表级的X，S发生冲突。**

**InnoDB的锁机制兼容情况如下：**
![InnoDB的锁机制兼容情况](https://user-gold-cdn.xitu.io/2018/6/7/163da3105cb54186?w=672&h=159&f=png&s=8700)

**当一个事务请求的锁模式与当前的锁兼容，InnoDB就将请求的锁授予该事务；反之如果请求不兼容，则该事物就等待锁释放。**

#### 23.5.死锁和避免死锁

**①InnoDB的行级锁是基于索引实现的，如果查询语句为命中任何索引，那么InnoDB会使用表级锁.** 此外，InnoDB的行级锁是针对索引加的锁，不针对数据记录，因此即使访问不同行的记录，如果使用了相同的索引键仍然会出现锁冲突。

②使用锁的时候，**如果表没有定义任何索引，那么InnoDB会创建一个隐藏的聚簇索引**并使用这个索引来加记录锁。

③此外，**不同于MyISAM总是一次性获得所需的全部锁，InnoDB的锁是逐步获得的，当两个事务都需要获得对方持有的锁，导致双方都在等待，这就产生了死锁。** 发生死锁后，InnoDB一般都可以检测到，并使一个事务释放锁回退，另一个则可以获取锁完成事务，我们可以采取以上方式避免死锁：

- **通过表级锁来减少死锁产生的概率；**
- **多个程序尽量约定以相同的顺序访问表（这也是解决并发理论中哲学家就餐问题的一种思路）；**
- **同一个事务尽可能做到一次锁定所需要的所有资源。**





### 24.大表优化问题

## 9.1.限定数据的范围

#### 9.2.读/写分离

经典的数据库拆分方案，主库负责写，从库负责读；

#### 9.3.垂直分区

- **垂直拆分的优点：** 可以使得列数据变小，在查询时减少读取的Block数，减少I/O次数。此外，垂直分区可以简化表的结构，易于维护。
- **垂直拆分的缺点：** 主键会出现冗余，需要管理冗余列，并会引起Join操作，可以通过在应用层进行Join来解决。此外，垂直分区会让事务变得更加复杂；

#### 9.4.水平分区



### 10.分库分表id主键处理

因为要是分成多个表之后，每个表都是从 1 开始累加，这样是不对的，我们需要一个全局唯一的 id 来支持。

生成全局 id 有下面这几种方式：

- **UUID**：不适合作为主键，因为太长了，并且无序不可读，查询效率低。比较适合用于生成唯一的名字的标示比如文件的名字。
- **数据库自增 id** : 两台数据库分别设置不同步长，生成不重复ID的策略来实现高可用。这种方式生成的 id 有序，但是需要独立部署数据库实例，成本高，还会有性能瓶颈。
- **利用 redis 生成 id :** 性能比较好，灵活方便，不依赖于数据库。但是，引入了新的组件造成系统更加复杂，可用性降低，编码更加复杂，增加了系统成本。
- **Twitter的snowflake算法** ：64位分为别标示时间和机器，但是如果机器上时间回滚会出问题。
- **美团的[Leaf](https://tech.meituan.com/2017/04/21/mt-leaf.html)分布式ID生成系统** ：Leaf 是美团开源的分布式ID生成器，能保证全局唯一性、趋势递增、单调递增、信息安全，里面也提到了几种分布式方案的对比，但也需要依赖关系数据库、Zookeeper等中间件。感觉还不错。美团技术团队的一篇文章：https://tech.meituan.com/2017/04/21/mt-leaf.html 。





### 11.重启 MySQL 数据库id问题

InnoDB 表只会把自增主键的最大 id 记录在内存中，所以重启之后会导致最大 id 丢失。





------

# 框架面试题

## 1.javaweb

### 1 . Servlet的运行过程？

Web容器加载Servlet并将其实例化后，Servlet生命周期开始，

容器运行其init()方法进行Servlet的初始化；

请求到达时调用Servlet的service()方法

service()方法会根据需要调用与请求对应的doGet或doPost等方法；

当服务器关闭或项目被卸载时服务器会将Servlet实例销毁，此时会调用Servlet的destroy()方法。



### 2. **讲解JSP中的四种作用域。** 

- page代表与一个页面相关的对象和属性。 
- request代表与Web客户机发出的一个请求相关的对象和属性。一个请求可能跨越多个页面，涉及多个Web组件；需要在页面显示的临时数据可以置于此作用域。 
- session代表与某个用户与服务器建立的一次会话相关的对象和属性。跟某个用户相关的数据应该放在用户自己的session中。 
- application代表与整个Web应用程序相关的对象和属性，它实质上是跨越整个Web应用程序，包括多个页面、请求和会话的一个全局作用域。

### 3.Servlet生命周期

1.创建对象->初始化->service()->doXXX()->销毁

2.对象创建只有一次，单例

初始化一次

销毁一次

3.关于线程安全

不安全的三个因素

①多线程的环境（有多个客户端，同时访问Servlet）

②多个线程共享资源，比如一个单例对象（Servlet是单例的）

③这个单例对象是有状态的（比如Servlet方法中采用全局变量，并且以该变量的运算结果作为下一步操作依据）



### 4.Cookie和Session的区别

1.存储位置不同

​	Session：服务端

​	Cookie：客户端

2.存储的数据格式不同

​		Session：value为对象，Object类型

​		Cookie：value为字符串，如果我们存储一个对象，对象转J son

3.存储数据的大小

​		Session：受服务器内存控制

​		Cookie：一般来说，最大为4K

4.生命周期不同：

​		Session：服务器端控制，默认30分钟。**关闭浏览器不会消失**。

​		Cookie：客户端控制，其实是客户端的一个文件，分2种情况：

​	①默认会话级Cookie，随浏览器关闭而消失

​	②比如设置7天免登陆，SetMaxAge

5.Cookie和Session之间的联系

http协议是无状态的协议，为了记住用户的状态，服务器会通过会话级的cookie来保存session标识。

JessionId保存session的id。



### 5.get 和 post 请求有哪些区别？

- get 请求会被浏览器主动缓存，而 post 不会。
- get 传递参数有大小限制，而 post 没有。
- post 参数传输更安全，get 的参数会明文限制在 url 上，post 不会。



### 6.转发和重定向的区别

1.转发

​		发生在服务器内部的跳转，对客户端来说就一次请求，request对象可以传递。

2.重定向

​		多次请求之间传递数据，需要session对象。



## 1.Spring

### 1.说说你对 Spring 的理解，非单例注入的原理？它的生命周期？循环注入的原理， aop 的实现原理，说说 aop 中的几个术语，它们是怎么相互工作的。

AOP与IOC的概念（即spring的核心）

a) IOC：

b) AOP：

核心组件：bean，context，core，单例注入是通过单例beanFactory进行创建，生命周期是在创建的时候通过接口实现开启，循环注入是通过后置处理器，aop其实就是通过反射进行动态代理，pointcut，advice等。



### 2.Spring框架中都用到了哪些设计模式？

1. 代理模式：在AOP和remoting中被用的比较多。
2. 单例模式：在spring配置文件中定义的bean默认为单例模式。
3. 模板方法模式：用来解决代码重复的问题。
4. 前端控制器模式：Spring提供了DispatcherServlet来对请求进行分发。
5. 依赖注入模式：贯穿于BeanFactory / ApplicationContext接口的核心理念。
6. 工厂模式：BeanFactory用来创建对象的实例。



### 3.你用过哪些重要的 Spring 注解？

@Controller - 用于 Spring MVC 项目中的控制器类。
@Service - 用于服务类。
@RequestMapping - 用于在控制器处理程序方法中配置 URI 映射。
@ResponseBody - 用于发送 Object 作为响应，通常用于发送 XML 或 JSON 数据作为响应。
@PathVariable - 用于将动态值从 URI 映射到处理程序方法参数。
@Autowired - 用于在 spring bean 中自动装配依赖项。
@Qualifier - 使用 @Autowired 注解，以避免在存在多个 bean 类型实例时出现混淆。
@Scope - 用于配置 spring bean 的范围。
@Configuration，@ComponentScan 和 @Bean - 用于基于 java 的配置。
@Aspect，@Before，@After，@Around，@Pointcut - 用于切面编程（AOP）。



### 4.@Component, @Controller, @Repository, @Service 有何区别？

- @Component：这将 java 类标记为 bean。它是任何 Spring 管理组件的通用构造型。spring
  的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中。
- @Controller：这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IoC 容器中。
- @Service：此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用
- @Service 而不是 @Component，因为它以更好的方式指定了意图。
- @Repository：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。



### 5.有哪些不同类型的依赖注入实现方式？

依赖注入是时下最流行的IoC实现方式，依赖注入分为

1. 接口注入
2. Setter方法注入
3. 构造器注入

其中接口注入由于在灵活性和易用性比较差，现在从Spring4开始已被废弃。

构造器依赖注入：构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参数代表一个对其他类的依赖。

Setter方法注入：Setter方法注入是容器通过调用无参构造器或无参static工厂 方法实例化bean之后，调用该bean的setter方法，即实现了基于setter的依赖注入。

#### 5.1构造方法注入

spring 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans.xsd
	    http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
        
	<bean id="bowl" class="constxiong.interview.inject.Bowl" />
	
	<bean id="person" class="constxiong.interview.inject.Person">
		<property name="bowl" ref="bowl"></property>
	</bean>
	
</beans>
```

```java
package constxiong.interview.inject;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class InjectTest {

	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext("spring_inject.xml");
		Person person = (Person)context.getBean("person");
		person.eat();
	}
}
```

**b) <bean> 节点 factory-method 参数指定静态工厂方法**

工厂类，静态工厂方法

```java
package constxiong.interview.inject;
 
public class BowlFactory {
 
	public static final Bowl getBowl() {
		return new Bowl();
	}
	
}
```

spring 配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans.xsd
	    http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
        
	<bean id="bowl" class="constxiong.interview.inject.BowlFactory" factory-method="getBowl"/>
	
	<bean id="person" class="constxiong.interview.inject.Person">
		<constructor-arg name="bowl" ref="bowl"></constructor-arg>
	</bean>
	
</beans>
```



**c) 非静态工厂方法，需要指定工厂 bean 和工厂方法**

工厂类，非静态工厂方法

```java
package constxiong.interview.inject;
 
public class BowlFactory {
 
	public Bowl getBowl() {
		return new Bowl();
	}
	
}
```

配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans.xsd
	    http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    
    <bean id="bowlFactory" class="constxiong.interview.inject.BowlFactory"></bean>   
	<bean id="bowl" factory-bean="bowlFactory" factory-method="getBowl"/>
	
	<bean id="person" class="constxiong.interview.inject.Person">
		<constructor-arg name="bowl" ref="bowl"></constructor-arg>
	</bean>
	
</beans>
```



#### 5.2 注解方式注入 

**bean 的申明、注册**

@Component //注册所有bean
@Controller //注册控制层的bean
@Service //注册服务层的bean
@Repository //注册dao层的bean

**bean 的注入**

@Autowired 作用于 构造方法、字段、方法，常用于成员变量字段之上。
@Autowired + @Qualifier 注入，指定 bean 的名称
@Resource JDK 自带注解注入，可以指定 bean 的名称和类型等

测试代码

**e) spring 配置文件，设置注解扫描目录**

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans.xsd
	    http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
        
	<context:component-scan base-package="constxiong.interview" />
	
</beans>
```

 

class Bowl

```java
package constxiong.interview.inject;
 
import org.springframework.stereotype.Component;
//import org.springframework.stereotype.Controller;
//import org.springframework.stereotype.Repository;
//import org.springframework.stereotype.Service;
 
@Component //注册所有bean
//@Controller //注册控制层的bean
//@Service //注册服务层的bean
//@Repository //注册dao层的bean
public class Bowl {
 
	public void putRice() {
		System.out.println("盛饭...");
	}
 
}
```



class Person

```java
package constxiong.interview.inject;
 
//import javax.annotation.Resource;
//
import org.springframework.beans.factory.annotation.Autowired;
//import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;
 
@Component //注册所有bean
//@Controller //注册控制层的bean
//@Service //注册服务层的bean
//@Repository //注册dao层的bean
public class Person {
 
	@Autowired
//	@Qualifier("bowl")
//	@Resource(name="bowl")
	private Bowl bowl;
 
	public void eat() {
		bowl.putRice();
		System.out.println("开始吃饭...");
	}
	
}
```





### 6.Spring的优点有哪些

- 非侵入式：基于Spring开发的应用中的对象可以不依赖于Spring的API
- 控制反转：IOC——Inversion of Control，指的是将对象的创建权交给 Spring 去创建。使用 Spring 之前，对象的创建都是由我们自己在代码中new创建。而使用 Spring 之后。对象的创建都是给了 Spring 框架。
- 依赖注入：DI——Dependency Injection，是指依赖的对象不需要手动调用 setXX 方法去设置，而是通过配置赋值。
- 面向切面编程：Aspect Oriented Programming——AOP
- 容器：Spring 是一个容器，因为它包含并且管理应用对象的生命周期
- 组件化：Spring 实现了使用简单的组件配置组合成一个复杂的应用。在 Spring 中可以使用XML和Java注解组合这些对象。
- 一站式：在 IOC 和 AOP 的基础上可以整合各种企业应用的开源框架和优秀的第三方类库（实际上 Spring 自身也提供了表述层的 SpringMVC 和持久层的 Spring JDBC）



### 7 什么是AOP

- 面向切面编程，供了模块化。通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP（面向对象编程）的延续，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

- AOP采取横向抽取机制，取代了传统纵向继承体系重复性代码

- 经典应用：事务管理、性能监视、安全检查、缓存 、日志等

- Spring AOP使用纯Java实现，不需要专门的编译过程和类加载器，在运行期通过代理方式向目标类织入增强代码

- AspectJ是一个基于Java语言的AOP框架，Spring2.0开始，Spring AOP引入对Aspect的支持，AspectJ扩展了Java语言，提供了一个专门的编译器，在编译时提供横向代码的织入

  

基于Java的主要AOP实现有：

1. aop底层将采用代理机制进行实现。

2. 接口 + 实现类 ：spring采用 jdk 的动态代理Proxy。

3. 实现类：spring 采用 cglib字节码增强。

   



### 10. Spring AOP 代理是什么？

(1) SpringAOP是动态代理来实现的。有两种代理方式：JDK动态代理与CGLIB动态代理
(2) JDK动态代理：是通过反射来接收被代理类，要求必须实现一个接口
(3) CGLIB动态代理：当被代理类没有实现一个接口的时候，就会使用CGLIB进行动态代理。CGLIB动态代理通过运行时动态生成被代理类的子类，运用继承的方式来实现动态代理。如果被代理类被final修饰了，那么就不能使用CGLIB进行动态代理了

代理是使用非常广泛的设计模式。简单来说，**代理是一个看其他像另一个对象的对象，但它添加了一些特殊的功能**。
Spring AOP是基于代理实现的。**AOP 代理是一个由 AOP 框架创建的用于在运行时实现切面协议的对象**。
Spring AOP默认为 AOP 代理使用标准的 JDK 动态代理。

这使得任何接口（或者接口的集合）可以被代理。

Spring AOP 也可以使用 CGLIB 代理。这对代理类而不是接口是必须的。
如果业务对象没有实现任何接口那么默认使用CGLIB



### 11.AOP实现方式

#### 4.1手动方式

###### 4.1.1JDK动态代理

- JDK动态代理 对“装饰者”设计模式 简化。使用前提：必须有接口

1.目标类：接口 + 实现类

```java
public interface UserService {
    public void addUser();
    public void updateUser();
    public void deleteUser();
}12345
```

2.切面类：用于存通知 MyAspect

```java
public class MyAspect { 
    public void before(){
        System.out.println("鸡首");
    }   
    public void after(){
        System.out.println("牛后");
    }
}12345678
```

3.工厂类：编写工厂生成代理

```java
public class MyBeanFactory {

    public static UserService createService(){
        //1 目标类
        final UserService userService = new UserServiceImpl();
        //2切面类
        final MyAspect myAspect = new MyAspect();
        /* 3 代理类：将目标类（切入点）和 切面类（通知） 结合 --> 切面
         *  Proxy.newProxyInstance
         *      参数1：loader ，类加载器，动态代理类 运行时创建，任何类都需要类加载器将其加载到内存。
         *          一般情况：当前类.class.getClassLoader();
         *                  目标类实例.getClass().get...
         *      参数2：Class[] interfaces 代理类需要实现的所有接口
         *          方式1：目标类实例.getClass().getInterfaces()  ;注意：只能获得自己接口，不能获得父元素接口
         *          方式2：new Class[]{UserService.class}   
         *          例如：jdbc 驱动  --> DriverManager  获得接口 Connection
         *      参数3：InvocationHandler  处理类，接口，必须进行实现类，一般采用匿名内部
         *          提供 invoke 方法，代理类的每一个方法执行时，都将调用一次invoke
         *              参数31：Object proxy ：代理对象
         *              参数32：Method method : 代理对象当前执行的方法的描述对象（反射）
         *                  执行方法名：method.getName()
         *                  执行方法：method.invoke(对象，实际参数)
         *              参数33：Object[] args :方法实际参数
         * 
         */
        UserService proxService = (UserService)Proxy.newProxyInstance(
                                MyBeanFactory.class.getClassLoader(), 
                                userService.getClass().getInterfaces(), 
                                new InvocationHandler() {

                                    @Override
                                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

                                        //前执行
                                        myAspect.before();

                                        //执行目标类的方法
                                        Object obj = method.invoke(userService, args);

                                        //后执行
                                        myAspect.after();

                                        return obj;
                                    }
                                });

        return proxService;
    }

}
```

4.测试

```java
	@Test
    public void demo01(){
        UserService userService = MyBeanFactory.createService();
        userService.addUser();
        userService.updateUser();
        userService.deleteUser();
    }
```

###### 4.1.2 CGLIB字节码增强

- 没有接口，只有实现类。
- 采用字节码增强框架 cglib，在运行时 创建目标类的子类，从而对目标类进行增强。

工厂类

```java
public class MyBeanFactory {

    public static UserServiceImpl createService(){
        //1 目标类
        final UserServiceImpl userService = new UserServiceImpl();
        //2切面类
        final MyAspect myAspect = new MyAspect();
        // 3.代理类 ，采用cglib，底层创建目标类的子类
        //3.1 核心类
        Enhancer enhancer = new Enhancer();
        //3.2 确定父类
        enhancer.setSuperclass(userService.getClass());
        /* 3.3 设置回调函数 , MethodInterceptor接口 等效 jdk InvocationHandler接口
         *  intercept() 等效 jdk  invoke()
         *      参数1、参数2、参数3：以invoke一样
         *      参数4：methodProxy 方法的代理
         *      
         * 
         */
        enhancer.setCallback(new MethodInterceptor(){

            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {

                //前
                myAspect.before();

                //执行目标类的方法
                Object obj = method.invoke(userService, args);
                // * 执行代理类的父类 ，执行目标类 （目标类和代理类 父子关系）
                methodProxy.invokeSuper(proxy, args);

                //后
                myAspect.after();

                return obj;
            }
        });
        //3.4 创建代理
        UserServiceImpl proxService = (UserServiceImpl) enhancer.create();

        return proxService;
    }

}
```



### 12.spring 容器的启动加载流程

​	首先解析 spring.xml 配置文件，把其中 `<bean>` 解析为 BeanDefinition, 存入beanFactory

​	执行beanFactory的后处理器

​	接下来 由 beanFactory 创建每个类对应的单例对象, 利用了反射根据类名创建每个类的实例对象（构造，初始化方法）

​	执行 bean 的后处理器, 它其中有两个方法，会在 bean 的初始化方法前后被调用
​	会把这些结果 存入 beanFactory 的 singletonObjects 这样一个map集合里

​	执行依赖注入
​	主要为了解决循环引用问题





### 13.为什么要使用 spring？

- spring 提供 ioc 技术，容器会帮你管理依赖的对象，从而不需要自己创建和管理依赖对象了，更轻松的实现了程序的解耦。
- spring 提供了事务支持，使得事务操作变的更加方便。
- spring 提供了面向切片编程，这样可以更方便的处理某一类的问题。
- 更方便的框架集成，spring 可以很方便的集成其他框架，比如 MyBatis、hibernate 等。



### 14.Spring AOP / AspectJ AOP 的区别？

Spring AOP属于运行时增强，而AspectJ是编译时增强。

Spring AOP基于代理（Proxying），而AspectJ基于字节码操作（Bytecode Manipulation）。

AspectJ相比于Spring AOP功能更加强大，但是Spring AOP相对来说更简单。如果切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择AspectJ，它比SpringAOP快很多。



### 16. 解释一下什么是 ioc？

ioc：Inversionof Control（中文：控制反转）是 spring 的核心，对于 spring 框架来说，就是由 spring 来负责控制对象的生命周期和对象间的关系。

简单来说，控制指的是当前对象对内部成员的控制权；控制反转指的是，这种控制权不由当前对象管理了，由其他（类,第三方容器）来管理。

依赖注入是时下最流行的IoC实现方式，依赖注入分为

1. 接口注入
2. Setter方法注入
3. 构造器注入



### 17. Spring中有哪些不同的通知类型

通知(advice)是你在你的程序中想要应用在其他模块中的横切关注点的实现。Advice主要有以下5种类型：

1. **前置通知(Before Advice)**: 在连接点之前执行的Advice，不过除非它抛出异常，否则没有能力中断执行流。使用 `@Before` 注解使用这个Advice。
2. **返回之后通知(After Retuning Advice)**: 在连接点正常结束之后执行的Advice。例如，如果一个方法没有抛出异常正常返回。通过 `@AfterReturning` 关注使用它。
3. **抛出（异常）后执行通知(After Throwing Advice)**: 如果一个方法通过抛出异常来退出的话，这个Advice就会被执行。通用 `@AfterThrowing` 注解来使用。
4. **后置通知(After Advice)**: 无论连接点是通过什么方式退出的(正常返回或者抛出异常)都会执行在结束后执行这些Advice。通过 `@After` 注解使用。
5. **围绕通知(Around Advice)**: 围绕连接点执行的Advice，就你一个方法调用。这是最强大的Advice。通过 `@Around` 注解使用。



### 18.spring封装bean的方式有哪些？

1. no：默认值 表示没有自动装配，应使用显示bean引用进行装配

2. byName：根据bean的名称进行注入

3. byType：根据类型进行注入

4. 构造函数：通过构造函数来注入依赖项 需要设置大量参数

5. autodetect：容器首先通过使用autowire装配，如果不能使用byType进行自动装配

   

### 19. spring 支持几种 bean 的作用域？

spring 支持 5 种作用域，如下：

- singleton：spring ioc 容器中只存在一个 bean 实例，bean 以单例模式存在，是系统默认值；
- prototype：每次从容器调用 bean 时都会创建一个新的示例，既每次 getBean()相当于执行 new Bean()操作；

- request：每次 http 请求都会创建一个 bean；
- session：同一个 http session 共享一个 bean 实例；
- global-session：用于 portlet 容器，因为每个 portlet 有单独的 session，globalsession 提供一个全局性的 http session。



### 20. spring 中的 bean 是线程安全的吗？

spring 中的 bean 默认是单例模式，spring 框架并没有对单例 bean 进行多线程的封装处理。

实际上大部分时候 spring bean 无状态的（比如 dao 类），所有某种程度上来说 bean 也是安全的，但如果 bean 有状态的话（比如 view model 对象），那就要开发者自己去保证线程安全了，最简单的就是改变 bean 的作用域，把“singleton”变更为“prototype”，这样请求 bean 相当于 new Bean()了，所以就可以保证线程安全了。

- 有状态就是有数据存储功能。
- 无状态就是不会保存数据。



### 21.spring 事务实现方式有哪些？

- 声明式事务：声明式事务也有两种实现方式，
- 基于 xml 配置文件的方式和注解方式（在类上添加 @Transaction 注解）。
- 编码方式：提供编码的形式管理和维护事务。



### 22.说一下 spring 的事务隔离

spring 有五大隔离级别，默认值为 ISOLATION_DEFAULT（使用数据库的设置），其他四个隔离级别和数据库的隔离级别一致：

ISOLATION_DEFAULT：用底层数据库的设置隔离级别，数据库设置的是什么我就用什么；

ISOLATION*READ*UNCOMMITTED：未提交读，最低隔离级别、事务未提交前，就可被其他事务读取（会出现幻读、脏读、不可重复读）；

ISOLATION*READ*COMMITTED：提交读，一个事务提交后才能被其他事务读取到（会造成幻读、不可重复读），SQL server 的默认级别；

ISOLATION*REPEATABLE*READ：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（会造成幻读），MySQL 的默认级别；

ISOLATION_SERIALIZABLE：序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。

**脏读** ：表示一个事务能够读取另一个事务中还未提交的数据。比如，某个事务尝试插入记录 A，此时该事务还未提交，然后另一个事务尝试读取到了记录 A。

**不可重复读** ：是指在一个事务内，多次读同一数据。

**幻读** ：指同一个事务内多次查询返回的结果集不一样。比如同一个事务 A 第一次查询时候有 n 条记录，但是第二次同等条件下查询却有 n+1 条记录，这就好像产生了幻觉。发生幻读的原因也是另外一个事务新增或者删除或者修改了第一个事务结果集里面的数据，同一个记录的数据内容被修改了，所有数据行的记录就变多或者变少了。



## 2.springMvc

### 1.说一下 spring mvc 运行流程？

- spring mvc 先将请求发送给 DispatcherServlet。
- DispatcherServlet 查询一个或多个 HandlerMapping，找到处理请求的 Controller。
- DispatcherServlet 再把请求提交到对应的 Controller。
- Controller 进行业务逻辑处理后，会返回一个ModelAndView。
- Dispathcher 查询一个或多个 ViewResolver 视图解析器，找到 ModelAndView 对象指定的视图对象。
- 视图对象负责渲染返回给客户端。



### 2. spring mvc 有哪些组件？

- 前置控制器 DispatcherServlet。
- 映射控制器 HandlerMapping。
- 处理器 Controller。
- 模型和视图 ModelAndView。
- 视图解析器 ViewResolver。



### 3.SpringMvc 怎么和 AJAX 相互调用的？

通过 Jackson 框架就可以把 Java 里面的对象直接转化成 Js 可以识别的 Json 对象具体步骤如下
1）加入 Jackson.jar
2）在配置文件中配置 json 的映射
3）在接受 Ajax 方法里面可以直接返回 Object,List 等,但方法前面要加上@ResponseB注解





## 3.Mybatis

### 1. MyBatis 中 #{}和 ${}的区别是什么？

`\#{}`是预编译处理，`${}`是字符替换。 在使用 `#{}`时，MyBatis 会将 SQL 中的 `#{}`替换成“?”，配合 PreparedStatement 的 set 方法赋值，这样可以有效的防止 SQL 注入，保证程序的运行安全。



### 2. MyBatis 有几种分页方式？

分页方式：逻辑分页和物理分页。

**逻辑分页：** 使用 MyBatis 自带的 RowBounds 进行分页，它是一次性查询很多数据，然后在数据中再进行检索。

**物理分页：** 自己手写 SQL 分页或使用分页插件 PageHelper，去数据库查询指定条数的分页数据的形式。



### 3.MyBatis 逻辑分页和物理分页的区别是什么？

- 逻辑分页是一次性查询很多数据，然后再在结果中检索分页的数据。这样做弊端是需要消耗大量的内存、有内存溢出的风险、对数据库压力较大。
- 物理分页是从数据库查询指定条数的数据，弥补了一次性全部查出的所有数据的种种缺点，比如需要大量的内存，对数据库查询压力较大等问题。



### 4. MyBatis 是否支持延迟加载？延迟加载的原理是什么？

MyBatis 支持延迟加载，设置 lazyLoadingEnabled=true 即可。

延迟加载的原理的是调用的时候触发加载，而不是在初始化的时候就加载信息。比如调用 a. getB(). getName()，这个时候发现 a. getB() 的值为 null，此时会单独触发事先保存好的关联 B 对象的 SQL，先查询出来 B，然后再调用 a. setB(b)，而这时候再调用 a. getB(). getName() 就有值了，这就是延迟加载的基本原理。



### 5.说一下 MyBatis 的一级缓存和二级缓存？

- 一级缓存：基于 PerpetualCache 的 HashMap 本地缓存，它的声明周期是和 SQLSession 一致的，有多个 SQLSession 或者分布式的环境中数据库操作，可能会出现脏数据。当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认一级缓存是开启的。
- 二级缓存：也是基于 PerpetualCache 的 HashMap 本地缓存，不同在于其存储作用域为 Mapper 级别的，如果多个SQLSession之间需要共享缓存，则需要使用到二级缓存，并且二级缓存可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现 Serializable 序列化接口(可用来保存对象的状态)。

开启二级缓存数据查询流程：二级缓存 -> 一级缓存 -> 数据库。

缓存更新机制：当某一个作用域(一级缓存 Session/二级缓存 Mapper)进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。



## 4.springboot

### 1.为什么要用 spring boot？

- 配置简单
- 独立运行
- 自动装配
- 无代码生成和 xml 配置
- 提供应用监控
- 易上手
- 提升开发效率



### 2. spring boot 核心配置文件是什么？

spring boot 核心的两个配置文件：

- bootstrap (. yml 或者 . properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，且 boostrap 里面的属性不能被覆盖；
- application (. yml 或者 . properties)：用于 spring boot 项目的自动化配置。



### 3. spring boot 有哪些方式可以实现热部署？

- 使用 devtools 启动热部署，添加 devtools 库，在配置文件中把 spring. devtools. restart. enabled 设置为 true；
- 使用 Intellij Idea 编辑器，勾上自动编译或手动重新编译。



### 4.springboot的工作机制：

SpringApplication的run方法的实现是我们本次旅程的主要线路，该方法的主要流程大体可以归纳如下：

1） 如果我们使用的是SpringApplication的静态run方法，那么，这个方法里面首先要创建一个SpringApplication对象实例，然后调用这个创建好的SpringApplication的实例方法。在SpringApplication实例初始化的时候，它会提前做几件事情：

- 根据classpath里面是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext）来决定是否应该创建一个为Web应用使用的ApplicationContext类型。
- 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationContextInitializer。
- 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationListener。
- 推断并设置main方法的定义类。

2） SpringApplication实例初始化完成并且完成设置后，就开始执行run方法的逻辑了，方法执行伊始，首先遍历执行所有通过SpringFactoriesLoader可以查找到并加载的SpringApplicationRunListener。调用它们的started()方法，告诉这些SpringApplicationRunListener，“嘿，SpringBoot应用要开始执行咯！”。

3） 创建并配置当前Spring Boot应用将要使用的Environment（包括配置要使用的PropertySource以及Profile）。

4） 遍历调用所有SpringApplicationRunListener的environmentPrepared()的方法，告诉他们：“当前SpringBoot应用使用的Environment准备好了咯！”。

5） 如果SpringApplication的showBanner属性被设置为true，则打印banner。

6） 根据用户是否明确设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为当前SpringBoot应用创建什么类型的ApplicationContext并创建完成，然后根据条件决定是否添加ShutdownHook，决定是否使用自定义的BeanNameGenerator，决定是否使用自定义的ResourceLoader，当然，最重要的，将之前准备好的Environment设置给创建好的ApplicationContext使用。

7） ApplicationContext创建好之后，SpringApplication会再次借助Spring-FactoriesLoader，查找并加载classpath中所有可用的ApplicationContext-Initializer，然后遍历调用这些ApplicationContextInitializer的initialize（applicationContext）方法来对已经创建好的ApplicationContext进行进一步的处理。

8） 遍历调用所有SpringApplicationRunListener的contextPrepared()方法。

9） 最核心的一步，将之前通过@EnableAutoConfiguration获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的ApplicationContext。

10） 遍历调用所有SpringApplicationRunListener的contextLoaded()方法。

11） 调用ApplicationContext的refresh()方法，完成IoC容器可用的最后一道工序。

12） 查找当前ApplicationContext中是否注册有CommandLineRunner，如果有，则遍历执行它们。

13） 正常情况下，遍历执行SpringApplicationRunListener的finished()方法、（如果整个过程出现异常，则依然调用所有SpringApplicationRunListener的finished()方法，只不过这种情况下会将异常信息一并传入处理）



------

# 计算机网络面试题

### 1.http协议和tcp协议的区别

**TCP协议对应于传输层，而HTTP协议对应于应用层，从本质上来说，二者没有可比性。Http协议是建立在TCP协议基础之上的，当浏览器需要从服务器获取网页数据的时候，会发出一次Http请求。**

1.支持客户/服务器模式。
2.简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快。
3.灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。
4.无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
5.无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

### 2.socket：

 Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

前人已经给我们做了好多的事了，网络间的通信也就简单了许多，但毕竟还是有挺多工作要做的。以前听到Socket编程，觉得它是比较高深的编程知识，但是只要弄清Socket编程的工作原理，神秘的面纱也就揭开了。
    一个生活中的场景。你要打电话给一个朋友，先拨号，朋友听到电话铃声后提起电话，这时你和你的朋友就建立起了连接，就可以讲话了。等交流结束，挂断电话结束此次交谈。  生活中的场景就解释了这工作原理，也许TCP/IP协议族就是诞生于生活中，这也不一定。

#### 2.1WebSocket

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

在 WebSocket API 中，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。

现在，很多网站为了实现推送技术，所用的技术都是 Ajax 轮询。轮询是在特定的的时间间隔（如每1秒），由浏览器对服务器发出HTTP请求，然后由服务器返回最新的数据给客户端的浏览器。这种传统的模式带来很明显的缺点，即浏览器需要不断的向服务器发出请求，然而HTTP请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，显然这样会浪费很多的带宽等资源。

HTML5 定义的 WebSocket 协议，能更好的节省服务器资源和带宽，并且能够更实时地进行通讯。



### 3.简述 tcp 和 udp的区别？

tcp 和 udp 是 OSI 模型中的运输层中的协议。tcp 提供可靠的通信传输，而 udp 则常被用于让广播和细节控制交给应用的通信传输。

两者的区别大致如下：

- tcp 面向连接，udp 面向非连接即发送数据前不需要建立链接；

- tcp 提供可靠的服务（数据传输），udp 无法保证；

- tcp 面向字节流，udp 面向报文；

- tcp 数据传输慢，udp 数据传输快；




### 4.http和https有什么区别？

1. https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用。

2. http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。

3. http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。

4. http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。所以说，HTTP 安全性没有 HTTPS高，但是 HTTPS 比HTTP耗费更多服务器资源。

5. HTTP协议运行在TCP之上，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份。

   

### 5.HTTPS的工作原理

客户端在使用HTTPS方式与Web服务器通信时有以下几个步骤，如图所示。

（1）客户使用https的URL访问Web服务器，要求与Web服务器建立SSL连接。

（2）Web服务器收到客户端请求后，会将网站的证书信息（证书中包含公钥）传送一份给客户端。

（3）客户端的浏览器与Web服务器开始协商SSL连接的安全等级，也就是信息加密的等级。

（4）客户端的浏览器根据双方同意的安全等级，建立会话密钥，然后利用网站的公钥将会话密钥加密，并传送给网站。

（5）Web服务器利用自己的私钥解密出会话密钥。

（6）Web服务器利用会话密钥加密与客户端之间的通信。

![img](https://pic002.cnblogs.com/images/2012/339704/2012071410212142.gif)





### 6.OSI，TCP/IP，五层协议的体系结构，以及各层协议

OSI分层 （7层）：物理层、数据链路层、网络层、传输层、会话层、表示层、应用层。
TCP/IP分层（4层）：网络接口层、 网际层、运输层、 应用层。
五层协议 （5层）：物理层、数据链路层、网络层、运输层、 应用层。
每一层的协议如下：
物理层：RJ45、CLOCK、IEEE802.3 （中继器，集线器）
数据链路：PPP、FR、HDLC、VLAN、MAC （网桥，交换机）
网络层：IP、ICMP、ARP、RARP、OSPF、IPX、RIP、IGRP、 （路由器）
传输层：TCP、UDP、SPX
会话层：NFS、SQL、NETBIOS、RPC
表示层：JPEG、MPEG、ASII
应用层：FTP、DNS、Telnet、SMTP、HTTP、WWW、NFS
每一层的作用如下：
物理层：通过媒介传输比特,确定机械及电气规范（比特Bit）
数据链路层：将比特组装成帧和点到点的传递（帧Frame）
网络层：负责数据包从源到宿的传递和网际互连（包PackeT）
传输层：提供端到端的可靠报文传递和错误恢复（段Segment）
会话层：建立、管理和终止会话（会话协议数据单元SPDU）
表示层：对数据进行翻译、加密和压缩（表示协议数据单元PPDU）
应用层：允许访问OSI环境的手段（应用协议数据单元APDU）



### 7.五层协议

应用层（DNS、**HTTP**、SMTP）

运输层（**TCP**、UDP，复用和分用的功能）

网络层（选择合适的网间路由和交换结点， 确保数据及时传送。 把运输层产生的报文段或用户数据报封装成分组和包进行传送。**IP协议**）

数据链路层（将网络层交下来的IP数据报转成帧）

物理层（实现相邻计算机节点之间比特流的透明传送）



### 8.三次握手四次挥手

- 客户端–发送带有 SYN 标志的数据包–一次握手–服务端

- 服务端–发送带有 SYN/ACK 标志的数据包–二次握手–客户端（确认序号为收到的序号加1 ）

- 客户端–发送带有带有 ACK 标志的数据包–三次握手–服务端（确认序号为收到的序号加1 ）

  

- 客户端-发送一个 FIN，用来关闭客户端到服务器的数据传送

- 服务器-收到这个 FIN，它发回一 个 ACK，确认序号为收到的序号加1 。和 SYN 一样，一个 FIN 将占用一个序号

- 服务器-关闭与客户端的连接，发送一个FIN给客户端

- 客户端-发回 ACK 报文确认，并将确认序号设置为收到序号加1



#### 8.1为什么连接的时候是三次握手，关闭的时候却是四次握手？

答：因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。



### 9.TCP 协议如何保证可靠传输

1. 应用数据被分割成 TCP 认为最适合发送的数据块。

2. TCP 给发送的每一个包进行编号，接收方对数据包进行排序，把有序数据传送给应用层。

3. 校验和： TCP 将保持它首部和数据的检验和。这是一个端到端的检验和，目的是检测数据在传输过程中的任何变化。如果收到段的检验和有差错，TCP 将丢弃这个报文段和不确认收到此报文段。

4. TCP 的接收端会丢弃重复的数据。

5. 流量控制： TCP 连接的每一方都有固定大小的缓冲空间，TCP的接收端只允许发送端发送接收端缓冲区能接纳的数据。当接收方来不及处理发送方的数据，能提示发送方降低发送的速率，防止包丢失。TCP 使用的流量控制协议是可变大小的滑动窗口协议。 （TCP 利用滑动窗口实现流量控制）

6. 拥塞控制： 当网络拥塞时，减少数据的发送。

7. ARQ协议： 也是为了实现可靠传输的，它的基本原理就是每发完一个分组就停止发送，等待对方确认。在收到确认后再发下一个分组。

8. 超时重传： 当 TCP 发出一个段后，它启动一个定时器，等待目的端确认收到这个报文段。如果不能及时收到一个确认，将重发这个报文段。

   

### 10.输入URL发生的事情

1.URL地址解析

2.DNS域名解析

3.客户端与服务端建立TCP  连接（三次握手）

4.把客户端信息（携带cookies）传递给服务端（发送HTTP请求）

5.服务端得到并处理请求（HTTP响应内容）

6.客户端渲染服务器返回的内容

7.和服务端断开TCP连接（四次挥手）



### 11.Dns 如何工作的

简单的说就是通过udp去查ip

主要就是先去本地找，本地没记录去本地运营商那里找，一般都有，如果没的话，就去根服务器找，然后再找到顶级，再找权威这样迭代下来，最终找到返回。解析过的都会在本地缓存着。



# 操作系统面试题：

我们经常会听到程序的用户态和内核态，一个程序从用户态进入了内核态。。。

## 1.什么是用户态和内核态

内核态和用户态到底指的是什么呢？我们这就解开其神秘面纱

所谓的用户态、内核态，实际上是处理器（cpu）的一种状态，在 cpu 状态字里面用 1bit 表示

### 1.1什么是用户态

也叫普通态，cpu 访问资源有限

### 1.2用户态的几个特点

1. cpu 访问资源有限
2. 程序可靠性、安全性`要求低`
3. 程序编写维护比较简单

### 1.3什么是内核态

也叫特权态，cpu 可以访问计算机的任何资源

### 1.4内核态的特点？

1. cpu 可以访问任何资源
2. 程序可靠性、安全性`要求高`
3. 编写维护成本比较高



------

## 2.为什么需要有用户态和内核态

那么，经过上面的解释，应该都了解了什么是用户态和内核态。

BUT！有没有想过，操作系统为什么要搞出用户态和内核态？

如果都处于一个态下，有什么问题吗？

想象一下，如果一个国家的所有人都能获得国家的机密资料、控制国家资源，那这个国家也就离崩溃不远了。

操作系统也是如此，所以我们要限制不用的程序访问资源的权限。

------

## 3.操作系统是如何控制不同态的权限的

要控制权限，必须要对程序发出的每一条指令进行检查。而这种检查被称为 `地址翻译`，这里不详细展开。内核态程序通过`绕过`地址翻译执行特权指令，从而访问所有资源。



------

## 4.程序应该运行在用户态还是内核态？

- 用户态

  - 能运行在用户态就运行在用户态
  - 涉及用户数据和应用的操作

- 内核态

  - 牵扯到计算机本体的操作
  - 对时序要求比较高的操作

  

------

## 5.用户态如何切换到内核态？

用户态程序 `陷入` 到内核态有 3 种方法：

1. 系统调用
2. 外围设备中断
3. 异常

具体处理过程：
通过 `软中断` 切换



# 网络安全面试题

### 1.什么是SQL注入攻击

攻击者在HTTP请求中注入恶意的SQL代码，服务器使用参数构建数据库SQL命令时，恶意SQL被一起构造，并在数据库中执行。
用户登录，输入用户名 lianggzone，密码 ‘ or ‘1’=’1 ，如果此时使用参数构造的方式，就会出现
select * from user where name = ‘lianggzone’ and password = ‘’ or ‘1’=‘1’

不管用户名和密码是什么内容，使查询出来的用户列表不为空。

如何防范SQL注入攻击使用预编译的PrepareStatement是必须的，但是一般我们会从两个方面同时入手。
Web端
1）有效性检验。
2）限制字符串输入的长度。
服务端
1）不用拼接SQL字符串。
2）使用预编译的PrepareStatement。
3）有效性检验。(为什么服务端还要做有效性检验？第一准则，外部都是不可信的，防止攻击者绕过Web端请求)
4）过滤SQL需要的参数中的特殊字符。比如单引号、双引号。

​		prepareStatement对象防止sql注入的方式是**把用户非法输入的单引号用\反斜杠做了转义**，从而达到了防止sql注入的目的

![img](https://img-blog.csdnimg.cn/20190305210744118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N6aDUwMA==,size_16,color_FFFFFF,t_70)



### 2.什么是XSS攻击

答; 跨站点脚本攻击，指攻击者通过篡改网页，嵌入恶意脚本程序，在用户浏览网页时，控制用户浏览器进行恶意操作的一种攻击方式。如何防范XSS攻击
1）前端，服务端，同时需要字符串输入的长度限制。
2）前端，服务端，同时需要对HTML转义处理。将其中的”<”,”>”等特殊字符进行转义编码。
防 XSS 的核心是必须对输入的数据做过滤处理。



### 3.什么是CSRF攻击

答; 跨站点请求伪造，指攻击者通过跨站请求，以合法的用户的身份进行非法操作。可以这么理解CSRF攻击：攻击者盗用你的身份，以你的名义向第三方网站发送恶意请求。CRSF能做的事情包括利用你的身份发邮件，发短信，进行交易转账，甚至盗取账号信息。

如何防范CSRF攻击:

安全框架，例如Spring Security。
token机制。在HTTP请求中进行token验证，如果请求中没有token或者token内容不正确，则认为CSRF攻击而拒绝该请求。
验证码。通常情况下，验证码能够很好的遏制CSRF攻击，但是很多情况下，出于用户体验考虑，验证码只能作为一种辅助手段，而不是最主要的解决方案。
referer识别。在HTTP Header中有一个字段Referer，它记录了HTTP请求的来源地址。如果Referer是其他网站，就有可能是CSRF攻击，则拒绝该请求。但是，服务器并非都能取到Referer。很多用户出于隐私保护的考虑，限制了Referer的发送。在某些情况下，浏览器也不会发送Referer，例如HTTPS跳转到HTTP。
1）验证请求来源地址；
2）关键操作添加验证码；
3）在请求地址添加 token 并验证。



### 4.DDos 攻击

客户端向服务端发送请求链接数据包，服务端向客户端发送确认数据包，客户端不向服务端发送确认数据包，服务器一直等待来自客户端的确认
没有彻底根治的办法，除非不使用TCP
DDos 预防：
1）限制同时打开SYN半链接的数目
2）缩短SYN半链接的Time out 时间
3）关闭不必要的服务

![img](https://img2020.cnblogs.com/blog/1849006/202005/1849006-20200520150334091-152505701.png)



### 5.**SYN攻击**

 		在三次握手过程中，服务器发送SYN-ACK之后，收到客户端的ACK之前的TCP连接称为半连接(half-open connect).此时服务器处于Syn_RECV状态.当收到ACK后，服务器转入ESTABLISHED状态.
Syn攻击就是 攻击客户端 在短时间内伪造大量不存在的IP地址，向服务器不断地发送syn包，服务器回复确认包，并等待客户的确认，由于源地址是不存在的，服务器需要不断的重发直 至超时，这些伪造的SYN包将长时间占用未连接队列，正常的SYN请求被丢弃，目标系统运行缓慢，严重者引起网络堵塞甚至系统瘫痪。
Syn攻击是一个典型的DDOS攻击。检测SYN攻击非常的方便，当你在服务器上看到大量的半连接状态时，特别是源IP地址是随机的，基本上可以断定这是一次SYN攻击.在Linux下可以如下命令检测是否被Syn攻击
netstat -n -p TCP | grep SYN_RECV
		一般较新的TCP/IP协议栈都对这一过程进行修正来防范Syn攻击，修改tcp协议实现。主要方法有SynAttackProtect保护机制、SYN cookies技术、增加最大半连接和缩短超时时间等.
但是不能完全防范syn攻击。



# 数据结构与算法面试题

## 树：

### 1.特点

1. 结构直观
2. 一棵树满足某种性质往往要求每个结点都满足



### 2.常考形状

1. 普通二叉树
2. 平衡二叉树
3. 完全二叉树
4. 二叉搜索树
5. 四叉树
6. 多叉树
7. 红黑树、自平衡二叉搜索树（意向职位需要使用时会考）



### 3.遍历操作及其应用场景

- 前序遍历（根左右）：树里搜索、创建一棵新树
- 中序遍历（左根右）：二叉搜索树
- 后序遍历（左右中）：对某节点进行分析时需要用到左子树和右子树的信息，即搜索信息时候从树的底部开始，就像修剪一棵树时，从叶子向根修剪





### 3.1满二叉树

　　一棵二叉树的结点要么是叶子结点，要么它有两个子结点（如果一个二叉树的层数为K，且结点总数是(2^k) -1，则它就是满二叉树。）



### 3.2完全二叉树

　　若设二叉树的深度为k，除第 k 层外，其它各层 (1～k-1) 的结点数都达到最大个数，第k 层所有的结点都**连续集中在最左边**，这就是完全二叉树。



### 3.3 平衡二叉树

　　它或者是一颗空树，或它的左子树和右子树的深度之差(平衡因子)的绝对值不超过1，且它的左子树和右子树都是一颗平衡二叉树。



### 3.4最优二叉树（哈夫曼树）

　　树的带权路径长度达到最小，称这样的二叉树为最优二叉树，也称为哈夫曼树(Huffman Tree)。哈夫曼树是带权路径长度最短的树，权值较大的结点离根较近。



### 3.5红黑树

红黑树(Red-Black Tree，简称R-B Tree)，它一种特殊的二叉查找树。

红黑树是特殊的二叉查找树，意味着它满足二叉查找树的特征：任意一个节点所包含的键值，大于等于左孩子的键值，小于等于右孩子的键值。

红黑树的特性:
(1) 每个节点或者是黑色，或者是红色。
(2) 根节点是黑色。
(3) 每个叶子节点是黑色。 [注意：这里叶子节点，是指为空的叶子节点！
(4) 如果一个节点是红色的，则它的子节点必须是黑色的。
(5) 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。



**注意**：

(01) 特性(3)中的叶子节点，是只为空(NIL或null)的节点。
(02) 特性(5)，确保没有一条路径会比其他路径长出俩倍。因而，红黑树是相对是接近平衡的二叉树。

![img](https://images0.cnblogs.com/i/497634/201403/251730074203156.jpg)

#### **1.6红黑树的Java实现(代码说明)**

红黑树的基本操作是**添加**、**删除**和**旋转**。在对红黑树进行添加或删除后，会用到旋转方法。为什么呢？道理很简单，添加或删除红黑树中的节点之后，红黑树就发生了变化，可能不满足红黑树的5条性质，也就不再是一颗红黑树了，而是一颗普通的树。而通过旋转，可以使这颗树重新成为红黑树。简单点说，旋转的目的是让树保持红黑树的特性。
旋转包括两种：**左旋** 和 **右旋**。下面分别对红黑树的基本操作进行介绍。

```java
public class RBTree<T extends Comparable<T>> {

    private RBTNode<T> mRoot;    // 根结点

    private static final boolean RED   = false;
    private static final boolean BLACK = true;

    public class RBTNode<T extends Comparable<T>> {
        boolean color;        // 颜色
        T key;                // 关键字(键值)
        RBTNode<T> left;    // 左孩子
        RBTNode<T> right;    // 右孩子
        RBTNode<T> parent;    // 父结点

        public RBTNode(T key, boolean color, RBTNode<T> parent, RBTNode<T> left, RBTNode<T> right) {
            this.key = key;
            this.color = color;
            this.parent = parent;
            this.left = left;
            this.right = right;
        }

    }

    ...
}
```

RBTree是红黑树对应的类，RBTNode是红黑树的节点类。在RBTree中包含了根节点mRoot和红黑树的相关API。
注意：在实现红黑树API的过程中，我重载了许多函数。重载的原因，一是因为有的API是内部接口，有的是外部接口；二是为了让结构更加清晰。



## 算法：

### 1.二分查找。

#### 非递归实现：

```java
public static int biSearch(int []array,int a){
    int lo=0;
    int hi=array.length-1;
    int mid;
    while(lo<=hi){
        mid=(lo+hi)/2;
        if(array[mid]==a){
            return mid+1;
        }else if(array[mid]<a){
            lo=mid+1;
        }else{
            hi=mid-1;
        }
    }
    return -1;
}
```

#### 递归实现：

```java
public static int sort(int []array,int a,int lo,int hi){
        if(lo<=hi){
            int mid=(lo+hi)/2;
            if(a==array[mid]){
                return mid+1;
            }
            else if(a>array[mid]){
                return sort(array,a,mid+1,hi);
            }else{
                return sort(array,a,lo,mid-1);
            }
        }
        return -1;
    }
```



### 排序方式

| 排序方法 | 时间复杂度（平均） | 时间复杂度（最坏) | 时间复杂度（最好) | 空间复杂度 | 稳定性 | 复杂性 |
| -------- | ------------------ | ----------------- | ----------------- | ---------- | ------ | ------ |
| 插入排序 | O(n2)              | O(n2)             | O(n)              | O(1)       | 稳定   | 简单   |
| 希尔排序 | O(nlog2n)          | O(n2)             | O(n)              | O(1)       | 不稳定 | 较复杂 |
| 选择排序 | O(n2)              | O(n2)             | O(n2)             | O(1)       | 不稳定 | 简单   |
| 堆排序   | O(nlog2n)          | O(nlog2n)         | O(nlog2n)         | O(1)       | 不稳定 | 较复杂 |
| 冒泡排序 | O(n2)              | O(n2)             | O(n)              | O(1)       | 稳定   | 简单   |
| 快速排序 | O(nlog2n)          | O(n2)             | O(nlog2n)         | O(nlog2n)  | 不稳定 | 较复杂 |
| 归并排序 | O(nlog2n)          | O(nlog2n)         | O(nlog2n)         | O(n)       | 稳定   | 较复杂 |
| 基数排序 | O(d(n+r))          | O(d(n+r))         | O(d(n+r))         | O(n+r)     | 稳定   | 较复杂 |



#### **冒泡排序**

```java
public class Bubbling {
    public static void BubblingSort(int[] arr){
        if (arr == null || arr.length < 2){
            return;
        }
        for (int end = arr.length - 1;end > 0;end--){
            for (int i = 0;i < end;i++){
                if (arr[i] > arr[i+1]){
                    swap(arr, i ,i +1);
                }
            }
        }
    }
    public static void swap(int[] arr,int i,int j){
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```



#### **选择排序：**

```java
public class Choose {
    public static  void select(int[] arr){
        if (arr == null || arr.length < 2){
            return;
        }
        for (int i = 0;i < arr.length - 1;i++){
            int minIndex = i;
            for (int j = i + 1;j < arr.length;j++){
                minIndex = arr[j] < arr[minIndex] ? j : minIndex;
            }
            swap(arr,i,minIndex);
        }
    }
    public static void swap(int[] arr,int i,int j){
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```

#### **插入排序：**

```java
private static int[] insertSort(int[] arr) {
	int temp;
    if (arr == null || arr.length < 2){
            return;
    }
    for (int i=1;i<arr.length;i++){
        //待排元素小于有序序列的最后一个元素时，向前插入
        if (arr[i]<arr[i-1]){
            temp = arr[i];
            for (int j=i;j>=0;j--){
                if (j>0 && arr[j-1]>temp) {
                    arr[j]=arr[j-1];
                }else {
                    arr[j]=temp;
                    break;
                }
            }
        }
    }
    return arr;
}
```

#### **快速排序：**

```java
private static int[] quickSort(int[] arr, int low, int high) {
    	if (arr == null || arr.length < 2){
            return;
        }
		if (low < high) {
			int middle = getMiddle(arr, low, high);
			//对左子序列进行排序
			quickSort(arr, low, middle - 1);
			//对右子序列进行排序
			quickSort(arr, middle + 1, high);
		}
		return arr;
}
private static int getMiddle(int []arr,int low,int high){
    int temp=arr[low];
		while (low<high) {
			while (low<high &&arr[high]>=temp) {
					high--;
			}
			arr[low]=arr[high];
			while(low<high && arr[low]<=temp){
				low++;
			}
			arr[high]=arr[low];
		}
		arr[low]=temp;
		return low;
}
```



#### **堆排序：**

```java
 public static void heapSort(int[] arr) {
        if (arr == null || arr.length < 2){
            return;
        }
        //构造大根堆
        heapInsert(arr);
        int size = arr.length;
        while (size > 1) {
            //固定最大值
            swap(arr, 0, size - 1);
            size--;
            //构造大根堆
            heapify(arr, 0, size);
        }
    }
    public static void heapInsert(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            //当前索引
            int currentIndex = i;
            //父结点索引
            int fatherIndex = (currentIndex - 1) / 2;
            //如果当前插入的值大于其父结点的值,则交换值，并且将索引指向父结点
            //然后继续和上面的父结点值比较，直到不大于父结点，则退出循环
            while (arr[currentIndex] > arr[fatherIndex]) {
                swap(arr, currentIndex, fatherIndex);
                currentIndex = fatherIndex;
                fatherIndex = (currentIndex - 1) / 2;
            }
        }
    }
//剩余的数构造成大根堆
public static void heapify(int[] arr, int index, int size) {
    int left = 2 * index + 1;
    int right = 2 * index + 2;
    while (left < size) {
        int largestIndex;
        if (arr[left] < arr[right] && right < size) {
            largestIndex = right;
        } else {
            largestIndex = left;
        }
        //比较父结点的值与孩子中较大的值，并确定最大值的索引
        if (arr[index] > arr[largestIndex]) {
            largestIndex = index;
        }
        //为大根堆时
        if (index == largestIndex) {
            break;
        }
        swap(arr, largestIndex, index);
        index = largestIndex;
        left = 2 * index + 1;
        right = 2 * index + 2;
    }
}
//交换数组中两个元素的值
public static void swap(int[] arr, int i, int j) {
     int temp = arr[i];
     arr[i] = arr[j];
     arr[j] = temp;
}
```



## 链表：

```java
package com.zxh.Demo;

/*
 * @author ：xiaobaibai
 * @Data：2020/11/30 21:36
 * @Describe:
 *      反转链表
 */

import java.util.ArrayList;
import java.util.Stack;

/**
 * 从尾到头遍历链表 输入一个链表，按链表值从尾到头的顺序返回一个ArrayList
 *
 * @author Administrator
 */
class ListNode {// 单链表节点构建
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}

public class Demo06 {

    static ListNode head = null;// 创建一个头节点

    public static void main(String[] args) {
        addNode(5);
        addNode(8);
        ArrayList<Integer> list = printListFromTailToHead(head);
        System.out.println(list);
    }

    // 队列和栈是一对好基友，从尾到头打印链表，当然离不开借助栈的帮忙啦
    // 所以，先把链表里的东西，都放到一个栈里去，然后按顺序把栈里的东西pop出来，就这么简单
    public static ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        Stack<Integer> stack = new Stack<Integer>();
        while (listNode != null) {
            stack.push(listNode.val);
            listNode = listNode.next;
        }
        ArrayList<Integer> list = new ArrayList<Integer>();
        while (!stack.isEmpty()) {
            list.add(stack.pop());
        }
        return list;
    }
   
    // input
    public static void addNode(int d) {
        ListNode newNode = new ListNode(d);
        if (head == null) {
            head = newNode;
        }
        ListNode tmp = head;
        while (tmp.next != null) {
            tmp = tmp.next;
        }
        tmp.next = newNode;
    }

    // delete
    public boolean deleteNode(int index) {
        if (index < 1 || index > length()) {
            return false;// 如果当前index在链表中不存在
        }
        if (index == 1) {// 如果index指定的是头节点
            head = head.next;
            return true;
        }
        int i = 2;
        ListNode preNode = head;// 前一个节点（从头节点开始）
        ListNode curNode = preNode.next;// 当前节点
        while (curNode != null) {
            if (i == index) {
                preNode.next = curNode.next;// 删除当节点，前节点连接到下节点
                return true;
            }
            preNode = curNode;
            curNode = curNode.next;
            i++;
        }
        return false;
    }

    // 返回节点长度

    public int length() {
        int length = 0;
        ListNode tmp = head;
        while (tmp != null) {
            length++;
            tmp = tmp.next;
        }
        return length;
    }

    // 链表反转
    public ListNode ReverseIteratively(ListNode head) {
        ListNode pReversedHead = head;
        ListNode pNode = head;
        ListNode pPrev = null;
        while (pNode != null) {
            ListNode pNext = pNode.next;
            if (pNext == null) {
                pReversedHead = pNode;
            }
            pNode.next = pPrev;
            pPrev = pNode;
            pNode = pNext;
        }
        this.head = pReversedHead;
        return this.head;
    }

    // 查找单链表的中间节点
    public ListNode SearchMid(ListNode head) {
        ListNode p = this.head, q = this.head;
        while (p != null && p.next != null && p.next.next != null) {
            p = p.next.next;
            q = q.next;
        }
        System.out.println("Mid:" + q.val);
        return q;
    }

    // 查找倒数 第k个元素
    public ListNode findElem(ListNode head, int k) {
        if (k < 1 || k > this.length()) {
            return null;
        }
        ListNode p1 = head;
        ListNode p2 = head;
        for (int i = 0; i < k; i++)// 前移k步
            p1 = p1.next;
        while (p1 != null) {
            p1 = p1.next;
            p2 = p2.next;
        }
        return p2;
    }

    // 排序
    public ListNode orderList() {
        ListNode nextNode = null;
        int tmp = 0;
        ListNode curNode = head;
        while (curNode.next != null) {
            nextNode = curNode.next;
            while (nextNode != null) {
                if (curNode.val > nextNode.val) {
                    tmp = curNode.val;
                    curNode.val = nextNode.val;
                    nextNode.val = tmp;
                }
                nextNode = nextNode.next;
            }
            curNode = curNode.next;
        }
        return head;
    }

    // 从尾到头输出单链表，采用递归方式实现

    public void printListReversely(ListNode pListHead) {
        if (pListHead != null) {
            printListReversely(pListHead.next);
            System.out.println("printListReversely:" + pListHead.val);
        }
    }

    // 判断链表是否有环，单向链表有环时，尾节点相同
    public boolean IsLoop(ListNode head) {
        ListNode fast = head, slow = head;
        if (fast == null) {
            return false;
        }
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
            if (fast == slow) {
                System.out.println("该链表有环");
                return true;
            }
        }
        return !(fast == null || fast.next == null);
    }

    // 找出链表环的入口

    public ListNode FindLoopPort(ListNode head) {
        ListNode fast = head, slow = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast)
                break;
        }
        if (fast == null || fast.next == null)
            return null;
        slow = head;
        while (slow != fast) {
            slow = slow.next;
            fast = fast.next;
        }
        return slow;
    }
}
```

## 算法应用题

题目 : 使用递归算法输出某个目录下及其子目录下所有文件. 

递归 : 自动调用自己 , 需要定义递归出口.

题目分析 : 参数为一个指定的目录 . 输出所有文件列表;

```java
public class TestPrintDirAndFiles {

    public static void main(String[] args) {
        print(new File("E:/"));
    }
    
    private static void print(File file) {
        System.out.println(file.getAbsolutePath());
        if (file.isDirectory()) {
            File[] files = file.listFiles();
            for (File f : files) {
                print(f);
            }
        }
    }
}
```



java实现杨辉三角：

```java
public class Demo04 {
    public static void main(String[] args) {
        int rows = 11;
        for (int i = 0; i < rows; i++) {
            int number = 1;
            // 打印空格字符串
            System.out.format("%" + (rows - i) * 2 + "s", "");
            for (int j = 0; j <= i; j++) {
                System.out.format("%4d", number);
                number = number * (i - j) / (j + 1);
            }
            System.out.println();
        }
    }
}
```

#### 在排序数组中查找元素的第一个和最后一个位置

```java
public class Solution {

    public int[] searchRange(int[] nums, int target) {
        if (nums.length == 0) {
            return new int[]{-1, -1};
        }
        int firstPosition = findFirstPosition(nums, target);
        int lastPosition = findLastPosition(nums, target);
        return new int[]{firstPosition, lastPosition};
    }


    private int findFirstPosition(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                // ① 不可以直接返回，应该继续向左边找，即 [left, mid - 1] 区间里找
                right = mid - 1;
            } else if (nums[mid] < target) {
                // 应该继续向右边找，即 [mid + 1, right] 区间里找
                left = mid + 1;
            } else {
                // 此时 nums[mid] > target，应该继续向左边找，即 [left, mid - 1] 区间里找
                right = mid - 1;
            }
        }

        // 此时 left 和 right 的位置关系是 [right, left]，注意上面的 ①，此时 left 才是第 1 次元素出现的位置
        // 因此还需要特别做一次判断
        if (left != nums.length && nums[left] == target) {
            return left;
        }
        return -1;
    }


    private int findLastPosition(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) {
                // 只有这里不一样：不可以直接返回，应该继续向右边找，即 [mid + 1, right] 区间里找
                left = mid + 1;
            } else if (nums[mid] < target) {
                // 应该继续向右边找，即 [mid + 1, right] 区间里找
                left = mid + 1;
            } else {
                // 此时 nums[mid] > target，应该继续向左边找，即 [left, mid - 1] 区间里找
                right = mid - 1;
            }
        }

        if (right != -1 && nums[right] == target) {
            return right;
        }
        return -1;
    }
}
```

#### 只出现一次的数字

```java
class Solution {
    public int singleNumber(int[] nums) {
        int single = 0;
        for (int num : nums) {
            single ^= num;
        }
        return single;	
    }
}
```



只有一个数字是重复的

```java
int FindOnceNum(int arr[],int len){
    if (arr==NULL||len<=0){
        throw std::exception("Invalid input.");
    }

    int ret=0;
    for(int i=0;i<len;i++){
        ret=ret^arr[i];
    }
    return ret;
}
```



## 数组和链表的区别：

数组吧，用的比较多的的就是他的那个下标了，更深入一点的话就是他的那些内存取键吧，数组的存储区间是连续的，而且他占用的内存也比较大，我感觉麻烦的就是需要提前定义好这个数组的大小，也就是大小固定，不扩容。空间利用率不高，空间复杂度小。主要特点就是像arraylist那样，查起来容易，增加删除困难

链表的话

就是非连续，非顺序的存储结构，它主要就是一些列节点组成，两部分：一个数存储数据元素的数据域，一个是存储下个节点地址的指针域。存储空间分散，占用内存比较宽松，空间复杂度小，时间复杂度大。底层如果比喻起来就像是一列火车吧，车厢，连接车厢的杆儿。他也是增删容易，但是查找相对与数组来说就比较困难了~。



# 前端面试题

### 1.比较一下Java和JavaSciprt。

基于对象和面向对象：Java是一种真正的面向对象的语言，即使是开发简单的程序，必须设计对象；

JavaScript是种脚本语言，它可以用来制作与网络无关的，与用户交互作用的复杂软件。它是一种基于对象（Object-Based）和事件驱动（Event-Driven）的编程语言，因而它本身提供了非常丰富的内部对象供设计人员使用。



- 解释和编译：Java的源代码在执行之前，必须经过编译。JavaScript是一种解释性编程语言，其源代码不需经过编译，由浏览器解释执行。（目前的浏览器几乎都使用了JIT（即时编译）技术来提升JavaScript的运行效率）
\- 强类型变量和类型弱变量：Java采用强类型变量检查，即所有变量在编译之前必须作声明；JavaScript中变量是弱类型的，甚至在使用变量前可以不作声明，JavaScript的解释器在运行时检查推断其数据类型。



### 2.什么是ajax请求？

```js
$.ajax({
     url:"/handle_Ajax/",
     type:"POST",
     data:{username:"Alex",password:123},
     success:function(data){
         console.log(data)
     },
```



### 3.简述jsonp及实现原理？

JSONP 是json用来跨域的一个东西。原理是通过script标签的跨域特性来绕过同源策略。

JsonP解决跨域只能发送get请求，并且实现起来需要前后端交互比较多。

JSONP的简单实现：创建一个回调函数，然后在远程服务上调用这个函数并且将JSON数据作为参数传递，完成回调。



### 4.列举Http请求中常见的请求方式？

![image-20200629084508965](C:\Users\33066\AppData\Roaming\Typora\typora-user-images\image-20200629084508965.png)



### 5.列举Http请求中的状态码？

1XX：指示信息–表示请求已接收，继续处理。

2XX Success(成功状态码)：成功–表示请求已被成功接收、理解、接受。

> 200 表示从客户端发来的请求在服务器端被正常处理
> 204 该状态码表示服务器接收的请求已成功处理，但在返回的响应报文中不含实体的主体部分
> 206 该状态码表示客户端进行了范围请求，而服务器成功执行了这部分的GET请求

3XX Redirection(重定向状态码)：重定向–要完成请求必须进行更进一步的操作。

> 301 永久性重定向
> 302 临时性重定向

4XX Client Error(客户端错误状态码)：客户端错误–请求有语法错误或请求无法实现。

> 400 该状态码表示请求报文中存在语法错误
> 401 该状态码表示发送的请求需要有通过HTTP认证的认证信息
> 403 该状态码表明对请求资源的访问被服务器拒绝了
> 404 该状态码表明服务器上无法找到请求的资源

5XX Server Error(服务器错误状态码)：服务器端错误–服务器未能实现合法的请求。

> 500 该状态码表明服务器端在执行请求时发生了错误。
> 503 该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。



### 6.vue优点？

答：轻量级框架：只关注视图层
简单易学：国人开发，中文文档，不存在语言障碍 ，易于理解和学习；
双向数据绑定：保留了angular的特点，在数据操作方面更为简单；
组件化：保留了react的优点，实现了html的封装和重用，在构建单页面应用方面有着独特的优势；
视图，数据，结构分离：使数据的更改更为简单，不需要进行逻辑代码的修改，只需要操作数据就能完成相关操作；



### 7.vue中的路由的拦截器的作用？

拦截器可以在请求发送前和发送请求后做一些处理。

vue路由拦截，针对要先登录才能进入的页面，判断是否有Token值，如果有则next()，否则跳转到登录页面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200522105602802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMzNjI4MQ==,size_16,color_FFFFFF,t_70)



### 8.列举vue的常见指令

1. 文本插值：{{ }} Mustache
2. DOM属性绑定： v-bind
3. 指令绑定一个事件监听器：v-on
4. 实现表单输入和应用状态之间的双向绑定：v-model
5. 控制切换一个元素的显示：v-if 和 v-else
6. 列表渲染：v-for
7. 根据条件展示元素：v-show



### 9. 什么是 vue 生命周期？有什么作用？

答：每个 Vue 实例在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等。同时在这个过程中也会运行一些叫做 生命周期钩子 的函数，这给了用户在不同阶段添加自己的代码的机会。（ps：生命周期钩子就是生命周期函数）例如，如果要通过某些插件操作DOM节点，如想在页面渲染完后弹出广告窗， 那我们最早可在mounted 中进行。

![image-20200628174125918](C:\Users\33066\AppData\Roaming\Typora\typora-user-images\image-20200628174125918.png)



# Linux面试题

## 1.Linux常用操作。

1：man rm———查看命令帮助

2：mkdir————-创建目录

3：touch————-创建文件

4：cd———————切换。

5：ls————查看目录

6：ls -lh——————查看目录详细

7：pwd—————-查看当前目录

8：vim—————-添加内容

9：echo———————追加内容

10：cat—————查看文件内容

11：mv————————-移动

12：cp——————-拷贝

13：mv——————重命名

15：find——-搜索

16：rm—————-删除数据

17：ping———————-查看能不能上网

19：tar cf ———————打压缩

20：tar xf——————-解压缩

21：chmod-------------修改文件权限



## 2.git常用命令

- 1:git init—————————初始化

- 2:git add .————————-从工作区，添加到版本库

- 3:git commit -m”xxx”————从暂存区，添加到分支

- 4:git status————————查看状态

- 5:git log —————————查看版本库的日志

- 6:git reflog————————查看所有日志

- 7:git reset —head 版本号—-切换

- 8:git stash————————-保存

- 9:git stash————————-将第一个记录从“某个地方”重新拿到工作区（可能有冲突）

- git stash list————————查看“某个地方”存储的所有记录

- git stash clear—————————-清空“某个地方”

- git stash pop——————————-将第一个记录从“某个地方”重新拿到工作区（可能有冲突）

- git stash apply ——————————编号,将指定编号记录从“某个地方”重新拿到工作区（可能有冲突）

- git stash drop ——————————编号 ，删除指定编号的记录

- 10:git branch dev—————创建分支

- 11:git branch -d dev———-删除分支

- 12:git checkout dev————切换分支

- 13:git merge dev—————-合并分支

- 14:git branch———————查看所有分支

- 15:git clone https:xxx——-克隆

- 16:git add origin https:xxx-起个别名

- 17:git push origin dev ——添加到dev分支

- 18:git pull origin master—拉代码

- 19:git fetch origin master-去仓库获取

- 20:git merge origin/master-和网上下的master分支合并

  

# 面试技巧

## 自我介绍

​		您好，我是来自天津科技大学的一名大三本科生-翟先浩，目前主要学习的是java。在大学期间对自己的学习路线还是比较清晰的。
​		大一开始学习编程语言，先学习了C语言，在大一寒假期间开始自学java，系统的学习了javase，javaee，数据库，前端html，css，js，以及前后端框架，并考取了一个国家工信部的java开发师证书

​		大二期间开始利用框架做项目，第一个项目是和朋友参加的创新创业项目，也就是简历上的那个微信小程序，和经管，机械，设计学院进行合作，这个项目也是在最近的天津市挑战杯中获得了银奖以及各类校级的比赛的一等奖。然后就是疫情期间，就是大二下学期，进行了一段时间的实习，是天津西青那边一家科技公司--霆客，当时也是java岗位，然后干了前后端个人独立开发，项目是为赛闻公司编写CRM管理系统。

​		目前的话，大三了嘛，主要就是担任一个校园工作室的后端负责人和一个社团的部长，主要还是技术方面吧。前一段时间也是有幸帮助学院设计学校双创中心网站以及工作室官网编写工作并担任负责人。现在也是在进一步提高自己的技术，把自己之前学过的知识内容做一个总结，憨实基础。希望可以在java这条路上可以做一个进一步的提高。



## 项目经验

​		项目经验的话，主要就是简历上的那三个吧。

​		先说第一个项目的难点吧，主要就是微信小程序嘛，对于微信授权和微信支付了解的不多，也是上他那个微信公众平台去查api，也没有学长带，就自己去查，哪里不懂去到处找老师问，也是通过这个微信小程序，了解到了一些现有api怎么发送请求然后获取相应信息。

​		后来是实习的时候，是根据已经建立好的架构，在此基础上进行改动，难点大概就是前端vue吧，当时还用了组件，之前没怎么用过，了解那个element组件用了挺长时间的，但是当组件搭好后，用起来就即美观又省事。

​		最近这个官网的话，遇到的问题主要就是和前端交互吧，由于我们都是分开来写的，彻底分离状态，所以对于数据的传输格式，包括服务器跨域，还有数据请求时的数据格式，请求头的设置之类的，都是在这次官网编写过程中遇到的。



## 数据库的使用

​			数据库的话，mysql用的比较多，在我看来的话，它既轻而且还免费，这是现在我个人觉得她作为主流框架的一个原因吧。

学习的话，我们学校目前开设的时sqlserver，这个数据库就感觉挺重量级的，第一次接触的时候，下载工具就用了十几个G，然后他那个版本用起来也让人不是特别舒服，算是个人喜好吧。

其他数据库的话，最开始我学习的时oralce，没有系统学习过mysql，当时学习oracle学了一个月，后来学mysql的时候发现CRUD时候的sql都一样。oracle当初学了，但是后来没怎么用过，也是在了解到他是付费的，如果一些特别大的项目的话，可以选择这个oracle数据库。

redis数据库的话，第一次接触就是做那个微信小程序吧，就是当时要实现一个功能，就是记录用户登录时间，并且在一段时间后过期。为了这个功能去学习的这们数据库，后来发现redis的使用，他可以做一些缓存处理，像一些什么订阅消息啦，购物订单啦， 都可以，他的底层是哪个key-value的结构嘛，一对一，对于一些数据的处理还是很友好的。



## 排序算法	

像冒泡，选择，插入比较简单，这里不做具体叙述了。

最近看的比较多的是

快排，他是对冒泡排序的一种改进吧，基本思想就是通过一趟排序奖要排序的数据分割成独立的两部分，其中以一部分比另外一部分的所有数据都要小，然后再按照此方法分别进行快速排序。这个过程也是用到了递归，递归一定次数之后，便称为了有序序列。

还有什么归并排序，桶排序，堆排序，希尔排序，基数排序，计数排序用的不是很多，然后得话还有一些什么不太出名的猴子排序和睡眠排序，挺好玩的，了解过一点。



## 集合的使用

​		集合的话，collection接口下的set和list，还有一个单独的接口是map	

​		set的话，这个集合本身是无序，不重复，他的里面又hashset，hashlistset。无序性（没下标），不能用for循环去遍历，可以用迭代器，.HashSet：增加删除时效率高，LinkedHashSet：查看检索时，效率比较高

​		list的话，他个那个set就正好相反，有序的，可重复。它里面有那个arraylist和linkedlist，这个arraylsit是数组的结构的，数组嘛，有下标，所以他的查找速度很快。然后是linkedlist，他是链表结构，这个arrlist和linklist我也是相对来记得，arraylist用来查找，linkedlist用来增加删除。哦，对，还有一句，这个arraylist和linkedlist都是线程不安全的

​		然后的话就是map吧，基于数组和链表进行存储数据，集合一一对应（），主要就是他的那个键值对的结构，hashmap了解的比较多，hashmap我用他的话，他是基于链表和数组的嘛，所以优秀之处还是挺多的。之前的话主要就是像项目中要返回一些json数据啦，会用到。前两天写一个基础题时也用到了hashmap，他主要是存储一个小车对象，程序是控制小车的移动，并记录小车的位置。运用面向对象的思想，由于不连数据库，所以如果是多个小车进行操作的话，那么hashmap的选择可以说是为我这个程序的扩容和扩展提供了极大的帮助，hashmap《interger，对象》，integer就是那个对象的id，运用此结构存储和读取对象，很方便。细致的话，也算是看过一点hashmap 的源码，他主要还是生成hashcode然后存到key中HashMap的基础就是一个线性数组，这个数组就是Entry[]，Map里面的内容都保存在Entry[]里面。

```java
//存储时:
int hash = key.hashCode();// 这个hashCode方法这里不详述,只要理解每个key的hash是一个固定的int值
int index = hash % Entry[].length;
Entry[index] = value;

//取值时:
int hash = key.hashCode();
int index = hash % Entry[].length;
return Entry[index];
```





synchronized的三种应用方式

（1）修饰实例方法
（2）修饰静态方法
（3）修饰代码块

这三种方法，synchronized用的比较多，也是写多线程的时候常用的，然后项目中并发处理，是用过一次并发处理，就是再实习的时候，他当时有一个功能是批量转移用户到另一个用户名下，然后用到了悲观锁。当时也是在组长的指导下吧，进行了注解的事务回滚以及对于方法的调用，然后判断此数据是否改变。后来也去查了悲观锁了乐观锁的使用，

乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在提交更新的时候会判断一下在此期间别人有没有去更新这个数据。

悲观锁：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻止，直到这个锁被释放。

数据库的乐观锁需要自己实现，在表里面添加一个 version 字段，每次修改成功值加 1，这样每次修改的时候先对比一下，自己拥有的 version 和数据库现在的 version 是否一致，如果不一致就不修改，这样就实现了乐观



# 模拟面试：

## 1.java常量和变量之间的区别

常量就是final修饰的量，在java规范中，变量名必须大写，代表常数

变量就是基本数据类型

全局变量可以不手动复制，java会自动初始话，局部变量必须赋值，否则报错



## 2.Integer与int的区别

integer是int的包装类，int是基本数据类型，

interger更多的用在对象的引用，int的初始值为0，integer的初始值是null。

举个简单的例子，如果要实现一个用户去参加某项比赛，如果这个人没有参加，那么这个对应的字段值应该是null，而不是int



## 3.== 和 equals 的区别是什么？

如果是基本数据类型的话，那么就是用==，equals会报错

如果是引用类型的话

那么== 比较的就是对象的地址，而equals比较的是地址里面的值

举个例子，String arr = “abc”，String arr01 = new String("abc");

如果是== 的话就false，如果是equals就相等，为什么呢，因为equals重写了String和Inetger方法

这种还要延申到另外一道题，就是hashcode，如果两个变量equals相等，那么hashcode一定相等，如果hash相等，那么这两个数不一定相等，原因是键值对不一定相等，他是通过hash算法实现的计算一个hash值，不同的内容有可能是得到相同的hash值,String底层是31实现的hashcode算法，为什么选择31呢，因为31是一个既不大也不小的质数，还可以被jvm的位运算优化



## 4.ﬁnal和abstract关键字的作用

final可以修饰方法，类，属性，修饰之后不能被改变

abstract可以修饰类和方法，修饰之后需要被继承和重写

final修饰栈内存中的引用不能别改变，但是修饰堆内存中的属性值，这个堆内存中的属性值依旧可以被改变



## 5.String 属于基础的数据类型吗？

不属于，String是属于引用对象类型的，这个比较简单，多说一点吧，在jdk1.8的时候String底层是使用char数组实现的，在1.9之后是使用byte数组实现的，String底层有一个final，也就是不可变的，如果想要经常修改字符串的值，那么就要引出另外两个东西，一个是Stringbuffer，一个是StringBuilder。

先说数组，这个数组的话，byte数组，那么我感觉这个String可以有一个长度，就是他底层返回的长度值是一个int类型的值，也就是20亿字节，而一个int所占的字节数是byte所占字节数的4倍，大于80亿字节的话，也就是8G的话，程序就会报错

然后刚才还是说了一个final，

一个是因为安全，不让他他被其他类改变

另外一个是共享常量池，他放在了字符串常量池中

然后是Stringbuffer和Stringbuilder，

String每次改变都会产生新数组，但是Stringbuilder和Stringbuilder不会

Stringbuffer是线程安全的，底层有syn，String和Stringbuilder是线程不安全的

Stringbuilder的效率比Stringbuffer要高

单线程的情况推荐Stringbuilder，多线程环境下推荐Stringbuffer



## 6.接口和抽象类有什么区别？

1.接口需要implements实现，抽象类是通过extends继承

2.抽象类可以有构造函数，接口不能有构造函数

3.抽象类有很多main方法，并且可以运行，但是接口不能有

4.类可以实现多个接口，但是只能继承一个抽象类

5.接口默认修饰符是public，抽象类可以是任意修饰符

6.抽象类不能直接实例化，普通类实现接口，其中方法都需要被实现

7.抽象类不能用final修饰，接口当中的常量，默认是public static final，就算不写的话也会默认是这样

8.实现接口的类如果是抽象类可以不实现接口里面的所有方法 



**设计**

1.抽象类：同一类事务的抽取，比如针对Dao层操作的封装，如BaseDao，BaseServiceImpl

2.接口：单体项目，分层开发，interface作为各层之间的纽带





## 7.索引是怎么实现的

1.myISAM引擎和InnoDB都使用B+Tree结构的索引。

2.myISAM的B+树叶结点存储的是数据地址，顾称为非聚集索引。

3.InnoDB的B+树叶节点直接存储的是数据，顾称为聚集索引。

4.InnoDB的辅助索引存储的是主键的值。



#### 7.1.索引类型

MySQL 的索引有两种分类方式：逻辑分类和物理分类。

 按照逻辑分类，索引可分为：

- **主键索引**：一张表只能有一个主键索引，**不允许重复**、**不允许为 NULL**；
- **唯一索引**：数据列**不允许重复**，允许为 NULL 值，一张表可有多个唯一索引，但是一个唯一索引只能包含一列，比如身份证号码、卡号等都可以作为唯一索引；
- **普通索引**：一张表可以创建多个普通索引，一个普通索引可以包含多个字段，允许数据重复，允许 NULL 值插入；
- **组合索引：**最左前缀法则
- **全文索引**：让搜索关键词更高效的一种索引。

按照物理分类，索引可分为：

- **聚集索引**：一般是表中的主键索引，如果表中没有显示指定主键，则会选择表中的第一个不允许为 NULL 的唯一索引，如果还是没有的话，就采用 Innodb 存储引擎为每行数据内置的 6 字节 ROWID 作为聚集索引。每张表只有一个聚集索引，因为聚集索引的键值的逻辑顺序决定了表中相应行的物理顺序。聚集索引在精确查找和范围查找方面有良好的性能表现（相比于普通索引和全表扫描），聚集索引就显得弥足珍贵，聚集索引选择还是要慎重的（一般不会让没有语义的自增 id 充当聚集索引）；

- **非聚集索引**：该索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同（非主键的那一列），一个表中可以拥有多个非聚集索引。

  

#### 7.2.MySQL索引的设计原则

   (1）对于经常查询的字段，建议创建索引。

（2）索引不是越多越好，一个表如果有大量索引，不仅占用磁盘空间，而且会影响INSERT，DELETE，UPDATE等语句的性能。

（3）避免对经常更新的表进行过多的索引，因为当表中数据更改的同时，索引也会进行调整和更新，十分消耗系统资源。

（4）数据量小的表建议不要创建索引，数据量小时索引不仅起不到明显的优化效果，对于索引结构的维护反而消耗系统资源。

（5）不要在区分度低的字段建立索引。比如性别字段，只有 “男” 和 “女” ，建索引完全起不到优化效果。

（6）当唯一性是某字段本身的特征时，指定唯一索引能提高查询速度。

（7）在频繁进行跑排列分组（即进行 group by 或 order by操作）的列上建立索引，如果待排序有多个，可以在这些列上建立组合索引。



#### 7.3什么情况下索引会失效？

①如果条件中有or，即使其中有条件带索引也不会使用(**这也是为什么尽量少用or的原因**)

②like查询是以%开头

③如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引。

⑤最佳左前缀法则，不能缺少索引项里面左边的那个字段。



## 8.关于集合

### 8.1 set：无序，不可重复

Set接口的实现类主要有：HashSet、TreeSet、LinkedHashSet等

hashset：哈希表，增加删除时效率高，此构造函数创建一个大小为16的容器，加载因子为0.75，扩容增量：原容量的 1 倍，为线程不安全的实现，如果有多个线程同时访问，修改，就需要在外部同步。来保证数据的幂等性，判断不等是使用的hash算法，感觉就是equals那套东西

（幂等是数据中得一个概念，表示N次变换和1次变换的结果相同。）

LinkedHashSet：双向链表，查看检索时，效率比较高，非线程安全的，数据的有序性



### 8.2 list： 有序，可重复

List接口的实现类主要有：ArrayList、LinkedList，Vector等

**ArrayList：**

数组结构，带下标，长度可变，查询速度快，增删速度较慢，线程不安全，put时扩容为10，增删时调用，加载因子是1，0.5倍+1，极消耗资源



**LinkedList：**

双向链表，线程不安全，查询速度较慢，增删速度快



**Vector：**

线程安全（底层是一个数组，方法加了synchronized，默认容量10，负载因子1，扩容1被）

缺点：

1.因为vector是线程安全的，所以效率低
2.Vector空间满了之后，扩容是一倍，而ArrayList仅仅是一半
3.Vector分配内存的时候需要连续的存储空间，如果数据太多，容易分配内存失败
4.只能在尾部进行插入和删除操作，效率低



### 8.3 map：

Map接口的实现类主要有：HashMap、TreeMap、Hashtable、ConcurrentHashMap等

**HashMap:**

1.7 会产生死循环、数据丢失、数据覆盖的问题，

1.8 中会有数据覆盖的问题

1. 数组+链表改成了数组+链表或红黑树；
2. 链表的插入方式从头插法改成了尾插法，简单说就是插入时，如果数组位置上已经有元素，1.7将新元素放到数组中，原始节点作为新节点的后继节点，1.8遍历链表，将元素放置到链表的最后；
3. 扩容的时候1.7需要对原数组中的元素进行重新hash定位在新数组的位置，1.8采用更简单的判断逻辑，位置不变或索引+旧容量大小；
4. 在插入时，1.7先判断是否需要扩容，再插入，1.8先进行插入，插入完成再判断是否需要扩容；



#### 11.8为什么要做这些优化呢

1. 防止发生hash冲突，链表长度过长，将时间复杂度由`O(n)`降为`O(logn)`;

2. 因为1.7头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环；

   A线程在插入节点B，B线程也在插入，遇到容量不够开始扩容，重新hash，放置元素，采用头插法，后遍历到的B节点放入了头部，这样形成了环，如下图所示：



默认情况下，数组大小为16，负载因子0.75

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWdrci5jbi1iai51ZmlsZW9zLmNvbS85ZDkyZGRkYS1lZmRiLTRmZjctYTlmYi00MTFjMTY5MzNkYmMucG5n?x-oss-process=image/format,png)

插入过程：

1. 判断数组是否为空，为空进行初始化;
2. 不为空，计算 k 的 hash 值，通过`(n - 1) & hash`计算应当存放在数组中的下标 index;
3. 查看 table[index] 是否存在数据，没有数据就构造一个Node节点存放在 table[index] 中；
4. 存在数据，说明发生了hash冲突(存在二个节点key的hash值一样), 继续判断key是否相等，相等，用新的value替换原数据(onlyIfAbsent为false)；
5. 如果不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；(如果当前节点是树型节点证明当前已经是红黑树了)
6. 如果不是树型节点，创建普通Node加入链表中；判断链表长度是否大于 8并且数组长度大于64， 大于的话链表转换为红黑树。
7. 插入完成之后判断当前节点数是否大于阈值，如果大于开始扩容为原数组的二倍。



HashMap 在底层将 key-value 当成一个整体进行处理，这个整体就是一个 Entry 对象。HashMap 底层采用一个 Entry[] 数组来保存所有的 key-value 对，当需要存储一个 Entry 对象时，会根据hash算法来决定其在数组中的存储位置，在根据equals方法决定其在该数组位置上的链表中的存储位置；当需要取出一个Entry时，也会根据hash算法找到其在数组中的存储位置，再根据equals方法从该位置上的链表中取出该Entry。



**ConcurrentHashMap:**

​		 ConcurrentHashMap使用分段锁，降低了锁粒度，让并发度大大提高。

 		ConcurrentHashMap的大部分操作和HashMap是相同的，例如初始化，扩容和链表向红黑树的转变等。

​		 但是，在ConcurrentHashMap中，利用一个CAS算法实现无锁化的修改值的操作，降低了锁代理的性能消耗。使用CAS和synchronized保证并发。synchronized只锁住当前链表或红黑树的首节点，这样是要hash不冲突，就不会产生并发冲突。

​		这个算法的基本思想就是不断地去比较当前内存中的变量值与你指定的一个变量值是否相等，如果相等，则接受你指定的修改的值，否则拒绝你的操作。因为当前线程中的值已经不是最新的值，你的修改很可能会覆盖掉其他线程修改的结果。这一点与乐观锁，SVN的思想是比较类似的。





## 9.关于排序：

| 排序方法 | 时间复杂度（平均） | 时间复杂度（最坏) | 时间复杂度（最好) | 空间复杂度 | 稳定性 | 复杂性 |
| -------- | ------------------ | ----------------- | ----------------- | ---------- | ------ | ------ |
| 插入排序 | O(n2)              | O(n2)             | O(n)              | O(1)       | 稳定   | 简单   |
| 希尔排序 | O(nlog2n)          | O(n2)             | O(n)              | O(1)       | 不稳定 | 较复杂 |
| 选择排序 | O(n2)              | O(n2)             | O(n2)             | O(1)       | 不稳定 | 简单   |
| 堆排序   | O(nlog2n)          | O(nlog2n)         | O(nlog2n)         | O(1)       | 不稳定 | 较复杂 |
| 冒泡排序 | O(n2)              | O(n2)             | O(n)              | O(1)       | 稳定   | 简单   |
| 快速排序 | O(nlog2n)          | O(n2)             | O(nlog2n)         | O(nlog2n)  | 不稳定 | 较复杂 |
| 归并排序 | O(nlog2n)          | O(nlog2n)         | O(nlog2n)         | O(n)       | 稳定   | 较复杂 |
| 基数排序 | O(d(n+r))          | O(d(n+r))         | O(d(n+r))         | O(n+r)     | 稳定   | 较复杂 |



### 快排的基本思想：

O(nlog2n)   O(nlog2n)  不稳定  较复杂

1. 选取其中的一个元素作为基准元素,将所有小于基准元素的元素都放在一边,比基准元素大的元素都放在另一边
2. 将依据基准元素分成两列的元素,使用上述的排序方法进行排列(递归调用)
3. 直到一层层递归都符合递归调用的结束条件,结束递归调用
4. 最终完成快速排序

```java
public class kuaiPai {

    public static void main(String[] args) {
        int[] arr = new int[]{5, 3, 4, 1, 6, 322, 66, 2, 78};
        quickSort(arr, 0, arr.length - 1);
        System.out.println(Arrays.toString(arr));
    }
 
 
    public static void quickSort(int[] arr, int start, int end) {
 
        //当开始位置小于结束位置时（数组有数据）  进行排序  也就是递归入口
        if (start < end) {
            //首先找到基准数  作为比较的标准数  取数组开始位置   从哪里开始  用哪个数当标准数 这个就是标准数
            int stard = arr[start];
            //记录需要进行排序的下标
            int low = start;
            int high = end;
 
            //循环比对比标准数大和小的数字
            while (low < high) {
                //如果标准数小于右边的数字  把右边的游标卡尺向左移动
                while (low < high && stard <= arr[high]) {
                    high--;
                }
                //如果标准数大于 右边的数字
                //用低位数字替换右边数字
                arr[low] = arr[high];
                //如果左边的数字比标准数小
                while (low < high && arr[low] <= stard) {
                    low++;
                }
                //如果左边的数字比标准数大
                //用左边数字替换右边数字
                arr[high] = arr[low];
            }
            //把标准数赋给高或者低所在的元素
            arr[low] = stard;
            //处理所有比标准数小的数字
            quickSort(arr, start, low);
            //处理所有比标准数大的数字
            quickSort(arr, low + 1, end);
        }
    }
}
```



### 堆排序的基本思想：

O(nlog2n)  O(nlog2n)  O(nlog2n)  O(1) 不稳定

1. 先把数组构成一个大顶堆
2. 然后把堆顶（数组最大），和数组最后一个元素交换，这样就把最大值放到了最后面
3. 数组长度n-1，再进行构造堆
4. 循环这样的方法

```java
public class HeapSort implements IArraySort {

    @Override
    public int[] sort(int[] sourceArray) throws Exception {
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);

        int len = arr.length;

        buildMaxHeap(arr, len);

        for (int i = len - 1; i > 0; i--) {
            swap(arr, 0, i);
            len--;
            heapify(arr, 0, len);
        }
        return arr;
    }

    private void buildMaxHeap(int[] arr, int len) {
        for (int i = (int) Math.floor(len / 2); i >= 0; i--) {
            heapify(arr, i, len);
        }
    }

    private void heapify(int[] arr, int i, int len) {
        int left = 2 * i + 1;
        int right = 2 * i + 2;
        int largest = i;

        if (left < len && arr[left] > arr[largest]) {
            largest = left;
        }

        if (right < len && arr[right] > arr[largest]) {
            largest = right;
        }

        if (largest != i) {
            swap(arr, i, largest);
            heapify(arr, largest, len);
        }
    }

    private void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

}
```



### 归并排序的基本思想：

归并排序  O(nlog2n)  O(nlog2n)  O(nlog2n)  O(n)  稳定

- 归并排序，首先把一个数组中的元素，按照某一方法，先拆分了之后，按照一定的顺序各自排列，然后再归并到一起，使得归并后依然是有一定顺序的 。
- 归并排序算法可以利用递归的思想或者迭代的思想去实现。首先我们先把一个无序的数组去拆分，然后利用一定的规则，去合并。类似于二叉树的结构。其总的时间复杂度为O( n log n)。空间复杂度为 S（nlogn）

```java
public class MergeSort implements IArraySort {

    @Override
    public int[] sort(int[] sourceArray) throws Exception {
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);

        if (arr.length < 2) {
            return arr;
        }
        int middle = (int) Math.floor(arr.length / 2);

        int[] left = Arrays.copyOfRange(arr, 0, middle);
        int[] right = Arrays.copyOfRange(arr, middle, arr.length);

        return merge(sort(left), sort(right));
    }

    protected int[] merge(int[] left, int[] right) {
        int[] result = new int[left.length + right.length];
        int i = 0;
        while (left.length > 0 && right.length > 0) {
            if (left[0] <= right[0]) {
                result[i++] = left[0];
                left = Arrays.copyOfRange(left, 1, left.length);
            } else {
                result[i++] = right[0];
                right = Arrays.copyOfRange(right, 1, right.length);
            }
        }

        while (left.length > 0) {
            result[i++] = left[0];
            left = Arrays.copyOfRange(left, 1, left.length);
        }

        while (right.length > 0) {
            result[i++] = right[0];
            right = Arrays.copyOfRange(right, 1, right.length);
        }

        return result;
    }

}
```



## 10. synchronized和volatile和lock

### synchronized

synchronized的底层也是一个基于CAS操作的等待队列

synchronized修饰成员方法时，默认的锁对象，就是当前对象

synchronized修饰静态方法时，默认的锁对象，当前的class对象，比如User.class

synchronized修饰代码块时，可以自己来设置锁对象，比如synchronized（this）

synchronized无需手动获取和释放锁，发生异常会自动解锁，不会出现死锁

synchronized可以保证变量修改的可见性和原子性，可能会造成线程阻塞

synchronize对线程的同步仅提供独占模式，而Lock即可以提供独占模式，也可以提供共享模式

synchronize的加锁和解锁的过程是由JVM管理

当一个线程使用synchronize获取锁时，若锁被其他线程占用着，那么当前只能被阻塞，直到成功获取锁。



问：锁和synchronized为何能保证可见性？
答：根据[JDK 7的Java doc](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/package-summary.html#MemoryVisibility)中对`concurrent`包的说明，一个线程的写结果保证对另外线程的读操作可见，只要该写操作可以由`happen-before`原则推断出在读操作之前发生。



### volatile

volatile修饰变量

synchronized和锁需要通过操作系统来仲裁谁获得锁，开销比较高，而volatile开销小很多。因此在只需要保证可见性的条件下，使用volatile的性能要比使用锁和synchronized高得多。

`volatile`关键字作用的是保证**可见性**和**有序性**，并不保证原子性。不会造成阻塞

Java内存模型的主要目标就是**定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的细节**。

`volatile`是通过编译器在生成字节码时，在指令序列中添加“**内存屏障**”来禁止指令重排序的。

JVM的实现会在volatile读写前后均加上内存屏障，在一定程度上保证有序性。

硬件层面的“**内存屏障**”：

- **sfence**：即写屏障(Store Barrier)，在写指令之后插入写屏障，能让写入缓存的最新数据写回到主内存，以保证写入的数据立刻对其他线程可见
- **lfence**：即读屏障(Load Barrier)，在读指令前插入读屏障，可以让高速缓存中的数据失效，重新从主内存加载数据，以保证读取的是最新的数据。
- **mfence**：即全能屏障(modify/mix Barrier )，兼具sfence和lfence的功能
- **lock 前缀**：lock不是内存屏障，而是一种锁。执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。

JMM层面的“**内存屏障**”：

- **LoadLoad屏障**： 对于这样的语句Load1; LoadLoad; Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
- **StoreStore屏障**：对于这样的语句Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
- **LoadStore屏障**：对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
- **StoreLoad屏障**： 对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。





### lock

Lock的加锁和解锁都是由java代码配合native方法

lock只能给代码块加锁

lock需要自己加锁或释放锁，如lock()和unlock()，如果忘记使用unlock（），则会出现死锁



Lock前缀指令和缓存一致性协议可以看出来，volatile的实现原理

1.Lock前缀的指令

Lock前缀的指令在多核处理器下会发生两件事情：

1)将当前处理器的缓存行的数据协会到系统内存。

2)这个写回内存的操作会使其他CPU缓存了该内存的地址的数据无效。

2.缓存一致性协议

在多处理器下，为了保证各个处理器的缓存是一致的，每个处理器都会通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了。当处理器发现自己缓存行对应的地址被修改，就会将当前处理器的缓存行设置为无效状态。当处理器对这个数据进行读写的时候，会重新把数据从内存中读取到处理器缓存中。





## 11.关于Spring

### 11.1解释一下什么是 ioc？

ioc：Inversionof Control（中文：控制反转）是 spring 的核心，对于 spring 框架来说，就是由 spring 来负责控制对象的生命周期和对象间的关系。

简单来说，控制指的是当前对象对内部成员的控制权；控制反转指的是，这种控制权不由当前对象管理了，由其他（类,第三方容器）来管理。

依赖注入是时下最流行的IoC实现方式，依赖注入分为

1. 接口注入
2. Setter方法注入
3. 构造器注入

### 11.2  什么是AOP

- 面向切面编程，供了模块化。通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP（面向对象编程）的延续，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

- AOP采取横向抽取机制，取代了传统纵向继承体系重复性代码

- 经典应用：事务管理、性能监视、安全检查、缓存 、日志等

- Spring AOP使用纯Java实现，不需要专门的编译过程和类加载器，在运行期通过代理方式向目标类织入增强代码

- AspectJ是一个基于Java语言的AOP框架，Spring2.0开始，Spring AOP引入对Aspect的支持，AspectJ扩展了Java语言，提供了一个专门的编译器，在编译时提供横向代码的织入

  

基于Java的主要AOP实现有：

1. aop底层将采用代理机制进行实现。
2. 接口 + 实现类 ：spring采用 jdk 的动态代理Proxy。
3. 实现类：spring 采用 cglib字节码增强。



## 12.http协议和https协议

1. https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用。

2. http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。

3. http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。

4. http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。所以说，HTTP 安全性没有 HTTPS高，但是 HTTPS 比HTTP耗费更多服务器资源。

5. HTTP协议运行在TCP之上，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份。



客户端在使用HTTPS方式与Web服务器通信时有以下几个步骤：

（1）客户使用https的URL访问Web服务器，要求与Web服务器建立SSL连接。

（2）Web服务器收到客户端请求后，会将网站的证书信息（证书中包含公钥）传送一份给客户端。

（3）客户端的浏览器与Web服务器开始协商SSL连接的安全等级，也就是信息加密的等级。

（4）客户端的浏览器根据双方同意的安全等级，建立会话密钥，然后利用网站的公钥将会话密钥加密，并传送给网站。

（5）Web服务器利用自己的私钥解密出会话密钥。

（6）Web服务器利用会话密钥加密与客户端之间进行通信。





## 13.tcp和udp

tcp 和 udp 是 OSI 模型中的运输层中的协议。tcp 提供可靠的通信传输

两者的区别大致如下：

- tcp 面向连接，udp 面向非连接即发送数据前不需要建立链接；

- tcp 提供可靠的服务（数据传输），udp 无法保证；

- tcp 面向字节流，udp 面向报文；

- tcp 数据传输慢，udp 数据传输快；





## 14.异常：

Throwable 是所有异常的父类，

它有两个直接子类:

Exception 又被继续划分为被检查的异常（checked exception）和运行时的异常（runtime exception即不受检查的异常）

Error 表示系统错误，通常不能预期和恢复（譬如 JVM 崩溃、内存不足等）； 恢复不是不可能但很困难的情况下的一种严重问题。比如说内存溢出。不可能指望程序能处理这样的情况。



常见的五个编译时异常:

SQLException 、

IOexception 、

 FileNotFoundException 、

 ClassNotFoundException 



运行时异常：

NullPointerException - 空指针引用异常
ClassCastException - 类型强制转换异常。
IllegalArgumentException - 传递非法参数异常。
ArithmeticException - 算术运算异常
ArrayStoreException - 向数组中存放与声明类型不兼容对象异常
IndexOutOfBoundsException - 下标越界异常
NegativeArraySizeException - 创建一个大小为负数的数组错误异常
NumberFormatException - 数字格式异常
SecurityException - 安全异常
UnsupportedOperationException - 不支持的操作异常



## 15.关于锁：

### 乐观锁：

- 乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在提交更新的时候会判断一下在此期间别人有没有去更新这个数据。

​		一般是在数据表中加上一个数据版本号 version 字段，表示数据被修改的次数。当数据被修改时，version 值会+1。当线程A要更新数据值时，在读取数据的同时也会读取 version 值，在提交更新时，若刚才读取到的 version 值与当前数据库中的 version 值相等时才更新，否则重试更新操作，直到更新成功。

实现：

主要就是两个步骤：冲突检测和数据更新。

### 悲观锁：

悲观锁：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻止，直到这个锁被释放。

**悲观锁主要分为[共享锁和排他锁](https://www.jianshu.com/p/b4731a7d255a)：**

- 共享锁【shared locks】又称为读锁，简称S锁。顾名思义，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。
- 排他锁【exclusive locks】又称为写锁，简称X锁。顾名思义，排他锁就是不能与其他锁并存，如果一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁，包括共享锁和排他锁，但是获取排他锁的事务是可以对数据行读取和修改。

在对记录进行修改前，先尝试为该记录加上排他锁(exclusive locks)。

如果加锁失败，说明该记录正在被修改，那么当前查询可能要等待或者抛出异常。具体响应方式由开发者根据实际需要决定。

如果成功加锁，那么就可以对记录做修改，[事务](https://www.jianshu.com/p/7e76ce65e3ad)完成后就会解锁了。

期间如果有其他对该记录做修改或加排他锁的操作，都会等待解锁或直接抛出异常。

**在乐观锁与悲观锁的选择上面，主要看下两者的区别以及适用场景就可以了。**
 1️⃣响应效率：如果需要非常高的响应速度，建议采用乐观锁方案，成功就执行，不成功就失败，不需要等待其他并发去释放锁。乐观锁并未真正加锁，效率高。一旦锁的粒度掌握不好，更新失败的概率就会比较高，容易发生业务失败。
 2️⃣冲突频率：如果冲突频率非常高，建议采用悲观锁，保证成功率。冲突频率大，选择乐观锁会需要多次重试才能成功，代价比较大。
 3️⃣重试代价：如果重试代价大，建议采用悲观锁。悲观锁依赖数据库锁，效率低。更新失败的概率比较低。
 4️⃣乐观锁如果有人在你之前更新了，你的更新应当是被拒绝的，可以让用户从新操作。悲观锁则会等待前一个更新完成。这也是区别。

### 可重入锁：

​			可重入就是说某个线程已经获得某个锁，可以再次获取锁而不会出现死锁。

- 可重入锁有

  - synchronized
  - ReentrantLock

  ### 使用ReentrantLock的注意点

  ReentrantLock 和 synchronized 不一样，需要手动释放锁，所以使用 ReentrantLock的时候一定要**手动释放锁**，并且**加锁次数和释放次数要一样**

  

## 16.线程安全

线程安全的话，用的不是很多，所以了解的不是怎么深入。主要线程安全还是以下三种方法实现吧；

- 方法一：使用安全类，比如 Java. util. concurrent 下的类。
- 方法二：使用自动锁 synchronized。
- 方法三：使用手动锁 Lock。

尽可能避免引起非线程安全的条件——共享变量。如果能从设计上避免共享变量的使用，即可避免非线程安全的发生，也就无须通过锁或者synchronized以及volatile解决原子性、可见性和顺序性的问题。

