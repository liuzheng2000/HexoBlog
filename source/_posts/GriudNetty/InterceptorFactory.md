---
title: InterceptorFactory
abbrlink: 2696642908
date: 2021-12-12 15:41:22
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
InterceptorFactory

```java
package com.github.jsoncat.core.aop.factory;

import com.github.jsoncat.annotation.aop.Aspect;
import com.github.jsoncat.annotation.aop.Order;
import com.github.jsoncat.common.util.ReflectionUtil;
import com.github.jsoncat.core.aop.intercept.BeanValidationInterceptor;
import com.github.jsoncat.core.aop.intercept.Interceptor;
import com.github.jsoncat.core.aop.intercept.InternallyAspectInterceptor;
import com.github.jsoncat.exception.CannotInitializeConstructorException;
import com.github.jsoncat.factory.ClassFactory;

import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.Set;
import java.util.stream.Collectors;

/**
 * 存放所有拦截器的工厂类
 *
 * @author shuang.kou
 * @createTime 2020年10月09日 22:24:00
 **/
public class InterceptorFactory {
    private static List<Interceptor> interceptors = new ArrayList<>();

    public static void loadInterceptors(String[] packageName) {
        // 获取指定包中实现了 Interceptor 接口的类
        Set<Class<? extends Interceptor>> interceptorClasses = ReflectionUtil.getSubClass(packageName, Interceptor.class);
        // 获取被 @Aspect 标记的类
        Set<Class<?>> aspects = ClassFactory.CLASSES.get(Aspect.class);
        // 遍历所有拦截器类
        interceptorClasses.forEach(interceptorClass -> {
            try {
                interceptors.add(interceptorClass.newInstance());
            } catch (InstantiationException | IllegalAccessException e) {
                throw new CannotInitializeConstructorException("not init constructor , the interceptor name :" + interceptorClass.getSimpleName());
            }
        });
        aspects.forEach(aClass -> {
            Object obj = ReflectionUtil.newInstance(aClass);
            Interceptor interceptor = new InternallyAspectInterceptor(obj);
            if (aClass.isAnnotationPresent(Order.class)) {
                Order order = aClass.getAnnotation(Order.class);
                interceptor.setOrder(order.value());
            }
            interceptors.add(interceptor);
        });
        // 添加Bean验证拦截器
        interceptors.add(new BeanValidationInterceptor());
        // 根据 order 为拦截器排序
        interceptors = interceptors.stream().sorted(Comparator.comparing(Interceptor::getOrder)).collect(Collectors.toList());
    }

    public static List<Interceptor> getInterceptors() {
        return interceptors;
    }
}
```





```java
  // 获取指定包中实现了 Interceptor 接口的类
        Set<Class<? extends Interceptor>> interceptorClasses = ReflectionUtil.getSubClass(packageName, Interceptor.class);
        // 获取被 @Aspect 标记的类
        Set<Class<?>> aspects = ClassFactory.CLASSES.get(Aspect.class);
```

![image-20211209220636221](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209220636221.png?x-oss-process=style/qingyun)![image-20211209220638105](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209220638105.png?x-oss-process=style/qingyun)

因为目前没有具体的用到  故不去往类中看





```java
interceptors.add(interceptorClass.getDeclaredConstructor().newInstance());
```

这里是实例化类  并添加  因为之前解析过类似的故不在解析





```java
aspects.forEach(aClass -> {
    Object obj = ReflectionUtil.newInstance(aClass);
    Interceptor interceptor = new InternallyAspectInterceptor(obj);
    if (aClass.isAnnotationPresent(Order.class)) {
        Order order = aClass.getAnnotation(Order.class);
        interceptor.setOrder(order.value());
    }
    interceptors.add(interceptor);
});
```

遍历 // 获取被 @Aspect 标记的类

 

```
Object obj = ReflectionUtil.newInstance(aClass);
```

生成实例 



```java
Interceptor interceptor = new InternallyAspectInterceptor(obj);
```

引入了一个新的类   InternallyAspectInterceptor

新建  .md 进行解析  





```java
// 根据 order 为拦截器排序
interceptors = interceptors.stream().sorted(Comparator.comparing(Interceptor::getOrder)).collect(Collectors.toList());
```

一个排序  根据Order进行排序   Order越小越在前







