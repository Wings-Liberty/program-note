#还没有复习 

# 多Realm认证和认证策略


在`subject.login(token);`的认证流程中`SecurityManager`会遍历持有的所有`Realm`的实现类

上述其实是由`AuthenticatingSecurityManager`（`SecurityManager`的实现类）的成员变量`Authenticator authenticator`实现的


可以通过调用

`securityManager.setRealm(shiroRealm);`或`securityManager.setRealms(CollectionUtils.asList(shiroRealm, secondRealm));`

将Realm的实现类添加到安全管理器


认证策略由`AuthenticatingSecurityManager`的成员变量指定

```java
/**
  * 设置认证策略
  * AtLeastOneSuccessfulStrategy（默认策略，如果有一个Realm通过认证就算认证成功）
  * AllSuccessfulStrategy（所有Realm通过认证才算认证成功）
  * FirstSuccessfulStrategy（当有一个Realm通过认证后就不再执行其余Realm的认证方法）
  */
modularRealmAuthenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());
```


## 会话管理


在Java Web中有Session的概念，但是Java Web的Session通常被建议只能在控制层用，不能在服务层用。且移动端不支持Session

Shiro提供的Session可以通过`subject.getSession();`获取。且推荐在各层都可以使用，且支持移动端，且在控制层调用`HttpServletSession`的`setAttribute()`方法后，Shior的Session也能获取到set的属性和属性值


# 记住我

