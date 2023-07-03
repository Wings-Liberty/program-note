
shell 提供了

- 声明，使用变量，变量赋值。变量类型
- 数据的输入和输出
- 声明和调用函数
- 条件控制，循环控制


比较特殊的一点是，shell 的变量可以来自环境变量

shell 里可以调用环境中的所有指令

[[../020 - 附件文件夹/The Linux Commend Line.pdf]]


# shell 环境

`printenv [var_name]` 可以显示环境变量

`set` 会显示环境变量和函数

`echo $VAR_NAME` 可以输出环境变量的值

如果 shell 环境中的一个成员既不可用 set 命令也不可用 printenv 命令显示，则这个变量是别名。输入不带参数的 alias 命令来查看它们


有些内置的环境变量如下

![[../020 - 附件文件夹/Pasted image 20230503133134.png|575]]
![[../020 - 附件文件夹/Pasted image 20230503133149.png|575]]


环境变量来自于启动系统时，会读取启动文件的配置脚本，脚本中声明的全局变量可以供登录 shell 使用，这就是环境变量

有些配置脚本不需要登录也会被读取，所以要区分哪些配置脚本是登录 shell 才会读取的

登录 shell 会读取一个或多个启动文件

![[../020 - 附件文件夹/Pasted image 20230503133402.png|550]]
![[../020 - 附件文件夹/Pasted image 20230503133411.png|550]]


脚本文件中声明的变量的作用于一般只在当前 shell 进程里，如果想让子进程都能访问，需要 `export $VAAR_NAME`

对于一些系统文件，在修改后需要用 `source filename` 或 `. filename` 命令使修改生效


> [!NOTE] . 命令
> 这个点（.）命令是 source 命令的同义词，一个 shell 内部命令，用来读取一个指定的 shell 命令文件，并把它看作是从键盘中输入的一样


# 脚本的格式和可执行权限

**格式**

脚本第一行应该指定 shell 的类型，比如用 bash，但是文件后缀名可以不写或随便写

```bash
#!/bin/bash
```

**可执行权限**

对于脚本文件，有两个常见的权限设置

- 权限为 755 的脚本，则每个人都能执行
- 权限为 700 的脚本，只有文件所有者能够执行

注意为了能够执行脚本，脚本必须是可读的


**可执行脚本的存放路径**

如果可执行脚本放在了 `PATH` 环境变量里，那可以直接在命令行或脚本中输入可执行脚本的名字，shell 会从 `PATH` 指定的目录里找文件


# vim 配置辅助编写脚本

vim 可以设置语法高亮，搜索高亮

- :syntax on。开启语法高亮
- :set syntax=sh。视文件为 sh 文件
- :set hlsearch。开启搜索高亮
- :set tabstop=4。设置 tab 的空格数量
- :set autoindent。设置换行时，新行和上一行的缩进位置相同
- :set number。显示行号

通过把这些命令（没有开头的冒号字符）添加到你的 ∼/.vimrc 文件中，这些改动会永久生效



# 输入和输出


## echo 输出

echo 输出，如果参数有引号，那么左右引号之间可以有回车，命令行不会收到换行就输出

## 输入

### 文档输入

```bash
command << token  
text  
token
```

command 是一个可以接受标准输入的命令名， token 是一个用来指示嵌入文本结束  
的字符串

```bash
#!/bin/bash
cat << _EOF_  
text
_EOF_
```

text 就会作为输入进入 cat 命令

token 必须在一行中单独出现，并且文本行中没有末尾的空格

here documents 中的单引号和双引号会失去它们在 shell 中的特殊含义，将其视为字面量使用

here documents 有时可以很方便编写一些执行远程命令的操作，比如用 ftp 获取文件需要执行一系列命令

```bash
ftp -n << _EOF_  
open $FTP_SERVER  
user anonymous me@linuxbox  
cd $FTP_PATH  
hash  
get $REMOTE_FILE  
bye  
_EOF_
```

如果我们把重定向操作符从“<<”改为“<<-”， shell 会忽略在此 here document 中开头  
的 tab 字符。这就能缩进一个 here document，从而提高脚本的可读性：

```bash
ftp -n <<- _EOF_  
	open $FTP_SERVER  
	user anonymous me@linuxbox  
	cd $FTP_PATH  
	hash  
	get $REMOTE_FILE  
	bye  
_EOF_
```


### read - 从标准输入读取数值

从标准输入读取单行数据，也可用重定向，读取指定文件里的单行数据


```bash
read [-options] [variable...]
```


> [!warning] read 不能和 管道符一块用
> 管道符会创建一个子 shell 执行后面的命令，所以 read 得到的值被赋值给 REPLY 或其他变量后，脚本中 read 后面的语句在父 shell 里执行，是获取不到子 shell 里的数据


有哪些选项可参考 UTool 的 Linux 命令手册

如果用了重定向，从文件里读一行数据，通常把一行数据里的多个 key 进行分割并赋值给 read 指定的多个变量，但分割符可能不是空格，可能是冒号，分号等其他分隔符

分隔符由，IFS 这个环境变量控制，比如现在要用 read 读取 /etc/passwd 中的内容，并成功地把字段分给不同的变量

```bash
#!/bin/bash  
# read-ifs: read fields from a file  
FILE=/etc/passwd  
read -p "Enter a user name > " user_name
# 读取到指定用户的信息
file_info=$(grep "^$user_name:" $FILE)  
if [ -n "$file_info" ]; then  
	IFS=":" read user pw uid gid name home shell <<< "$file_info"
	...
fi
```


> [!NOTE] 上述对 IFS 值的修改是暂时的，整行命令执行完后 IFS 会恢复为原值
> Shell 允许在一个命令之前立即发生一个或多个变量赋值。这些赋值为跟随着的命令更改环境变量。这个赋值的影响是暂时的；只是在命令存在期间改变环境变量


> [!NOTE] <<< 操作符
> <<< 操作符指示一个 here 字符串。一个 here 字符串就像一个 here 文档，只是比较简短，由单个字符串组成


### 获取命令行参数

以 `$数字` 方式可获取命令行传过来的参数

数字的范围是 0-9

- 如果希望获取第 9 个以上的参数，用 `${10}`，`${11}` 方式获取
- `$0` 的值是执行程序的路径名，可能是相对路径，也可能是绝对路径
- `$#` 是参数个数。`$0` 不是用户输入的，所以不计数
- 一般用 `basename $0` 方式获取当前正在执行的文件名，`PROGNAME=$(basename $0)`



> [!NOTE] shift 命令能解决命令行参数过多问题
> 一般用不到，除非命令行参数很多，而且需要用循环方式对每个参数做一样的处理。所以暂时不展开 shift 的讨论

除了用 `$数字` 方式访问命令行指定位置参数外，还能直接获取到整个参数列表

![[../020 - 附件文件夹/Pasted image 20230503134714.png|550]]

举个例子

```bash
#!/bin/bash  
# posit-params3 : script to demonstrate $* and $@  
print_params () {  
	echo "\$1 = $1"  
	echo "\$2 = $2"  
	echo "\$3 = $3"  
	echo "\$4 = $4"  
}  
pass_params () {  
	echo -e "\n" '$* :';     print_params $*
	echo -e "\n" '"$*" :';   print_params "$*"
	echo -e "\n" '$@ :';     print_params $@
	echo -e "\n" '"$@" :';   print_params "$@"
}  
pass_params "word" "words with spaces"
```

执行结果是

```bash
$* :  
$1 = word  
$2 = words  
$3 = with  
$4 = spaces  
"$*" :  
$1 = word words with spaces  
$2 =  
$3 =  
$4 =  
$@ :  
$1 = word  
$2 = words
$3 = with  
$4 = spaces  
"$@" :  
$1 = word  
$2 = words with spaces  
$3 =  
$4 =
```

据经验分析，`"$@"` 用的多

# 变量


## 声明变量

直接在脚本里声明变量名和值，值类型有字符串和数字两种

```bash
my_var="this is my var"
```

此外要求变量名

- 可由字母数字字符（字母和数字）和下划线字符组成
- 第一个字符必须是一个字母或一个下划线
- 不允许出现空格和标点符号

此外，常量命名常用全大写，变量命名常用全小写

## 读取变量值

```bash
echo $my_var

echo ${my_var}
```

如果此变量没有值或没有被声明过，变量值就是空


展开式仍能使用

![[../020 - 附件文件夹/Pasted image 20230503135224.png]]



## 变量的作用域

默认声明的变量能在当前 shell 进程里用，如果一个脚本声明了一个变量，当前 shell 执行其他脚本时也能获取到这个变量之前赋的值

如果希望变量只能在一个函数里存在，可以声明为局部变量 `local var_name`

如果希望变量不仅能在本 shell 进程里被访问，还能在子进程里被访问，在声明完变量后用 `export var_name`

## 数组类型变量

### 声明和删除方式

```bash
a[1]=foo
echo ${a[1]}

a=("foo1" "foo2")

a=([1]="foo1" [5]="foo5")

```

或用 `declare` 命令声明一个

```bash
declare -a a
```

**删除**一个数组需要 `unset` 命令

```bash
# 删除数组 foo
unset foo

# 删除数组中单个元素
unset 'foo[2]'
```

直接给一个数组赋空值不会清空数组内容

![[../020 - 附件文件夹/Pasted image 20230503135918.png]]

任何引用一个不带下标的数组变量，则指的是数组元素 0：

![[../020 - 附件文件夹/Pasted image 20230503135935.png]]


### 遍历数组

在 for 循环中有两种方式指定数组集合，一种用 `*`，一种用 `@`

```
[me@linuxbox ~]$ animals=("a dog" "a cat" "a fish")  
[me@linuxbox ~]$ for i in ${animals[*]}; do echo $i; done  
a  
dog  
a  
cat  
a  
fish  
[me@linuxbox ~]$ for i in ${animals[@]}; do echo $i; done  
a  
dog  
a  
cat  
a  
fish  
[me@linuxbox ~]$ for i in "${animals[*]}"; do echo $i; done  
a dog a cat a fish  
[me@linuxbox ~]$ for i in "${animals[@]}"; do echo $i; done  
a dog  
a cat  
a fish
```

表示法 `${animals[*]} `和 `${animals[@]}` 的行为是一致的  

直到它们被用引号引起来

### 获取数组长度

```bash
a[100]=foo

# 获取数组中有值的元素个数
echo ${#a[@]}

# 获取某个元素的字符串长度
echo ${#a[100]}
```

### 获取数组元素的值和对应的下标

```bash
# 声明一个数组
foo=([2]=a [4]=b [6]=c)

# 输出元素值
for i in "${foo[@]}"; do echo $i; done

# 输出每个有值的元素在数组中的下标
for i in "${!foo[@]}"; do echo $i; done
```

### 在数组末尾添加元素

```bash
# 声明一个数组
foo=(a b c)
echo ${foo[@]}

# 追加元素
foo+=(d e f)
echo ${foo[@]}
```

### 关联数组 / 哈希表

关联数组优点 kv 哈希表的意思，key 可以是字符串

![[../020 - 附件文件夹/Pasted image 20230503140340.png]]


# 函数

## 函数声明

函数有两种声明方式

```bash
function name {  
	commands  
	return  
}  


name () {  
	commands  
	return  
}
```

为了使函数调用被识别出是 shell 函数，而不是被解释为外部程序的名字，所以在脚本中 shell **函数定义必须出现在函数调用之前**

## 函数调用

```bash
func_name params

$(func_name params)
```

## 函数返回值


`$?` 表示上一个 shell 脚本 / 命令的执行结果，0 为成功，非 0（1~255）为失败

函数可以调用 `exit` 命令或 `return` 命令退出函数并返回执行状态

# 条件控制


## if 的书写格式

```bash
if commands; then  
	commands  
[elif commands; then  
	commands...]
[else  
	commands]  
fi
```


> [!NOTE] `elif` 是 `else if` 的简写，两种用哪种都行



条件列表返回 `true` 还是 `false` 有两种评判标准

- 如果条件列表的 command 是一般的命令，命令返回值为空等价于 `false`，不为空等价于 `true`
- 条件列表如果用 test 语句，就会根据 test 返回的 boolean 判断

shell 提供了两个极其简单的内部命令，它们不做任何事情，除了以一个零或 1 退出状态来终止执行。 `true` 命令总是执行成功，而 `false` 命令总是执行失败

```
[me@linuxbox ~]$ if true; then echo "It's true."; fi  
It's true.  
[me@linuxbox ~]$ if false; then echo "It's true."; fi  
[me@linuxbox ~]$
```

## test 表达式

`test expression` 有两种格式，比较流行的是

```bash
[ expression ]
```

expression 是一个表达式，其执行结果是 true 或者是 false。当表达式为真时，这个  
test 命令返回一个零退出状态，当表达式为假时， test 命令退出状态为 1


> [!NOTE] expression 和左右两边的中括号之间的一个空格是必须的
> 不加会报错



`[ expression ]` 有三大类表达式

- 文件表达式
- 字符串表达式
- 整形表达式

### 文件表达式

- 一个文件：是否存在，是文件还是目录，是否可读可写可执行，是否是块文件，字符文件，符号连接....
- 两个文件：是否相同，file1 创建时间是否早于 file2

![[../020 - 附件文件夹/Pasted image 20230503141729.png|700]]
![[../020 - 附件文件夹/Pasted image 20230503141740.png|700]]



> [!NOTE] 在 test expression 里用变量时，通常加双引号
> ```bash
> if [ -e "$FILE" ]; then
> ...
> fi
> ```
> 引号并不是必需的，但这是为了防范空参数。如果 $FILE 的参数展开是一个空值，就会导致一个错误

### 字符串表达式

![[../020 - 附件文件夹/Pasted image 20230503142105.png|700]]



> [!warning] \>，< ，(  和 ) 表达式操作符必须用引号引起来（或者是用反斜杠转义）
> 如果不这样，它们会被 shell 解释为重定向操作符

### 整形表达式

![[../020 - 附件文件夹/Pasted image 20230503142221.png|650]]![[../020 - 附件文件夹/Pasted image 20230503142242.png|650]]


> [!NOTE] -z 对整形变量也有效
> 如果想检查整形变量是否存在，也能用双引号加 -z 进行检查，比如 `[-z "$INT"]`，因为变量值不存在时会返回空字符串


## 加强版 test - `[[]]`

`test expression` 有时候不够打，于是有了加强版的 test，也是最推荐的方式，语法如下

```bash
[[ expression ]]
```


> [!NOTE] expression 和左右两边的中括号之间的一个空格是必须的
> 不加会报错


`[[ ]]` 命令非常相似于 test 命令（它支持所有的表达式），但是增加了一个重要的新的字符串表达式 - 正则表达式匹配结果检查

### 正则匹配条件判断

```bash\
[[string1 =~ regex]]
```

比如检查变量是否是数字

```bash
if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
	...
fi
```

### == 对路径名展开的支持

`[[ ]]` 添加的另一个功能是 == 操作符支持类型匹配

```bash
FILE=foo.bar
if [[ $FILE == foo.* ]]; then
	...
fi
```

## 加强支持整数条件判断的表达式 - `(())`

```bash
((expression))
```

expression 和小括号之间没有空格

`(( ))` 被用来执行算术真测试。如果算术计算的结果是非零值，则一个算术真测试值为真

在 `(())` 里可以直接用 `>`，`<`，`==` 等符号。之前在 `[]` 和 `[[]]` 里只能用 `-eq`，`-lt` 等


## 逻辑表达式

![[../020 - 附件文件夹/Pasted image 20230503143059.png|600]]


这些操作符不仅可以直接用在 test 里，也能用其短路的特性到多行命令的执行里

比如

```bash
mkdir temp && cd temp
```

只有 `mkdir temp` 执行成功后才会执行下一个命令

再比如

```bash
[ -d temp ] || mkdir temp
```

当 temp 目录不存在时，再执行创建 tmep 目录


# 循环控制

## while 循环和 until 循环

while 循环的格式

```bash
while [ test ]; do  
	...
done
```

此外，还支持 `continut` 直接进入下一次循环，`break` 跳出当前循环


until 循环的格式为

```bash
until [ test ]; do  
	...
done
```

while 和 until 的区别在于

- while，当满足 test 时，继续执行循环
- until，当不满足 test 时，继续执行循环


## 循环读取文件数据行

```bash
while read distro version release; do
	...
done < distros.txt
```

重定向操作符放置到 done 语句之后。循环将使用 read 从重定向文件中读取字段

这个 read 命令读取每个文本行之后，将会退出，其退出状态为零，直到到达文件末尾。到时候，它的退出状态为非零数值，因此终止循环


循环的数据可以来自标准输入，文件，管道

如果循环的 test 用了 read 函数

- 不指定输入源时，默认输入来自标准输入 - 键盘
- 如果读取文件，如上述示例，重定向操作符放到 done 语句之后
- 如果希望数据来自管道，可以这样

```bash
sort -k 1,1 -k 2n distros.txt | while read distro version release; do
	...
done
```


## for 循环

语法格式为

```bash
for variable [in words]; do  
	commands  
done
```


`words` 就是被遍历的集合

举个例子

```bash
for i in A B C D; do echo $i; done

for i in {A..D}; do echo $i; done

for i in distros*.txt; do echo $i; done

for i in $(strings $1); do
```

这个集合可以是

- 直接枚举
- `{}` 表示集合
- 路径展开
- 命令替换
- ...

**如果省略掉 for 命令的可选项 words 部分， for 命令会默认处理位置参数**

for 循环也支持 C 语言风格的语法格式

```bash
for (( expression1; expression2; expression3 )); do  
	commands  
done

# 举个例子
for (( i=0; i<5; i=i+1 )); do  
	echo $i  
done
```

# 流程控制 - case 分支

case 的格式如下

```bash
case $var_name in
	val1)
		...
		;;
	val2)
		...
		;;
	*)
		...
		;;
esac
```

val 可以是

- 整形或字符串字面量，比如 `strVal)`
- 字符集。比如 `\w)`
- 占位符。比如 `??)`，`*)`，`*.txt`

这里的 `*` 不是正则的 `*`，其含义表示任意数量任意个字符

val 中还能有 `|` 或运算符

```bash
#!/bin/bash  
read -p "enter word > "  
case $REPLY in  
	[[:alpha:]]) echo "is a single alphabetic character." ;;  
	[ABC][0-9])  echo "is A, B, or C followed by a digit." ;;  
	???)         echo "is three characters long." ;;
	a|b)         echo "is a or b string"
	*.txt)       echo "is a word ending in '.txt'" ;;  
	*)           echo "is something else." ;;
esac
```

之前上述 case 只能实现匹配到一个条件，匹配到后就不再尝试匹配后面的条件

现在可以把 `;;` 替换为 `;;&`，在匹配上一个条件并执行内容后，还会继续尝试向后匹配

# 参数展开

参数展开指用参数的引用，运行时引用被替换为指定参数，或指定参数集合

## 参数展开

```bash
${xxx}
```

**基本参数展开**

`$a`，`${a}` 能展开一个变量的值，用 `{}` 可以应对分割变量名和其他字面量问题

- `${var:-default}` 如果 var 不存在或为空，返回 default 的字面量
- `${var:default}` 如果 var 不存在或为空，返回 default 的字面量，并赋值给 var
- `${var:+default}` 如果 var 不为空，返回 default 的字面量，但不给 var 赋值

==注意：位置参数或其它的特殊参数不能以这种方式赋值==


**变量名的参数展开**

`${!prefix*}`

`${!prefix@}`

返回以 prefix 开头的已有变量名

**字符串展开 / 函数**

- `${#parameter}` 获取字符串长度
- `${parameter:offset}` 获取第 offset 到 end 的子字符串
- `${parameter:offset:length}` 获取第 offset 到 offset
 \+ length 的子字符串
- `${parameter#pattern}` 掐头，去掉字符串开头匹配正则的子字符串
- `${parameter##pattern}` 掐头，去掉字符串开头匹配正则的子字符串

\# 形式清除最短的匹配结果，而 \#\# 模式清除最长的匹配结果
 
- `${parameter%pattern}` 和 `${parameter%%pattern}` 是去尾

此外还有 replace 等展开

## 算数展开

```
$(( expression ))
```

![[../020 - 附件文件夹/Pasted image 20230503143513.png]]

也支持 `+=`，`-=`，`/=` ，`*=`，`++`，`--` 等运算符，位运算符

![[../020 - 附件文件夹/Pasted image 20230503143525.png|500]]

还支持 `&=`，`|=` 等运算符

支持逻辑运算符

![[../020 - 附件文件夹/Pasted image 20230503143540.png|500]]

# 静态检查工具

https://blog.csdn.net/weixin_46048542/article/details/120626915

ShellCheck

