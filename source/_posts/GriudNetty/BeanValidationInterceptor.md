---
title: BeanValidationInterceptor
abbrlink: 910739970
date: 2021-12-12 15:45:59
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
BeanValidationInterceptor

```java
package com.github.jsoncat.core.aop.intercept;

import com.github.jsoncat.annotation.validation.Validated;
import org.hibernate.validator.HibernateValidator;
import org.hibernate.validator.messageinterpolation.ParameterMessageInterpolator;

import javax.validation.ConstraintViolation;
import javax.validation.ConstraintViolationException;
import javax.validation.Valid;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;
import java.lang.annotation.Annotation;
import java.util.Arrays;
import java.util.Set;

public class BeanValidationInterceptor extends Interceptor {
    private final Validator validator;

    public BeanValidationInterceptor() {
        ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
                .configure()
                .messageInterpolator(new ParameterMessageInterpolator())
                .buildValidatorFactory();
        this.validator = validatorFactory.getValidator();
    }

    @Override
    public boolean supports(Object bean) {
        return (bean != null && bean.getClass().isAnnotationPresent(Validated.class));
    }

    @Override
    public Object intercept(MethodInvocation methodInvocation) {
        Annotation[][] parameterAnnotations = methodInvocation.getTargetMethod().getParameterAnnotations();
        Object[] args = methodInvocation.getArgs();
        for (int i = 0; i < args.length; i++) {
            boolean isNeedValidating = Arrays.stream(parameterAnnotations[i])
                    .anyMatch(annotation -> annotation.annotationType() == Valid.class);
            if (isNeedValidating) {
                Set<ConstraintViolation<Object>> results = validator.validate(args[i]);
                if (!results.isEmpty()) {
                    throw new ConstraintViolationException(results);
                }
            }
        }
        return methodInvocation.proceed();
    }
}
```





```java
public BeanValidationInterceptor() {
    ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
            .configure()
            .messageInterpolator(new ParameterMessageInterpolator())
            .buildValidatorFactory();
    this.validator = validatorFactory.getValidator();
}
```

初始化操作

这里的主要作用是  初始化了校验类

![image-20211210211727789](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211210211727789.png?x-oss-process=style/qingyun)

接口 方法的返回   Validator

```
//    @NotBlank(message =)   验证字符串非null，且长度必须大于0
//    @Email  被注释的元素必须是电子邮箱地址
//    @Length(min=,max=)  被注释的字符串的大小必须在指定的范围内
//    @NotEmpty   被注释的字符串的必须非空
//    @Range(min=,max=,message=)  被注释的元素必须在合适的范围内
//Hibernate Validator提供的校验注解  类校验器
```

添加了一个类校验器