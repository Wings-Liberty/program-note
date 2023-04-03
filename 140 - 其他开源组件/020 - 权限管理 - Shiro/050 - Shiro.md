#还没有复习 

1. 执行流程，SecurityManager，Realm，Subject
2. 内置过滤器都干了什么事，都有哪些内置过滤器，如何自定义过滤器
3. 过滤器添加和执行流程（ShiroFilterFactoryBean如何根据url定制的过滤器链，过滤器链又是如何拦截请求的，先执行Shiro的过滤器后执行Spring中其他的过滤器）


# 入门

## 主要作用

---

> 用于认证，授权，加密，会话管理，web集成，缓存

## 官网

[官网]( https://shiro.apache.org/ )

## 核心类

- Subject  代表当前user
- ShiroSecurityManager 管理所有前者
- Realm 和数据打交道


```java
// 快速开始的核心代码
Subject currentUser = SecurityUtils.getSubject();
Session session = currentUser.getSession();
currentUser.isAuthenticated();
currentUser.hasRole("schwartz");
currentUser.isPermitted("lightsaber:wield");
currentUser.logout();
```


## 使用

### 注册三大对象

```java
@Configuration
public class ShiroConfig {

    /*
    三大核心对象
     */

    // ShiroFilterFactoryBean
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("manager") DefaultWebSecurityManager manager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        //关联manager
        bean.setSecurityManager(manager);
        return bean;
    }

    // DefaultWebSecurityManager
    @Bean(name = "manager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("getMyRealm") MyRealm myrealm){
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        //关联realm 
        manager.setRealm(myrealm);
        return manager;
    }

    // Realm
    @Bean
    public MyRealm getMyRealm(){
        return new MyRealm();
    }

}
```


### 添加拦截器

```java
//ShiroFilterFactoryBean
@Bean
public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("manager") DefaultWebSecurityManager manager){
	ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
	//关联manager
	bean.setSecurityManager(manager);

	/*
	拦截属性
	anon  无需认证
	authc 认证才嫩访问
	user  开启了 记住我 功能后才能访问
	perms 有某个资源权限后才能访问
	role  有某个角色权限后才能访问
	 */
	
	/*
	添加过滤器
	添加拦截
			filterMap.put("/user/add", "authc");
			filterMap.put("/user/update", "authc");
	 */
	Map<String, String> filterMap = new LinkedHashMap<>();
	filterMap.put("/user/*", "authc");

	bean.setFilterChainDefinitionMap(filterMap);

	//设置被拦截后跳转到指定的url
	bean.setLoginUrl("/login");

	return bean;
}
```


## 案例

登陆案例（使用认证技术）

> 用户角度
>
> - 未登陆就进入，跳转到指定页面
> - 登陆，成功或失败，成功后跳进指定界面，失败后携带错误提示跳转到登陆页

> 后端角度
>
> - 设置拦截器，没有权限的人无法进入add/update
>
> - 被拦截后跳入登陆页面
> - controller接收username和password
> - 在controller中获取当前用户和令牌
> - 使用用户的登陆方法，令牌作为参数
>   - 在登陆方法中执行认证方法
>     - 在认证方法中获取数据库信息，和传来的令牌信息进行校验


# 开始


## 核心类

`Subject` 代表一个用户

`ShiroSecurityManager` Shiro的安全管理器，管理过滤器，管理认证和授权的流程

`Realm`  从db中查询用户真实数据并封装到`Realm`对象中，对`Subject`进行认证和授权时就使用此对象进行密码比对，鉴权等


## 实现流程

具体代码在  Boot-Shiro  里

Spring整合Shiro
1. 在web.xml中配置class=org.springframework.web.filter.DelegatingFilterProxy，filter-name=shiro。过滤所有请求/**的过滤器

2. 向ioc中添加一个DefaultWebSecurityManager，并设置一个Realm的实现类

3. 向容器中添加一个Realm的实现类，供2中的DefaultWebSecurityManager使用

4. 向ioc中加入一个LifecycleBeanPostProcessor和一个DefaultAdvisorAutoProxyCreator

5. 向ioc中加入一个AuthorizationAttributeSourceAdvisor并为其配置2中的DefaultWebSecurityManager

6. 向ioc中加入ShiroFilterFactoryBean并为其配置DefaultWebSecurityManager。同时可以配置loginUrl，UnauthorizedUrl，SuccessUrl等
   还可以配置shiroFilterFactoryBean.setFilterChainDefinitions("/login = anon \n /** = authc");
   表示/login 可以使用匿名身份访问 其他接口需要被认证才能访问   anno表示匿名  authc表示需要被认证
   这个bean的名字必须是shiroFilter


## 流程浅析

1. 分析Spring整合Shiro的第1和第6步
    DelegatingFilterProxy会作为代理类，从ioc中找一个bean作为真正的过滤器去过滤
    这个bean的名字为filter-name或者为DelegatingFilterProxy手动设置targetBeanName。targetBeanName的优先级更高
2. 分析ShiroFilterFactoryBean的setFilterChainDefinitions
    url = 拦截器名[参数] （形如：/login = anon 等号两边的空格可有可无）
    anno是Shiro里内置的拦截器
    url的匹配方式支持ant   ? 表示一个字符 * 表示多个字符 **表示当前/下所有url
    后声明的url和过滤器不会覆盖先声明的（第一次优先匹配）


## 认证流程（可以参考或直接调试QuickStart）

1. 调用静态方法`SecurityUtils.getSubject();`获取Subject，调用isAuthenticated判断是否被认证过，如果没有认证过，创建以一个AuthenticationToken调用login(token)进行认证
   token包含前端传来的用户名和密码
2. login实际上会调用SecurityManager的方法进行认证并返回一个新的Subject
3. Spring整合Shiro的第2步的Realm的实现类包含查库获取用户真正的用户名和密码并将结果封装到AuthenticationInfo对象中
   SecurityManager进行验证时会获取到Realm的列表，遍历Realm判断他们是否支持当前传来的AuthenticationToken，如果支持就执行上述流程
4. 密码在哪里匹配？AuthenticationToken中包含前端传来的用户名密码，AuthenticationInfo中包含Realm从db中获取的真正的用户名密码
   ps：为Realm设置加密器credentialMatcher可以为密码加密（自动为前端传来的密码加密，不对查到的db的密码进行加密），还可以设置加密次数（使用加密算法进行多次加密）
        加密所需要的盐用  ByteSource.Util.bytes(String str)  str值通常唯一的，通常使用id或username


## 登出

1. 设置 url = logout  （登出过滤器）  这个url不需要自己创建，随便设置一个即可
登出过滤器会调用Subject的logout方法


## 多Realm认证

1. 添加更多Realm的实现类
2. 创建一个ModularRealmAuthenticator，注入容器并作为参数设置到Spring整合Shiro第二步创建的DefaultWebSecurityManager中作为成员变量
3. 将新建的Realm也设置到DefaultWebSecurityManager中
   ps：1. 将多个Realm设置到ModularRealmAuthenticator中去掉了DefaultWebSecurityManager中的realm的配置，而不是设置到DefaultWebSecurityManager中
    2. 或将reaml配置给DefaultWebSecurityManager都行
    推荐后者。原因1. 做授权时会从DefaultWebSecurityManager中读取realm而不是ModularRealmAuthenticator
    2. 在创建DefaultWebSecurityManager时如果它的成员变量包含含有ModularRealmAuthenticator那就把realm赋给ModularRealmAuthenticator
    不管realm在哪，最终认证时都是从ModularRealmAuthenticator中获取realm的。
       但是授权时会从DefaultWebSecurityManager中获取realm的


## 认证策略

在ModularRealmAuthenticator中设置

新建一个ModularRealmAuthenticator对象并放到ioc中

## 授权

授权根据Realm的一个抽象类的方法实现的。你需要做的就是创建类并继承它，实现方法即可

ps：权限注解应该加在Controller层，因为权限注解会创建代理对象。通常Service层会使用事务注解，如果权限注解也加在Service层会出现代理的代理情况，会出现类型转换异常


## 会话管理

HttpSession通常规定只能在Controller层中使用

Shiro的Session能在Service层中使用

而在Controller层中HttpSession调用的setAttribute设置的kv

在Shiro的Session中也可以使用getAttribute获取到

说明Shiro的Session和HttpSession有关联关系

## 补充：

SessionDao 将会话持久化到db中
会话验证调度器，用于验证会话是否过期


## 缓存
Realm缓存和Session缓存


## 到此为止还没有实现请求每次都携带token，后端专门有过滤器校验token


## 详解

[参考博客分析Shiro内置过滤器](https://www.cnblogs.com/LeeScofiled/p/10511948.html)