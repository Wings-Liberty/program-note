#还没有复习 

以`XmlWebApplicationContext`为例，分析容器初始化过程中**一个**`BeanDefinition`的定位，加载和注册过程



- `Resource`定位。使用`ResourceLoader`定位到资源
- 加载。`BeanDefinitionReader`解析资源中的bean的定义信息并创建`BeanDefinition`对象
- 注册。使用`BeanDefinitionRegistry`，在容器内部将`BeanDefinition`对象注册进来



# 定位



## 定位的栈

以下是调用`refresh()`方法定位`Resource`的栈信息

```
loadBeanDefinitions:219, AbstractBeanDefinitionReader (org.springframework.beans.factory.support)
loadBeanDefinitions:194, AbstractBeanDefinitionReader (org.springframework.beans.factory.support)
loadBeanDefinitions:125, XmlWebApplicationContext (org.springframework.web.context.support)
loadBeanDefinitions:94, XmlWebApplicationContext (org.springframework.web.context.support)
refreshBeanFactory:133, AbstractRefreshableApplicationContext (org.springframework.context.support)
obtainFreshBeanFactory:621, AbstractApplicationContext (org.springframework.context.support)
refresh:522, AbstractApplicationContext (org.springframework.context.support)
...
```



## 定位流程

调用`BeanDefinitionReader`持有的`ResourceLoader`对象的`Resource getResource(String var1);`方法获取到`Resource`

ps：在`Applicationcontext`的继承体系中，`Applicationcontext`实现了`ResourceLoader`，所以容器也是一个`ResourceLoader`。通常`BeanDefinitionReader`持有的`ResourceLoader`对象就是当前使用的容器





```
loadBeanDefinitions:94, XmlWebApplicationContext (org.springframework.web.context.support)
```

下面的方法对应此栈，方法实现在`XmlWebApplicationContext`

```java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    // 创建一个XmlBeanDefinitionReader，用于定位Bean定义信息
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    beanDefinitionReader.setEnvironment(getEnvironment());
    // XmlWebApplicationContext的继承体系中实现了ResourceLoader，所以this是一个ResourceLoader
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    // Allow a subclass to provide custom initialization of the reader,
    // then proceed with actually loading the bean definitions.
    initBeanDefinitionReader(beanDefinitionReader);
    loadBeanDefinitions(beanDefinitionReader);
}
```

上述方法创建了一个`BeanDefinitionReader`，并将容器作为`ResourceLoader`传递进去





```
loadBeanDefinitions:219, AbstractBeanDefinitionReader (org.springframework.beans.factory.support)
```

下面的方法对应此栈，调用方法的是上一步创建的`XmlBeanDefinitionReader`对象，方法实现在`AbstractBeanDefinitionReader`中

```java
public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
    ResourceLoader resourceLoader = getResourceLoader();
    if (resourceLoader == null) {
        throw new BeanDefinitionStoreException("...");
    }

    if (resourceLoader instanceof ResourcePatternResolver) {
        try {
            // 至此，完成BeanDefinition的Resource的定位。获取到Resource
            Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
            // BenaDefinition的加载和注册在此方法中
            int loadCount = loadBeanDefinitions(resources);
            if (actualResources != null) {
                for (Resource resource : resources) {
                    actualResources.add(resource);
                }
            }
            return loadCount;
        }
        catch (IOException ex) {
            throw new BeanDefinitionStoreException("...", ex);
        }
    }
    else {
        // 至此，完成BeanDefinition的Resource的定位。获取到Resource
        Resource resource = resourceLoader.getResource(location);
        // BenaDefinition的加载和注册均在此方法中
        int loadCount = loadBeanDefinitions(resource);
        if (actualResources != null) {
            actualResources.add(resource);
        }
        if (logger.isDebugEnabled()) {
            logger.debug("...");
        }
        return loadCount;
    }
}
```





# 加载



## 加载的栈

```
// 此方法包含BeanDefinition的加载和注册
processBeanDefinition:305, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
parseDefaultElement:196, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
parseBeanDefinitions:175, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
doRegisterBeanDefinitions:148, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
registerBeanDefinitions:98, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
registerBeanDefinitions:507, XmlBeanDefinitionReader (org.springframework.beans.factory.xml)
// 此方法包含了BeanDefinition加载的两大步骤（解析xml文件创建Document对象，BeanDefinition加载和注册）
doLoadBeanDefinitions:391, XmlBeanDefinitionReader (org.springframework.beans.factory.xml)
loadBeanDefinitions:335, XmlBeanDefinitionReader (org.springframework.beans.factory.xml)
loadBeanDefinitions:303, XmlBeanDefinitionReader (org.springframework.beans.factory.xml)
loadBeanDefinitions:187, AbstractBeanDefinitionReader (org.springframework.beans.factory.support)
// 这里包含BeanDefinition的定位，同时也包含BeanDefinition的加载和注册的入口方法
loadBeanDefinitions:223, AbstractBeanDefinitionReader (org.springframework.beans.factory.support)
...
refresh:522, AbstractApplicationContext (org.springframework.context.support)
...
```



## 加载流程

加载分为两步

1. 使用XML的解析器获取到`Document`对象（需要解析xml文件是因为bean的定义信息是在xml文件里写的）
2. 使用`DocumentReader`按照`Spring`定义`Bean`的规则解析xml文件并创建`BeanDefinition`对象。创建的`BeanDefinition`对象会先被`BeanDefinitionHolder`封装起来



```
doLoadBeanDefinitions:391, XmlBeanDefinitionReader (org.springframework.beans.factory.xml)
```

下面的方法对应此栈，方法实现在`XmlBeanDefinitionReader`中

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource) throws BeanDefinitionStoreException {
    try {
        // 解析xml文件，创建Document对象
        Document doc = doLoadDocument(inputSource, resource);
        // BeanDefinition的加载和注册
        return registerBeanDefinitions(doc, resource);
    }
    catch (Throwable ex) {
        throw ...
    }
}
```



```
parseBeanDefinitions:175, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
```

下面的方法对应此栈。获取xml中的根节点，获取根节点下的所有子节点，遍历

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
            
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
                        // 解析根节点下的子节点
						parseDefaultElement(ele, delegate);
					}
					else {
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}
```



```
processBeanDefinition:305, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
```

下面的方法对应此栈。方法的调用者`DefaultBeanDefinitionDocumentReader`是由`XmlBeanDefinitionReader`创建的

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    // 至此，完成BeanDefinition的加载。创建BeanDefinition对象并将结果交给BeanDefinitionHolder持有
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // BeanDefinition的注册在此方法中
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("Failed to register bean definition with name '" +
                                     bdHolder.getBeanName() + "'", ele, ex);
        }
        // Send registration event.
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```



# 注册



## 注册的栈

```
registerBeanDefinition:792, DefaultListableBeanFactory (org.springframework.beans.factory.support)
registerBeanDefinition:150, BeanDefinitionReaderUtils (org.springframework.beans.factory.support)
processBeanDefinition:310, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
parseDefaultElement:196, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
parseBeanDefinitions:175, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
doRegisterBeanDefinitions:148, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
registerBeanDefinitions:98, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
registerBeanDefinitions:507, XmlBeanDefinitionReader (org.springframework.beans.factory.xml)
// 此方法包含了BeanDefinition加载的两大步骤（解析xml文件创建Document对象，BeanDefinition加载和注册）
doLoadBeanDefinitions:391, XmlBeanDefinitionReader (org.springframework.beans.factory.xml)
...
```



## 注册流程

以kv键值对的方式将beanName和`BeanDefinition`对象put到容器内部的一个map中（类型是`ConcurrentHashMap`）



```
processBeanDefinition:310, DefaultBeanDefinitionDocumentReader (org.springframework.beans.factory.xml)
```

下面的方法对应此栈

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    // 创建BeanDefinition并将其包装在BeanDefinitionHolder中
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // 将BeanDefinition注册到容器中
            // 方法的第二个参数是DefaultListableBeanFactory对象
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("...", ele, ex);
        }
        // 发布注册事件
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```





```
registerBeanDefinition:150, BeanDefinitionReaderUtils (org.springframework.beans.factory.support)
```

下面的方法对应此栈。这里的`BeanDefinitionRegistry`的实际类型是`DefaultListableBeanFactory`

```java
public static void registerBeanDefinition(BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry) throws BeanDefinitionStoreException {

    String beanName = definitionHolder.getBeanName();
    // 调用BeanDefinitionRegistry（Bean定义信息注册中心）将BeanDefinition以kv形式注册到容器中。k是beanName，v是对象
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String alias : aliases) {
            // 把别名也注册到容器中
            registry.registerAlias(beanName, alias);
        }
    }
}
```





```
registerBeanDefinition:792, DefaultListableBeanFactory (org.springframework.beans.factory.support)
```

下面的方法对应此栈。方法实现在`DefaultListableBeanFactory`，方法由`BeanDefinitionRegistry`接口定义

```java
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition) throws BeanDefinitionStoreException {

    if (beanDefinition instanceof AbstractBeanDefinition) {
        try {
            ((AbstractBeanDefinition) beanDefinition).validate();
        }
        catch (BeanDefinitionValidationException ex) {
            throw ...
        }
    }

    BeanDefinition oldBeanDefinition;

    // 先用beanName从容器中获取BeanDefinition
    oldBeanDefinition = this.beanDefinitionMap.get(beanName);
    // 如果容器中确实已经存在同名BeanDefinition对象，根据之前设置的能否覆盖同名对象进行操作
    if (oldBeanDefinition != null) {
        // 如果设置的是  不能覆盖同名对象，抛异常
        if (!isAllowBeanDefinitionOverriding()) {
            throw new ...
        }
        else if (oldBeanDefinition.getRole() < beanDefinition.getRole()) {
           ...
        }
        else if (!beanDefinition.equals(oldBeanDefinition)) {
            ...
        }
        else {
            ...
        }
        // 如果 能覆盖同名对象，将beanDefinition对象put到map中
        // 这个map就是容器中BeanDefinition的持有者，
        this.beanDefinitionMap.put(beanName, beanDefinition);
    }
    else {
        if (hasBeanCreationStarted()) {
            synchronized (this.beanDefinitionMap) {
                // 将beanDefinition对象put到map中
                this.beanDefinitionMap.put(beanName, beanDefinition);
                List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
                updatedDefinitions.addAll(this.beanDefinitionNames);
                updatedDefinitions.add(beanName);
                this.beanDefinitionNames = updatedDefinitions;
                if (this.manualSingletonNames.contains(beanName)) {
                    Set<String> updatedSingletons = new LinkedHashSet<>(this.manualSingletonNames);
                    updatedSingletons.remove(beanName);
                    this.manualSingletonNames = updatedSingletons;
                }
            }
        }
        else {
            // 将beanDefinition对象put到map中
            this.beanDefinitionMap.put(beanName, beanDefinition);
            this.beanDefinitionNames.add(beanName);
            this.manualSingletonNames.remove(beanName);
        }
        this.frozenBeanDefinitionNames = null;
    }

    if (oldBeanDefinition != null || containsSingleton(beanName)) {
        resetBeanDefinition(beanName);
    }
}
```

此方法会将`BeanDefinition`对象put到map中



至此，一个`BeanDefinition`注册到容器中的任务就完成了