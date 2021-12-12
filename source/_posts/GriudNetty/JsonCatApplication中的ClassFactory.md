---
title: JsonCatApplication中的ClassFactory
abbrlink: 2719058893
date: 2021-12-12 15:39:08
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---


### JsonCatApplication   中的 ClassFactory类

```Java
package com.github.jsoncat.factory;

import com.github.jsoncat.annotation.aop.Aspect;
import com.github.jsoncat.annotation.ioc.Component;
import com.github.jsoncat.annotation.springmvc.RestController;
import com.github.jsoncat.common.util.ReflectionUtil;

import java.lang.annotation.Annotation;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author shuang.kou
 * @createTime 2020年09月30日 15:26:00
 **/
public class ClassFactory {
    public static final Map<Class<? extends Annotation>, Set<Class<?>>> CLASSES = new ConcurrentHashMap<>();
    public static void loadClass(String[] packageName) {

        Set<Class<?>> restControllers = ReflectionUtil.scanAnnotatedClass(packageName, RestController.class);
        Set<Class<?>> components = ReflectionUtil.scanAnnotatedClass(packageName, Component.class);
        Set<Class<?>> aspects = ReflectionUtil.scanAnnotatedClass(packageName, Aspect.class);
        CLASSES.put(RestController.class, restControllers);
        CLASSES.put(Component.class, components);
        CLASSES.put(Aspect.class, aspects);
    }
}

```





主要方法  

```java
public static void loadClass(String[] packageName) {
    Set<Class<?>> restControllers = ReflectionUtil.scanAnnotatedClass(packageName, RestController.class);
    Set<Class<?>> components = ReflectionUtil.scanAnnotatedClass(packageName, Component.class);
    Set<Class<?>> aspects = ReflectionUtil.scanAnnotatedClass(packageName, Aspect.class);
    CLASSES.put(RestController.class, restControllers);
    CLASSES.put(Component.class, components);
    CLASSES.put(Aspect.class, aspects);
}
```

packageName   传进来的是一个包名





```java
Set<Class<?>> restControllers = ReflectionUtil.scanAnnotatedClass(packageName, RestController.class);
```



牵扯到了另一个类  同样引另外一个 md文件

ReflectionUtil  类



##### 传入的注解  RestController.class    Component.class  Aspect.class

```java
Set<Class<?>> restControllers = ReflectionUtil.scanAnnotatedClass(packageName, RestController.class);
Set<Class<?>> components = ReflectionUtil.scanAnnotatedClass(packageName, Component.class);
Set<Class<?>> aspects = ReflectionUtil.scanAnnotatedClass(packageName, Aspect.class);
```

这个方法是获取所有这种方法的类

RestController.class    Component.class  Aspect.class

这是三个自定义类



因为比较简单  所以放在这边



```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface RestController {
    String value() default "";
}
```

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Component {
    String name() default "";
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Aspect {
    String value() default "";
}
```

类似于三个自定义注解





### 可以测试下  

这里是   RestController 使用到的类   那么我们 来 看 这个的返回值是不是如此

![image-20211209140933898](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209140933898.png?x-oss-process=style/qingyun)

![image-20211209141200398](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209141200398.png?x-oss-process=style/qingyun)



使用范围是   com.github.demo  正好是此六个类

0 = {Class@1204} "class com.github.demo.aop.StudentController"
1 = {Class@1205} "class com.github.demo.circularDependency.CircularDependencyController"
2 = {Class@1206} "class com.github.demo.config.ConfigController"
3 = {Class@1207} "class com.github.demo.sms.SmsController"
4 = {Class@1208} "class com.github.demo.user.UserController"
5 = {Class@1209} "class com.github.demo.validation.CarController"





#####  Component  使用到的类   

![image-20211209143650779](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209143650779.png?x-oss-process=style/qingyun)





加载了这八个类

0 = {Class@1272} "class com.github.demo.aop.ascept.HeadMasterAspect"
1 = {Class@1273} "class com.github.demo.aop.ascept.TeacherAspect"
2 = {Class@1274} "class com.github.demo.aop.StudentServiceImpl"
3 = {Class@1275} "class com.github.demo.circularDependency.CircularDependencyAImpl"
4 = {Class@1276} "class com.github.demo.circularDependency.CircularDependencyBImpl"
5 = {Class@1277} "class com.github.demo.circularDependency.CircularDependencyCImpl"
6 = {Class@1278} "class com.github.demo.sms.AliSmsServiceImpl"
7 = {Class@1279} "class com.github.demo.sms.QiNiuSmsServiceImpl"
8 = {Class@1280} "class com.github.demo.user.UserService"







#####  Aspect使用到的类

![image-20211209143822073](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209143822073.png?x-oss-process=style/qingyun)



加载了两个类

0 = {Class@1272} "class com.github.demo.aop.ascept.HeadMasterAspect"
1 = {Class@1273} "class com.github.demo.aop.ascept.TeacherAspect"





依据目前来看  我们并不知道这些类的作用   所以先不去这些类中解析  如果遇到 则进入解析



```java
CLASSES.put(RestController.class, restControllers);
CLASSES.put(Component.class, components);
CLASSES.put(Aspect.class, aspects);
```



放入了  这里的 类加载器  （）

也就是

```
public static final Map<Class<? extends Annotation>, Set<Class<?>>> CLASSES = new ConcurrentHashMap<>();
```

Map格式



## 本类的主要作用

加载了 某些注解下的类  返回了 一个 Map<Class<? extends Annotation>, Set<Class<?>>> CLASSES