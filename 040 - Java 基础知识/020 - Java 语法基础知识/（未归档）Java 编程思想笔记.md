写这个笔记的目的不是为了达到只看笔记不需要书就能完全回忆起javaSE的所有知识而写的——java编程思想（巨tm厚）

# 第一遍

> 目的在于，复习旧知识和查漏补缺，比如：原来浅学且学过后基本没用过的注解，枚举类等知识
>
> 为接下来学习SpringBoot原理/源码做准备
>
> 跳过所有与垃圾回收机制有关的对象清理知识
>
> **忽略已经掌握的知识，以掌握不熟练的知识为主，所以简单的知识将不再记录**

## 第一章 导论

- 对于大多数程序来说不用关心聚合和组合的区别

- 继承的两种关系is-a（子类单纯地继承了父类，纯粹替代）,is-like-a（子类继承父类后添加了自己的变量和方法）

  纯粹替代是一种理想状态，不会因为向上转型而无法调用子类自己的方法。但是往往不这样用继承，而且多数情况下这样用也并不一定好

- java是弱语言（因为多态的存在，程序只有在运行的时候才能知道调用的是谁的方法，这样的语言叫做弱语言）

- java是单继承结构，类都继承Object，这样有利于垃圾回收机制回收垃圾（这样所有的类都可以看做object）

- java因为垃圾回收机制，而不存在内存泄漏（不需要手动销毁对象，而c/c++就需要）

- 面向对象编程的基本目的是，让上层代码只操纵父类的引用



## 第二章 对象

- java也有作用域，虽然不能在自己作用域外调用对象，但是被new出来的对象在作用域外还继续存在着，只是它唯一的引用不能使用了
- 类里的基本类型的成员数据都是有默认值的，如boolean的默认值是false 
- BigInteger类，可以操作的数字大小没有限制，但是速度会慢一些
- BigDecimal类，可以操作没有大小限制的浮点数




## 第三章 操作符

- 可变对象和不可变对象

  例，String的对象时不可变对象，每次赋值其实是创建了一块新的内存放新的值将引用指向新的值放弃旧的值。

  可变对象相反

  ```java
  String a = "a";
  String b = "b";
  a = b;
  b = "c";
  //那么a = "b", b = "c",  "a"就是不再被引用的对象，会被垃圾回收器清理掉
  
  //但是在数组中，修改数组里的数据后，通过其他引用也可以访问修改过的数据，例如String[]里的数据修改过后，其他引用指向的数据也会改变
  
  Student s1 = new Student("tom");
  Student s2 = new Student("cat");
  s1 = s2;
  s2.name = "jack";
  //那么s1和s2的名字都会变成jack
  ```

  而不可变对象线程安全（csdn）

- 方法的值传递和引用传递

  当方法参数为基本类型时，是将外部变量值拷贝到局部变量中而进行逻辑处理的，故方法是不能修改原基本变量的。

  当方法参数为包装类型（Boolean, Integer...）时结果也是一样的

  对于引用类型的方法参数，会将外部变量的引用地址，复制一份到方法的局部变量中，两个地址指向同一个对象。所以如果通过操作副本引用的值，修改了引用地址的对象，此时方法以外的引用此地址对象也会被修改。（两个引用，同一个地址，任何修改行为2个引用同时生效）
  
- 基本类型变量的比较可以直接使用==，包装类的比较就得使用.equals()，对象的equals默认是比较引用的是否是一块内存，不过你可以重写equals方法

- 直接常量，忽略

- 按位/移位操作符，忽略

- 强制类型转换，扩展转换，问题不大；窄化转换，可能会丢失数据

- 逗号操作符，例，for(int i = 0, j = i+1; i <= 10; i++)




## 第四章 控制执行流程

- 标签，原来是和goto一起用的，其实continue和break也可以使用，因为到现在也没见过这种用法，所以不学



## 第五章 初始化和清理

- 重载方法时，即使参数表里仅仅是参数的顺序不同也合法。但是不建议这样做，这样会导致代码难以维护

- 如果传入的实参小于形参，那么实参的数据类型会被提升

  例：传入一个int类型的实参，但是只有接收double类型参数的方法，那么就会调用这个方法；传的是char，找不到对应方法，程序会自动先找是int类型形参的方法

- this——指调用这个方法的对象

- 使用this可以做到在构造方法中调用构造方法，但是在一个构造方法中此法只能使用一次，且必须在第一句

- 跳过finalize()

- 静态变量在对象第一次加载的时候初始化

- 静态代码块，在程序启动时就自动执行的，只执行一次

- 非静态代码块，每次创建对象时在构造方法前调用非静态代码块

- 参数列表中的可变参数，既可以传入数个数据，也可以直接传入一个数组，也可以不传参数

  ```java
   public static int add(int x, int... args);
  ```
  
  
  
  **如果你尝试有多个只有一个可变参数的重载函数，这样会有隐患**具体见p105



## 第六章 访问权限控制

- 为了解决使用不同包下的类名相同的类，应该将类放在一internet域名（逆序）命名下的包里，因为域名是独一无二的
- import和import static的区别，后者可以直接使用导入的类的静态或public方法（方法名）而不需要写  类名.方法名
- 类的访问权限只能是public或包访问权限（即什么都不写）
- 当构造器是private时，在该类的static方法里仍可以new该对象

|                      | private | 默认访问权限 | protected | public |
| -------------------- | ------- | ------------ | --------- | ------ |
| 类本身               | 是      | 是           | 是        | 是     |
| 相同包中子类         | 否      | 是           | 是        | 是     |
| 相同包中的非子类     | 否      | 是           | 是        | 是     |
| 不同包中的子类       | 否      | 否           | 是        | 是     |
| 不同包中的非子类子类 | 否      | 否           | 否        | 是     |



## 第七章 复用类

- 创建一个子类的对象时，该子类对象里包含了一个父类的子对象。这个父类对象和你直接new一个父类是一样的。二者区别在于，后者来自于外部，前者的子对象被包装在子类对象的内部
- 定义final变量也可以不赋值，只要在使用它之前给它赋值即可，一般在构造器中给他赋值（前提是你没给它赋初值）。当然，赋完值后不能改变
- final变量不能有setter方法
- final参数，在参数表中使用final，意味着你不能修改参数的值
- final方法，不能被子类重重写
- final类，不能被继承



## 第八章 多态

- 多态是用来消除耦合的

- 父类的f方法是私有的，那么子类可以自己写一个f方法，但是当使用父类的引用创建一个子类对象时，这将使你的对象无法调用f方法

- 使用父类引用的子类对象直接访问和父类重名的公有变量/域会直接访问到父类的变量/域

- 创建一个有父类，有其他类的对象（子对象）时，调用的构造器的顺序

  先调用父类里的子对象的构造器，再调用父类的构造器，再按顺序调用其他类里的构造器，最后调用子类的构造器
  
- 如果父类的构造器中调用一个子类覆盖过的方法，那么new那个子类的时候调用的父类的构造方法执行的将会是子类重写了的方法，而不会是父类自己的方法。这有时会坑到爹。所以父类构造器中调用的尽可能是父类自己的final和private的方法

- 使用父类引用的子类对象，无法调用子类自己添加的方法




## 第九章 接口

- 如果一个类要成为父类，那么如果没有特殊情况，他应该成为接口而不是成为一个父类或抽象类
- 接口只能继承接口（extends），不能实现接口（implents）。且如果要继承多个接口，可以使用extends继承多个接口，用逗号分开即可（java是单继承，只能继承一个普通的类，但是可以引用多个“父类”接口）
- 应该避免继承的类和实现的接口里有相同名字相同的方法
- 接口里的变量默认带有final和static的修饰符
- 在接口里定义常量组（以前的做法），枚举类也可以做到。建议使用枚举类
- 接口中的变量不能是“空final”（？？）
- 接口中的变量不是接口的一部分，它们的值被存储在解耦的静态存储区域内（没明白）
- 接口可以嵌套在接口或类中，嵌套在类里的接口可以定义为private。这样的话，实现了这个接口的类，就无法在其他地方向上转型（因为那个接口只能在它所在的类里使用）。嵌套在其他接口里的接口默认是public，且不能改为private
- 接口的default关键字。接口里的方法前加上default后，并完善方法的方法体。这样该接口的实现类也可以不用必须实现该方法（此条知识本书中没有说。个人认为，这样整的像是抽象类，区别就是接口可以实现多个，抽象类只能继承一个）



## 第十章 内部类

- 普通的内部类，如果要在别处被使用，需要具体指明这个内部类的全包名。如：

  ```java
  OuterClassName.InnerClassName innerClass = outerClass.getInnerClass();
  ```

- 内部类的对象可以访问其外围为的对象的所有成员。内部类还拥有其外围类的所有元素的访问权

- 内部类里使用OuterClass.this，其返回值就是这个内部类对象的外部类对象的引用

- 内部类对象的创建必须依赖一个外部类对象，否则无法创建。静态内部类除外

  ```java
  OuterClass.InnerClass inner = outerClass.new InnerClass();
  ```
  
- protected的内部类只能其外部类，外部类的子类，同一包下的类能用

- 使用普通内部类的原因

  1. 内部类实现了某个接口，就能返回这个接口的引用供其他人使用
  2. 解决一个问题时，你需要一个类辅助你帮你解决，但是你并不希望这个类时公共可用的（单独一个.java文件必须有一个公共可用的类）
  
- 匿名内部类。参照p195或网上资料

第十章的10.5及其后面的章节暂时不看。因为到现在没用过。



## 第十一章 持有对象

> 这一章讲常用的容器。set，map，list、queue等

- List的引用的LinkedList的对象，因为向上转型的原因，导致对象无法使用LinkedList自己的方法。这时将不建议使用向上转型



**Set**（没有重复的元素）

- HashSet无序，储存元素速度最快
- TreeSet，自定义存储顺序
- LinkedHasjSet，按照被添加的顺序存对象

**Map**（元素保存的顺序和插入的顺序不一致）

- HashMap，存最快
- TreeMap，自定义存储顺序
- LinkedHashMap，按照被插入的顺序保存对象，同时保留了HashMap的查询速度

**List**

- ArrayList，随机刚问元素块，插入和删除元素慢
- LinkedList，插入和删除元素速度快。随机访问元素慢

**java.util里的Queue不推荐使用**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190722182630847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzUxMDUyMg==,size_16,color_FFFFFF,t_70)



## 第十二章 通过异常处理错误

- RunTimeException 运行时异常/不受检查的异常。当方法里需要抛出 继承该类的异常类时 不需要在方法头后声明可能会抛出的异常，也不需要try-catch
- 在执行try-catch-finally块中执行return前会执行finally里的语句
- java异常机制的遗憾：异常丢失。try-finally块中抛出异常后由于没有catch所以即使抛出了异常也没有任何输出
  异常的限制。
- 子类重写父类的方法时不能抛出父类方法的异常之外的异常。这是为了多态。父类引用的子类对象调用方法时，表面上调用的是父类方法（但是运行时，因为动态绑定，调用的是子类的方法）所以视它可能抛出的异常都是父类方法里写的异常
- 子类的父类和实现的接口中有重名且抛不同异常的方法，此时子类重载该方法时只能按照父类的方法来抛异常（因为多态）
- 子类构造器异常说明必须包含父类构造器的异常说明，子类构造器还可以添加父类构造器异常说明以外的异常



## 第十三章 字符串

- 使用string追加字符串时，编译器会自动调用stringbuilder再把最终生成tostring   但是并不是说就可以随便用string进行字符串连接。因为里面还是会出现多次创建steingbuilder对象的问题。所以尽量使用steingbuilder执行字符串的连接
  在append方法中如果使用了如，s. append(a+":"+b)这会导致编译器另创建一个stringbuilder对象处理括号里的字符串连接

- java里能重载运算符，但是默认不能使用（因为设计者觉得不需要重载运算符），String的+就被重载过
  

*p287～p312包含的是正则表达式和string类的常用方法，暂时跳过，等第一遍过完后再看或等学spring源码时再看*



## 第十四章 类型信息

> 反射的一部分

- Class对象，编译一个类就会产生一个Class对象（更恰当的说，Class的对象被保存在了同名的.class文件里）

- 类被加载时会自动static块里的代码，但是执行的顺序在这里不讨论——p315

- ? extends Number，指的是Number的子类

- ? super Son， 指的是Son的父类

- instanceof只能用于对象和类的命名类型进行比较，而不能与Class的对象进行比较

- A instanceof B，意思是A是B的对象吗或者A是B的子类的对象吗；而==或equals只有你是这个类吗 的意思

- Class对象是多例的  Class是一个泛型类  **Class<type>对象是单例的**

  所以用==比较class对象是可以获得true的
  
- RTTI（Run Time Type Information）和反射的区别是，前者在编译时打开和检查.class文件，后者在运行时打开和检查.class文件

- 动态代理模式中，创建一个代理需要一个类加载器（可以通过已经被加载的对象获取），一个你希望该代理实现的接口列表（不是类或抽象类），一个InvocationHandler接口的实现（里面就是aop中的切面/代理拿到类后要做的事）

- 对于反射而言，处理除了不能修改final变量的值以外，不管是调用私有方法还是访问私有内部类，匿名类获取类或方法声明的泛型的占位符的标识符（最后一个没毛用）都能做到

- getDeclaredMethods()可以拿到反射类中的公共方法、私有方法、保护方法、默认访问权限方法，但不获得继承的方法。

  getMethods可以拿到反射类及其父类中的所有公共方法。



## 第十五章 泛型

- 类的泛型声明紧跟在类名后，方法的泛型声明在返回类型前

- 如果只使用泛型方法就能取代整个类的泛型化，就应该只使用泛型方法。因为这样能使代码更清晰

- 调用泛型方法时一般不显示声明泛型类型，如要需要，那么泛型声明应该出现在方法名前

- 子类声明的泛型可以包含父类声明的所有的泛型，并添加自己的泛型

- ArrayList<String>.getClass和ArrayList<Integer>.getClass使用==后结果是true，原因：泛型的擦除，在运行时，前面的两个例子的类型被认为是相同的

- 泛型的边界，之前提过，就是<T extends Father> <T super Son>，

  值得一提的是<T extends Father & MyInterface>，这里的extends后面还可以跟接口，所以这里看起来有点像多继承（其实不是）
  
- 类的泛型和继承（暂时跳过，一会再学）

  

**PS**：擦除问题不再深究，到此结束。细节下一遍再说

- p390没有看懂
- 泛型通配符，暂时不看。第二遍再看



## 第十六章 数组

- 数组，直接创建，通过整型下标访问、修改元素。效率比其他容器高，但是长度固定。一般数组知道这个就行了

- 问题，对象数组支持存入向上转型的类吗？支持。Father的数组能存Son类的对象（Father是Son类的父类）

```java
public static void main(String[] args) {
    Random random = new Random(47);

    int[][][] a = new int[random.nextInt(7)][][];
    for(int i = 0; i < a.length; i++){
        a[i] = new int[random.nextInt(5)][];
        for(int j = 0; j < a[i].length; j++){
            a[i][j] = new int[random.nextInt(5)];
        }
    }
    System.out.println(Arrays.deepToString(a));
  }
  /*
  第一个new创建了数组，其第一位的长度是有随机数确定的，其他维的长度没有定义
  位于for循环内的第二个new则会决定第二维的长度
  直到碰到第三个new第三维的长度才得以确定
  */
```



## 第十九章 枚举类型

- 枚举类不能继承或被继承，但是可以实现接口

- 由于枚举类不能被继承，所以有时想对已有的枚举类进行继承扩展是行不通的，但是可以使用接口组织枚举类（创建一个接口，在接口里定义多个枚举类）枚举类向上转型为它实现的接口也是可以的

- 枚举类的每个每枚举对象里都可以有一些值，这些值存在枚举类里定义的变量中。这些值对应的变量根据构造器的参数列表而定

  ```java
  public enum MyEnum {
      FIRST("first",1), SECOND("second",3), THIRD("third",6);
  
      private int i;
      private String des;
  
      MyEnum(String des1, int i) {
          this.des = des1;
          this.i = i;
      }
  }
  ```

枚举类看到p600



## 第二十章 注解

> 注解就像是标签，在标签里写一些东西，给予被贴上标签的东西予以说明
>
> 由于对注解不够了解，这里将详说注解

### 注解的定义

它的形式就像接口一样。这样就创建了名为TestAnnotation的注解

```java
public @interface TestAnnotation {
}
```

### 注解的使用

就像是把之前写好的一个标签贴在这里一样

```java
@TestAnnotation
public class Test {
}
```

### 元注解

注解到注解上的注解（5个）

> @Retention（指定注解存活时间）
>
> @Documented（将注解中的元素包含到javadoc中）
>
> @Target（指定注解运用的地方）
>
> @Inherited（是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。）
>
> @Repeatable（被这个注解注解过的注解，可以在它所使用的地方多次使用）



#### @Retention

- RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
- RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。
- RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
}
```

#### @Documented



#### @Inherited

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}
@Test
public class A {}
public class B extends A {}
//注解 Test 被 @Inherited 修饰，之后类 A 被 Test 注解，类 B 继承 A,类 B 也拥有 Test 这个注解
```

#### @Repeatable

```java
@interface Persons {
    Person[]  value();
}
@Repeatable(Persons.class)
@interface Person{
    String role default "";
}
/*
如果Person注解没有被@Repeatable注解过的话，就不能像这样在同一个地方使用多次
而 @Repeatable 后面括号中的类相当于一个容器注解
*/
@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan{
}
```

#### @Target

注解可以作用的地方

- ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
- ElementType.CONSTRUCTOR 可以给构造方法进行注解
- ElementType.FIELD 可以给属性进行注解
- ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
- ElementType.METHOD 可以给方法进行注解
- ElementType.PACKAGE 可以给一个包进行注解
- ElementType.PARAMETER 可以给一个方法内的参数进行注解
- ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举



### 注解的属性

注解的属性也叫成员变量。注解只有成员变量（也可以设置默认值），没有方法

```java
//这样定义注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
    int id();
    String msg();
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
    public int id() default -1;
    public String msg() default "Hi";
}

//这样使用注解
@TestAnnotation(id=3,msg="hello annotation")
public class Test {
}
```

### 注解的提取

也就是写的注解里的变量/元素怎么在运行时获取

注解通过反射获取

Class对象的isAnnotationPresent()判断类是否应用了某个注解

getAnnotation() / 获getAnnotations() 获取指定/所有注解对象

```java
//这样用
@TestAnnotation()
public class Test {
    public static void main(String[] args) {
        boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);
        if ( hasAnnotation ) {
            TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);
            System.out.println("id:"+testAnnotation.id());
            System.out.println("msg:"+testAnnotation.msg());
        }
    }
}
/*输出结果
id:-1
msg:
*/
```

如果一个注解要在运行时被成功提取，那么 @Retention(RetentionPolicy.RUNTIME) 是必须的

### 注解的使用场景

注解一般都是和反射一起用

获取到class对象/其方法/其变量检查有没有使用指定的注解，如果用了，就执行定义好的业务逻辑



## 要解决的问题

- p284有关于汇编的应用，如果学了汇编，自行学习

- 为什么不能创建泛型数组？？？？？？？

  ````java
  // 以下两种形式创建泛型数组都不行
  Pair<String>[] p=new Pair<String>[10]; // 报错
  
  T[] t = new T[12] // 报错
      
  // 对于第二种，只能这样创建。注：泛型'Key'是'<Key extends Comparable<Key>>'
  Key[] pq = (Key[]) new Comparable[max];
  ````




## 暂时不解决的知识盲区

- 包装类的自动装箱机制
- 类从加载到初始化一个对象的过程中，static块，static变量和static final变量初始化的执行顺序

