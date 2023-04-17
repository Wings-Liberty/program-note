#还没有复习 

1. 重要组件
2. Spring MVC 的执行流程
3. 拦截器
4. 统一异常处理器
5. 如何接收各种参数（json，文件，表单...，日期数据？。可变长度的文件数？？）
6. SpringMVC 的父子容器？？
7. 如何定制 MVC —— WebMvcConfigurationSupport



- [x] 图灵
- [ ] 黑马
- [ ] 尚硅谷




# Spring MVC 处理请求方式

Spring MVC 基于 Servlet 实现，由无状态的 DispatcherServlet 接收并分发请求给用户自定义的处理器

用户自定义的处理器接收参数和返回数据的方式非常灵活多变，所以框架处理请求时在自定义处理器外添加了一层适配器。适配器用于适配各种入参和返回值，其用于解析请求里的参数并绑定到用户自定义处理器方法的参数上，还负责统一封装处理器的返回结果

![Pasted image 20220706133703](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220706133703.png)


| 组件名       | 类名                    | 提供方 | 作用                                         |
| ------------ | ----------------------- | ------ | -------------------------------------------- |
| 前端控制器   | DispatcherServlet       | 框架   | 接收请求，处理响应结果                       |
| 处理器映射器 | HandlerMapping          | 框架   | 根据请求 URL，找到对应的 Handler             |
| 处理器适配器 | HandlerAdapter          | 框架   | 调用处理器的方法                             |
| 处理器       | Handler 又名 Controller | 开发者 | 接收用户请求数据，调用业务方法处理请求       |
| 视图解析器   | ViewResolver            | 框架   | 视图解析，把逻辑视图名称解析成真正的物理视图 |
| 视图         | View                    | 开发者 | 将数据展现给用户                             |


# 三大组件


## 处理器映射器

HandlerMapping 用于保存 key -> Handler 的映射

常用 HttpRequestHandlerMapping 实现保存 url -> Handler 的映射


## 处理器适配器

常用 RequestMappingHandlerAdapter 实现请求，响应实体类和处理器的适配

让参数类型和返回值类型非常灵活的自定义处理器能适配请求，响应的封装


## 视图解析器

处理器的返回结果会先被封装为 ModelAndView，它包含了接口返回的数据和视图

后端常用的视图是 JSON 视图，MappingJackon2JsonView 会把数据模型封装称 Json 格式的数据传输


把 ModelAndView 解析并放到响应体里需要视图解析器，SpringMVC 采用类似 Java 类加载器的委派方式，让视图解析器链多个视图解析器解析，解析成功就返回，解析失败就交给下一个解析器

## 处理器的三种返回结果的处理方式

处理器可以返回 String，void，ModelAndView

如果返回 String，借助视图解析器，可以在指定位置为我们找到指定扩展名的视图。比如 .jsp，.html


> [!NOTE] String 引起的页面跳转方式
> 默认是请求转发，可以通过 "redirect:" 前缀控制其使用重定向


如果返回的是 void，最终也会向客户端返回一个响应。但如果处理器内部对响应做了修改，也能影响响应结果

如果返回的不是 void，适配器最终都会返回一个 ModelAndView 对象。处理器返回 String 时返回结果最终也会被封装到 ModelAndView 对象里

# 处理器绑定请求参数方式

参数传递方式主要有：

- query params（url 拼接参数）
- application/x-www-form-urlencoded
- multipart/form-data
- application/json

处理器参数列表可以根据实际情况，用以下五种参数类型接收请求中的数据

- 普通参数
- POJO 参数
- 嵌套 POJO 参数
- 数组类型参数
- 集合类型参数

Spring MVC 提供了一些注解帮助接收参数

- @RequestParam
- @RequestBody
- @PathVariable


上述内容看似繁多，其实不多


## 按照 Content-Type 分类

- query params（url 拼接参数）
- application/x-www-form-urlencoded
- multipart/form-data

上述 3 种 content-type 传递的参数都可以是简单的 kv，所以可以直接在处理器参数列表声明同名，同类型参数

对于非同名参数和参数是否必选，可以用 @RequestParam 解决

但 @RequestParam 能接收 query parameters, form data, and parts in multipart requests

@RequestBody 能接收 applicaation/json 

## 按照参数类型传递

- 普通参数
- POJO 参数
- 嵌套 POJO 参数
- 数组类型参数
- 集合类型参数

下列参数全部没有 @RequestParam 注解

### 发送基本数据类型

```java
@GetMapping("/simple/raw")  
public String testRaw(int i, String s, boolean b) {  
    String res = "int i = " + i + "; String s = " + s + "; boolean b = " + b;  
    System.out.println(res);  
    return res;  
}
```

![Pasted image 20220706214317](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220706214317.png)

无论是 GET 还是 POST

当传参方式是 query param、application/x-www-form-urlencoded 或 multipart/form-data 时结果都是下述所示

- 如果没有为 i 传参，服务器报错
- 如果没有为 b 传参，b 默认值为 false
- 如果没有为 s 传参，s 默认值为 null

> [!warning] 后端接口参数最好用包装类型，不用基本数据类型
> 因为有些基本数据类型参数没有收到值时会报错，而不是被设置为零值
> 
> 如果参数时包装类型，当参数没有收到值时，默认值为 null


> [!warning] 当两种方式都用时，multipart/form-data 会覆盖 query param
> multipart/form-data 里的 kv 会覆盖 query param 的 kv，但如果 form-data 不会用空值覆盖自己没有传递的 kv


> [!error] Postman 无法正确发送 GET 下的 application/x-www-form-urlencoded
> Postman 会把数据放到请求体里，而不会数据拼接到 url 里，但浏览器会。这就导致 Postman 发送 GET 请求，application/x-www-form-urlencoded 格式时，数据被放到请求体里。因为规范规定 GET 请求没有请求体，所以服务器不会主动从 GET  请求的请求体里获取数据，所以服务器获取不到 Postman 发的数据，但能从浏览器那获取到

当传参方式是 application/x-www-form-urlencoded 时，如果处理器方法接口参数没有 @RequestParam 注解，后端就收不到数据


### POJO 类型

```java
@PostMapping("/pojo/user")  
public User testUser2(User user) {  
    return user;  
}
```

以 query param，application/x-www-form-urlencoded 或 multipart/form-data 方式传递的参数中，同名参数会被封装到对象里

![Pasted image 20220707113916](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220707113916.png)

而接口参数在没有 @RequestBody 是，是收不到以 application/json 方式传来的数据的

POJO 类型传参的工作原理是：为参数列表中的 POJO 创建实例，然后把请求中的参数 set 到 POJO 对象里。所以即使没有为参数列表的 POJO 对象传递任何参数，对象也会被创建，只是它的成员变量都是空值而已（如果构造方法没有赋初值的话）

### 嵌套 POJO 参数

```java
@PostMapping("/pojo/userwithcar")  
public UserWithCar testUserWithCar2(UserWithCar user) {  
    return user;  
}
```

嵌套 POJO 参数：请求参数名与形参对象属性名相同，按照对象层次结构关系即可接收嵌套 POJO 属性参数

![Pasted image 20220707115103](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220707115103.png)

PS：以 query param，application/x-www-form-urlencoded 或 multipart/form-data 方式都可以

### 基本数据类型的数组类型传参

```java
@PostMapping("/array/str")  
public String testArrayStr(String[] strs){  
    return Arrays.toString(strs);  
}
```

以 query param，application/x-www-form-urlencoded 或 multipart/form-data 方式时，请求里的参数名相同时，参数就能被封装到

![Pasted image 20220707122927](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220707122927.png)


> [!NOTE] 不支持 POJO 数组传参
> query param，application/x-www-form-urlencoded 或 multipart/form-data 方式下不支持 POJO 数组传参。想传必须用 Json

### 集合类型

```java
@PostMapping("/list/str")  
public String testListStr(@RequestParam List<String> strs){  
    System.out.println(strs.getClass());  
    return Arrays.toString(strs.toArray());  
}
```

和 POJO 类型相同，SpringMVC 会先把 strs 当作 POJO，并调用 List 的构造方法

结果 List 是接口，没有构造方法。所以需要为其加上 @RequestParam 注解，集合的默认实现类是 ArrayList

## 接受 Json 参数和返回 Json 响应

前后端常用 json 方式传参：前端把 json 格式的数据放到请求体里，后端把 json 格式的数据放到响应体里

现在我们只关心后端如何获取请求体里的 json 以及如何把处理器返回值变为 json 后放到响应体里

```java
@PostMapping("/json/user") 
@ResponseBody
public User testJsonUser(@RequestBody User user){  
    return user;  
}
```

- @RequestBody 表示从请求体里取数据

- @ResponseBody 表示把处理器方法的返回值放到响应体里

如果没有 @RequestBody，处理器就不会从请求体里**取 json 数组**（但还能绑定 query param 和两种表单格式的请求数据）

如果有 @RequestBody 但请求体里的参数不是 json，比如是两种表单格式，就会抛错，并返回 415 Unsupported Media Type

@RequestBody + Json 方式还能接受对象数组和集合

```java
@PostMapping("/json/list/user")  
public List<User> testJsonUser2(@RequestBody List<User> users){  
    return users;  
}  
  
@PostMapping("/json/array/user")  
public String testJsonUser3(@RequestBody User[] users){  
    return Arrays.toString(users);  
}
```

这两个接口均能接受以下 json 数据

```json
[
    {
        "name": "12",
        "age": 12
    },
    {
        "name": "22",
        "age": 22
    }
]
```


> [!NOTE] Title
> 纯 Spring-MVC 中处理 json 时需要加入 com.fasterxml.jackson.core 的 jackson-databind 依赖


## 上传文件和 Multipart 请求

HTTP 对于上传文件的方式参考[这里](https://zhuanlan.zhihu.com/p/120834588)

如果不想看，就看这里的总结：multipart/form-data 最适合作文件上传

```java
@PostMapping("/file/one")  
public String testFile(MultipartFile file){
	// 获取文件名 - filename，请求中文件的 key - name，文件类型
    System.out.println("originalFilename = " + file.getOriginalFilename());  
    System.out.println("name = " + file.getName());  
    System.out.println("contentType = "+ file.getContentType());  
    return file.getContentType();  
}
```

SpringMVC 会把文件上传接口视为特殊的接口，只有 multipart request 才能请求这个接口

如果请求没有携带文件，那么 SpringMVC 会抛异常，认为这个请求不是 multipart request，向客户端返回 500

如果请求携带了文件，但接口方法参数上有些文件对象没有绑定到文件，就无法执行处理器（而不是把文件对象设为 null 或仅创建空对象后继续执行处理器方法），向客户端返回 400


能用数组或集合接收多个文件

```java
@PostMapping("/file/array")  
public String testFile2(MultipartFile[] files){  
    for (MultipartFile file : files) {
	    System.out.println("originalFilename = " + file.getOriginalFilename());
    }  
    return "success";  
}  
  
@PostMapping("/file/list")  
public String testFile3(@RequestParam List<MultipartFile> files){  
    for (MultipartFile file : files) {  
        System.out.println("originalFilename = " + file.getOriginalFilename());
    }  
    return "success";  
}
```

![Pasted image 20220707141010](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220707141010.png)

发送的请求如下

![Pasted image 20220707141213](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220707141213.png)

## 类型转换器 - Converter

请求里的数据之所以能被取出来，并转换为处理器方法参数列表上同名变量的指定类型，是因为有合适的类型转换器

如果请求中的参数 key1 对应的值想要转为 ClassA 就需要有相应的类型转换器

实现一个 Converter 接口并直接加到容器里即可生效

Converter 要求，如果转换器不能转换参数到指定类型就返回 null，以便下一个转换器继续尝试工作



## 日期参数

前端发送的日期参数格式有很多种，时间戳，字符串

字符串表示日期的方式又有很多种：yyyy/MM/dd，MM/yyyy/dd，yyyy-MM-dd ...

假设请求中日期是字符串格式

Java 中的日期类型又有很多种：Date，LocalDate

如果是 Date，可以用 @DateTimeFormat 注解自定义请求参数中的日期格式

```java
@GetMapping("/date/format")
public Date testDate2(@DateTimeFormat(pattern = "yyyy-MM-dd") Date date){  
	// SpringMVC 默认能解析请求参数里 yyyy/MM/dd 格式的字符串
	// 但对于其他格式的字符串就需要注解的帮助
    System.out.println(date);  
    return date;  
}
```

如果传入 2022-07-08 或 2022-7-8 都能被转换为 Date

但 Spring MVC 没有内置 String -> LocalDate，LocalTime，LocalDateTime 的转换器

所以需要自行实现 Converter 接口

```java
@Configuration  
public class DateformatConfig {  
  
    @Bean  
    public Converter<String, LocalDateTime> localDateTimeConverter() {  
        // 使用 lambda 表达式有问题，暂未解决  
        return new Converter<String, LocalDateTime>() {  
            @Override  
            public LocalDateTime convert(String s) {  
                return StringUtils.isEmpty(s) ? null : LocalDateTime.parse(s, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));  
               }  
        };  
    }  
  
    @Bean  
    public Converter<String, LocalDate> localDateConverter() {  
        return new Converter<String, LocalDate>() {  
            @Override  
            public LocalDate convert(String s) {  
                return StringUtils.isEmpty(s) ? null : LocalDate.parse(s, DateTimeFormatter.ofPattern("yyyy-MM-dd"));  
            }  
        };  
    }  
  
    @Bean  
    public Converter<String, LocalTime> localTimeConverter() {  
        return new Converter<String, LocalTime>() {  
            @Override  
            public LocalTime convert(String s) {  
                return StringUtils.isEmpty(s) ? null : LocalTime.parse(s, DateTimeFormatter.ofPattern("HH:mm:ss"));  
            }  
        };  
    }  
}
```


> [!warning] 自行实现的转换器会严格遵守正则表达式
> 比如 2022-7-7 会被认为不是 yyyy-MM-dd 格式，所以需要注意下

参考[这里](https://juejin.cn/post/6970363896460738590)和[这里](https://blog.csdn.net/SIMBA1949/article/details/103163794)




> [!NOTE] 注意:SpringMVC的配置类把@EnableWebMvc当做标配配置上去，不要省略


处理器方法列表中

get  Query_Params 中中文编码有问题，修改 tomcat 配置
post  x-www-form-urlencoded 中中文编码有问题，添加字符编码过滤器

CharacterEncodingFilter

# 支持 ant 风格路径和占位符路径

Ant 风格资源地址支持 3 种匹配符

- ?：匹配文件名中的一个字符

- \*：匹配文件名中的任意字符

- \*\* 匹配多层路径

```java
@RequestMapping(value="/api/user/*/info")
```

占位符路径

```java
@RequestMapping(value="/api/user/{userId}")
public void test(@PathVariable Integer userId){
}
```

# 处理器方法获取上下文信息

处理器方法除用于接收请求，还能绑定请求和响应的元数据，Spring 的一些上下文对象（比如 Spring Seurity 的一些内置对象）


## 获取请求头 - @RequestHeader

```java
@RequestMapping(value="/header")
public String test(@RequestHeader(value="Accept-Language") String al){
	System.out.println("testRequestHeader - Accept-Language："+al);
	return "success";
}
```

## 获取请求里的 Cookie - @CookieValue

```java
@RequestMapping("/cookie")
public String test(@CookieValue("JSESSIONID") String sessionId) {
	System.out.println("testCookieValue: sessionId: " + sessionId);
	return "success";
}
```


## 获取请求，响应和会话实体

在处理器方法参数列表上声明这些类型的对象，SpringMVC 就能为其绑定到这些对象

HttpServletRequest，HttpServletResponse，HttpSession


## 获取上下文

比如获取[[04 - Spring MVC#数据绑定的架构设计|数据绑定结果 BindingResult]]


> [!warning] BindingResult 对象只能在 `@ModelAttribute`，`@Valid`，`@RequestBody`或者`@RequestPart` 注解修饰的参数后声明
> ```java
> public String validRange(@Range(min = 1, max = 10, message = "越界了") Integer num, BindingResult result)
> ```
> 像这样写，启动项目时就会报错，因为 num 没有被上述 3 种注解修饰



# 拦截器和统一异常处理器

## DispatcherServlet 内部的拦截器

拦截器在 DispatcherServlet 里执行，而过滤器是在执行 Servlet 前后执行

为 DispatcherServlet 添加过滤器只需要把 HandlerInterceptor 的实现类加到容器里即可

拦截器的执行时机为

- preHandle：执行处理器前

- postHandle：执行处理器后

- afterCompletion：DispatcherServlet 处理完请求后被调用

## 拦截器链执行顺序

拦截器链的实现方式类似装饰器 + 拦截器链，所以 preHandle 的按顺序执行，postHandle 和 afterCompletion 按逆序执行


> [!help] SpringBoot 下如何控制拦截器在链中的位置
> 除了直接把拦截器直接加到容器中外，怎么指定它在拦截器链中的位置呢？



## 统一异常处理器和异常处理器

处理器执行时抛出异常时，HandlerExceptionResolver 会为其寻找一个最近的异常处理器

异常处理器是被 @ExceptionHandler 标注的方法，方法可以写在控制器里，或统一异常处理器里

异常处理器需要在 @ExceptionHandler#value 上标注会响应哪些异常

```java
@ExceptionHandler( value={ ArithmeticException.class } )
```

统一异常处理器是被 @ControllerAdvice 标注的类，里面的 @ExceptionHandler 方法会被解析为异常处理器

当处理器抛出异常时，HandlerExceptionResolver 会现在控制器里找异常处理器，如果找不到再从统一异常处理器里找。寻找异常处理器时，会找最符合当前异常的那个，如果没有，就找其父类

异常处理器的参数可以是异常本身，返回值可以是 ModelAndView，也可以是 void


> [!NOTE] @RestControllerAdvice = @ControllerAdvice + @ResponseBody


## 内置的异常处理解析器

HandlerExceptionResolver 就是异常处理解析器的顶级接口

它的作用是：内置一些异常处理器，接受用户自定义的异常处理器，异常处理器处理完异常后根据解析器的特性向响应中加入一些信息

举个例子，有些请求有问题会导致后端出现异常，当出现异常时

后端控制台有时会直接抛出异常，有时会只打印一些警告或错误日志，有时会没有任何输出

此外，前端有时会收到 500，有时会收到 400 等各种错误响应码

这就是异常处理解析器的工作，有些特定的异常解析器会为响应加上合适的响应码和各种响应头，有些异常解析器遇到异常后会寻找有正确响应的 “错误页面”，而不是返回错误响应

### 处理自定义异常处理器 - ExceptionHandlerExceptionResolver

负责注册，管理控制器里 @ExceptionHandler 标注的和 @ControllerAdvice 统一异常处理器中 @ExceptionHandler 标注的异常处理器

### 配置合适的错误响应码 - ResponseStatusExceptionResolver

当处理器抛出了异常 ex，且 ExceptionHandlerExceptionResolver 没有处理 ex，且 ex 或其父类被 @ResponseStatus 标注了

那就又 ResponseStatusExceptionResolver 处理，并把 @ResponseStatus#value 声明的响应码放到响应里

框架里并没有用 @ResponseStatus 标注的异常，这个解析器常用于为那些用户自定义的异常设置合适的响应码


> [!NOTE] @ResponseStatus 也可以标注在异常处理器方法上
> 这样就不需要再创建自定义异常了


### 为常见异常填充错误响应 - DefaultHandlerExceptionResolver

举几个简单的例子

- NoSuchRequestHandlingMethodException
- HttpRequestMethodNotSupportedException
- HttpMediaTypeNotSupportedException
- HttpMediaTypeNotAcceptableException
- MissingPathVariableException

这个解析器会为这些常见设置合适的错误响应码和响应头，这些错误通常都是由客户端传参或请求异常导致的，所以解析器没必要再把异常向外抛，由解析器内部消化即可


### 为异常设置逻辑视图 - SimpleMappingExceptionResolver

遇到异常时，尝试返回逻辑视图，逻辑视图最终又被解析为物理视图。此解析器常用于前后端不分离时，返回错误页面时使用

## 异常处理器实现方式和执行时机

异常处理器在控制器的执行器链之外

异常处理也是对响应结果的处理，所以异常处理器是在拦截器的 postHandler 执行之后，在 拦截器的 afterCompletion 执行之前执行


> [!NOTE] 异常处理是在 DispatcherServlet 里被调用的
> 具体来说是在 processDispatchResult 方法里被调用的

## 404 异常的特殊处理

SpringMVC 默认 404 是不走统一异常处理器的，所以遇到 404 时不能在统一异常处理器中定制返回值

Spring MVC 处理 404 的流程可以[参考这里](https://www.jb51.net/article/200169.htm)




# 数据绑定流程

数据的绑定流程指请求中的数据被解析再被赋值给处理器方法参数的过程

请求里的数据被绑定到处理器的方法上需要做这几件事

- 类型转换：请求里的数据转为指定类型，比如 String 类型的数据要被赋值给 Ingeger 前就得做类型转换

- 数据格式化：比如把字符串类型的日期格式化后，再转换到 Date 对象了里

- 数据校验：如果处理器方法的参数上有正则校验，空值校验等校验注解，就需要对数据进行校验

其中数据的格式化本质上也是类型转换，比如 String -> Date

## 数据绑定的架构设计

所以数据绑定的整个过程如下

![Pasted image 20220707224322](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220707224322.png)

①  Spring MVC 主框架将 ServletRequest 对象及目标方法的入参实例传递给 WebDataBinderFactory 实例，以创建 **DataBinder** 实例对象

②  DataBinder 调用装配在 Spring MVC 上下文中的 **ConversionService** 组件进行**数据类型转换、数据格式化**工作。将 Servlet 中的请求信息填充到入参对象中

③  调用 **Validator** 组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果 **BindingData** 对象

④  Spring MVC 抽取 **BindingResult** 中的入参对象和校验错误对象，将它们赋给处理方法的响应入参

如果上述中某个步骤出现错误，异常处理器会就接收，比如 DefaultHandlerExceptionResolver 获取 BindingResult 后把那些参数在类型转换或校验过程中的异常原因放到响应体里

Spring MVC 通过反射机制对目标处理方法进行解析，将请求消息绑定到处理方法的入参中。数据绑定的核心部件是 **DataBinder**


## 类型转换

Spring 提供了 3 种实现类型转换器的方式

- Converter\<S, T\>：将 S 类型对象转为 T 类型对象

- ConverterFactory：将相同系列多个 “同质” Converter 封装在一起。如果希望将一种类型的对象转换为另一种类型及其子类的对象（例如将 String 转换为 Number 及 Number 子类（Integer、Long、Double 等）对象）可使用该转换器工厂类

- GenericConverter：会根据源类对象及目标类对象所在的宿主类中的上下文信息进行类型转换

Converter 在[[04 - Spring MVC#类型转换器 - Converter|这里]]说过


> [!NOTE] 数据格式化的逻辑在类型转换器中实现
> 比如自定义 String -> LocalDate，LocalTime，LocalDateTime。实现可以参考[[04 - Spring MVC#日期参数|这里]]

对属性对象的输入/输出进行格式化，从其本质上讲依然属于 “类型转换” 的范畴。

Spring 在格式化模块中定义了一个实现 ConversionService 接口的

**FormattingConversionService** 实现类，该实现类扩展了 GenericConversionService，因此它**既具有类型转换的功能，又具有格式化的功能**

FormattingConversionService 拥有一个 **FormattingConversionServiceFactroyBean** 工厂类，后者用于在 Spring 上下文中构造前者，FormattingConversionServiceFactroyBean 内部已经注册了 :

- NumberFormatAnnotationFormatterFactroy：支持对数字类型的属性使用 **@NumberFormat** 注解

- JodaDateTimeFormatAnnotationFormatterFactroy：支持对日期类型的属性使用 **@DateTimeFormat** 注解



## 数据校验

### 校验注解 quick start

要求处理器接收的参数必须符合非空，正则等一些规范时，SpringMVC 提供了校验器校验参数是否合法的框架

SpringMVC 用 LocalValidatorFactoryBean 创建的校验器校验参数信息， LocalValidatorFactoryBean 同时支持 JSR303 提供的数据校验注解和 Hibernate Validator 提供的扩展校验注解

开启 @EnableMVC 后，自动配置类会向容器中添加 LocalValidatorFactoryBean，由它来完成数据校验

然后在实体类指定校验方式，在处理器方法参数列表上标明哪些参数需要被校验

```java
@Data // lombok
public class RegisterUserVo {
	@NotEmpty
	String username;

	@NotEmpty
	String password;
	
	@Range(min = 5, max = 10)  
	Integer age;
}
```

```java
@PostMapping("/user/register")
public String register(@Valid RegisterUserVo user) {
	System.out.println(user);
	return "success";
}
```

### 基础类型参数的校验和复杂类的校验

校验注解可以直接用在参数列表上

```java
public String validRange(@Range(min = 1, max = 10, message = "越界了") Integer num)
```

此外还能把校验注解直接加在 String，Date 等类型的类上


> [!NOTE] 也能直接对 @PathVariable 标注的基础类型用校验注解


如果参数是复杂的 Java Bean 就必须加 @Valid 或 @Validated 注解

```java
public String postRegister(@Validated(UserVO.ADD.class) UserVO user);

public String postRegister(@Valid RegisterUserVo user);
```


> [!NOTE] 如果 @Validated 想要校验对象上所有指定了任意分组
> 让所有分组接口都实现一个顶级接口 Default，再用 @Validated(Default.class) 即可

常见的校验注解有

| 注解         | 作用                                                     |
| ------------ | -------------------------------------------------------- |
| @Null        | 被注释的元素必须为 null                                  |
| @NotNull     | 被注释的元素必须不为 null                                |
| @AssertTrue  | 被注释的元素必须为 true                                  |
| @AssertFalse | 被注释的元素必须为 false                                 |
| @Min         | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @Max         | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin  | 被注释的元素必须是个数字,其值必须大于等于指定的最小值    |
| @DecimalMax  | 被注释的元素必须是个数字，其值必须个子等手指定的最大值   |
| @Size        | 被注释的元素的大小必须在指定的范围内                     |
| @Digits      | 被注释的元素必须是，其值必须在可接受的范围内             |
| @Past        | 被注释的完素必须是过去的时间                             |
| @Future      | 被注释的元素必须是一个将来的日期                         |
| @Pattern     | 被注释的元素必须符合指定的正则表达式                     |
| @Email       | 被注释的元素必须是电子邮箱地址                           |
| @Length      | 被注释的字符串的大小必须在指定的范围内                   |
| @NotEmpty    | 被注释的字符串的必须非空                                 |
| @Range       | 被注释的元素必须在合适的范围内                           | 


### 分组校验

有些 VO 会在多个接口里作为参数，在注册接口里，希望 UserVO 的用户名密码不为空且符合正则；在修改用户信息接口里，希望 UserVO 的邮箱不为空

分组校验就用于这种一个 VO 在不同接口里需要遵守不同校验规则的情况

```java
@Data  
public class RegisterUserVo {  
  
    @NotEmpty(groups = {GetGroup.class})  
    String username;  
  
    @NotEmpty(groups = {GetGroup.class})  
    String password;  
  
    @Range(min = 1, max = 4, groups = {GetGroup.class})  
    @Range(min = 5, max = 10, groups = {PostGroup.class})  
    Integer age;  

	interface PostGroup{}

	interface GetGroup{}
}
```

@Validated 支持分组校验，@Valid 不支持，所以不能用 @Valid

```java
@PostMapping("/user/register")  
public String postRegister(@Validated(RegisterUserVo.PostGroup.class) RegisterUserVo user) {  
    System.out.println(user);  
    return "success";  
}  
  
@GetMapping("/user/register")  
public String getRegister(@Validated(RegisterUserVo.GetGroup.class) RegisterUserVo user) {  
    System.out.println(user);  
    return "success";  
}
```

@Validated 是 Spring 的注解，它只会对实体类中属于特定分组的校验注解生效。如果实体类的成员没有用校验注解指定分组或分组和 @Validated 指定的分组不一致，校验注解将不会生效 


> [!NOTE] groups 里的类对象只能是接口，不能是普通类或抽象类
> 通常这些接口可以被声明在实体类里，这样便于管理


### 嵌套验证

```java
@Data
public class User {

    @NotNull(groups = {Save.class, Update.class})
    @Length(min = 2, max = 10, groups = {Save.class, Update.class})
    private String userName;

    @NotNull(groups = {Save.class, Update.class})
    @Length(min = 6, max = 20, groups = {Save.class, Update.class})
    private String password;

    /**
     * 此时DTO类的对应字段必须标记@Valid注解
     */
    @Valid
    @NotNull(groups = {Save.class, Update.class})
    private Job job;

    @Data
    public static class Job {
        @NotNull(groups = {Save.class, Update.class})
        @Length(min = 2, max = 10, groups = {Save.class, Update.class})
        private String jobName;
    }

    // 分组校验接口声明
    public interface Save {}
    public interface Update {}

}
```

@Validated 不支持嵌套校验，@Valid 支持，所以希望被校验的成员变量只能加 @Valid，但接口的参数列表上用 @Validated 或 @Valid 都可

且如果在接口参数列表里用 @Validated，校验嵌套对象时，分组校验仍能生效

```java
    @PostMapping
    public Object saveUser(@RequestBody @Validated(User.Save.class) User user) {
        // 校验通过，才会执行业务逻辑处理
        return "ok";
    }
```

参考自 [Validated数据校验，看这一篇就够了](https://localhost80.blog.csdn.net/article/details/112974137)

> [!warning] 对嵌套成员要用 @NotNull，不要用 @NotEmpty
> 以上述代码为例，@NotEmpty 的校验时机是在创建 Job 对象后，为 Job 对象的成员赋值前。所以校验器得知 user 的 job 不为 null，但 job 的成员均为空，所以校验不通过


> [!note] 如果嵌套成员是 list 或数组，还能用 @Valid
> ```java
> @Data  
> public class UserWithCars {  
>
>  @Length(min = 1, max = 20)  
>  private String name;  
>
>  @Size(max = 2)  
>  @Valid  
>  private List\<Car\> cars;  
>
> }
> ```


### 校验集合或数组

@Validated 不能接口参数列表上的校验集合或数组元素，@Valid 能

```java
@RestController  
@Validated  
public class TestController {

	@PostMapping("/user/list/valid")   
	public String getUserList(@RequestBody List<@Valid RegisterUserVo> users) {  
	    System.out.println(users);  
	    return "success";  
	}  
	  
	@PostMapping("/user/array/valid")  
	public String postUserArray(@RequestBody @Valid RegisterUserVo[] users) {  
	    System.out.println(Arrays.toString(users));  
	    return "success";  
	}
}
```

需要先用 @Validated 注解类（不知道是什么意思），再用 @Valid 注解数组或集合泛型变量上

参考自[Spring Validation 校验集合新方法，竟然在泛型参数里加注解？？](https://juejin.cn/post/7044444094948458533)

### 自定义校验注解

如果 JSR 303 提供的校验注解不能满足业务，可以自定义校验注解

比如期望某个参数的取值要在枚举值里

先大致了解下 SpringMVC 的注解校验处理流程：校验器里被注册了一些注解，校验参数时，如果注解被注册到了某些校验器里，就用这些校验器校验

因此，自定义校验注解需要做这些事

- 自定义校验注解

- 自定义校验器

- 把校验注解注册到校验器里（不用把校验器放容器里）

```java
/**  
 * 校验注解，作用在 String 类型的对象上 </br>  
 * 作用是让对象的取值仅限于指定枚举值  
 */  
@Retention(RetentionPolicy.RUNTIME)  
@Constraint(validatedBy = EnumValidator.class)  
@Target({ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.PARAMETER})  
public @interface EnumValue {  
  
    String[] values() default {};  
  
    String message() default "只能在指定的枚举值里取值";  
  
    /**  
     * 作为校验注解，下面的这两个属性是必须的。如果没有这些属性就会报错，比如： Enums contains Constraint annotation, but does not contain a payload parameter.  
     */    Class<?>[] groups() default { };  
    Class<? extends Payload>[] payload() default { };  
}
```

```java
public class EnumValidator implements ConstraintValidator<EnumValue, String> {  
  
    private final HashSet<String> set = new HashSet<>();  
  
    @Override  
    public void initialize(EnumValue constraintAnnotation) {  
        set.addAll(Arrays.asList(constraintAnnotation.values()));  
    }  
  
    @Override  
    public boolean isValid(String value, ConstraintValidatorContext context) {  
        System.out.println("校验器获取到请求参数 : " + value);  
        return set.contains(value);  
    }  
}
```

可以直接用校验注解，也可以用 @Valid 或 @Validated

```java
@PostMapping("/valid/pojo/modeltype")  
public String myValidator(@Valid ModelType modelType){  
    System.out.println(modelType);  
    return "success";  
}  
  
@PostMapping("/valid/simple/modeltype")  
public String simpleModelType(@EnumValue(values = {"client", "transfer"}) String value){  
    System.out.println(value);  
    return "success";  
}
```

当数据校验未通过时

- 第一种方式下，默认返回 400 响应
- 第二种方式下，默认返回 500 响应



# SpringMVC 的详细处理流程

调试，数据绑定，处理处理，异常处理

1. Spring MVC 提供了接收请求，处理请求，返回响应的一整套逻辑和处理这些逻辑
的组件。其核心在 DispatcherServlet 执行逻辑业务，WebMvcConfigurer 提供解析请求，绑定参数，创建响应等内部行为
2. 上述的组件大多都是可自定义的，WebMvcConfigurer 提供了配置这些组件到上述流程中的入口

@EnableMVC 注解就是向容器添加了一个 WebMvcConfigurer 实现类

直接向容器中加入一个 DispatcherServlet 或像 SpringBoot那样用 AbstractDispatcherServletInitializer 创建一个 DispatcherServlet 和 子容器



# 父子容器

参考[父子容器](https://segmentfault.com/a/1190000039761203)

[父子容器](https://zhuanlan.zhihu.com/p/63212113)

