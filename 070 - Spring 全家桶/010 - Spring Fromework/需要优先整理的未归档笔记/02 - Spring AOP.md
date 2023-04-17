#还没有复习 

# 注解式开发 AOP

**流程**

1.  导入依赖
2. 定义业务逻辑方法（这里假设要进行 aop 的方法名为" f "）
3. 定义切面类（假设类命名为Aspect）
4. 配置类上添加 `@EnableAspectJAutoProxy`

```xml
<!-- 在 Spring 下用这个依赖 -->
<dependency>
	<groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
</dependency>

<!--在 SpringBoot 下使用这个依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```


### 通知注解

通知方法。该注解下的方法的参数表里都能获取 `JoinPoint` 对象，用于获取目标类执行的方法的信息。不过这个对象必须放在参数表的第一个

```java
@Aspect
@Component
public class Aspect {

	@Pointcut("execution(public void com.test.web.*(..))")  
	public void pointCut(){};  

    @Before(value="pointCut()")
    public void before() {
        ...
    }

    @Around(value="pointCut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        ...
        Object retVal = pjp.proceed();
        ...
        return retVal;
    }

	@After(value="pointCut()")
	public void after(){
		...
	}

	@AfterReturning(value="pointCut()", returning="result")
	public void afterReturning(Object result){
	    ...
	}
	
	@AfterThrowing(value="poitCut()", throwing="exception")
	public void afterThrowing(Exception exception){
	    ...
	}
```



### 公共的切点表达式

凡是符合切点表达式的方法都会被匹配上，并创建 aop 代理对象

公共切点表达式有两种写法
- 字符串表达式。符合 execution 表达式要求的目标都会被创建 aop
- 注解。类或方法带有目标注解时会被认为匹配上表达式

```java
@Pointcut("execution(public void com.imakerlab.web.*(..))") // 这是切点表达式的内容
// @Pointcut("@annotation(com.example.annotation.AOP)") 指定注解
public void pointCut(){}; // 方法名是切点表达式的名字

// 如果希望外部的切点表达式，就需要补全切点表达式方法的全包名
// 比如：@After("xx.xx.xx.pointCut()")
@After("pointCut()")  
public vodi after(){  
    ...  
}
```

切点表达式中
- `*` 表示所有
- `..` 在参数列表中表示同名方法的所有重载方法


- advisor 其内部包含若干个 advice，并决定拦截哪些方法
- advice 其是拦截时负责执行具体实现的组件


# AOP 实现流程
AOP 的执行流程包含 3 部分
- 识别出所有的 Aspect 切面类
- 为需要进行 AOP 的 bean 创建代理对象
- 在调用 bean 的方法时，调用代理对象的方法


## @EnableAspectJAutoProxy 注解的介绍

 `@EnableAspectJAutoProxy` 会向容器中添加一个 `AnnotationAwareAspectJAutoProxyCreator`

此 bean 的继承体系如图

![](https://oscimg.oschina.net/oscnet/up-f18d0274c515709c36a20daf1e410222091.png)

`AnnotationAwareAspectJAutoProxyCreator` 是一个 `BeanPostProcessor` 和 `InstantiationAwareBeanPostProcessor`

这两个接口方法的实现在 `AbstractAutoProxyCreator` 里

- `InstantiationAwareBeanPostProcessor` -> `postProcessBeforeInstantiation` 执行识别出所有的 Aspect 切面类

- `BeanPostProcessor` -> `postProcessAfterInitialization` 为需要进行 AOP 的 bean 创建代理对象




## 识别出所有的 Aspect 切面类

每次创建 bean 时都会调用 `InstantiationAwareBeanPostProcessor` -> `postProcessBeforeInstantiation` 

所以，用缓存方式只进行一次 Aspect 切面类的识别。伪代码如下

```java
if(aspectNames == null){
	buildAspect();
}
return;
```

![[../../91 - 静态资源/Drawing 2022-06-16 11.10.17.excalidraw]]

```java
public List<Advisor> buildAspectJAdvisors() {  
	if (this.aspectBeanNames == null) { // 如果是第一次执行就命中不到缓存
		List<Advisor> advisors = new ArrayList<>();  
        String[] beanNames = beanNamesForTypeIncludingAncestors(); // 获取所有 beanName
		for (String beanName : beanNames) {  
			if (this.advisorFactory.isAspect(beanType)) { // 如果 bean 有 @Aspect 就识别它
				this.aspectBeanNames.add(beanName);  
				List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory); // 识别出通知方法 method/advice 和切入表达式，并封装两者到一个 advisors 里（一个 Aspect 切面类可以有多个 通知方法）
				 this.advisorsCache.put(beanName, classAdvisors);    
				 advisors.addAll(classAdvisors);  
			}
			return advisors;  
		}  
	} 
	List<Advisor> advisors = new ArrayList<>();  
	for (String aspectName : aspectNames) {  
      advisors.addAll(this.advisorsCache.get(aspectName)); 
	}
	return advisors;  
}
```

1. 识别出所有的 Aspect 切面类
2. 识别出 Aspect 切面类中所有的通知方法和方法使用的切面表达式，并为每个通知方法和切入点表达式创建一个 advisor 对象
3. 缓存所有 advisor，以便下次不再重新执行整个过程



> [!NOTE] `@PointCut` 不会被单独识别
> 不单独为 `@PointCut` 修饰的方法创建 advisor 对象，因为切入点表达式作为 advisor 的一部分被封装到每个 advisor 中，而不是被单独作为一个 advisor 去识别


##  为需要进行 AOP 的 bean 创建代理对象

每次创建完 bean 实例后都会调用 `BeanPostProcessor` -> `postProcessAfterInitialization` 

获取到所有的 advisor 后，根据每个 advisor 的切入点表达式判断 bean 是否需要被 aop 增强，如果需要就创建代理对象

![[../../91 - 静态资源/02 - Spring AOP 2022-06-16 11.38.54.excalidraw]]

```java
List<Advisor> candidateAdvisors = findCandidateAdvisors(); // 获取到所有的 advisor
for (Advisor candidate : candidateAdvisors) {  
   if (canApply(candidate, clazz)) { // 判断 bean 是否能匹配每个 advisor 的切入点表达式
      eligibleAdvisors.add(candidate);  
   }  
}
```

判断分两个阶段

```java
canApply(pca.getPointcut(), targetClass, hasIntroductions);
```

```java
public static boolean canApply(Pointcut pc, Class<?> targetClass) {  
	// 第一阶段：bean 是否符合切入点表达式所匹配的类
	if (!pc.getClassFilter().matches(targetClass)) {  
	    return false;  
	}  

	// 第二阶段：如果有任意一个方法符合切入点表达式，说明这个 bean 应该被 advisor 增强 
	MethodMatcher methodMatcher = pc.getMethodMatcher();
	Method[] methods = getAllDeclaredMethods(clazz); // 获取到 bean 的所有方法
    for (Method method : methods) {  
        if (methodMatcher.matches(method, targetClass)) {
	        return true;  
	    }  
    } 
   return false;
}
```


> [!quote] 参考
> [[../../91 - 静态资源/03-Spring-AOP源码.pdf|Spring-AOP 源码分析]]


# 缺少动态代理体系的介绍和代理对象的方法调用方式