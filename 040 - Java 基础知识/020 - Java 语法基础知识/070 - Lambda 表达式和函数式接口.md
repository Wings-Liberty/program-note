
# lambda 表达式

> lambda 表达式，也称“闭包”
>
> 以下将 lambda 表达式简称为 “表达式”

`lambda` 表达式能代表一个接口的**匿名**的实现类

## 表达式模板和可简化部分

**lambda 表达式模板**

```
(parameters) ->{ statements; }
```

**可简化部分**

- 省略参数类型：如果编译器可静态自动识别参数列表的参数类型，则参数列表的参数类型可以省略
- 省略小括号：如果只有一个参数，参数列表两边的小括号可省略`()`
- 省略大括号：如果方法体只有一句话，可省略方法体的大括号`{}`
- 省略 `return` 关键字：如果方法体只有一句话，且这句话是 `return statements`，可省略 `return` 关键字


## lambda 表达式中变量的作用域

参考[[../../021 - 离线网页备份文件夹/Lambda 表达式的作用域.html|Lambda表达式的作用域]]

在表达式中使用域外的**局部变量**（**方法体内声明的变量和外部方法参数列表里的变量**）需要注意

- 不能在表达式中修改这些域外的变量的**值**，**基本数据类型和对象都不行**
- 在表达式中用到的域外变量被隐式声明为了 `final`（原因见下述代码示例）
- 表达式的参数列表中参数名不能和域外变量同名

基于前两个注意事项，建议将这些在表达中被用到的**域外局部变量都显式声明为 final**

```java
public static void main(String[] args) {
    int i = 3;
    MyInterface myInterface = msg -> {
        System.out.println(String.valueOf(msg + i));
    };
    myInterface.hello("hello");
    i = 4; // 1. 表达式中用到了这个域外变量 2. 尝试在域外修改这个变量  这行会导致报错，原因如下
}
```

```
Build Output（编译结果）:
java: 从 lambda 表达式引用的本地变量必须是最终变量或实际上的最终变量
```

也就是说在表达式中用到的**域外局部变量**不能再被修改

**但表达式内部对静态变量和所在类的实例对象的成员变量可读可写**


## 表达式中的this

`lambda` 表达式中的 `this` 是 `lambda` 表达式代码所在的类，而不是 `lambda` 所表示的函数式接口实例


## 表达式中的闭包

如下，新线程执行时，repeatHessage 方法可能已经退出，方法的局部变量 text 和 count 就会消失，而 lambda 会自行在其内部保存这两个变量的值，以便表达式运行时使用。表达式保存的 text 和 count 不是表达式声明的，所以称为自由变量

含有自由变量的代码块被称为 “闭包”

```java
class Test{
    public static void main(String[] args) {
        repeatMessage("Hello world", 20);
    }
    public static void repeatHessage(String text,int count){
        Runnable r = () -> {
            for(int i = 0; i < count；i++){
                System.out.printLn(text);
            }
            Thread.yield();
            new Thread(r).start();
        }
    }
}
```

为防止在方法还未退出时方法和表达式内部同时对自由变量进行修改，造成数据不一致问题。所以才会有强制规定**域外局部变量**为隐式 final，且不能别修改

因为静态变量和是实例对象的成员变量所属的类对象和类的实例对象不会因方法的退出而消失，所以表达式不需要保存其变量的副本，所以表达式对静态变量和实例对象的成员变量可读可写（哦耶）


## 函数式接口的声明

> 一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口
>
> 接口的静态方法和接口默认继承的Object的方法不算

在接口上添加 `@FunctionInterface` 注解，它会自动帮你检查接口是否复合函数式接口的规范，如果不符合它会报错


# 方法引用 - method reference

`JDK8` 的方法引用，又称双冒号语法，`Java` 的语法糖

方法引用不能独立存在，它等价于提供方法参数的 `lambda` 表达式。方法引用最终会转换为某个函数式接口实例

三种用法：`object::instanceMethod`，`Class::staticMethod`，`Class::instanceMethod`

```java
public class Test {

    public static void  printVal(String str){
        System.out.println("print value : "+str);
    }

    public static void main(String[] args) {
        // 创建一个List
        List<String> al = Arrays.asList("a","b","c","d");

        // 1. 使用增强for循环输出
        for (String a: al) {
            AcceptMethod.printVal(a);
        }

        // 2. 使用 lambda 表达式
        al.forEach(x->{
            Test.printVal(x);
        });

        // 3. 使用双冒号
        al.forEach(Test::printVal);

        // 使用双冒号语法和下面是等价的
        Consumer<String> methodParam = AcceptMethod::printValur; // 方法参数
        al.forEach(x -> methodParam.accept(x));// 方法执行accept

    }
}
```


`this::function` 代表这个函数对象的方法，也可以用 `MyClass::function` 的方式

`this::fuction` 不可以用在 static 方法上，`MyClass::function` 是用在类的 static 方法中


# 构造器引用

使用方式 `Class::new`

```java
ArrayList<String> names = Arrays.asList(array);
Stream<Person> stream = names.stream().map(Person::new);
List<Person> people = stream.col1ect(Col1ectors.toList());
```

对于有参构造器，构造器引用是 `Function`

对于无参构造器，构造器引用时 `Consumer`


# 函数式编程


## 函数式接口

`java.util.function` 包下的四大函数式接口

| 函数式接口                | 方法                | 用途                                   |
| ------------------------- | ------------------- | -------------------------------------- |
| Consumer<\T> 消费型接口   | `void accept(T t)`  | 接收 T 类型对象的方法                  |
| Supplier<\T> 供给型接口   | `T get()`           | 返回类型为 T 的对象                    |
|Function<\T, \R> 函数型接口| `R apply(T t)`      | 接收 T 类型对象，返回 R 类型对象的方法 |
| Predicate<\T>断定型接口   | `boolean test(T t)` | 接收 T 类型对象，返回 boolean 值的方法 |

下面是用 `lambda` 实现的简单示例

```java
// 函数型接口
Function<String, Integer> function = str -> str.length();
System.out.println(function.apply("函数型接口"));

// 供给型接口
Supplier<String> supplier = () -> "供给型接口";
System.out.println(supplier.get());

// 消费型接口
Consumer<String> consumer = str-> System.out.println(str);
consumer.accept("消费型接口");

// 断言型接口
Predicate<String> predicate = (str)-> "断言型接口".equals(str);
System.out.println(predicate.test("断言型接口"));
```

[还有很多其他函数式接口](https://www.runoob.com/java/java8-functional-interfaces.html)


# Lambda 受检异常处理

```java
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5, 6, 0);  

integers.forEach(i -> {
	try {  
		System.out.println(50 / i);  
	} catch (ArithmeticException e) {  
		System.err.println( "Arithmetic Exception occured : " + e.getMessage());  
	}  
});
```

Consummer 的 accept 方法中没有显式声明异常

```java
void accept(T t);
```

所以 lambda 写 accept 时需要在 lambda 里用 try-catch 捕捉异常，这让 lambda 失去了简洁性

比如我们可以自定义函数式接口声明异常

```java
@FunctionalInterface  
interface UncheckedFunction<T, R> {  
	R apply(T t) throws Exception;  
}
```

但如果在 foreach 里写 UncheckedFunction 的 lambda 就会导致抛出的异常抛给 foreach

foreach 没有处理异常的功能，而且 foreach 只接收 JDK 的 Consumer，Function 等接口
不支持我们自定义的 UncheckedFunction

所以需要一个适配器，把我们的 UncheckedFunction 转为 JDK 的 Function

```java
public class Try {  
	public static <T, R> Function<T, R> of(UncheckedFunction<T, R> mapper) {  
		Objects.requireNonNull(mapper);  
		return t -> {
			try {  
				return mapper.apply(t);  
			} catch (Exception e) {  
				throw Exceptions.unchecked(e);  
			}
		};  
	}

	@FunctionalInterface  
	public interface UncheckedFunction<T, R> {  
		R apply(T t) throws Exception;  
	}
}
```

