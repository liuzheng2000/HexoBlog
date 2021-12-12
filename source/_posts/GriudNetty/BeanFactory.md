---
title: BeanFactory
abbrlink: 4126343555
date: 2021-12-12 15:46:35
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
#### BeanFactory

我们先看下具体的代码

```java
package com.github.jsoncat.core.ioc;

import com.github.jsoncat.annotation.ioc.Component;
import com.github.jsoncat.annotation.springmvc.RestController;
import com.github.jsoncat.common.util.ReflectionUtil;
import com.github.jsoncat.core.aop.factory.AopProxyBeanPostProcessorFactory;
import com.github.jsoncat.core.aop.intercept.BeanPostProcessor;
import com.github.jsoncat.core.config.ConfigurationFactory;
import com.github.jsoncat.core.config.ConfigurationManager;
import com.github.jsoncat.exception.DoGetBeanException;
import com.github.jsoncat.factory.ClassFactory;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public final class BeanFactory {

    // ioc bean 容器
    public static final Map<String, Object> BEANS = new ConcurrentHashMap<>(128);

    private static final Map<String, String[]> SINGLE_BEAN_NAMES_TYPE_MAP = new ConcurrentHashMap<>(128);

    public static void loadBeans() {
        ClassFactory.CLASSES.get(Component.class).forEach(aClass -> {
            String beanName = BeanHelper.getBeanName(aClass);
            Object obj = ReflectionUtil.newInstance(aClass);
            BEANS.put(beanName, obj);
        });
        ClassFactory.CLASSES.get(RestController.class).forEach(aClass -> {
            Object obj = ReflectionUtil.newInstance(aClass);
            BEANS.put(aClass.getName(), obj);
        });
        BEANS.put(ConfigurationManager.class.getName(), new ConfigurationManager(ConfigurationFactory.getConfig()));
    }

    public static void applyBeanPostProcessors() {
        BEANS.replaceAll((beanName, beanInstance) -> {
            BeanPostProcessor beanPostProcessor = AopProxyBeanPostProcessorFactory.get(beanInstance.getClass());
            return beanPostProcessor.postProcessAfterInitialization(beanInstance);
        });
    }

    public static <T> T getBean(Class<T> type) {
        String[] beanNames = getBeanNamesForType(type);
        if (beanNames.length == 0) {
            throw new DoGetBeanException("not fount bean implement，the bean :" + type.getName());
        }
        Object beanInstance = BEANS.get(beanNames[0]);
        if (!type.isInstance(beanInstance)) {
            throw new DoGetBeanException("not fount bean implement，the bean :" + type.getName());
        }
        return type.cast(beanInstance);
    }

    public static <T> Map<String, T> getBeansOfType(Class<T> type) {
        Map<String, T> result = new HashMap<>();
        String[] beanNames = getBeanNamesForType(type);
        for (String beanName : beanNames) {
            Object beanInstance = BEANS.get(beanName);
            if (!type.isInstance(beanInstance)) {
                throw new DoGetBeanException("not fount bean implement，the bean :" + type.getName());
            }
            result.put(beanName, type.cast(beanInstance));
        }
        return result;
    }

    private static String[] getBeanNamesForType(Class<?> type) {
        String beanName = type.getName();
        String[] beanNames = SINGLE_BEAN_NAMES_TYPE_MAP.get(beanName);
        if (beanNames == null) {
            List<String> beanNamesList = new ArrayList<>();
            for (Map.Entry<String, Object> beanEntry : BEANS.entrySet()) {
                Class<?> beanClass = beanEntry.getValue().getClass();
                if (type.isInterface()) {
                    Class<?>[] interfaces = beanClass.getInterfaces();
                    for (Class<?> c : interfaces) {
                        if (type.getName().equals(c.getName())) {
                            beanNamesList.add(beanEntry.getKey());
                            break;
                        }
                    }
                } else if (beanClass.isAssignableFrom(type)) {
                    beanNamesList.add(beanEntry.getKey());
                }
            }
            beanNames = beanNamesList.toArray(new String[0]);
            SINGLE_BEAN_NAMES_TYPE_MAP.put(beanName, beanNames);
        }
        return beanNames;
    }
}
```



从Application中调用的代码

```java
public static void loadBeans() {
    ClassFactory.CLASSES.get(Component.class).forEach(aClass -> {
        String beanName = BeanHelper.getBeanName(aClass);
        Object obj = ReflectionUtil.newInstance(aClass);
        BEANS.put(beanName, obj);
    });
    ClassFactory.CLASSES.get(RestController.class).forEach(aClass -> {
        Object obj = ReflectionUtil.newInstance(aClass);
        BEANS.put(aClass.getName(), obj);
    });
    BEANS.put(ConfigurationManager.class.getName(), new ConfigurationManager(ConfigurationFactory.getConfig()));
}
```



```
 ClassFactory.CLASSES.get(Component.class).forEach(aClass -> {
        String beanName = BeanHelper.getBeanName(aClass);
        Object obj = ReflectionUtil.newInstance(aClass);
        BEANS.put(beanName, obj);
    });
```

这里是一个 Lambda 表达是的方式     ClassFactory.CLASSES.get(Component.class)  和之前一样

获取了这个的  含有此注解 Component.class  类的集合

```
forEach(aClass -> { }
```

进行遍历且 操作 

![image-20211209194223683](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209194223683.png?x-oss-process=style/qingyun)

因为是 Lambda 表达式  所以这里 一个一个更替





```
String beanName = BeanHelper.getBeanName(aClass);
```

##### 引入了一个新的类  BeanHelper

```java
package com.github.jsoncat.core.ioc;

import com.github.jsoncat.annotation.ioc.Component;

/**
 * @author shuang.kou
 * @createTime 2020年10月07日 21:23:00
 **/
public class BeanHelper {

    /**
     * get the bean name
     *
     * @param aClass target class
     * @return the bean name
     */
    public static String getBeanName(Class<?> aClass) {
        String beanName = aClass.getName();
        if (aClass.isAnnotationPresent(Component.class)) {
            Component component = aClass.getAnnotation(Component.class);
            beanName = "".equals(component.name()) ? aClass.getName() : component.name();
        }
        return beanName;
    }
}
```

此类仅有一个方法  故直接解析

getBeanName(Class<?> aClass)   

传入一个类    aClass.isAnnotationPresent(Component.class)  判断类的类型

```
Component component = aClass.getAnnotation(Component.class);  //反射构造类
```

  beanName = "".equals(component.name()) ? aClass.getName() : component.name();

判断`注解名`是否有自定义    `是返回定义名`  ` 不是返回类名`



### 跳转至  ReflectionUtil 之前使用过

Object obj = ReflectionUtil.newInstance(aClass);

让我们进入  ReflectionUtil  来解析方法

前往另一个md文件

这里的主要作用是  `实例化` 了 对象



```java
BEANS.put(beanName, obj);
```

这里的作用是  将  Bean名   和 实例化  对象放入

![image-20211209200859249](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209200859249.png?x-oss-process=style/qingyun)







```java
ClassFactory.CLASSES.get(RestController.class).forEach(aClass -> {
    Object obj = ReflectionUtil.newInstance(aClass);
    BEANS.put(aClass.getName(), obj);
});
```

这里也是  实例化  RestController.class 注解下的类  



![image-20211209201224157](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209201224157.png?x-oss-process=style/qingyun)

具体的实例化如上





```java
BEANS.put(ConfigurationManager.class.getName(), new ConfigurationManager(ConfigurationFactory.getConfig()));
```

直接放入的一个类

去查看下

ConfigurationManager

引入至另一个 ConfigurationManager  的类

```java
new ConfigurationManager(ConfigurationFactory.getConfig())
```

调用了有参构造函数   （参数的传值是一个新的类）

```java
ConfigurationFactory.getConfig()
```

故在开一个   .md文件   ConfigurationFactory.md







#### BeanFactory.applyBeanPostProcessors(); 在Application中的调用

```java
public static void applyBeanPostProcessors() {
    BEANS.replaceAll((beanName, beanInstance) -> {
        BeanPostProcessor beanPostProcessor = AopProxyBeanPostProcessorFactory.get(beanInstance.getClass());
        return beanPostProcessor.postProcessAfterInitialization(beanInstance);
    });
}
```

将Contorller中的类也改为了AOP形式





#### getBeansOfType(Class<T> type)

```java
public static <T> Map<String, T> getBeansOfType(Class<T> type) {
    Map<String, T> result = new HashMap<>();
    String[] beanNames = getBeanNamesForType(type);
    for (String beanName : beanNames) {
        Object beanInstance = BEANS.get(beanName);
        if (!type.isInstance(beanInstance)) {
            throw new DoGetBeanException("not fount bean implement，the bean :" + type.getName());
        }
        result.put(beanName, type.cast(beanInstance));
    }
    return result;
}
```

调用的返回了空数组