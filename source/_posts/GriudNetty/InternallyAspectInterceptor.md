---
title: InternallyAspectInterceptor
abbrlink: 1416760160
date: 2021-12-12 15:40:38
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
### InternallyAspectInterceptor

```java
package com.github.jsoncat.core.aop.intercept;

import com.github.jsoncat.annotation.aop.After;
import com.github.jsoncat.annotation.aop.Before;
import com.github.jsoncat.annotation.aop.Pointcut;
import com.github.jsoncat.common.util.ReflectionUtil;
import com.github.jsoncat.core.aop.lang.JoinPoint;
import com.github.jsoncat.core.aop.lang.JoinPointImpl;
import com.github.jsoncat.core.aop.util.PatternMatchUtils;

import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Objects;

public class InternallyAspectInterceptor extends Interceptor {

    private final Object adviceBean;
    private final HashSet<String> expressionUrls = new HashSet<>();
    private final List<Method> beforeMethods = new ArrayList<>();
    private final List<Method> afterMethods = new ArrayList<>();

    public InternallyAspectInterceptor(Object adviceBean) {
        this.adviceBean = adviceBean;
        init();
    }

    private void init() {
        for (Method method : adviceBean.getClass().getMethods()) {
            Pointcut pointcut = method.getAnnotation(Pointcut.class);
            if (!Objects.isNull(pointcut)) {
                expressionUrls.add(pointcut.value());
            }
            Before before = method.getAnnotation(Before.class);
            if (!Objects.isNull(before)) {
                beforeMethods.add(method);
            }
            After after = method.getAnnotation(After.class);
            if (!Objects.isNull(after)) {
                afterMethods.add(method);
            }
        }
    }

    @Override
    public boolean supports(Object bean) {
        return expressionUrls.stream().anyMatch(url -> PatternMatchUtils.simpleMatch(url, bean.getClass().getName())) && (!beforeMethods.isEmpty() || !afterMethods.isEmpty());
    }

    @Override
    public Object intercept(MethodInvocation methodInvocation) {
        JoinPoint joinPoint = new JoinPointImpl(adviceBean, methodInvocation.getTargetObject(),
                methodInvocation.getArgs());
        beforeMethods.forEach(method -> ReflectionUtil.executeTargetMethodNoResult(adviceBean, method, joinPoint));
        Object result = methodInvocation.proceed();
        afterMethods.forEach(method -> ReflectionUtil.executeTargetMethodNoResult(adviceBean, method, result, joinPoint));
        return result;
    }
}
```





```java
public InternallyAspectInterceptor(Object adviceBean) {
    this.adviceBean = adviceBean;
    init();
}
```

进入时，调用的方法  

相当于初始化了一个  InternallyAspectInterceptor  类    adviceBean =  传入的值



```java
private void init() {
    for (Method method : adviceBean.getClass().getMethods()) {
        Pointcut pointcut = method.getAnnotation(Pointcut.class);
        if (!Objects.isNull(pointcut)) {
            expressionUrls.add(pointcut.value());
        }
        Before before = method.getAnnotation(Before.class);
        if (!Objects.isNull(before)) {
            beforeMethods.add(method);
        }
        After after = method.getAnnotation(After.class);
        if (!Objects.isNull(after)) {
            afterMethods.add(method);
        }
    }
}
```

主要的初始化方法

给   adviceBean   中的方法进行了分类

 ![image-20211209222513821](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209222513821.png?x-oss-process=style/qingyun)

这就是分类出来的 ，当然这只是例子





```java
// 添加Bean验证拦截器
interceptors.add(new BeanValidationInterceptor());
// 根据 order 为拦截器排序
interceptors = interceptors.stream().sorted(Comparator.comparing(Interceptor::getOrder)).collect(Collectors.toList());
```





### interceptors.add(new BeanValidationInterceptor());

牵扯到了一个新的类

```java
new BeanValidationInterceptor()
```

引入新的md文件解析

