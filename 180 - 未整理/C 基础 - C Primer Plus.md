#还没有复习 

# Clion 补充

[让 Clion 下一个 C 的工程中同时存在多个 main 函数](https://blog.csdn.net/justinzwd/article/details/85206640)



# 第 1 章：初识 C 语言

> C 语言是编译型语言

## C 语言的优点

**高效性**
C 语言具有汇编语言才有的微调控能力，可通过微调控以获得最大运行速度或最有效地使用内存

**可移植性**
用和特定类型的 CPU 匹配的编译器就能把 C 语言源代码编译为能执行的可执行文件

可移植性：完美的可移植性程序是，其源代码无需修改就能在不同计算机系统中成功编译的程序

## C 语言标准

- K&R 定义了 C 语言的标准。有些编译器都需要声称是否支持 K&R 的实现
- ANSI/ISO 标准，被称为 C89 （比较常用的标准）
- ANSI/ISO 又推出标准，被称为 C99（比较常用的标准）
- 此外还有 C11，C90

## 编译源代码

编译器是把源代码转换称可执行代码的程序。可执行代码是计算机的机器语言表示的代码

编译器还负责把多个源程序和库文件合并为一个可执行程序文件（由链接器程序连接库函数）

编译器还会检查 C 语言程序的语法

## 编译机制

C 编程的基本策略是，用程序把源代码文件转换为可执行文件（其中包含可直接运行的机器语言代码）

C 语言通过编译和链接两个步骤来完成这一过程。编译器把源代码转换成中间代码，链接器把中间代码和其他代码合并，生成可执行文件。通过这种方式，如果只更改某个模块，不必因此重新编译其他模块

另外，链接 器还将你编写的程序和预编译的库代码合并

中间文件有多种形式。最普遍的一种形式即把源代码转换为机器语言代码，并把结果放在目标代码文件（或简称目标文件  .o 文件） 中
虽然目标文件中包含机器语言代码，但是不能直接运行该文件。因为目标文件中储存的是编译器翻译的源代码，还不是一个完整的程序
目标代码还缺少库函数

链接器的作用是，把目标代码、系统的标准启动代码和库代码合并成一个文件，即可执行文件。对于库代码，链接器只会把程序中要用到的库函数代码提取出来

<center>编译器和链接器</center>



简而言之，目标文件和可执行文件都由机器语言指令组成的。然而，目 标文件中只包含编译器为你编写的代码翻译的机器语言代码，可执行文件中还包含你编写的程序中使用的库函数和启动代码的机器代码



## C 语言的编程机制

C编程的基本策略是，用程序把源代码文件转换为可执行文件（其中包 含可直接运行的机器语言代码）

典型的C实现通过编译和链接两个步骤来完成这一过程。编译器把源代码转换成中间代码，链接器把中间代码和其他代码合并，生成可执行文件

C 使用这种分而治之的方法方便对程序进行**模块化**，可以独立编译单独的模块，稍后再用链接器合并已编译的模块。通过这种方式，如果只更改某个模块，不必因此重新编译其他模块。另外，链接器还将你编写的程序和预编译的库代码合并



Nginx 就有效地利用 C 语言的模块化设计和动态库的设计，使得 Nginx 能通过使用一个 C 文件/库文件/模块文件实现向正在运行中的 Nginx 插拔功能模块



# 第 2 章：C语言概述

<center>C 程序刨析</center>



```c
#include<stdio.h>
```

该行告诉编译器把 stdio.h 中的内容包含在当前程序中。相当于把 stdio.h 文件中的所有内容都放到该行所在的位置。这是一个文本替换操作，由预处理器完成

include 指令是 C 预处理器的指令。文件开头的`#include<..>`被称为头文件



**变量命名规则**

变量名可有小写字母，大写字母，数字和下划线。变量名第一个字符必须是字符或下划线，不能是数字

通常库文件中的变量名以下划线开头，为避免混淆，推荐自己命名的变量名不要以下划线开头



**函数的声明和使用**

函数原型也称为函数声明，函数声明应该放在头文件下。通常函数原型后就是 main 函数，再后面就是函数实现（函数实现可以放在函数声明后的任何位置，但为方便阅读，通常把主函数放到最前面，其他的函数实现放到后面）

函数声明和函数实现的参数列表中可以只写参数的数据类型而不写参数名

```c
#include<stdio.h>

// 函数声明/函数原型
void f(void);

// 主函数
int main(void){
    f();
    return 0;
}

// 函数实现
void f(void){
    // ... 
}
```



# 第 3 章：数据和 C

> 目标：介绍以下内容
>
> - 关键字：int、short、long、unsigned、char、float、double、_Bool、\_Complex、\_Imaginary
> - 运算符：sizeof()
> - 函数：scanf()
> - 整数类型和浮点数类型的区别
> - 如何书写整型和浮点型常数，如何声明这些类型的变量
> - 如何使用printf()和scanf()函数读写不同类型的值



C 语言提供两大系列的多综合功能数据类型：整数类型和浮点数类型



## 数据：数据类型关键字

<center>C 语言的数据类型关键字</center>



_Bool 类型表示布尔值（true或false），\_complex 和 \_Imaginary 分别表示复数和虚数

欲用`bool`类型，需要引入`stdbool.h`头文件



**浮点数的字面值表示法**

浮点数与数学中实数的概念差不多。2.75、3.16E7、7.00 和 2e-8 都是浮点数。注意，在一个值后面加上一个小数点，该值就成为一个浮点值。所以，7是整数，7.00是浮点数

3.16E7 表示3.16×107（3.16 乘以 10 的 7 次方）。其中， 107=10000000，7被称为10的指数

浮点数和整数的储存方案不同。计算机把浮点数分成小数部分和指数部分来表示，而且分开储存这两部分



<center>以浮点格式（十进制）储存π的值</center>


计算机的浮点数不能表示区间内所有的值。浮点数通常只是实际值的近似 值。例如，7.0可能被储存为浮点值6.99999



## C 语言基本数据类型





- 为了得到某个类型或某个变量在特定平台上的准确大小，可以使用 **sizeof** 运算符
- 没有被赋初值的基本数据类型变量会有一个随机的垃圾值
- 浮点数字面值在缺省的情况下都是 double 类型的，除非它后面跟一个 L 或 l 表示它是一个 long double 类型的值，或跟 F 或 f 表示它是一个 float 类型的值。ll 表示 long long ，u/U 表示 unsigned long long
- `_Bool`类型（C 语言里布尔类型的关键字居然是这样的...）C 语言的布尔类型，原则上变量只占一位
- 计算机中的浮点数和整数在本质上不同，其存储方式和运算过程有很大区别



### 整形溢出

当整型变量的值达到最大值后再加 1

如果是无符号数，溢出，值变为 0

如果是有符号数，溢出，值变为最小值（因为有符号数最大值的最高位为 0，表示正数，+1 后最高位表示负数）

short 类型“可能”比 int 类型占用的空间少，long 类型“可能”比 int 类型占用的空间多？因为 C 只规定了 short 占用的存储空间不能多于 int ，long 占用的存储空间不能少于 int



### 浮点数

<center>记数法示例</center>



float，double，long double

- C 语言要求 float 至少能表示 6 位有效数字，且取值范围至少是 10<sup>-37</sup>～10<sup>+37</sup>（通常，系统储存一个浮点数要占用32位。其中8位用于表示指数的值和符号，剩下 24 位用于表示非指数部分（也叫作尾数或有效数）及其符号）
- double（双精度）。double 类型和 float 类型的最小取值范围相同，但至少必须能表示 10 位有效数字。一般情况下，double 占用 64 位。一些系统将多出的 32 位全部用来表示非指数部分（非指数部分位数变多意味着提高了精度）
- long double 类型至少与 double 类型的精度相同



浮点数常量可以这样表示：-1.56E+12，2.87e-3 。其中 + 可以省略

- 默认情况下，编译器假定浮点型常量是 double 类型的精度。在浮点数后面加上 f 或 F 后缀可覆盖默认设置，编译器会将浮点型常量看作 float 类型



### 浮点数的上溢和下溢

浮点数上溢会导致浮点数变为无穷大

浮点数下溢会导致精度损失

特殊浮点数 NaN。出现在未被定义的结果中，比如 asin(x) ，x > 1 导致 x 超过 asin 的定义域，结果返回 NaN

浮点数还会出现舍入错误：f = f +1; f = f - 1; 结果 f 和初始值不同

关于浮点数的标准，在 IEEE 中定义了一套标准（408 里会学到）



### 其他类型

从基本类型衍生的其他类型，包括数组、指针、结构和联合

基本数据类型由11个关键字组成：int、long、short、unsigned、char、float、double、signed、\_Bool、\_Complex 和 _Imaginary



## 输入和输出中的小细节

- `scanf()`不会读取输入流中的回车/换行符。scanf 遇到输入流中的空格后会停止读取数据并抛弃空格
- `printf()`把输出发送到一个叫作缓冲区（buffer）的区域，缓冲区中的内容再不断被发送到屏幕上。C 标准明确规定了何时把缓冲区中的内容发送到屏幕：当缓冲区满、遇到换行字符或需要输入的时候（从缓冲区把数据发送到屏幕或文件被称为刷新缓冲区）



  <center>标准输出的格式</center>

| 格式 | 含义               |
| ---- | ------------------ |
| %d   | 十进制形式的整形值 |
| %o   | 八进制形式的整形值 |
| %x   | 十六进制的整形值   |
| %g   | 浮点类型           |
| %c   | 表示一个字符       |
| %s   | 表示一个字符串     |
| \n   | 换行               |



<center>标准输入中的格式</center>

| 格式 | 对应的变量类型 |
| ---- | -------------- |
| %ld  | long           |
| %f   | float          |
| %lf  | double         |
| %c   | char           |
| %s   | char 数组      |
| %hd  | short          |



# 第 4 章：字符串和格式化输入输出

> 目标：介绍以下内容
>
> - 函数：strlen() 
> - 关键字：const 
> - 字符串：如何创建、存储字符串
> - 如何使用 strlen() 函数获取字符串的长度
> - 用 C 预处理器指令 `#define` 和 ANSIC 的 const 修饰符创建符号常量



## 字符串

C 语言没有专门用于储存字符串的变量类型，字符串都被储存在 char 类型的数组中。数组由连续的存储单元组成

<center>数组中的字符串</center>



数组末尾位置的字符 \0。这是空字符（null character），C 语言用它标记字符串的结束。计算机可以自己处理这些细节

字符串，无论是表示成字符常量还是储存在字符数组中，都以一个叫做空字符的隐藏字符结尾


## strlen 和 sizeof

sizeof 获取的是指定的数据类型或变量所占用的空间大小（以字节为单位）

strlen 获取的是字符串变量/ char 数组中字符串的长度

这意味着，想要知道 char 数组占用的字节数就用 sizeof，想要知道 char 数组保存的字符串的长度就用 strlen


## 常量和 C 预处理器

**用宏定义的常量/明示常量**

```c
#define NAME value // value 可以是数字的字面量，也可以是带有单引号的字符常量，带有双引号的字符串常量
```

在预处理阶段，预处理器会将程序中和宏的 NAME 相同的非字面量的字符串做文本替换，最终编译器拿到的源代码是做过替换的代码


**用 const 定义的常量**

C90标准新增了const关键字，用于限定一个变量为只读 


**一些头文件中的明示常量**

limits.h 头文件中包含了 INT_MAX 等 XXX_MAX 的明示常量。因为不同计算机中，int，long，float，double 等基本数据类型的表示范围和占用字节数可能不同，所以这个头文件中会根据计算机的不同为这些明示常量赋值


## scanf 和 printf


### printf

请求 printf() 函数打印数据的指令要与待打印数据的类型相匹配。例如，打印整数时使用%d，打印字符时使用%c。这些符号被称为转换（conversion specification），它们指定了如何把数据转换成可显示的形式。 

ANSI C标准为 printf() 提供了转换说明

<center>printf 的转换说明</center>



<center>printf 的转换说明修饰符和标记</center>




**转换说明的意义**

数据都是以二进制被保存在内存和外存中，转换说明的作用是让程序用指定的格式翻译这些二进制数据，以人类能接受的方式表达这些二进制数据


**1. 转换不匹配**：暂时跳过

**2. printf() 的返回值**：返回打印字符的个数（注意计算针对所有字 符数，包括空格和不可见的换行符（\n））。如果有输出错误， printf() 则返回一个负值（在写入文件时很常用）

**3. 打印长字符串**

```c
printf("Hello, young lovers, wherever you are.");
printf("Hello, young " "lovers" ", wherever you are.");
```

上述两种打印方式是等价的


### scanf

<center>ANSI C中 scanf() 的转换说明</center>



<center>scanf()转换说明中的修饰符</center>





**1. 从 scanf() 角度看输入**

scanf() 函数每次读取一个字符，跳过所有的空白字符，直至遇到第 1个非空白字符才开始读取

**2. 格式字符串中的普通字符**

scanf() 函数允许把普通字符放在格式字符串中。除空格字符外的普通字符必须与输入字符串严格匹配

**3. scanf() 的返回值**

scanf() 函数返回成功读取的项数。如果没有读取任何项，且需要读取一个数字而用户却输入一个非数值字符串，scanf() 便返回 0。当 scanf() 检测到“文件结尾”时，会返回EOF


### printf() 和 scanf() 的*修饰符

printf 和 scanf 指定数据宽度时，如果数据宽度是动态的，可以用 * 代替，* 真正的值可以用一个变量表示

```c
printf("The number is :%*d:\n", width, number);
```

width 的值就是 * 的值，number 才是整形的值


# 第 5 ~ 9 章：语法基础


## 运算符

- 赋值运算符左侧必须引用一个存储位置
- “指针”，可用于指向一个存储位置
- 赋值表达式的值是赋值运算符左侧运算对象的值


## 数组和字符串




## 处理字符的函数库

C 有一系列专门处理字符的函数，ctype.h头文件包含了这些函数的原型。这些函数接受一个字符作为参数，如果该字符属于某特殊的类别，就返回一个非零值（真）；否则，返回0（假）



## 三元运算符

C 提供了三目运算符`expression1 ? expression2 : expression3;`


## switch-case

```c
switch (整型表达式)
{
    case 常量1:
        语句 <--可选
        break;
    case 常量2:
        语句 <--可选
        break;
    default : <--可选 
        语句 <--可选
}
```


## 杂项

- 强制类型转换  type()  如：`(char)a`

- C99标准新增了可代替逻辑运算符的拼写，它们被定义在ios646.h头文件中。如果在程序中包含该头文件，便可用and代替&&、or代替||、not代替!

  ​	为何C不直接使用and、or和not？因为C一直坚持尽量保持较少的关键字


## 字符输入/输入和输入验证

scanf，printf，getchar，putchar

getchar()读取每个字符，包括空格、制表符和换行符；而 scanf() 在读取数字时则会跳过空格、 制表符和换行符


### 缓冲区

对于输入字符后非立即回显的程序来说，用户的输入会先被保存在缓冲区。只用用户按下回车后，缓冲区的数据才会被送到目标中

缓冲分两类：完全缓冲 I/O，行缓冲 I/O

- 完全缓冲 I/O：当数据填满缓冲区后，缓冲区才会被刷新。通常用于文件操作
- 行缓冲 I/O：当遇到换行符的输入后，刷新缓冲区。通常用于用户和界面的交互

conio.h 头文件提供了无缓冲的 I/O 函数


## 文件结尾

键盘输入 ctrl + z 等价于 EOF


## 函数


### 函数的声明于调用

函数通常被这样定义和使用

- 函数原型（function prototype）在 main 函数前声明函数，告诉编译器函数的类型
- 函数调用（function call）在函数中用函数名调用函数，表明在此处执行函数
- 函数定义（function definition）在 main 函数下实现函数原型，明确地指定了函数要做什么

函数的声明有两种方式：

- 在 main 函数前声明函数原型，在 main 后实现函数原型
- 在 main 函数前创建函数的实现


杂项

- 函数原型指明了函数的返回值类型和函数接受的参数类型。 这些信息称为该函数的签名（signature）
- 函数原型的参数列表中的参数变量名可以省略不写，只写参数类型
- 一个 C 程序文件的函数头包括：预处理指令和函数声明
- 函数类型指的是返回值的类型
- 对于无参的函数，最好在其参数列表里使用 void 关键字表名确实没有参数（这个 ANSI 标准的历史有关）

### 递归

[递归示意图](D:\workspace\Drawio-workspace\递归示意图.drawio)

递归题目：编写一个函数，输入一个十进制整数，打印这个整数的二进制数

```c
#include <stdio.h>

void to_binary(unsigned long n);

int main(void) {
    unsigned long number;
    printf("Enter an integer (q to quit):\n");
    while (scanf("%lu", &number) == 1) {
        printf("Binary equivalent: ");
        to_binary(number);
        putchar('\n');
        printf("Enter an integer (q to quit):\n");
    }
    printf("Done.\n");
    return 0;
}

void to_binary(unsigned long n) /* 递归函数 */
{
    int r;
    r = n % 2;
    if (n >= 2) to_binary(n / 2);
    putchar(r == 0 ? '0' : '1');
}
```


### 头文件

如果把 main() 放在第 1 个文件中，把函数定义放在第 2 个文件中，那么第 1 个文件仍然要使用函数原型。把函数原型放在头文件中，就不用在每次使 用函数文件时都写出函数的原型。C 标准库就是这样做的，例如，把 I/O 函数原型放在 stdio.h 头文件中，把数学函数原型放在 math.h 头文件中

把函数原型和已定义的字符常量（#define 定义的常量）放在头文件中是一个良好的编程习惯


### 使用头文件

main.c 中引用`#include<a.h>`

`#include<a.h>` 中定义函数原型和常量（#define）

a.c 中引用`#include<a.h>`并实现其头文件中的函数原型


### 查找地址：& 运算符


```c
int void main(void){
    int n = 10;
    printf("%p", &p); // 以十六进制的格式打印变量 p 在内存中的地址
}
```


### 指针简介

& 是取地址符；* 是取值符（也叫间接运算符，也叫解引用操作）

& 通常被用于普通变量，来获取普通变量的地址

\* 通常被用于指针变量，来获取指针变量指向的地址中保存的数据

指针是一个新的类型，而不是一个简单的保存整数的整数类型


# 第 10 章：数组和指针


## 数组

> C 把数组看作是派生类型，因为数组是建立在其他类型的基础上

### 初始化数组

未被初始化的数组中元素的默认值是垃圾值（随机值；这个普通的基本类型变量相同）

但如果数组中有被初始化过的元素，那么其他元素会被赋值为 0


> 到目前为止，声明和使用的变量都属于自动存储类别的变量（在函数中被声明和使用，且声明时未使用 static 关键字）
>
> 在这里提到存储类别的原因是，不同的存储类别有不同的属性，所以不能把本章的内容推广到其他存储类别。对于一些其他存储类别的变量和数 组，如果在声明时未初始化，编译器会自动把它们的值设置为0

### 指定初始化器

> C99 增加了一个新特性：指定初始化器（designated initializer）。利用该特性可以初始化指定的数组元素


```c
int arr[6] = { [5] = 212 }; // 把 arr[5] 初始化为 212
```


### 数组下标越界

数组下标越界会导致数据异常或程序终止运行


### 指定数组大小的方式

数组的大小能使用整形常量表达式指定。整形常量表达式有：整形数字的字面值，sizeof 表达式，#define 定义的整形常量（const 声明的常量不算整形常量表达式）

C99 后可使用变量指定数组的大小


### 多维数组

<center>二维数组</center>



## 数组和指针

先声明一个数组的定义

```c
int arr[10] = {0};
int *p = arr;
```

arr 和 &arr[0] 的值都是数组的首个元素的地址。他们可以被赋给一个 int 类型的指针（`int *p`）

指针变量 +1 指指针指向下一个存储单元，而不是下一个字节的地址。比如 int 类型的指针 +1，指针会向后移动 4 个字节，因为当前设备中 int 类型的变量占用 4 个字节（这是为什么必须声明指针所指向对象类型的原因之一）


- 指针变量 +1 是指`p = p+1`或`p++`，而不是`(*p)++`
- `*`的优先级非常高
- 数组和指针在很大程度上可以互换。`p[3]`等价于`*(p+3)`等价于`arr[3]`



## 函数，数组和指针

函数原型中需要指针变量或数组变量时，函数调用中既可以传指针也可以传数组

数组和指针在很大程度上可以互换，但指针可以做 +1 操作，数组不可以

```c
int arr[10] = {0,1,2,3,4,5,6,7,8,9,};
int *p = arr;

p++; // 正确
arr++; // 语法上就错了，根本不能通过语法检查。更不要说编译
```


### 指针操作

以下说明指针的几种基本操作，此外还可以使用比较运算符来比较指针

- 把一个地址赋值给指针变量
- 获取指针的值（指针指向的地址）
- 获取指针指向的地址的数据
- 获取指针变量本身的地址
- 指针和整型常量的加减法：指针加减常数 n 等于指针向前或向后移动 n 个元素/存储单元，而不是 n 个字节
- 指针和指针的减法：得到两个指针相差了几个元素/存储单元，而不是两个指针相差了几个字节
- 指针和指针的比较：同类型的指针才能进行比较，比较的内容是指针的值，也就是指针指向的地址


如果对未被初始化过的指针用`*`取数据，或把一个常量赋值给`*p`，这会使内存管理出现严重错误。因为指针指向的地址是垃圾值，常量将会被保存在垃圾值所表示的地址中


## 保护数组中的数据

传递地址会导致一些问题。C 通常都按值传递数据，因为这样做可以保证数据的完整性

其实传地址也是按值传递数据。因为传地址时，形参和实参都是指针，形参是实参的副本，但是实参的值和形参的值都是地址，所以形参操作的数据就是实参指向的数据


```c
// 不能修改 p 所指向的数据，但可以修改 p 指向的地址
int const *p;

// 不能让 p 指向其他地址，但可以通过 p 修改它所指向的数据
int *const p;
    
// 既不能修改 p 指向的地址，也不能修改 p 指向的数据
int const *const p;
```


对于声明为 const 的数组，不能用数组名 + 下标 修改数组中元素的值

```c
void f(const int arr[]);

void main(void){
    const int arr[] = {1,2,3,4,5};
}
```


- 用 const 变量或非 const 变量的地址为 const 的指针为其赋值是合法的
- 只能把非 const 变量的地址赋给普通指针，不能把 const 变量的地址赋值给普通指针（这个规则非常合理。否则，通过指针就能改变const数组中的数据）
- **不要把 const 数组作为实参传递给需要非 const 声明的数组的函数。**C 规定，使用非const标识符修改 const 数据导致的结果是未定义的


## 指针和多维数组

先声明一个二维数组

```c
int zippo[4][2]; /* 内含int数组的数组 */
```

首先需要知道以下几点

- zippo 的值是数组第一个元素的地址
- zippo[n] 的值是一个数组，所以 zippo[n] 的值是这个数组的第一个元素的地址，即 &zippo[n]\[0]

根据以上两点，得出以下结论

- zippo 的值等于 &zippo[0]，即第一个数组的地址。也就是指针的地址
- zippo[0] 的值等于 &zippo[0]\[0]，即数组的第一个元素的地址

简而言之，zippo[0] 是一个占用一个int大小对象的地址，而zippo是一个占用两个int大小对象的地址。由于这个整数和内含两个整数的数组都开始于同一个地址，所以 zippo 和 zippo[0] 的值相同


**对 zippo 和 zippo[0] 指针自增操作**

zippo[0] 的值等于 &zippo[0]\[0]，即数组的第一个元素的地址。对 zippo[0] 自增，即让指针指向下一个元素的地址，也就是 &zippo[0]\[1]

但是 zippo 自增，其指向的是下一个数组的地址，也就是 zippo[1] 的地址，也就是 &zippo[1]\[0]


**对 zippo 和 zippo 指针进行解引用**

*zippo[0] 等价于 zippo[0]\[0]

*zippo 等价于 &zippo[0]\[0] 也等价于 zippo[0]。\*\*zippo 才等价于 \*&zippo[0]\[0] 也就是 zippo[0]\[0]



小结：有此可得出可以有地址的地址，指针的指针这种骚操作


### 指向多维数组的指针

声明一个指向多维数组的指针

```c
int zippo[4][2] = { {2,4}, {6,8}, {1,3}, {5, 7} };

int (* pz)[2]; // pz指向一个内含两个int类型值的数组

pz = zippo;

int * pax[2]; // pax是一个内含两个指针元素的数组，每个元素都指 向int的指针
```


### 指针间的兼容性

指针之间的赋值比数值类型之间的赋值要严格。例如，不用类型转换就可以把 int 类型的值赋给 double 类型的变量，但是两个类型的指针不能这样做


如果两个不同类型的指针间进行赋值，编译时会报错，但可能会允许程序运行，不过数据肯定会出错


指向指针的指针

```c
int *p = ...;
int **p2; // 一个指向指针的指针

p2 = &p1;
```


### 函数和多维数组

可以这样在函数原型里声明数组形参

```c
// 声明一个 指向数组（内含4个int类型值）的指针。可以这样声明函数的形参
void f( int (* pt)[4] );
// 如果当且仅当pt是一个函数的形式参数时，可以这样声明
// 第 1 个方括号是空的。空的方括号表明 pt 是一个指针
void f( int pt[][4] );
```


多维数组在函数中的声明规范

```c
int sum2(int ar[][], int rows); // 错误的声明

int sum2(int ar[][4], int rows); // 有效声明

int sum2(int ar[3][4], int rows); // 有效声明，但是3将被忽略
```

```c
// 一般而言，声明一个指向 N 维数组的指针时，只能省略最左边方括号中的值：
int sum4d(int ar[][12][20][30], int rows);
```


## 变长数组（VLA）

变长数组指的是声明数组时，可以使用变量初始化数组的大小。在 C99 前只能用常量指定数组的大小

但变长数组有一些限制。变长数组必须是自动存储类别，这意味着无论在函数中声明还是作为函数形参声明，都不能使用static或extern 存储类别说明符（第12章介绍）。而且，不能在声明中初始化它们、


PS：数组的定义必须是声明在块中的自动存储类别数组


```c
int quarters = 4;
int regions = 5;
double sales[regions][quarters]; // 声明一个变长数组（VLA）
```

```c
// 声明一个带有变长数组形参的函数原型
int sum2d(int rows, int cols, int ar[rows][cols]); // ar是一个变长数组 （VLA）

// C99/C11 标准规定，可以省略原型中的形参名，但是在这种情况下，必须用星号来代替省略的维度
int sum2d(int, int, int ar[*][*]); // ar是一个变长数组（VLA），省略了维度形参名
```

变长数组允许动态内存分配，这说明可以在程序运行时指定数组的大小。普通 C 数组都是静态内存分配，即在编译时确定数组的大小。由于数组大小是常量，所以编译器在编译时就知道了。第12章将详细介绍动态内存分配


## 复合字面量

```c
// 这是一个普通数组的声明
int diva[2] = {10, 20};
```

```c
// 这是一个有两个 int 类型元素的数组复合字面量
(int [2]){10, 20} // 复合字面量

(int []){50, 20, 90} // 内含3个元素的复合字面量
```


复合字面量就是一个匿名数组。对于匿名数组的用法有两种，要么马上用它，要么把它作为右值赋给类型相同的数组变量

- 把复合字面量赋值给数组变量

```c
int * pt1;
pt1 = (int [2]) {10, 20};

int (*pt2)[4]; // 声明一个指向二维数组的指针，该数组内含2个数组 元素，每个元素是内含4个int类型值的数组 
pt2 = (int [2][4]) { {1,2,3,-9}, {4,5,6,-8} };
```

- 把复合字面量作为实参传递给数组

````c
int sum(const int ar[], int n); // 函数原型

void main(void){
    int total3; 
	total3 = sum((int []){4,4,4,5,5,5}, 6);
    return;
}
````


复合字面量具有块作用域（第 12 章将介绍相关内容），这意味着一旦离开定义复合字面量的块，程序将无法保证该字面量是否存在。也就是说，复合字面量的定义在最内层的花括号中


# 第 11 章：字符串和字符串函数

> 本章介绍以下内容： 
>
> - 函数：gets()、gets_s()、fgets()、puts()、fputs()、strcat()、strncat()、strcmp()、strncmp()、strcpy()、strncpy()、sprintf()、strchr()
> - 创建并使用字符串
> - 使用 C 库中的字符和字符串函数，并创建自定义的字符串函数
> - 使用命令行参数 


## 表示字符串和字符串 I/O

字符串是以空字符（\0）结尾的char类型数组。数组又和指针有密切的关系


### 在程序中定义字符串


字符串有以下几种表示方式

```c
// 宏定义
#define MSG "I am a symbolic string constant."

/// char 数组
char words[] = "I am a string in an array.";

// char 指针
const char * pt1 = "Something is pointing at me.";

// 字符串常量的串联性质。和 Java 不同，C 语言连接字符串不需要 '+' ，而是两个字符串间加一个空格即代表连接
char mesg [] = "Things should be as simple as possible," " but not simpler.";
```


PS：puts() 函数在 stdio.h 下，只能显示字符串，且会在字符串末尾添加换行符


字符串常量属于静态存储类别（static storage class），这说明如果在函数中使用字符串常量，该字符串只会被储存一次，在整个程序的生命期内存在，即使函数被调用多次。用双引号括起来的内容被视为指向该字符串储存位置的指针


标准的字符串初始化形式很繁琐，且需要手动追加 \0

```c
const char m1[40] = { 'L','i', 'm', 'i', 't', ' ', 'y', 'o', 'u', 'r', 's', 'e', 'l', 'f', ' ', 't', 'o', ' ', 'o', 'n', 'e', ' ','l', 'i', 'n', 'e', '\'', 's', ' ', 'w', 'o', 'r','t', 'h', '.', '\0' };
```

注意最后的空字符。没有这个空字符，这就不是一个字符串，而是一个字符数组


<center>初始化数组</center>



所有未被使用的元素都被自动初始化为 0（这里的 0 指的是 char 形式的空字符，不是数字字符 0 ）


### 指针和字符串

> 指针。懵逼的开始

字符数组名和其他数组名一样，是该数组首元素的地址。因此，假设有下面的初始化： 

```c
char car[10] = "Tata";
```

那么，以下表达式都为真： 

```c
car == &car[0]、*car == 'T'、*(car+1) == car[1] == 'a'
```


用字面量声明一个字符串后，字符串都作为可执行文件的一部分储存在数据段中。当把程序载入内存时，也载入了程序中的字符串。字符串储存在静态存储区（static memory）中

- 如果这个字符串字面量作为右值赋给了 char  数组。程序在开始运行时才会为该数组分配内存。此时，才将字符串拷贝到数组中（第 12 章将详细讲解）。即数组持有的字符串是副本
- 如果这和字符串字面量作为右值赋给了 char 指针。char 指针将指向原始字符串的首地址。字符串字面量被视为 const 数据，所以应该使用 const 的 char 指针接受字符串字面量

即：初始化数组把静态存储区的字符串拷贝到数组中，而初始化指针只把字符串的地址拷贝给指针


根据以上特性，看个易错的例子

```c
#include <stdio.h>
void main(void){
    char word[] = "frame";
    word[1] = 'l';
    printf("%s", word);
    return;
}
```

输出为 flame。没问题，没毛病

但是如果用 char 指针声明字符串，就有问题了

```c
#include <stdio.h>
void main(void){
    char *word = "frame";
    word[1] = 'l';
    printf("%s", word);
    return;
}
```

编译器可能不会报错，但是绝对会出问题。因为 char 指针指向的地址是保存在静态存储区的字符串常量的第一个字符的地址，`word[1] = 'l';`表示让 &word[1] 的值为 'l' 的地址，这 TM 显然不对


出现上述问题的原因是 char 数组保存的字符串是静态存储区的字符串的副本；而 char 指针直接保存静态存储区的字符串常量的地址

为防止出现上述错误，建议在把指针初始化为字符串字面量时使用 const 限定符


**对于需要对字符进行修改的字符串应该用 char 数组接收字符串常量**


~~PS：sizeof 计算 char 数组名时，结果是 char 数组占用的全部字节数；计算 char 指针时，结果是指针占用的字节数（如果用 char 指针声明了二维数组，则 sizeof 计算的结果是 2 * 一维数组中包含的指针数量）~~

~~如：`char *arr[5];`经`sizeof(arr)`计算得到的大小为 40，因为 arr 是一个保存两个 char 的指针，而 arr 是一个含有 5 个数组的数组~~


char 以数组方式声明的字符串数组在内存中呈矩形排列，每个字符串占用的字节数都相同，且字符串在内存中占用的位置是连续的

但 char 以指针方式声明的字符串数组的指针元素所指向的字符串不必存在连续的内存中

<center>矩形数组和不规则数组</center>



**如果要用数组表示一系列待显示的字符串，请使用指针数组，因为它比二维字符数组的效率高**

如果要改变字符串或为字符串输入预留空间，不要使用指向字符串字面量的指针


## 字符串输入

> 如果想把一个字符串读入程序，首先必须预留储存该字符串的空间，然后用输入函数获取该字符串


- scanf() 和 %s 只能读取一个单词，当其遇到空格和换行符后便结束读取字符串

- gets() 函数能读取一整行字符串，直到遇到换行符，并丢弃换行符，在字符串末尾追加空字符（'\0'）使其成为字符串

  缺点：如果输入的字符串长度超过了接收字符串的 char 数组的大小，溢出的字符串可能会覆盖掉其他空间的数据。所以 gets 已被 C11 弃用

- fgets() ， gets_s() 是 gets() 的替代品


### fgets

fgets() 是作用于指定流的 I/O 函数。不丢弃读取到的换行符

```c
#include <stdio.h>
#define STLEN 14
int main(void) {
    char words[STLEN];
    puts("Enter a string, please.");
    fgets(words, STLEN, stdin);
    fputs(words, stdout);
    puts("Done.");
    return 0;
}
```

> 系统使用缓冲的 I/O。这意味着用户在按下 Return 键之前，输入都被储存在临时存储区（即，缓冲区）中。按下 Return 键就在输入中增加了一个换行 符，并把整行输入发送给 fgets()
>
> 对于输出，fputs() 把字符发送给另一个缓冲区，当发送换行符时，缓冲区中的内容被发送至屏幕上


### gets_s()

编译器不一定支持。所以还是 fgets() 靠谱

gets_s() 和 fgets() 的区别在于它只能接收 stdin 的输入。且如果读取到最大字符数还没有读到换行符后会把数组的首位置空（'\0'）表示丢弃输入，以确保不会发生和 gets() 一样的问题


### scanf()

scanf() 和 gets() 或 fgets() 的区别在于它们如何确定字符串的开始和结束

从第1个非空白字符作为字符串的开始

如果使用%s转换说明，以下一个空白字符（空行、空格、制表符或换行符）作为字符串的结束（字符串不包括空白字符）

如果指定了字段宽度，如 %10s，那么 scanf() 将读取 10 个字符或读到第 1 个空白字符停止（先满足的条件即是结束输入的条件）


scanf() 和 gets() 类似，如果输入行的内容过长，scanf()也会导致数据溢出。不过，在%s转换说明中使用字段宽度可防止溢出


## 字符串输出

> C 有 3 个标准库函数用于打印字符串：put()、fputs() 和 printf()


### put()

- puts() 在显示字符串时会自动在其末尾添加一个换行符
- puts() 遇到空字符（'\0'）后就停止输出（不要用 puts() 输出一个没有空字符的  char 数组）


### fputs()

fputs() 不会在输出的末尾添加换行符

PS：注意，gets() 丢弃输入中的换行符，但是 puts() 在输出中添加换行符。另一方面，fgets() 保留输入中的换行符，fputs() 不在输出中添加换行符


## 自定义输入输出函数

可用 getchar() 和 putchar() 自定义输入输出函数


## 字符串函数

[string.h 库](https://www.runoob.com/cprogramming/c-standard-library-string-h.html)

绝大多数处理字符串的函数原型都在`string.h`下

- strlen：计算字符串长度。遇到空字符就停止计数
- strcat：拼接两个字符串。第一个实参被变为拼接后的字符串，函数返回值为第一个实参的指针
- strncat：拼接两个字符串。需要指定一个长度，当拼接的字符串长度达到指定长度或遇到空字符后停止拼接
- strcmp：比较两个字符串的内容（不是比较地址）。相同返回 0，不同则返回非 0，返回值字典序之差
- strncmp：比较两个字符串的内容。需要指定一个长度，在比较两个字符串时，可以 比较到字符不同的地方，也可以只比较第 3 个参数指定的字符数
- strcpy：复制字符串的内容。把第二个实参的字符串的值复制给第一个实参
- strncpy：函数的第 3 个参数指明可拷贝的最大字符数
- sprintf：和 printf 类似，但其输出流指向一个 char 指针，而不是 stdout。此函数声明在 stdio.h 中


通常不带 n 的字符串函数都是不安全的，没有检查目标字符串的容量是否足够。带 n 的字符串函数通常都使用一个额外的参数指定一个长度来限制操作的长度

C 库中还有以下主要几个函数可用

- 返回指向s字符串首位置的指针：`char *strchr(const char * s, int c);`
- 返回s字符串中c字符的最后一次出现的位置：`char *strrchr(const char * s, int c);`
- 在字符串中查找指定范围内的任意一个字符：`char *strpbrk(const char * s1, const char * s2);`
- 返回指向s1字符串中s2字符串出现的首位置：`char *strstr(const char * s1, const char * s2); `

PS：这里有一个语法错误

```c
char str[20] = "123";
str = "wqr"; // 语法错误
```


## ctype.h 库

[C 标准库 - <ctype.h>](https://www.runoob.com/cprogramming/c-standard-library-ctype-h.html)

<img src="D:\image\blog\Snipaste_2021-04-20_18-35-55.png" alt="Snipaste_2021-04-20_18-35-55" style="zoom:80%;" />


## 命令行参数

把 C 程序编译为可执行的二进制文件后，便可以用指令 + 参数调用程序。C 语言用 main 的参数列表接收命令行传递来的参数

通常 argv[0] 是程序的名字，argv[0] 后才是参数。所以参数的数量 = 1（程序名） + 命令行传来的参数数量

```c
// 第一个参数是命令行收到的参数数量，argv 是保存命令行传来的参数的 char 数组
int main(int argc, char *argv[])
{
    int count;

    printf("The command line has %d arguments:\n", argc - 1);
    for (count = 1; count < argc; count++)
        printf("%d: %s\n", count, argv[count]);
    printf("\n");

    return 0;
}
```

Windows10 下的运行结果

```
λ main1.exe param1 param2 param3
The command line has 3 arguments:
1: param1
2: param2
3: param3
```

系统用空格表示一个字符串的结束和下一个字符串的开始


## 操作字符串指针的技巧

使用这种循环能遍历字符串，因为字符串以空字符结尾。while 认为空字符为 false

```c
while(*p){
    p++;
}
```


## 把字符串转换为数字

atoi()、atol() 和 atof() 函数把字符串形式的数字分别转换成 int、long 和 double类型的数字

strtol()、strtoul() 和 strtod() 函数把字符串形式的数字分别转换成 long、unsigned long 和 double 类型的数字


# 第 12 章：存储类别、链接和内存管理

> 本章介绍以下内容： 
>
> - 关键字：auto、extern、static、register、const、volatile、restricted、 \_Thread_local、\_Atomic
> - 函数：rand()、srand()、time()、malloc()、calloc()、free()
> - 如何确定变量的作用域（可见的范围）和生命期（它存在多长时间）
> - 设计更复杂的程序 


## 存储类别

程序在内存中创建的数据叫对象。程序只能通过给对象一个标识符（给对象起一个名字），再通过标识符才能访问对象。也就是说标识符是访问对象的方式

由标识符和运算符组成的表达式不一定是标识符比如`*p`

指定对象的表达式被称为左值，变量名可以既是标识符也是左值。而`p1 + 2 * p2`就不是左值也不是标识符，`*(p1 + 2 * p2)`是一个左值，因为它指定了特定内存位置的值

作用域和链接可以用来描述标识符，标识符的作用域和链接表明了程序的哪些部分可以使用这些标识符

不同的存储类别有不同的存储器、作用域和链接

这些属性可以限制标识符的使用范围。标识符可以在源代码的多文件中共享、可用于特定文件的任意函数中、可仅限于特定函数中使用，甚至只在函数中的某部分使用



### 作用域

一个C变量的作用域可以是块作用域、函数作用域、函数原型作用域或文件作用域、

- 块作用域：只要是`{}`内的，都可以叫做块的作用域。现在包括 for ，while 等的判断条件所在的`()`也属于块作用域
- 函数作用域（function scope）仅用于goto语句的标签
- 函数原型作用域（function prototype scope）用于函数原型中的形参名。形参名的作用域仅限制在函数原型的参数列表中，只有在变长数组中，形参名才有用
- 文件作用域的范围最大。凡是定义在函数外的变量都具有文件作用域，它的标识符可在本文件的所有地方使用。这些变量也叫全局变量


为什么要有这些作用域？用翻译单元和文件，这两个词的概念体会一下

编译源代码文件后，多个源代码文件会组成一个文件

举个例子：

有 main 函数的源文件叫 main.c ，这个文件用 `#include` 引入了多个头文件。这个文件被称为翻译单元。当一个变量具有文件作用域时，它的实际可见范围是整个翻译单元


### 链接

> C 变量有 3 种链接属性：外部链接、内部链接或无链接
>
> 具有块作用域、函数作用域或函数原型作用域的变量都是无链接变量。这意味着这些变量属于定义它们的块、函数或原型私有
>
> 具有文件作用域的变量可以是外部链接或内部链接。外部链接变量可以在多文件程序 / 翻译单元 中使用，内部链接变量只能在一个翻译单元中使用


- 有`static`关键字修饰的文件作用域变量是内部链接
- 无`static`关键字修饰的文件作用域变量是内部链接


### 存储期

> 作用域和链接描述了标识符的可见性。存储期描述了通过这些标识符访问的对象的生存期
>
> C对象有4种存储期：静态存储期、线程存储期、自动存储期、动态分配存储期


- 具有静态存储期的变量，其生命周期在程序运行期间一直存在。文件作用域变量具有静态存储期
- 具有线程存储期的变量，其生命周期和所在线程的生命周期相同。可用`_Thread_local`关键字指定变量的生命周期为线程存储期
- 块作用域的变量通常有自动存储期，程序进入块时自动为变量分配内存，离开块时释放内存。变长数组的内存分配是在程序执行到变长数组的声明处时才分配的。给块作用域变量添加`static`关键字，可使其具有静态存储期属性

<center>5种存储类别</center>



### 自动变量

- 可用`auto`关键字显式声明自动变量、
- 外层块和内层块中用同名变量，外层块的变量会被隐藏
- 自动变量不会被初始化，自动变量需要被显式初始化，否则其值是一个垃圾值（随机值）



### 寄存器变量

- 可用`register`关键字显式声明寄存器变量
- 寄存器变量被保存在寄存器中，所以无法获取其地址
- 寄存器变量是否在寄存器中由计算机决定，计算机可能会对其置之不理，导致其降级为普通的自动变量。即使如此也不能获取到它的地址
- 寄存器变量也可在函数原型中声明
- 由于寄存器的位数有限，所以可能没有足够的空间存储 double 类型的变量



### 块作用域的静态变量

- 块作用域的静态变量的默认值为 0
- 块作用域的静态变量只会被初始化一次，再次执行到初始化语句时会跳过不执行
- 不能在函数的形参中使用`static`
- 块作用域的静态变量也被称为内部静态存储类别



### 外部链接的静态变量

- 为了指出该函数使用了外部变量，可以在函数中用关键字`extern`再次声明。在函数中不加`extern`声明的变量默认时自动变量，会暂时隐藏外部变量
  - 在函数外对外部变量的声明叫定义式声明。在函数内加`extern`的声明叫引用式声明，它指示编译器去别处查询其定义，而不会为其分配内存空间。同时表明引用式声明并不会初始化变量，否则会直接报错
  - 基于上条描述，不要用关键字extern创建外部定义，只用它来引用现有的外部定义
- 如果一个源代码文件使用的外部变量定义在另一个源代码文件中，则必须用`extern`在该文件中声明该变量
- 外部链接的静态变量的默认值为 0；只能使用常量表达式初始化文件作用域变量



### 内部链接的静态变量

- 内部链接的静态变量的作用域在同一个文件中的所有函数中
- 也可以在函数中使用`extern`对内部链接的静态变量进行引用式声明



### 多文件

举例说明上述几种存储类别在多个文件中的应用

parta.c

```c
// parta.c --- various storage classes
#include <stdio.h>
void report_count();
void accumulate(int k);
int count = 0;       // file scope, external linkage

int main(void)
{
    int value;       // automatic variable
    register int i;  // register variable
    
    printf("Enter a positive integer (0 to quit): ");
    while (scanf("%d", &value) == 1 && value > 0)
    {
        ++count;     // use file scope variable
        for (i = value; i >= 0; i--)
            accumulate(i);
        printf("Enter a positive integer (0 to quit): ");
    }
    report_count();
    
    return 0;
}

void report_count()
{
    printf("Loop executed %d times\n", count);
}
```

partb.c

```c
// partb.c -- rest of the program
#include <stdio.h>

extern int count;       // reference declaration, external linkage

static int total = 0;   // static definition, internal linkage
void accumulate(int k); // prototype


void accumulate(int k)  // k has block scope, no linkage
{
    static int subtotal = 0;  // static, no linkage
    
    if (k <= 0)
    {
        printf("loop cycle: %d\n", count);
        printf("subtotal: %d; total: %d\n", subtotal, total);
        subtotal = 0;
    }
    else
    {
        subtotal += k;
        total += k;
    }
}
```

不要惊讶 parta.c 中没有 accumulate() 函数的实现且未用 `#include` 引用 partb.c 就能调用 partb.c 中 accumulate() 的实现。因为 gcc 用链接器的链接将其链接到一块


### 函数的存储类别

> 函数也有存储类别，可以是外部函数（默认）或静态函数。C99 新增了第 3 种类别——内联函数
>
> 外部函数可以被其他文件的函数访问，但是静态函数只能用于其定义所在的文件

- `static`声明的函数是静态函数，只能在它所在的文件中被调用
- `extern`声明的函数表明当前文件中使用的函数被定义在别处。除非使用 static 关键字，否则一般函数声明都默认为 extern


### 案例

rand() 基于种子生成随机数的函数，srand() 重置种子的函数都在 stdlib.h 头文件中

使用 `#include<stdlib.h>` 后即可使用


## 分配内存：malloc() 和 free()

> 在确定用哪种存储类别后，根据已制定好的内存管理规则，将自动选择其作用域和存储期

使用 malloc() 和 free() 能把内存的分配的大小和内存释放的时机掌握在自己手中

- malloc() 向 OS 请求指定内存空间（以字节为单位），并返回首地址。可用指针接收返回的地址
- free() 根据指针参数释放内存空间。free() 只能释放 malloc() 和 calloc() 申请来的内存空间
- calloc() 和 malloc() 类似，也用来申请内存空间并返回首地址。不过 calloc() 会把申请来的地址上的数据提前置为 0

如果没有及时调用 free() 释放内存，可能会导致内存泄漏


### 动态分配内存和变长数组

```c
int (* p3)[m]; // 要求支持变长数组

p3 = (int (*)[m]) malloc(n * m * sizeof(int)); // n×m 数组（要求支持变长数组）
```


### 存储类别和动态内存分配

理想化模型。可以认为程序把它可用的内存分为 3部分：一部分供具有外部链接、内部链接和无链接的静态变量使用；一部分供自动变量使用；一部分供动态内存分配

自动变量的生命周期决定了自动变量的内存通常作为栈来处理，这意味着新创建的变量按顺序加入内存，然后以相反的顺序销毁

动态内存分配的空间可能是在一个函数中申请到的，在另一个函数中释放的，所以内存管理比较碎片。所以对动态内存的管理速度比自动变量使用的栈管理方式的速度要慢一些 

动态分配的数据占用的内存区域通常被称为内存堆或自由内存


## ANSI C 类型限定符

const（不可变），volatile（易变的），restrict（用于提高编译器优化），_Atomic（用于支持并发，在 stdatomic.h）

C99 为类型限定符增加了一个新属性：它们现在是幂等的（idempotent）。意思是可以在一条声明中多次使用同一个限定符，多余的限定符将被忽略

- restrict 跟 Java 的指令重排 的目的类似，通过修改语句的执行顺序或内容，修改为能执行处等效果的但性能更好的语句

- _Atomic。存在于头文件 stdatomic.h 和 threads.h（包含了管理线程的方法）

  声明为`_Atomic ` 类型的变量会称为原子类型的变量。当一个线程对一个原子类型的对象执行原子操作时，其他线程不能访问该对象。类似于对一个指定的对象进行上锁


# 第 13 章：文件输入/输出

> 本章介绍以下内容： 
>
> 函数：fopen()、getc()、putc()、exit()、fclose() （打开文件，读写单个字符到流中，关闭文件，退出程序）
>
> fprintf()、fscanf()、fgets()、fputs() （从指定流中读写文件）
>
> rewind()、fseek()、ftell()、fflush() （重置读写指针，）
>
> fgetpos()、fsetpos()、feof()、ferror() 
>
> ungetc()、setvbuf()、fread()、fwrite() 
>
> 如何使用C标准I/O系列的函数处理文件 
>
> 文件模式和二进制模式、文本和二进制格式、缓冲和无缓冲I/O 
>
> 使用既可以顺序访问文件也可以随机访问文件的函数


## 与文件进行通信


### 文件是什么

C把文件看作是一系列连续的字节，每个字节都能被单独读取


### 文本模式和二进制模式

C提供两种文件模式：文本模式和二进制模式


- 文本内容和二进制内容

  如果文件内容是基于某种字符编码，如 ASCⅡ 编码，那它就是文本文件；如果文件内容是机器代码或二进制数字（如 long 类型的数字），那它就是二进制文件

- 文本文件格式和二进制文件格式

  文本文件的格式除使用某种字符编码外，还用某种规则规定换行符或文件结束符

- 文本模式和二进制模式

  C 语言提供的两种访问文件的模式。文本模式下，一些特殊字符会被转为不可见字符，比如换行符，文件内容结束符（EOF，ctrl+Z）；二进制模式下，这些特殊字符不会被做处理


### I/O 的级别

I/O 有两个级别（即处理文件访问的两个级别）

- 底层I/O（low-level I/O）使用操作系统提供的基本I/O服务
- 标准高级I/O（standard high-level I/O）用 C 库的标准包和 stdio.h 头文件定义


### 标准文件

C程序会自动打开 3 个**文件**，它们被称为标准输入（stdin）、标准输出（stdout）和标准错误输出（stderr）

- 标准输入为程序提供输入（默认是键盘），它是 getchar() 和 scanf() 使用的**文件**
- 程序通常输出到标准输出（默认是显示器），它是 putchar()、puts() 和 printf() 使用的**文件**


## 标准 I/O

- exit() 函数会关闭所有打开的文件并结束程序。其函数声明在 stdlib.h 中。在主函数中执行`return 0;`和执行`exit(0);`的效果是一样的


### fopen 函数

```c
_iobuf *fopen(const char *_Filename, const char *_Mode);
```

返回的文件指针并不指向实际的文件，而是指向这个文件在内存中的缓冲区

<center>fopen()的模式字符串</center>



新的C11新增了带 x 字母的写模式

- 以写模式打开文件，把现有文件的长度截为 0，但如果文件打开失败，文件内容不会丢失
- 以 x 打开的文件会被上锁，是线程安全的


### getc() 和 putc() 函数

getc() 和 putc() 函数与getchar()和putchar()函数类似，可以从流中读取一个字符或写入一个字符。不同的是，要告诉 getc()和putc()函数使用哪一个文件


### 文件结尾

文件以 EOF 结尾，所以通常这样读取文件

```c
int ch;
FILE * fp;
fp = fopen("wacky.txt", "r");
while (( ch = getc(fp)) != EOF) 
{
    putchar(ch); //处理输入 
}
```


### fclose() 函数

fclose(fp) 函数关闭 fp 指定的文件，必要时刷新缓冲区


## 文件 I/O


### fprintf() 和 fscanf()

```c
int fprintf(_iobuf *_File, const char *_Format, ...);
int fscanf(_iobuf *_File, const char *_Format, ...);
```


指定一个流再进行输入和输出。比如下面这个

```c
#include <stdio.h>
#include <stdlib.h>
#define MAX 40

int main(void)
{
    FILE *fp;
    char words[MAX];

    if ((fp = fopen("wordy", "a+")) == NULL)
    {
        fprintf(stdout,"Can't open \"words\" file.\n");
        exit(1);
    }
     
    puts("Enter words to add to the file; press the Enter");
    while (gets(words) != NULL  && words[0] != '\0')
        fprintf(fp, "%s ", words);
     
    puts("File contents:");
    rewind(fp);           /* go back to beginning of file */
    while (fscanf(fp,"%s",words) == 1)
        puts(words);
     
    if (fclose(fp) != 0)
        fprintf(stderr,"Error closing file\n");
     
    return 0;
}
```

说明 fp 中有一个指针，指向当前文件内容中某个字符的位置。对文件的读写都是从这个位置开始的。将其称之为 读写指针

所以需要从头读取文件时应该使用 rewind() 函数让读写指针回到文件头

fopen 的 a+ 模式下，读写指针默认指向文件内容的最后一个字符



### fgets() 和 fputs()

```c
char *fgets(char *_Buf, int _MaxCount, _iobuf *_File);
int fputs(const char *_Str, _iobuf *_File);
```

fgets() 读取到 EOF ，换行符，_MaxCount 后停止读取。如果读到 EOF 则返回一个空指针，这样便于判断文件读取完毕

fgets() 不会丢掉换行符，fputs() 不会为输出添加换行符



### 随机访问 fseek() 和 ftell()

对特定文件随机访问是能随意控制读写指针的位置

- fseek() 用于修改读写指针的位置

```c
// 第 1 个参数是文件指针，第 3 个参数是读写指针的起始位置，第 2 个参数是读写指针相对第 3 个参数的偏移量
// 最终位置 = 起始位置 + 偏移量
int fseek(_iobuf *_File, long _Offset, int _Origin);
```

<center>文件的起始点模式（第 3 个参数）</center>



这些常量被定义在 stdio.h 中

```c
#define SEEK_CUR 1
#define SEEK_END 2
#define SEEK_SET 0
```

- ftell() 函数返回的是当前的位置距文件开始处的字节数

```c
long ftell(_iobuf *_File);
```


### fgetpos() 和 fsetpos() 函数

和 fseek() ，ftell() 相比。这两个函数能指定的偏移量和返回的位置能表示的范围更大，即能更好地随机访问文件大小很大的文件

```c
int fgetpos(FILE * restrict stream, fpos_t * restrict pos);
int fsetpos(FILE *stream, const fpos_t *pos);
```

int 的返回值。0 表示成功，非 0 表示失败

fgetpos 会将获取到的位置信息写入第二个实参——fpos_t 指针变量中


## 标准 I/O 的机理

> 通常，使用标准I/O的第1步是调用fopen()打开文件（前面介绍过，C程序会自动打开3种标准文件）。fopen()函数不仅打开一个文件，还创建了一 个缓冲区（在读写模式下会创建两个缓冲区）以及一个包含文件和缓冲区数据的结构。另外，fopen()返回一个指向该结构的指针，以便其他函数知道如 何找到该结构。假设把该指针赋给一个指针变量fp，我们说fopen()函数“打开一个流”。如果以文本模式打开该文件，就获得一个文本流；如果以二进 制模式打开该文件，就获得一个二进制流。这个结构通常包含一个指定流中当前位置的文件位置指示器。除此之外，它还包含错误和文件结尾的指示器、一个指向缓冲区开始处的指针、一个文件标识符和一个计数（统计实际拷贝进缓冲区的字节数）。 
>
> 我们主要考虑文件输入。通常，使用标准I/O的第2步是调用一个定义在 stdio.h 中的输入函数，如fscanf()、getc()或 fgets()。一调用这些函数，文件中的数据块就被拷贝到缓冲区中。缓冲区的大小因实现而异，一般是512字节或是它的倍数，如4096或16384（随着计算机硬盘容量越来越大，缓冲区的大小也越来越大）。最初调用函数，除了填充缓冲区外，还要设置fp所指向的结构中的值。尤其要设置流中的当前位置和拷贝进缓冲区的字节数。通常，当前位置从字节0开始。在初始化结构和缓冲区后，输入函数按要求从缓冲区中读取数据。在它读取数据时，文件位置指示器被设置为指向刚读取字符的下一个字符。由于stdio.h系列的所有输入函数都使用相同的缓冲区，所以调用任何一个函数都将从上一次函数停止调用的位置开始。 
>
> 当输入函数发现已读完缓冲区中的所有字符时，会请求把下一个缓冲大小的数据块从文件拷贝到该缓冲区中。以这种方式，输入函数可以读取文件中的所有内容，直到文件结尾。函数在读取缓冲区中的最后一个字符后，把 结尾指示器设置为真。于是，下一次被调用的输入函数将返回EOF。 
>
> 输出函数以类似的方式把数据写入缓冲区。当缓冲区被填满时，数据将被拷贝至文件中。


## 其他标准 I/O 函数

- `int ungetc();` 把c指定的字符放回输入流中

- `int fflush(FILE *fp);` 引起输出缓冲区中所有的未写入数据被发送到fp指定的输出文件。这个过程称为刷新缓冲区

- `int setvbuf(FILE * restrict fp, char * restrict buf, int mode, size_t size);`为流指定一个缓冲区，这个缓冲区就是 buf 实参。mode 指定缓冲区是完全缓冲还是行缓冲，size 指定 char 数组（buf）的大小

- fread()和 fwrite 用于读写二进制文件的函数

- `size_t fwrite(const void * restrict ptr, size_t size, size_t nmemb,FILE * restrict fp); ` 向指定地址写入二进制数据

  把 ptr 中 nmemb 个大小为 size 的数据块写入 fp。void 类型的指针表示不确定的类型的指针，跟 Java 里的 Object 有点像

- `size_t fread(void * restrict ptr, size_t size, size_t nmemb,FILE * restrict fp);` 从指定地址中读取二进制数据

  把 fp 中 nmemb 个大小为 size 的数据块读到 ptr 中

-  int feof(FILE \*fp) 和int ferror(FILE \*fp) 用于判断函数返回 EOF 是因为读到文件的 EOF 结尾还是因为读取错误

  当上一次输入调用检测到文件结尾时，feof() 函数返回一个非零值，否则返回0。当读或写出现错误，ferror()函数返回一个非零值，否则返回 0

<center>ungets() 函数</center>



**fwrite() 函数**

```c
// 把一块 256 字节的数据从 buffer 写入文件
char buffer[256];
fwrite(buffer, 256, 1, fp);
```

```c
// 把一块 256 字节的数据从 buffer 写入文件
double earnings[10];
fwrite(earnings, sizeof(double), 10, fp);
```

**fread() 函数**

```c
// 恢复上例中保存的内含10个double类型值 的数组
double earnings[10];
fread(earnings, sizeof (double), 10, fp);
```


### 一个程序示例

指定一个 dest 文件，指定多个 source 文件。把 source 文件中的数据依次追加到 dest 文件中

```c
/* append.c -- 把文件附加到另一个文件末尾 */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define BUFSIZE 4096
#define SLEN 81

void append(FILE *source, FILE *dest);
char *s_gets(char *st, int n);

int main(void) {

    FILE *fa, *fs; // fa 指向目标文件，fs 指向源文件
    int files = 0; // 附加的文件数量
    char file_app[SLEN]; // 目标文件名
    char file_src[SLEN]; // 源文件名
    int ch;
    puts("Enter name of destination file:");
    s_gets(file_app, SLEN);
    if ((fa = fopen(file_app, "a+")) == NULL) {
        fprintf(stderr, "Can't open %s\n", file_app);
        exit(EXIT_FAILURE);
    }
    if (setvbuf(fa, NULL, _IOFBF, BUFSIZE) != 0) {
        fputs("Can't create output buffer\n", stderr);
        exit(EXIT_FAILURE);
    }
    puts("Enter name of first source file (empty line to quit):");
    while (s_gets(file_src, SLEN) && file_src[0] != '\0') {
        if (strcmp(file_src, file_app) == 0) 
            fputs("Can't append file to itself\n", stderr);
        else if ((fs = fopen(file_src, "r")) == NULL) 
            fprintf(stderr, "Can't open %s\n", file_src);
        else {
            if (setvbuf(fs, NULL, _IOFBF, BUFSIZE) != 0) {
                fputs("Can't create input buffer\n", stderr);
                continue;
            }
            append(fs, fa);
            if (ferror(fs) != 0) 
                fprintf(stderr, "Error in reading file %s.\n", file_src);
            if (ferror(fa) != 0) 
                fprintf(stderr, "Error in writing file %s.\n", file_app);
            fclose(fs);
            files++;
            printf("File %s appended.\n", file_src);
            puts("Next file (empty line to quit):");
        }
    }
    printf("Done appending.%d files appended.\n", files);
    rewind(fa);
    printf("%s contents:\n", file_app);
    while ((ch = getc(fa)) != EOF) 
        putchar(ch);
    puts("Done displaying.");
    fclose(fa);
    return 0;
}

void append(FILE *source, FILE *dest) {
    size_t bytes;
    static char temp[BUFSIZE]; // 是保存在静态存储区中的变量，只分配一次
    while ((bytes = fread(temp, sizeof(char), BUFSIZE, source)) > 0)
        fwrite(temp, sizeof(char), bytes, dest);
}

char *s_gets(char *st, int n) {
    char *ret_val;
    char *find;
    ret_val = fgets(st, n, stdin);
    if (ret_val) {
        find = strchr(st, '\n'); // 查找换行符
        if (find) // 如果地址不是NULL，
            *find = '\0'; // 在此处放置一个空字符 
        else while (getchar() != '\n') continue;
    }
    return ret_val;
}
```


### 另一个程序示例

向一个文件中写入多个 double 类型的值，再读取文件并指定要读取第几个 double 类型的数据（用偏移量计算数据在文件中的位置）

```c
#include <stdio.h>
#include <stdlib.h>

#define ARSIZE 1000

int main() {
    double numbers[ARSIZE];
    double value;
    const char *file = "numbers.dat";
    int i;
    long pos;
    FILE *iofile; // 创建一组 double类型的值
    for (i = 0; i < ARSIZE; i++)
        numbers[i] = 100.0 * i + 1.0 / (i + 1);
    // 尝试打开文件
    if ((iofile = fopen(file, "wb")) == NULL) {
        fprintf(stderr, "Could not open %s for output.\n", file);
        exit(EXIT_FAILURE);
    }
    // 以二进制格式把数组写入文件
    fwrite(numbers, sizeof(double), ARSIZE, iofile);
    fclose(iofile);
    if ((iofile = fopen(file, "rb")) == NULL) {
        fprintf(stderr, "Could not open %s for random access.\n", file);
        exit(EXIT_FAILURE);
    }
    // 从文件中读取选定的内容
    printf("Enter an index in the range 0-%d.\n", ARSIZE - 1);
    while (scanf("%d", &i) == 1 && i >= 0 && i < ARSIZE) {
        pos = (long) i * sizeof(double); // 计算偏移量
        fseek(iofile, pos, SEEK_SET); // 定位到此处
        fread(&value, sizeof(double), 1, iofile);
        printf("The value there is %f.\n", value);
        printf("Next index (out of range to quit):\n");
    }
    // 完成
    fclose(iofile);
    puts("Bye!");
    return 0;
}
```


### 小结

如果要在不损失精度的前提下保存或恢复数值数据，用二进制模式以及 fread() 和 fwrite() 函数

如果打算保存文本信息并创建能在普通文本编辑器查看的文本，请使用文本模式和函数（如 getc() 和 fprintf() ）

标准 I/O 包自动创建输入和输出缓冲区以加快数据传输，也可以用 setvbuf 函数自定义一个缓冲区


# 第 14 章：结构和其他数据类型

> 本章介绍以下内容： 
>
> 关键字：struct、union、typedef 
>
> 运算符：.、-> 
>
> 什么是C结构，如何创建结构模板和结构变量 
>
> 如何访问结构的成员，如何编写处理结构的函数 
>
> 联合和指向函数的指针


## 建立结构声明

结构体定义格式（结构体的标记/结构体的名字可有可无。有点匿名类的感觉）

```c
struct [struct_name] {	// 这叫结构体的标记
    char values[10];	// 这叫结构体的布局
}
```

结构体变量的声明

```c
struct struct_name var_name;	// 用结构体的标记创建一个结构体变量
```

访问结构体的成员变量

```c
puts(var_name.values);
```


PS：结构体也叫模板。但这个模板和 C++ 中的模板不是一个东西

- 结构体可以定义在函数外，也可以定义在函数内。定义在函数内的结构体的标记只能在函数内使用


## 创建结构体变量


### 初始化结构

```c
struct struct_name var_name{
    "I am values."
};
```


### 结构体的初始化器

可用 .field_name 为结构体的指定成员变量赋值

```c
struct struct_name var_name{
    .values = "I am values.";
};
```


## 指向结构体的指针

- 结构体类型的指针指向一个结构体的数组变量首地址时，执行 +1 操作得到的地址是数组中下一个元素（结构体变量）的地址
- 结构体指针可以用`(*p_struct).field`访问成员变量，也可以用`p_struct->field`访问成员变量


## 结构体的特性

- 结构体变量之间可以相互赋值，数组变量不能
- const 修饰的结构体，其成员变量不能被修改
- 要谨慎地在结构体中使用指向字符串常量的 char 类型的指针


## 伸缩性数组成员

在结构体的定义中，允许最后一个是数组类型的成员变量时不用指定数组的大小，例如

```c
struct struct_name {
    int value;
    char arr[];
}
```

数组可以后期用 malloc() 函数分配内存。这种数组叫伸缩性数组

但是有这种数组的结构体之间不嫩互相赋值或拷贝。也不要把这种结构体作为另一个结构体的成员变量或创建这种结构体的数组类型（所以还是直接用指针比较舒服）


## 把结构内容保存到文件中

- 用 fprintf() 把结构保存到对象中效果不好，因为为便于读取，所以需要让结构中的字段尽可能用相同的宽度
- 用 fwrite() 把结构以二进制的方式保存起来，但其数据不具可以移植性


## 联合简介

联合的定义声明

```c
union union_name {
    int int_name;
    char char_name;
    double double_name;
}
```

联合变量的声明

```c
union union_name var_name;
```

联合的语法和结构的语法相同，但联合变量中同一时间只能保存一个成员变量

当联合变量保存了 int_name 后，其他两个成员变量均不能使用。因为联合变量只申请了一个成员变量的内存空间，空间大小以成员变量中占用字节数最多的数据类型为准

联合的功能是用一个变量保存不同类型的数据


## 枚举类型

枚举类型的声明

```c
enum spectrum {
    red, orange, yellow, green, blue, violet
};
```

枚举变量的声明

```c
enum spectrum color;
```

枚举类型的声明中的成员其本质是 int 类型的数据，不过枚举使得这些 int 类型数据可用一个符号常量访问。这有利于提高代码的可阅读性

枚举类型的赋值

枚举类型中的符号常量的默认值由 0 开始递增，也可以显式赋值

```c
enum levels {
    low = 100, medium = 500, high = 2000
};
```


## typedef

typedef 可以为某一类型自定义名称。typedef 由编译器解释，而不是预处理器

typedef 定义的作用域取决于 typedef 定义所在的位置

```c
typedef char * STRING;
STRING name, sign; // 相当于 char * name, * sign;
```

typedef 作用于结构体后可以简化结构体变量的创建

```c
typedef struct complex { 
    float real; float imag; 
} COMPLEX;
```

此后用`COMPLEX var_name;`就等价于`struct complex var_name;`

## 函数和指针

通常指向函数的指针常用来作为某个函数的参数。就和 Java 的函数式编程类似

指向变量的指针用对变量名取地址的方式来赋值

指向函数的指针用函数的函数签名来赋值

指向函数的指针的声明

```c
void ToUpper(char *); // 把字符串中的字符转换成大写字符

// 创建一个指向 ToUpper 函数的指针
void (*pf)(char *); // pf 是一个指向函数的指针
```

指向函数的指针的赋值

```c
pf = ToUpper; // 让 pf 指向 ToUpper() 函数
```

指向函数的指针的调用

```c
(*pf)(str);
pf(str);
```

推荐第一种

# 第 15 章：位操作

> 本章介绍以下内容： 
>
> 运算符：～、&、|、^、 
>
> <<、>> 
>
> &=、|=、^=、>>=、<<= 
>
> 二进制、十进制和十六进制记数法（复习） 
>
> 处理一个值中的位的两个C工具：位运算符和位字段 
>
> 关键字：\_Alignas、\_Alignof


## 二进制数、位和字节


### 二进制整数

1 个字节有 8 位，可表示 255 种状态

### 有符号数

如何表示有符号整数取决于硬件，而不是C语言。也许表示有符号数最简单的方式是用 1 位（如，高阶位）储存符号，只剩下7位表示数字本身

这种方法的缺点是有两个 0：+0 和 -0


## 位运算符

```
& 按位与 双目运算符 相同位均为1，结果为1；否则为0
	00000000 00000000 00000000 00000101
&	11111111 11111111 11111111 11111100
---------------------------------------
	00000000 00000000 00000000 00000101
```

```
| 按位或 双目运算符 相同位有一个为1，结果为1；都是0，结果是0
	00000000 00000000 00000000 00000101
|	11111111 11111111 11111111 11111100
---------------------------------------
	11111111 11111111 11111111 11111101
```

```
~ 按位取反 单目运算符 1变0，0变1
~	0000000 00000000 00000000 00000101
--------------------------------------
	1111111 11111111 11111111 11111010
```

```
^ 按位异或 双目运算符 相同位值不同，结果为1；否则为0
	00000000 00000000 00000000 00000101
^	11111111 11111111 11111111 11111100
---------------------------------------
	11111111 11111111 11111111 11111001
```

```
补充：
1. 异或运算满足交换律和结合律
如果：a^b=c
那么：a=b^c，b=a^c
2. 0和 0或1的异或结果都是0，所以0和任何数n 异或的结果都是 n
3. 布尔值之间也可以进行上述4种位运算，返回值也是布尔值。在位运算时逻辑上true=1，false=0
```


假设	`A = 0011 1100`

​			`B = 0000 1101`

```
<< 左移 双目运算符 二进制向左移动指定个数，右边补0
A << 2
res: 1111 0000
```


```
>> 右移 双目运算符 二进制向右移动指定个数。补位方式一：高位是0，左边就补0；高位是1，左边就补1；补位方式二：直接补0
A >> 2
res: 0000 1111
```


```
>>> 无符号右移 双目运算符 二进制向右移动指定个数。左边补0
A >>> 2
res: 0000 1111
```



## 位运算的用途

假设有个 1 字节的二进制串叫 source ，希望对 source 进行一些位运算将其变为 dest

### 掩码

掩码和 & 一起使用。通常用来 source 中你希望看到的位

比如，一个字节中，你只想看到 1 号位，那就把 mask 设为`00000010`

在和 mask 进行 & 运算后只有 1 号位保持原值，其余位均被置 0

```
source &= mask

	10100110 (source)
&	00000010 (mask)
---------------------
  	00000010 (dest)
```



### 打开位

目的：把 source 中的指定位变为 1，其余位不变

方式：把希望 source 变为 1 的位置在 mask 中对应的位置上设为 1，其余位为 0，在进行 | 运算

如希望 1 号位被设为 1

```
source |= mask

	01001001 (source)
|	00000010 (mask)
---------------------
	01001011 (dest)
```



###  关闭位

目的：把 source 中的指定位变为 0，其余位不变

方式：把希望 source 变为 0 的位置在 mask 中对应的位置上设为 0，其余位为 1，在进行 & 运算

如希望 1 号位被设为 0

```
source &= mask

	01001011 (source)
&	11111101 (mask)
---------------------
	01001001 (dest)
```



### 位切换

目的：把 source 中指定的位从 1 变为 0 或从 0 变为 1

方式：把希望 source 进行位切换的位置在 mask 中对应的位置上设为 1，其余位为 0，在进行 ^ 运算

如希望 1 号位进行位切换

```
source ^= mask

	01001011 (source)
^	00000010 (mask)
---------------------
	01001001 (dest)
```



### 检查位的值

目的：检查 source 指定位号的值是否为 1

方式：用掩码将关注的位保留下来，其余位置为 0。再用 == 与 mask 比较，检查指定位是否均为 1

希望检查 1 ，3 号位是否为 1

```
source &= mask1

	01001011 (source)
&	00001010 (mask)
---------------------
	00001010 (dest)
	
source == mask (true)
```



## 位字段和对齐特性

跳过。有点复杂



# 第 16 章：C 预处理器和 C 库

> 本章介绍以下内容： 
>
> 预处理指令：#define、#include、#ifdef、#else、#endif、#ifndef、#if、\#elif、#line、#error、#pragma
>
> 关键字：\_Generic、\_Noreturn、_Static_assert
>
> 函数 / 宏：sqrt()、atan()、atan2()、exit()、atexit()、assert()、memcpy()、memmove()、va_start()、va_arg()、va_copy()、va_end()
>
> C 预处理器的其他功能
>
> 通用选择表达式
>
> 内联函数
>
> C 库概述和一些特殊用途的方便函数



C 预处理器在程序执行之前查看程序（故称之为预处理器）。根据程序中的预处理器指令，预处理器把符号缩写替换成其表示的内容



## 翻译程序的第一步

在预处理之前。首先，编译器把源代码中出现的字符映射到源字符集

然后删除多余的换行符。用空格替换掉注释等



## 明示常量：#define



宏被替换为替换体的过程叫宏替换

宏定义中还能包含其他宏（一些编译器不支持这种嵌套）

```c
#define TWO 2
#define FOUR  TWO*TWO
```

```c
#define PX printf("X is %d.\n", x)
printf(FMT, x);
```



## 在 `#define` 中使用参数

在 `#define` 中使用参数可以创建外形和作用与函数类似的类函数宏

<center>函数宏定义的组成</center>



```c
#define SQUARE(X) X*X
z = SQUARE(2);
```

这看上去像函数调用，但是它的行为和函数调用完全不同

建议不要用宏函数



## 文件包含：#include

当预处理器发现 `#include` 指令时，会查看后面的文件名并把文件的内容包含到当前文件中，即替换源文件中的 `#include` 指令。这相当于把被包含文件的全部内容输入到源文件 `#include` 指令所在的位置

```c
#include <stdio.h> // ←查找系统目录 
#include "hot.h" // ←查找当前工作目录 
#include "/usr/biff/p.h" // ←查找/usr/biff目录
```



通常结构体的定义，明示常量 `#define` 的定义，typedef 的类型名定义，函数原型的定义都被放在 .h 头文件中。函数原型的实现放在同名的 .c 文件中



另外，还可以使用头文件声明外部变量供其他文件共享

在函数的源文件定义文件作用域变量

```c
int status = 0; // 该变量具有文件作用域，在源代码文件
```

在头文件中引用上述变量

```c
extern int status; // 在头文件中
```

这行代码会出现在包含了该头文件的文件中，这样使用该系列函数的文件都能使用这个变量



## 其他指令



### `#undef`

\# undef：取消指定的宏定义

```c
#define LIMIT 400  
// 取消 LIMIT 的定义
#undef LIMIT
```



### 从 C 预处理器的角度看已定义

\#define 宏的作用域从它在文件中的声明处开始，直到用 `#undef` 指令取消宏为止，或延伸至文件尾



如果宏通过头文件引入，那么 `#define` 在文件中的位置取决于 `#include` 指令的位置



预定义宏，如__DATE__和__FILE__。这些宏一定是已定义的，而且不能取消定义



### 条件编译

指令告诉编译器根据编译时的条件执行或忽略信息（或代码）块



**`#ifdef`、`#else` 和 `#endif` 指令**

```c
#ifdef XXX // 如果宏 XXX 是已被定义的就执行下面的指令
...
#else // 如果宏 XXX 是未被定义的，就执行下面的指令
...
#endif // 结束 ifdef-else
```

ifdef-else 之间的指令可以是其他的预处理指令也可以是 C 的源代码。未被“执行”的源代码的含义是它们不会被编译而不是它们不会被执行



**`#ifndef` 指令**

可以和 `#else`，`#endif` 一起使用。其含义和 `#ifdef` 相反。如果 `#ifndef` 后面的宏未被定义就执行下面的指令

\#ifndef 指令通常用于防止多次包含一个文件

```c
#ifndef _STDIO_H 
#define _STDIO_H 
// 省略了文件的内容 
#endif
```

通常引用某个头文件后会定义一个同名的宏，用 `#ifndef` 检查宏是否存在就能知道和宏同名的头文件是否被引用过

头文件对应的宏的命名格式为：字母全大写，用下划线代替头文件中的点。标准头文件对应的宏以下划线开头



**`#if` 和 `#elif` 指令**

```c
#if A==1
...
# elif B==2
...
# elif B==3
...
# else
...
#endif
```

`#if` 和 `#elif` 是带有具体的条件判断语句的条件编译指令



### 预定义宏

<center>预 定 义 宏</center>



PS：`__func__`是当前执行的方法的字符串字面量



### `#line` 和 `#error`

- \#line 指令重置 \_\_LINE__ 和 \_\_FILE\_\_ 宏报告的行号和文件
- `#error` 让预处理器发送错误信息，并中断编译。通常和 `#if` 搭配使用



### 泛型选择

C 语言中用于支持泛型的语言特性。用宏定义（#define）实现。目前不需要此特性，略



## 内联函数

普通函数的调用会创建调用栈，管理栈需要消耗性能

内联函数的功能就是把调用函数的地方直接替换为函数体。把函数体代码直接放到调用处，而非真正调用一个函数

为了使用内联函数，需要让内联函数的实现和函数的调用在同一文件中。所以内联函数的实现通常被放在 .h 头文件中，这样才能让所有需要内联函数的文件调用内联函数。为此内联函数的原型应该添加`inline`和`static`

一般头文件中只放函数原型，但是内联函数是特例，因为内联函数具有内部链接



## _Norereturn 函数

用此关键字的函数，在被调用后不会返回到主函数。比如 exit() 函数，执行完 exit() 后直接退出程序，而非返回主函数



## 数学库

math.h头文件包含许多有用的数学函数

<center>ANSI C标准的一些数学函数</center>



**类型变体**

上述数学函数多用 double 做形参，但 long 转 double会有精度损失，float 转 double 会降低计算速度

为解决此问题，提供了接收不同类型参数的函数。sqrtf() 是 sqrt() 的 float 版本，sqrtl() 是 sqrt() 的 long 版本 ...

跳过 泛型宏，tgmath.h



## 通用工具库

通用工具库：stdlib.h

rand()、srand()、malloc() 和 free() 函数都在 stdlib.h 中

**atexit()**

接收函数型指针，用于注册函数。在调用 exit() 后会执行 atexit() 注册的函数

atexit() 注册的函数应该不带任何参数且返回类型为 void

注意，即使没有显式调用 exit()，main() 结束时会隐式调用 exit()


**qsort()**

快速排序

```c
void qsort(void *_Base, size_t _NumOfElements, size_t _SizeOfElements, int (*)(const void *,const void *) _PtFuncCompare);
```


# todo

- 指针类型的数组进行排序的优点是什么

 
