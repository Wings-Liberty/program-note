## 基础

---



### 注意事项

- 第1行和第2行是标准注释，第1行注释可以让这个hello.py文件直接在Unix/Linux/Mac上运行，第2行注释表示.py文件本身使用标准UTF-8编码；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```

- **Python中常用的是匈牙利命名法**



### 和java的不同之处

- 每句后面不需要加 ' ; ' ，正因如此python对缩进要求很严格

- 尽量使用四个空格而不是Tab键

- python的整数和浮点数没有大小限制

  对于浮点数，如果数字很大就直接表示为inf（无限大）

- None——空值

- 两种除号 //（商只取整数部分，小数部分要么没有要么为.0，哪怕是式子里有小数） 和 /（商也有小数部分）

- python的变量不用声明类型（是很方便），但是正因如此，把变量作为参数传给函数时，可能因为参数类型和传入的类型不一致而抛异常

- python的函数返回值可以有多个，但是最后会把返回值放到tuple里

- 赋值语句a, b = b, a + b  不是a = b, b = a+b，因为a+b的a的值是a=b之前的值

- 函数里还能定义函数

- python中用and不用&&

### 输入输出

```python
# 输出
print()
# 输出，带逗号后，会有空格隔开
print("asd", "asdf", "as") 

#输入
input()
# 声明变量并获取输入值，声明变量不需要声明变量类型
name = inpuut()
```



#### 格式化输出

| 占位符 | 替换内容     |
| ------ | ------------ |
| %d     | 整数         |
| %f     | 浮点数       |
| %s     | 字符串       |
| %x     | 十六进制整数 |

> 还有其他%2d、%02d 、%.3f等和java一样，都是可以用的

举个例子

```python
age = input('输入年龄:')
gender = input('输入年级:')
print('Age: %s, Gender: %s' % (age, gender))
```



### format()格式化字符串

它会用传入的参数依次替换字符串内的占位符`{0}`、`{1}`……，不过这种方式写起来比%要麻烦得多，{n}  n的后面可以写格式化的东西，就像下面的例子一样

```python
name = input('输入名字:')
grade = float(input('输入成绩:'))
print('Hello, {0}, 成绩提升了 {1:.1f}'.format(name, grade))
```





### 转义字符和不转义字符

```python
# 转义字符和java一样

# 不转义字符的方法

print(r'\\\t\\')  # 输出结果：\\\t\\
```



### 编码和解码

str和bytes互换

```python
# 输出字符的编码
print(ord("A"))
# 输出结果：  65

# 输出该编码所代表的的字符
print(chr(65)) 
# 输出结果： A

```

对于字节类型的变量

```python
# 使用b的前缀表示后面的内容将变为字节
x = b'ABC'
```

举个例子

```python
# 编码
print('ABC'.encode('ascii'))
# 输出结果：  b'ABC'
print('中文'.encode('utf-8'))
# 输出结果： b'\xe4\xb8\xad\xe6\x96\x87'
```

```python
# 解码
print(b'ABC'.decode())
# 输出结果： ABC
print(b'\xe4\xb8\xad\xe6\x96\x87'.decode())
# 输出结果： 中文
```

当bytes中有小部分无效字符时直接解码会报错，忽略无效字符即可

```python
b'\xe4\xb8\xad\xff'.decode('utf-8', errors='ignore')
# 输出结果： 中
```



### 常用内置函数和自定义函数

#### 常用内置函数

```python
# 计算字符数，一个英文占1个字节，一个中文占3个字节
len('asdf')
# 输出结果： 4
```

```python
# 生成一个[0,n)的整数列表
range(n)
```

```python
# hlep(函数名) 函数名不带小括号， help函数自带输出，不用放在print函数里
help(abs)
```

```python
# 获取绝对值
abs(-20)

# 求最大值，可接受任意个参数
max()

# 把十进制数转换为十六进制数
print(hex(10))
# 输出结果： 0xa

# 变量类型检查，检查x是不是指定的变量类型，返回布尔值
isinstance(x, (int, float))

# 使用其他函数
import math
math.sin()
math.cos()
math.pi

# 把字符串的所有字母都转换为小写
'ASDF'.lower()
```

```python
# 强制类型转换
int('123')
# 结果： 123
int(12.34)
# 结果： 12
float('12.34')
# 结果： 12.34
str(1.23)
# 结果： '1.23'
str(100)
# 结果： '100'
bool(1)
# 结果： True
bool('')
# 结果： False
```

```python
# 获取对象的多有属性和函数
dir(stu)

# 可和getattr()、setattr()以及hasattr()一起使用
```





把函数赋给变量

```python
a = abs # 变量a指向abs函数
a(-1) # 所以也可以通过a调用abs函数
```



#### 自定义函数

```python
# 这是hello.py文件里的所有代码
def my_function(x):
    print('我的输出的是', x)
```

```python
# 这里是test.py文件
import hello
hello.my_function(3)
# 输出结果： 3
```

```python
# 自定义函数的参数表可以直接给参数赋值，并返回多个值
def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny
```

##### 位置参数

普通的必传参数

##### 默认参数

函数的参数表里可以给参数默认值，调用该函数时，这个参数如果没传就使用默认值。默认参数在参数表的最后面（和C++一样）

  ```python
  # 当有多个默认参数，而且有的默认参数传值，有的没有时，可以这样用
  enroll('Adam', 'M', city='Tianjin')
  # 指定传的参数是赋给哪个变量的
  ```

  **函数里的默认参数**java里的static变量一样，调用多次的函数使用的默认参数变量其实是一个

  解决方法在参数表里给默认参数赋None值，然后在下面的代码里赋默认值

  

##### 可变参数

接收任意数量的参数，0个也可以

```python
def calc(*numbers):
    pass

# 可以像这样传参，函数接收参数后把参数放大tuple里
calc()
calc(1, 2)
# 也可以直接把一个现成的list或tuple传进去，不过实参前面要加上*（星号）
calc(*list)
```



##### 关键字参数

作用：可传0个或任意个参数。

和可变参数不同的是，关键字参数参数需要有参数名，例：name='myname', email='myemail'

```python
def myFunction(name, age, **key):
    print(name)
    print(age)
    print(key)

# 调用函数的时候，关键字参数可传可不传。传的话必须是key=value的形式传
myFunction('as', 12, city='qwer')
```

##### 命名关键字参数

作用：限制只有指定的key才能传进关键字参数

```python
def person(name, age, *, city, job):
    print(name, age, city, job)
    
# 这样调用
person('tom', 12, 'city':'tianin', 'job':'coder')
```

只能传入key是city和job的键值对

````python
# 如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符*了
def person(name, age, *args, city, job):
    print(name, age, args, city, job)
````



**参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数。**

### 递归函数

#### 尾递归

尾递归是指，在函数返回的时候，调用自身本身，并且，return语句不能包含表达式。

#### 其他递归

**pass语句**

用来占位的，什么都不做

```python
# 定义一个空函数，什么都不做，但是如果没有pass程序就跑不起来
def nop():
    pass

# 也可以用在其他地方
if age >= 18:
    pass
```





### 特殊注释

第一行注释是为了告诉Linux/OS X系统，这是一个Python可执行程序，Windows系统会忽略这个注释；

第二行注释是为了告诉Python解释器，按照UTF-8编码读取源代码，否则，你在源代码中写的中文输出可能会有乱码。

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```



### list和tuple（中括号和小括号）



**list**

list和java的数组比较像

能通过下标访问，下标为负数时，如-2表示访问倒数第二个元素

但是，list是可变列表，可以添加和删除元素，不是想java的数组一样初始化后因为数组大小固定就不能额外添加元素

```python
# 初始化一个列表
list = ['Michael', 'Bob', 'Tracy']


# 在指定下标的元素前插入一个元素
classmates.insert(0, 'Tom')

# 在列表末尾添加一个元素
classmates.append('Tom')

# 删除指定下标的元素，不传参数默认删除列表最后一个参数
classmates.pop(0)

# 替换，直接用下标访问，修改元素
classmates[0] = 'Cat'

# 排序
sort()
```



**tuple**

tuple一旦初始化就不能修改

它没有inset(), append(), pop()方法，也不能通过访问下标修改元素（元素只能在初始化时定下来，以后并不能修改）

它只能通过下标访问元素

注：

```python
# 这句有歧义，所以python把这个小括号就算小括号，t表示的是数字1，而不是tuple列表
t = (1)

# 在末尾价格逗号这样t就是tuplo列表了
t = (1,)
```

当tuple里面有list时，list的元素是可变的

tuple就像是个指针，它的指向不会变，指的还是哪个list，但是list里的元素可以变



### 顺序逻辑

- if

  ```java
  age = 3
  if age >= 18:
      print('adult')
  elif age >= 6:
      print('teenager')
  else:
      print('kid')
  ```

  ```python
  # 只要x是非零数值、非空字符串、非空list等，就判断为True，否则为False
  if x:
      print('True')
  ```

- for  in循环

  ```python
  names = ['Michael', 'Bob', 'Tracy']
  for name in names:
      print(name)
  ```

  ```python
  sum = 0
  for x in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:
      sum = sum + x
  print(sum)
  ```

- while

  ```python
  sum = 0
  n = 99
  while n > 0:
      sum = sum + n
      n = n - 2
  print(sum)
  ```

- break, contunue

  这俩以前咋用，现在咋用



### dict和set（大括号和小括号加str或list）

- dict就是map

  ```python
  # 创建dict
  d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
  # 输出结果： 95
  print(d['Michael'])
  
  # 检查dict中有没有指定的key，输出True或False
  print('Thomas' in d)
  # 或者
  d.get('Thomas')
  # 删除指定key-value
  d.pop('Bob')
  ```

  dict的key必须是**不可变对象**

- set

  ```python
  # 创建一个set需要一个list或字符串
  # s = set([6, 1, 2, 2, 3, 3])
  s = set('012345678910')
  print(s)
  ```
  

  set是无序，没有重复的元素
  
  - 通过`add(key)`方法可以添加元素到set中
  - 通过`remove(key)`方法可以删除元素

set可以做数学集合的运算

如：

  ```python
  # 交集
  s1 | s2
  # 并集
  s1 & s2
  ```

### 高级特性

#### 切片Slice

作用：可以方便循环输出指定范围的元素，或截取指定范围内的元素

```python
L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']
print(L[0:3])
# 输出结果：['Michael', 'Sarah', 'Tracy']

print(L[-2:])
# 输出结果：['Bob', 'Jack']
print(L[-2:-1])
# 输出结果：['Bob'] 范围是左闭右开
```

```python
L = list(range(100))

print(L[:10])
# 取钱十个数，输出结果：[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# 取后十个数
print(L[-10:])

# 前十个数，每两个取一个
print(L[:10:2])

# 所有数，每5个取一个
print(L[::5])

# 原样复制一个list
list = L[:]

# [a:b:c] a是数组开始的下标，b是结束下标，c是每个c个数获取一个值
# 		  a为空，表示获取的数组从下标0开始，b为空表示到数组最后一个元素结束，c为空，默认值为1
```

tuple和str也能这样用，tuple这样用得到的结果还是tuple

#### 迭代

就是迭代器模式

dict迭代输出的shikey而不是vaule

```python
# 迭代dict中的key
for key in d

# 迭代dict中的value
for value in d.values()

# 迭代dict中的key和value
for k, v in d.items()

# 迭代时输出list的元素的下标
for i, value in enumerate(['A', 'B', 'C']):
     print(i, value)
        
# 判断指定的类型是否可以迭代
from collections import Iterable
print(isinstance('abc', Iterable)) # str是否可迭代

# 遍历多个元素
for x, y in [(1, 1), (2, 4), (3, 9)]:
     print(x, y)
```

#### 列表生成器

作用：生成list，使用中括号加for  in

`可实现全排列`

```python
# 创建一个[1*1, 2*2 ... 9*9, 10*10]
list = [x * x for x in range(1, 11)]
print(list)

# 使用两层循环实现全排列
list = [m + n for m in 'ABC' for n in 'XYZ']

# 使用列表生成器，列出当前目录下的所有文件和目录名
import os
print([d for d in os.listdir('.')])
```

#### 生成器generator

作用：当列表生成器生成的元素过多，会泄露大量内存时，使用生成器，生成器会在访问下一个元素时才生成元素，而不是在初始化时就生成，使用小括号加for  in

```python
# 生成器生成list和列表生成器生成list初始化的区别就是生成器使用()，列表生成器使用[]
g = (x * x for x in range(10))

# 生成器访问下一个元素时用过next()函数实现的
print(next(g))
```

#### yield

generator的函数，在每次调用`next()`的时候执行，遇到`yield`语句返回，再次执行时从上次返回的`yield`语句处继续执行。

```python
def odd():
    print('step 1')
    yield
    print('step 2')
    yield
    print('step 3')
    yield

o = odd()
next(o)
next(o)
next(o)

# 也可以这样调用
for o in odd():
    o
```

```python
但是用for循环调用generator时，发现拿不到generator的return语句的返回值。如果想要拿到返回值，必须捕获StopIteration错误，返回值包含在StopIteration的value中：

g = fib(6)
while True:
    try:
		x = next(g)
         print('g:', x)
	except StopIteration as e:
         print('Generator return value:', e.value)
         break
```

#### 迭代器

Iterable——可迭代对象

Iterator——迭代器

list，str，set等是可迭代对象但不是迭代器

生成器都是迭代器

但可以通过

```python
iter([1, 2, 3, 4, 5]) # 获取迭代器对象
```



### 函数式编程

函数名也是变量，是一个function类型的变量

高阶函数：把函数作为参数传入函数

#### map()/reduce()

> map，一次将元祖作为参数执行指定方法并把返回值放进新的list中，最后组成新的list
>
> reduce，将所有元素作为参数，执行指定方法，返回最后的结果

```python
# map函数把迭代器里的所有元素都传进函数里，并把返回值放到迭代器里
>>> def f(x):
...     return x * x
...
>>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> list(r)
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

```python
# reduce函数一次对迭代器里的元素执行函数，并把返回结果作为参数再进行函数运算
>>> from functools import reduce
>>> def add(x, y):
...     return x + y
...
>>> reduce(add, [1, 3, 5, 7, 9])
25
```



#### filter

参数为一个函数，一个序列

作用：对序列的所有元素都执行一次，根据函数的返回值（true或false）决定元素是否保留下来，filter的返回值就是保留下来的元素组成的序列

#### sorted

参数为一个序列，一个函数（可选）

作用：对序列的所有元素都执行一次函数，把返回结果放进序列里再进行排序，原序列中的元素的顺序按照排好序的序列排序并作为返回值返回

默认排序规则是按照ascall码值从小到大排序

```python
>>> sorted([36, 5, -12, 9, -21])
[-21, -12, 5, 9, 36]

>>> sorted([36, 5, -12, 9, -21], key=abs)
[5, 9, -12, -21, 36]
```





#### 返回参数

```python
# 函数里定义函数，传参数并调用函数，返回值是内函数的名时，得到的返回结果是方法和传的参数
# 通过def类型的变量调用方法时会使用之前的参数作为参数进行计算
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax = ax + n
        return ax
    return sum

f = lazy_sum(1, 3, 5, 7, 9)
print(f())
# 输出结果：25
```

返回函数的闭包有点不明白



#### 匿名函数

就是lambda表达式

`语法: lambda 参数列表 : 函数体` 如下：

````python
list(map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9]))
````

```python
def build(x, y):
    return lambda: x * x + y * y
```



#### 装饰器

就是java的aop或者说是代理模式

可以用来做日志输出

```python
def log(func):
    def wrapper(*args, **kw):
        # 函数对象有一个__name__属性，可以拿到函数的名字
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper

# @语法
@log
def now():
     print('2015-3-25')

now()
# 相当于分两步调用 
# 1. now = log(now)
# 2. now();
```



#### 偏函数

作用：把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。

```python
# functools模块
int2 = functools.partial(int, base=2)

# 以上代码等同于
def int2(x, base=2):
    return int(x, base)
```



### 模块

> 一个.py文件就是一个模块（Module）

和java一样，py可以有包package，且每个包都需要有一个`__init__.py`文件，表明这个包使py的，否则包会被当成一个普通文件夹（Pycharm会动生成: ) ）



- 一般的变量和函数是public的，可直接用全包名+变量名或函数名使用
- 以_或__为前缀的变量和函数是私有的，不应该被在别的地方使用（不是像java的private那样强制的，只是不建议，而不能阻止使用者使用）
- `__XX__`这样命名的变量和方法是特殊的



### 面向对象

#### 类和对象

创建类

```python
# class关键字 + 类名 +（父类）
class Student(object):
    # 这是构造函数的写法，参数列表中必须有self（作用和java的this一样）
    # 类的成员变量在构造函数里声明即可
    def __init__(self, name, score):
        self.name = name
        self.score = score

    def print_score(self):
        print('%s: %s' % (self.name, self.score))
        
    def print_score(std):
		print('%s: %s' % (std.name, std.score))
```

```python
# 如果类里啥也没有就这样写
class Student(object):
    pass
```

```python
# 
class Student(object):
    name = "myname"
```



创建对象和使用对象方法

```python
# 创建对象不使用new关键字
bart = Student('Bart Simpson', 59)
lisa = Student('Lisa Simpson', 87)
bart.print_score()
lisa.print_score()
```

写类的成员方法

成员方法需要传self，不传Pycharm会报红，但是可以运行



#### 访问权限

类中的成员变量的访问权限可以用__作为前缀

再类中，以\_\_为前缀的变量确实是私有变量，外部不可以使用  对象名.变量名 访问（强制的，访问会报错），但可以使用其他手段访问，如  stu._Student__name  stu是一个Student

总的来说就是，Python本身没有任何机制阻止你干坏事，一切全靠自觉。

```python
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score
    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))
```

**Python中常用的是匈牙利命名法**



#### 继承和多态

`继承和java一样，子类又父类的方法，子类可重写父类的方法，不过Python不需要写@Override注解`

isinstance()函数，和java一样，检查类型是否匹配

```python
# 场景：Animal是父类，Dog和Cat是子类
# b是Dog时结果是true，是Animal时结果是false
isinstance(b, Dog)
```

Python有多继承，但是不建议用（挺好）

type()函数，输出对象的实际类型



#### 获取对象信息

类似`__xxx__`的属性和方法在Python中都是有特殊用途的，比如`__len__`方法返回长度。在Python中，如果你调用`len()`函数试图获取一个对象的长度，实际上，在`len()`函数内部，它自动去调用该对象的`__len__()`方法，所以，下面的代码是等价的：

```python
class MyDog(object):
    def __len__(self):
        return 100

dog = MyDog()
len(dog)
100
```

#### 动态赋予属性

```python
class Student(object):
    pass

s = Student()
s.name = 'Michael' # 动态给实例绑定一个属性
print(s.name)
Michael
```

**\__slots__**

```python
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

绑定指定范围以外的属性时会报错



#### @property

作用：不使用setter，但是设置属性时执行一些业务逻辑

```python
class Student(object):
    @property # getter方法的变形
    def score(self):
        return self._score
    @score.setter # setter方法的变形
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
        
>>> s = Student()
>>> s.score = 60 # OK，实际转化为s.set_score(60)
>>> s.score # OK，实际转化为s.get_score()
60
>>> s.score = 9999
Traceback (most recent call last):
  ...
ValueError: score must between 0 ~ 100!
```

```python
class Student(object):

    @property
    def birth(self):
        return self._birth

    @birth.setter
    def birth(self, value):
        self._birth = value

    @property
    def age(self):
        return 2015 - self._birth
```

#### 定制类

其实就是重写各种方法

| 函数名        | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| `__str__`     | 就是java里的toString，类重写这个方法即可实现                 |
| `__repr__()`  | 通常`__str__()`和`__repr__()`代码都是一样的<br />区别是上个函数是在print中被调用<br />这个函数在直接写一个对象时被调用（整句只有一个对象名）<br />通常在写好`\__str__` 后   \__repr__ = \__str__即可 |
| `__iter__`    | 迭代器的方法，直接返回自己self即可                           |
| `__getitem__` | 可实现对象能像数组一样通过下标访问元素                       |
| `__getattr__` | 用于处理使用  对象名.属性名  但是实际上属性并不存在时的情况  |
| `__call__`    | 重写这个函数后可将对象作为def类型的对象  调用重写的call函数体中的业务逻辑  如： bean()<br />可自定义call函数的参数列表 |

关于call函数，它模糊了对象和函数的界限，因此可使用callable()函数判断对象是不是可以调用的函数

```python
>>> callable(Student())
True
>>> callable(max)
True
>>> callable([1, 2, 3])
False
>>> callable(None)
False
>>> callable('str')
False
```



#### 枚举类

```python
from enum import Enum

Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))
```

```python
# 这样我们就获得了Month类型的枚举类，可以直接使用Month.Jan来引用一个常量，或者枚举它的所有成员：

for name, member in Month.__members__.items():
    print(name, '=>', member, ',', member.value)
```

或手动写枚举类的子类

```python
from enum import Enum, unique

@unique # 用来检查是否包含重复元素
class Weekday(Enum):
    Sun = 0 # Sun的value被设定为0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6
```



#### 元类

> 有点像java的泛型

```python
>>> from hello import Hello
>>> h = Hello()
>>> h.hello()
Hello, world.
>>> print(type(Hello))
<class 'type'>
>>> print(type(h))
<class 'hello.Hello'>
```

Class  的类型是 type

> type可以做到，不写class类的代码就可以创建一个自定义的类对象

```python
>>> def fn(self, name='world'): # 先定义函数
...     print('Hello, %s.' % name)
...
# 				自定义类名  父类      成员方法列表
>>> Hello = type('Hello', (object,), dict(hello=fn)) # 创建Hello class
>>> h = Hello()
>>> h.hello()
Hello, world.
>>> print(type(Hello))
<class 'type'>
>>> print(type(h))
<class '__main__.Hello'>
```

要创建一个class对象，`type()`函数依次传入3个参数：

1. class的名称；
2. 继承的父类集合，注意Python支持多重继承，如果只有一个父类，别忘了tuple的单元素写法；
3. class的方法名称与函数绑定，这里我们把函数`fn`绑定到方法名`hello`上。

**元类**

可以创建出类的类，如果说对象是类的实例化的话，那么类就是元类的实例化

暂略，教程说元类基本用不到



### 异常处理

先上例子

```python
try:
    print('try...')
    r = 10 / 0
    print('result:', r)
except ZeroDivisionError as e:
    print('except:', e)
finally:
    print('finally...')
print('END')
```



#### 捕获异常

和java不同的是，Python除了try-except-finally 还有else可以加

```python
try:
    print('try...')
    r = 10 / int('2')
    print('result:', r)
except ValueError as e:
    print('ValueError:', e)
    logging.exception(e)
except ZeroDivisionError as e:
    print('ZeroDivisionError:', e)
else:
    # 当没有异常时会执行else中的代码
    print('no error!')
finally:
    # 不管有没有异常，finally的代码都会执行，这就是和else的区别
    print('finally...')
print('END')
```

Python的异常的顶层父类是`BaseException`，其捕捉异常机制和java一样，父类的exception会连同子类一起捕获

`logging.exception(e)`能输出全部的错误信息

#### 抛出异常

python抛异常不需要提前声明

使用`raise`关键字抛异常即可

```python
# err_raise.py
class FooError(ValueError):
    pass

def foo(s):
    n = int(s)
    if n==0:
        raise FooError('invalid value: %s' % s)
    return 10 / n

foo('0')
```



### 测试



#### 日志

```python
import logging
logging.basicConfig(level=logging.INFO) # 设置日志输出级别
logging.info('n = %d' % n)
```



#### 测试类

1. ```python
   import unittest
   ```

2. 写测试方法，导入`unittest`模块后，继承了unittest.TestCase的类中，所有以test开头的函数名的函数均被视为测试方法

   ```python
   import unittest
   
   from mydict import Dict
   
   class TestDict(unittest.TestCase):
   
       def test_init(self):
           d = Dict(a=1, b='test')
           self.assertEqual(d.a, 1)
           self.assertEqual(d.b, 'test')
           self.assertTrue(isinstance(d, dict))
   
       def test_key(self):
           d = Dict()
           d['key'] = 'value'
           self.assertEqual(d.key, 'value')
           
   # 要在最后加上这两行，测试方法才能正常使用
   if __name__ == '__main__':
       unittest.main()
   ```



```python
import unittest

class TestDict(unittest.TestCase):

    # 测试方法运行前执行的方法
    def setUp(self):
        print('setUp...')
	# 测试方法运行后执行的方法
    def tearDown(self):
        print('tearDown...')

    def test_init(self):
        print("test_init")

    def test_key(self):
        print("test_key")

if __name__ == '__main__':
    unittest.main()
```



### IO流

> 读写文件

#### 读文件

```python
# 打开文件，读文件
f = open("D:\workspace\\1.txt", "r")
str = f.read()
print(str)
f.close()
```

```python
# 或逐行读取
for line in f.readlines():
    print(line.strip()) # 把末尾的'\n'删掉
```

在open函数参数说明

 `r表示可读，b表示读取二进制文件（图片视频可读取）`

`encoding关键字可指定所读文件的编码格式，如open(xx,xx,encoding="gbk")`

#### 写文件

使用open函数的时候写明获取文件的写权限

`open(xxx,"w")`

写操作`f.write(str)`

#### StringIO和BytesIO

暂略

#### 操作文件和目录

暂略

#### 序列化

json格式的序列化和反序列化

```python
import json
d = dict(name='Bob', age=20, score=88)
json.dumps(d)

json_str = '{"age": 20, "score": 88, "name": "Bob"}'
json.loads(json_str)
```

对于自定义的类无法向上述一样序列化的解决方案

将对象编程dict即可

```python
json.dumps(s, default=lambda obj: obj.__dict__)
```



### 进程和线程

暂略

