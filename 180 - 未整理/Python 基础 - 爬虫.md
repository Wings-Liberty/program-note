> 万事先搜库

## 快速开始

**获取百度首页的html**

```python
import urllib.request

# 设置请求
request = urllib.request.Request("http://www.baidu.com")

# 向url发请求，并获取响应
response = urllib.request.urlopen(request)

# 读取响应
html = response.read()

print(html)
```



## User-Agent



## 正则表达式

Pattern 对象的一些常用方法主要有：

> - match 方法：从起始位置开始查找，一次匹配
> - search 方法：从任何位置开始查找，一次匹配
> - findall 方法：全部匹配，返回列表
> - finditer 方法：全部匹配，返回迭代器
> - split 方法：分割字符串，返回列表
> - sub 方法：替换

这里用pattern的math方法举例

1. 写正则表达式，创建`pattern`对象
2. 使用`pattern`的方法`pattern.match`匹配目标字符串，返回一个`Math`对象
3. 使用返回的`math`对象进行骚操作

```python
import re

# 将正则表达式编译成 Pattern 对象
pattern = re.compile(r'\d+')

m = pattern.match('3ne125twthree34four', 3, 10)
print(m)
print(m.group(0))  # 可省略 0
print(m.start(0))  # 可省略 0
print(m.end(0))  # 可省略 0
print(m.span(0))  # 可省略 0
```

```python
import re

pattern = re.compile(r'([a-z]+) ([a-z]+)', re.I)  # re.I 表示忽略大小写
m = pattern.match('Hello World Wide Web')

print(m)  # 匹配成功，返回一个 Match 对象

print(m.group(0))  # 返回匹配成功的整个子串
print('Hello World')
print(m.span(0))  # 返回匹配成功的整个子串的索引
print(m.group(1))  # 返回第一个分组匹配成功的子串
print(m.span(1))  # 返回第一个分组匹配成功的子串的索引
print(m.group(2))  # 返回第二个分组匹配成功的子串
print(m.span(2))  # 返回第二个分组匹配成功的子串
print(m.groups())  # 等价于 (m.group(1), m.group(2), ...)

# m.group(3)  # 不存在第三个分组
```

pattern的search方法用法和match用法一样，但是前者查匹配字符串是整个字符串匹配的，后者只能从指定位置开始匹配，如果一开始不匹配，将直接报None



## PXml

==教程上用的是pxml，现在貌似用的都是pyquery==

> 用于解决在处理获取到的html/xml时，处理困难的问题

pxml有一套自己的语法，其作用是相对于原始的处理html/xml我方法相比 更方便地获取元素和属性



**使用lxml模块**

可修正不规范的html代码

```python
#利用etree.HTML，将字符串解析为HTML文档
html = etree.HTML(text)

# 按字符串序列化HTML文档
result = etree.tostring(html)

print(result)

# 读取外部文件 hello.html，能将不规范的html自动修复
html = etree.parse('./hello.html')
result = etree.tostring(html, pretty_print=True)

print(result)
```



**获取标签和属性**

```python
result = html.xpath('//li')
```



## Json和JsonPath

```python
import json
```

`json`模块有四个功能`dumps`、`dump`、`loads`、`load`

1. `json.loads()`，json转python
2. `json.dumps()`python转json
3. `json.dump()`python转json并存入文件
4. `json.load`读取文件中的json并转为python



**JsonPath（了解）**

> 用于解析json的工具，就像是lxml和xml的关系
>