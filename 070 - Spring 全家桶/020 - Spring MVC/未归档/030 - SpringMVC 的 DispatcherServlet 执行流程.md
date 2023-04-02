#还没有复习 

# Spring MVC做了什么

![[../../../020 - 附件文件夹/Pasted image 20230402121731.png|500]]

# DispatcherServlet 结构分析

![[../../../020 - 附件文件夹/Pasted image 20230402121742.png|275]]

`FrameworkServlet`实现了`HttpServlet`

请求发送过来后，调用`HttpServlet`的`service`方法，`service`方法调用`doXxx`方法

`doXxx`最后都会调用以下方法

```java
processRequest(request, response);
```

在`processRequest`方法的方法体中，重点在于调用了

```java
doService(request, response);
```

此方法是抽象方法，它的实现在`DispatcherServlet`中

在`doService`方法的方法体中，重点在于调用了

```java
doDispatch(request, response);
```



处理请求的核心业务逻辑就在`doDispatch`方法中，接下来将针对这个方法进行分析



问：那`processRequest`和`doService`这两个方法都做了什么？

答：在请求域中执行各种set，get方法添加数据，做一些配置工作

# doDispatch 方法的执行流程

![[../../../020 - 附件文件夹/Pasted image 20230402121804.png|500]]

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    // 请求对象
    HttpServletRequest processedRequest = request;
    // 执行器链（里面有XXXController的对象，目标方法对象，拦截器组成的list）
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    // 处理异步请求的管理器（这里不做重点）
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        // 用于接收执行目标方法返的视图对象
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            // 检查请求是否是文件上传请求，如果是，将请求对象重新包装为文件上传的请求
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // 获取执行器链
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                // 如果找不到接口映射就执行这个方法
                noHandlerFound(processedRequest, response);
                return;
            }

            // 使用执行器链获取到目标方法的适配器（将目标方法封装到Handler中）
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }

            // 执行拦截器的preHandle方法
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // 执行目标方法（其中包含对请求参数的 数据绑定，数据格式化，数据校验）
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            // 如果mv不为空但是其成员变量view为空，就给view赋值，值为此次请求的
            applyDefaultViewName(processedRequest, mv);
            
            // 执行拦截器的postHandle方法
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            ...
        }
        catch (Throwable err) {
            ...
        }
        // 渲染视图并返回视图（执行完此方法后，视图将渲染完毕）
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        ...
    }
    catch (Throwable err) {
       ...
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // 执行拦截器的afterCompletion方法
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```





# getHandler方法

在上述`doDispatch`方法中获取到了执行器链

```java
mappedHandler = getHandler(processedRequest);
```

接下来将针对此方法分析，它是怎么根据`processedRequest`获取到执行器链的



`processedRequest`的成员变量`requestDispatcherPath`的值为此次请求的目标地址。如："/hello"



```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        for (HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            // 如果映射处理器能获取到执行器链，那么直接返回结果
            if (handler != null) {
                return handler;
            }
        }
    }
    return null;
}
```

`this.handlerMappings`中的对象是什么时候有的？

`DispatcherServlet`执行`initStrategies`方法，创建九大组件时创建的



`HandlerMapping`（映射处理器）会根据请求中的地址找到对应的接口并包装成一个`HandlerExecutionChain`（执行器链）

上述方法体中就是在遍历XXXHandlerMapping对象，找到合适的映射处理器后，根据url找到所需要的XXXCotroller的bean实例并将其封装到执行器链中



# getHandlerAdapter方法

在上述`doDispatch`方法中获取到`HandlerAdapter`对象

```java
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```

接下来将针对此方法分析，它是怎么根据`mappedHandler.getHandler()`返回的HandlerMethod对象获取到适配器的



```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
    if (this.handlerAdapters != null) {
        for (HandlerAdapter adapter : this.handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
    }
    throw new ServletException("...");
}
```



# ha.handle方法

在上述`doDispatch`方法中使用获取到的`HandlerAdapter`对象执行目标方法，并把目标方法的返回值封装到`ModelAndView`对象中

```java
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```



在`ha.handle`的方法体中重点在于调用了

```java
return handleInternal(request, response, (HandlerMethod) handler);
```

在`handleInternal`的方法体中重点在于调用了

```java
mav = invokeHandlerMethod(request, response, handlerMethod);
```

在`invokeHandlerMethod`的方法体中重点在于调用了

```java
invocableMethod.invokeAndHandle(webRequest, mavContainer);
```

在`invokeAndHandle`的方法体中重点在于调用了

```java
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
```

这里的`returnValue`是目标方法真正的返回值。这个返回值将被封装到`ModelAndView`对象中



- 上述在`DispatchServlet`中获取`this.handlerMappings`和`this.handlerAdapters`时，里面都是有对象的。这些对象是什么时候被放进去的？

  `DispatchServlet`调用`onRefresh`方法时会调用`initStrategies(context);`方法初始化九大组件

  ```java
  protected void initStrategies(ApplicationContext context) {
      initMultipartResolver(context);
      initLocaleResolver(context);
      initThemeResolver(context);
      initHandlerMappings(context);
      initHandlerAdapters(context);
      initHandlerExceptionResolvers(context);
      initRequestToViewNameTranslator(context);
      initViewResolvers(context);
      initFlashMapManager(context);
  }
  ```

- 在上述`doDispatch`方法的执行流程中，哪个方法最为复杂？

  `ha.handle`执行目标方法最为复杂。这也是为什么执行目标方法不是使用执行器链，而是单独创建了一个适配器类来执行

  执行目标方法首先要获取到`Method`对象，重点是，获取目标方法的参数。

  参数绑定，参数格式化，参数校验这些流程比较复杂





# 绑定接口方法中参数的流程

`XXXXMethodArgumentResolver`



执行目标方法时重点在于用反射获取到目标方法，从请求中获取参数，将参数传入`method`对象。执行方法

重点在于获取参数。因为获取参数的流程有 数据绑定，数据校验，数据格式化三大部分

```java
@Nullable
public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
                               Object... providedArgs) throws Exception {

    // 绑定参数。其中包含了数据绑定，数据校验，数据格式化的所有流程
    Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
    if (logger.isTraceEnabled()) {
        logger.trace("Arguments: " + Arrays.toString(args));
    }
    return doInvoke(args);
}
```



- 数据绑定

  绑定数据。接口方法中使用`@PathVariable`，`@RequestBody`，`@RequestParam` 绑定数据。有时注解上为参数设置了默认值，设置request为true或false等

- 数据校验。如果使用了`@Valid`。程序又会对参数进行校验

- 数据格式化。当使用了类型转换器时，又涉及到数据的类型转换或数据格式化

接下来将针对`Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);`的执行进行分析

```
supportsParameter:73, PathVariableMethodArgumentResolver (org.springframework.web.servlet.mvc.method.annotation)
getArgumentResolver:133, HandlerMethodArgumentResolverComposite (org.springframework.web.method.support)
supportsParameter:102, HandlerMethodArgumentResolverComposite (org.springframework.web.method.support)
getMethodArgumentValues:163, InvocableHandlerMethod (org.springframework.web.method.support)
```



```java
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

    // 记录了所有参数的信息，包括参数的类型，参数使用的注解有哪些
    MethodParameter[] parameters = getMethodParameters();
   	...

    Object[] args = new Object[parameters.length];
    // 遍历所有参数，逐一进行数据绑定，校验，格式化
    for (int i = 0; i < parameters.length; i++) {
        ...
        // 遍历参数解析器，如果没有合适的参数解析器就抛异常
        if (!this.resolvers.supportsParameter(parameter)) {
                throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
        }    
        try {
            // 使用合适的参数解析器解析参数的值
            args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
        }
        catch (Exception ex) {
           ...
        }
    }
    return args;
}
```

上述代码中`this.resolvers.supportsParameter(parameter)`的`this.resolvers`包含了一堆参数解析器

名字都是`XXXMethodArgumentResolver`和`XXXMapMethodArgumentResolver`

比如

`RequestBodyMethodArgumentResolver`，`RequestBodyMapMethodArgumentResolver`

`RequestParamMethodArgumentResolver`，`RequestParamMapMethodArgumentResolver`

`PathVariableMethodArgumentResolver`，`PathVariableMapMethodArgumentResolver`



使用获取到的参数解析器执行以下方法，最终返回参数

```java
// 使用了@RequestParam和@PathVariable的参数执行以下方法解析参数
public final Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

    // 获取当前参数的一些信息，这里的信息包含注解中指定的默认值
    NamedValueInfo namedValueInfo = getNamedValueInfo(parameter);
    MethodParameter nestedParameter = parameter.nestedIfOptional();

    Object resolvedName = resolveStringValue(namedValueInfo.name);
    if (resolvedName == null) {
        ...
    }

    // 获取到参数的值。完成数据绑定，接下来将对参数进行数据校验和数据格式化
    Object arg = resolveName(resolvedName.toString(), nestedParameter, webRequest);
    // 没有传此参数，就将默认值赋给参数
    if (arg == null) {
        if (namedValueInfo.defaultValue != null) {
            arg = resolveStringValue(namedValueInfo.defaultValue);
        }
        else if (namedValueInfo.required && !nestedParameter.isOptional()) {
            handleMissingValue(namedValueInfo.name, nestedParameter, webRequest);
        }
        arg = handleNullValue(namedValueInfo.name, arg, nestedParameter.getNestedParameterType());
    }
    else if ("".equals(arg) && namedValueInfo.defaultValue != null) {
        // 如果参数为空且设置了默认指，就把默认值赋给参数
        arg = resolveStringValue(namedValueInfo.defaultValue);
    }

    // 数据校验和数据格式化
    if (binderFactory != null) {
        // 1. 创建DataBinder对象
        WebDataBinder binder = binderFactory.createBinder(webRequest, null, namedValueInfo.name);
        try {
            // 使用类型转化器进行数据类型转换，格式化等
            arg = binder.convertIfNecessary(arg, parameter.getParameterType(), parameter);
        }
        catch (ConversionNotSupportedException ex) {
            ...
        }
    }

    handleResolvedValue(arg, namedValueInfo.name, parameter, mavContainer, webRequest);

    return arg;
}
```

① Spring MVC 主框架将 ServletRequest  对象及目标方法的入参实例传递给 WebDataBinderFactory 实例，以创建  DataBinder  实例对象

② DataBinder 调用装配在 Spring MVC 上下文中的  ConversionService 组件进行 数据类型转换、数据格式化 工作。将 Servlet 中的请求信息填充到入参对象中

③ 调用 Validator 组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果  BindingData 对象

④ Spring MVC 抽取 BindingResult中的入参对象和校验错误对象，将它们赋给处理方法的响应入参


# 拦截器



创建拦截器

```java
@Configuration
public class InterceptorConfig extends WebMvcConfigurationSupport {
    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/*");
    }
}
```



```java
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("执行拦截器的preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```


# WebMvcConfigurationSupport

```java
public class WebMvcConfigurationSupport implements ApplicationContextAware, ServletContextAware
```

这个类是用来拓展Spring MVC的。

![[../../../020 - 附件文件夹/Pasted image 20230402121821.png|500]]

Spring MVC除了九大组件外还有其他几个重要的组件

- 拦截器

- 静态资源处理器

- 参数解析器（用于将请求种的参数绑定到接口方法的参数列表中）

- 异常处理器

- `Formatters`数据格式化的处理器，和数据类型转化器差不多。这个可以用于将字符串转为预期的日期格式或将日期转为预期的字符串等

- `HttpMessageConverters`作用如下图

![[../../../020 - 附件文件夹/Pasted image 20230402121832.png]]

可自定义这个类的子类，重写这些方法。向springmvc中添加参数解析器，拦截器，转换器，添加静态资源映射等



```java
protected void addResourceHandlers(ResourceHandlerRegistry registry)
```

静态资源处理器。

网关接口有两种。一种映射到程序的接口，由程序接口处理请求后返回处理结果。另一种是映射到静态资源

此方法就是用来做接口与静态资源的绑定，例如

```java
protected void addResourceHandlers(ResourceHandlerRegistry registry) {
    // 访问程序的 /file/xxxx接口时，如果/webapp/xlc/static/upload/image/目录下有xxxx资源就返回静态资源
    registry
        .addResourceHandler("/file/**")
        .addResourceLocations("file:" + "/webapp/xlc/static/upload/image/");
}
```

