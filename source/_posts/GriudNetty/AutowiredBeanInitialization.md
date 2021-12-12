---
title: AutowiredBeanInitialization
abbrlink: 1171857368
date: 2021-12-12 15:50:14
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
### AutowiredBeanInitialization



```java
package com.github.jsoncat.core.ioc;

import com.github.jsoncat.annotation.config.Value;
import com.github.jsoncat.annotation.ioc.Autowired;
import com.github.jsoncat.annotation.ioc.Qualifier;
import com.github.jsoncat.common.util.ObjectUtil;
import com.github.jsoncat.common.util.ReflectionUtil;
import com.github.jsoncat.core.aop.factory.AopProxyBeanPostProcessorFactory;
import com.github.jsoncat.core.aop.intercept.BeanPostProcessor;
import com.github.jsoncat.core.config.ConfigurationManager;
import com.github.jsoncat.exception.CanNotDetermineTargetBeanException;
import com.github.jsoncat.exception.InterfaceNotHaveImplementedClassException;

import java.lang.reflect.Field;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author shuang.kou
 * @createTime 2020年10月19日 10:08:00
 **/
public class AutowiredBeanInitialization {
    private final String[] packageNames;

    public AutowiredBeanInitialization(String[] packageNames) {
        this.packageNames = packageNames;
    }

    //二级缓存（解决循环依赖问题）
    private static final Map<String, Object> SINGLETON_OBJECTS = new ConcurrentHashMap<>(64);

    public void initialize(Object beanInstance) {
        Class<?> beanClass = beanInstance.getClass();
        Field[] beanFields = beanClass.getDeclaredFields();
        // 遍历bean的属性
        if (beanFields.length > 0) {
            for (Field beanField : beanFields) {
                if (beanField.isAnnotationPresent(Autowired.class)) {
                    Object beanFieldInstance = processAutowiredAnnotationField(beanField);
                    String beanFieldName = BeanHelper.getBeanName(beanField.getType());
                    // 解决循环依赖问题
                    beanFieldInstance = resolveCircularDependency(beanInstance, beanFieldInstance, beanFieldName);
                    // AOP
                    BeanPostProcessor beanPostProcessor = AopProxyBeanPostProcessorFactory.get(beanField.getType());
                    beanFieldInstance = beanPostProcessor.postProcessAfterInitialization(beanFieldInstance);
                    ReflectionUtil.setField(beanInstance, beanField, beanFieldInstance);
                }
                if (beanField.isAnnotationPresent(Value.class)) {
                    Object convertedValue = processValueAnnotationField(beanField);
                    ReflectionUtil.setField(beanInstance, beanField, convertedValue);
                }
            }
        }
    }

    /**
     * 处理被 @Autowired 注解标记的字段
     *
     * @param beanField 目标类的字段
     * @return 目标类的字段对应的对象
     */
    private Object processAutowiredAnnotationField(Field beanField) {
        Class<?> beanFieldClass = beanField.getType();
        String beanFieldName = BeanHelper.getBeanName(beanFieldClass);
        Object beanFieldInstance;
        if (beanFieldClass.isInterface()) {
            @SuppressWarnings("unchecked")
            Set<Class<?>> subClasses = ReflectionUtil.getSubClass(packageNames, (Class<Object>) beanFieldClass);
            if (subClasses.size() == 0) {
                throw new InterfaceNotHaveImplementedClassException(beanFieldClass.getName() + "is interface and do not have implemented class exception");
            }
            if (subClasses.size() == 1) {
                Class<?> subClass = subClasses.iterator().next();
                beanFieldName = BeanHelper.getBeanName(subClass);
            }
            if (subClasses.size() > 1) {
                Qualifier qualifier = beanField.getDeclaredAnnotation(Qualifier.class);
                beanFieldName = qualifier == null ? beanFieldName : qualifier.value();
            }

        }
        beanFieldInstance = BeanFactory.BEANS.get(beanFieldName);
        if (beanFieldInstance == null) {
            throw new CanNotDetermineTargetBeanException("can not determine target bean of" + beanFieldClass.getName());
        }
        return beanFieldInstance;
    }

    /**
     * 处理被 @Value 注解标记的字段
     *
     * @param beanField 目标类的字段
     * @return 目标类的字段对应的对象
     */
    private Object processValueAnnotationField(Field beanField) {
        String key = beanField.getDeclaredAnnotation(Value.class).value();
        ConfigurationManager configurationManager = (ConfigurationManager) BeanFactory.BEANS.get(ConfigurationManager.class.getName());
        String value = configurationManager.getString(key);
        if (value == null) {
            throw new IllegalArgumentException("can not find target value for property:{" + key + "}");
        }
        return ObjectUtil.convert(beanField.getType(), value);
    }

    /**
     * 二级缓存解决循环依赖问题
     */
    private Object resolveCircularDependency(Object beanInstance, Object beanFieldInstance, String beanFieldName) {
        if (SINGLETON_OBJECTS.containsKey(beanFieldName)) {
            beanFieldInstance = SINGLETON_OBJECTS.get(beanFieldName);
        } else {
            SINGLETON_OBJECTS.put(beanFieldName, beanFieldInstance);
            initialize(beanInstance);
        }
        return beanFieldInstance;
    }

}
```





```java
    public AutowiredBeanInitialization(String[] packageNames) {
        this.packageNames = packageNames;
    }
```

先取出来这个对象

进行配置类的加载





```java
public void initialize(Object beanInstance) {
    Class<?> beanClass = beanInstance.getClass();
    Field[] beanFields = beanClass.getDeclaredFields();
    // 遍历bean的属性
    if (beanFields.length > 0) {
        for (Field beanField : beanFields) {
            if (beanField.isAnnotationPresent(Autowired.class)) {
                Object beanFieldInstance = processAutowiredAnnotationField(beanField);
                String beanFieldName = BeanHelper.getBeanName(beanField.getType());
                // 解决循环依赖问题
                beanFieldInstance = resolveCircularDependency(beanInstance, beanFieldInstance, beanFieldName);
                // AOP
                BeanPostProcessor beanPostProcessor = AopProxyBeanPostProcessorFactory.get(beanField.getType());
                beanFieldInstance = beanPostProcessor.postProcessAfterInitialization(beanFieldInstance);
                ReflectionUtil.setField(beanInstance, beanField, beanFieldInstance);
            }
            if (beanField.isAnnotationPresent(Value.class)) {
                Object convertedValue = processValueAnnotationField(beanField);
                ReflectionUtil.setField(beanInstance, beanField, convertedValue);
            }
        }
    }
}
```







```
Class<?> beanClass = beanInstance.getClass();
Field[] beanFields = beanClass.getDeclaredFields();
```



Object getClass() 方法用于获取对象的运行时对象的类。

getDeclaredFields()获得某个类的所有申明的字段，即包括public、private和proteced，

  但是不包括父类的申明字段。 





```Java
if (beanField.isAnnotationPresent(Autowired.class)) {
    Object beanFieldInstance = processAutowiredAnnotationField(beanField);
    String beanFieldName = BeanHelper.getBeanName(beanField.getType());
    // 解决循环依赖问题
    beanFieldInstance = resolveCircularDependency(beanInstance, beanFieldInstance, beanFieldName);
    // AOP
    BeanPostProcessor beanPostProcessor = AopProxyBeanPostProcessorFactory.get(beanField.getType());
    beanFieldInstance = beanPostProcessor.postProcessAfterInitialization(beanFieldInstance);
    ReflectionUtil.setField(beanInstance, beanField, beanFieldInstance);
}
```

判断是否有自动注入的类  Autowired.class  也就是有这个@Autowired的字符串

```java
Object beanFieldInstance = processAutowiredAnnotationField(beanField);
```

调用了本类的processAutowiredAnnotationField



```java
private Object processAutowiredAnnotationField(Field beanField) {
    Class<?> beanFieldClass = beanField.getType();
    String beanFieldName = BeanHelper.getBeanName(beanFieldClass);
    Object beanFieldInstance;
    if (beanFieldClass.isInterface()) {
        @SuppressWarnings("unchecked")
        Set<Class<?>> subClasses = ReflectionUtil.getSubClass(packageNames, (Class<Object>) beanFieldClass);
        if (subClasses.size() == 0) {
            throw new InterfaceNotHaveImplementedClassException(beanFieldClass.getName() + "is interface and do not have implemented class exception");
        }
        if (subClasses.size() == 1) {
            Class<?> subClass = subClasses.iterator().next();
            beanFieldName = BeanHelper.getBeanName(subClass);
        }
        if (subClasses.size() > 1) {
            Qualifier qualifier = beanField.getDeclaredAnnotation(Qualifier.class);
            beanFieldName = qualifier == null ? beanFieldName : qualifier.value();
        }

    }
    beanFieldInstance = BeanFactory.BEANS.get(beanFieldName);
    if (beanFieldInstance == null) {
        throw new CanNotDetermineTargetBeanException("can not determine target bean of" + beanFieldClass.getName());
    }
    return beanFieldInstance;
}
```





```java
Class<?> beanFieldClass = beanField.getType();
```



```
beanFieldClass.isInterface()    //需要注入接口类型？？
```

**java.lang.Class.isInterface()** 确定指定的Class对象表示一个接口类型。



```java
Set<Class<?>> subClasses = ReflectionUtil.getSubClass(packageNames, (Class<Object>) beanFieldClass);
```

让我们进入 ReflectionUtil 来看下方法的实现

主要目的是获取到了接口的实现类

![image-20211211170350643](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211211170350643.png?x-oss-process=style/qingyun)





```java
if (subClasses.size() == 0) {
    throw new InterfaceNotHaveImplementedClassException(beanFieldClass.getName() + "is interface and do not have implemented class exception");
}
```

实现类为0  抛出异常



```java
if (subClasses.size() == 1) {
    Class<?> subClass = subClasses.iterator().next();
    beanFieldName = BeanHelper.getBeanName(subClass);
}
```

实现类为1  先获取到类

然后  通过  BeanHelper.getBeanName(subClass)  获取类名

BeanHelper是一个单独的类  较为简单  故直接拿出   主要是获取类名

```java
package com.github.jsoncat.core.ioc;

import com.github.jsoncat.annotation.ioc.Component;

/**
 * 完
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





```java
if (subClasses.size() > 1) {
    Qualifier qualifier = beanField.getDeclaredAnnotation(Qualifier.class);
    beanFieldName = qualifier == null ? beanFieldName : qualifier.value();
}
```

大于1 时   获取指定的ClassName  找不到则返回Bean的名字





```java
beanFieldInstance = BeanFactory.BEANS.get(beanFieldName);
```

根据Name获取到Bean中的正在运行

```
        if (beanFieldInstance == null) {
            throw new CanNotDetermineTargetBeanException("can not determine target bean of" + beanFieldClass.getName());
        }
        return beanFieldInstance;
```

为空 则抛出异常  不为空 则返回







解决循环依赖问题

```java
beanFieldInstance = resolveCircularDependency(beanInstance, beanFieldInstance, beanFieldName);
```

这里调用了本地方法

```java
/**
 * 二级缓存解决循环依赖问题
 */
private Object resolveCircularDependency(Object beanInstance, Object beanFieldInstance, String beanFieldName) {
    if (SINGLETON_OBJECTS.containsKey(beanFieldName)) {
        beanFieldInstance = SINGLETON_OBJECTS.get(beanFieldName);
    } else {
        SINGLETON_OBJECTS.put(beanFieldName, beanFieldInstance);
        initialize(beanInstance);
    }
    return beanFieldInstance;
}
```

至少执行两边这里  感觉有错误

我先向下写

第一次执行的时候

![image-20211211174401555](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211211174403598.png?x-oss-process=style/qingyun)

注定为空 进入一遍初始化

![image-20211211174516583](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211211174516583.png?x-oss-process=style/qingyun)

肯定有上一次加入的 进行获取

![image-20211211174604853](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211211174604853.png?x-oss-process=style/qingyun)

返回了出来



```java
BeanPostProcessor beanPostProcessor = AopProxyBeanPostProcessorFactory.get(beanField.getType());
```

进入一遍这里



```java
package com.github.jsoncat.core.aop.factory;

import com.github.jsoncat.core.aop.intercept.BeanPostProcessor;
import com.github.jsoncat.core.aop.intercept.CglibAopProxyBeanPostProcessor;
import com.github.jsoncat.core.aop.intercept.JdkAopProxyBeanPostProcessor;


public class AopProxyBeanPostProcessorFactory {

    /**
     * @param beanClass 目标类
     * @return beanClass 实现了接口就返回jdk动态代理bean后置处理器,否则返回 cglib动态代理bean后置处理器
     */
    public static BeanPostProcessor get(Class<?> beanClass) {
        if (beanClass.isInterface() || beanClass.getInterfaces().length > 0) {
            return new JdkAopProxyBeanPostProcessor();
        } else {
            return new CglibAopProxyBeanPostProcessor();
        }
    }
}
```

判断使用那个代理类

![image-20211211174842709](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211211174842709.png?x-oss-process=style/qingyun)

现在使用CglibAopProxxyBeanPostProcessor



```java
beanFieldInstance = beanPostProcessor.postProcessAfterInitialization(beanFieldInstance);
```

进行代理构造

```java
package com.github.jsoncat.core.aop.intercept;

import com.github.jsoncat.core.aop.factory.InterceptorFactory;

public abstract class AbstractAopProxyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessAfterInitialization(Object bean) {
        Object wrapperProxyBean = bean;
        //链式包装目标类
        for (Interceptor interceptor : InterceptorFactory.getInterceptors()) {
            if (interceptor.supports(bean)) {
                wrapperProxyBean = wrapBean(wrapperProxyBean, interceptor);
            }
        }
        return wrapperProxyBean;
    }

    public abstract Object wrapBean(Object target, Interceptor interceptor);
}
```

这里是抽象类

进行代理层的判断
