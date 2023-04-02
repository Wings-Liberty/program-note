#还没有复习 

容器在加载`BeanDefinition`时会调用`bd.getPropertyValues().addPropertyValue(pv);`成员变量的定义信息添加到`BeanDefinition`对象中

容器在`AbstractAutowireCapableBeanFactory`的`doCreateBean()`方法中调用`populate()`方法执行依赖注入的逻辑

执行`populate()`方法时会调用`mbd.getPropertyValues()`获取`BeanDefinition`对象中的成员变量的定义信息，根据此信息进行依赖注入



```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    if (bw == null) {
        ...
    }

	...

    // 获取bean定义信息中的成员变量定义信息
    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

    if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME ||
        mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);

        // Add property values based on autowire by name if applicable.
        if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }

        // Add property values based on autowire by type if applicable.
        if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }

        pvs = newPvs;
    }

    ...

    if (pvs != null) {
        // 应用成员变量的定义信息
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}
```



`applyPropertyValues(beanName, mbd, bw, pvs);`——`bw.setPropertyValues()`——`setPropertyValue()`

遍历成员变量定义信息，逐个注入

容器根据成变量定义信息使用`getBean()`获取到bean，再使用反射调用`setterXxx()`方法或其他方法将bean注入到bean中