---
title: RouteMethodMapper
abbrlink: 1511266000
date: 2021-12-12 15:36:01
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
#### RouteMethodMapper

```java
package com.github.jsoncat.core.springmvc.factory;

import com.github.jsoncat.annotation.springmvc.GetMapping;
import com.github.jsoncat.annotation.springmvc.PostMapping;
import com.github.jsoncat.annotation.springmvc.RestController;
import com.github.jsoncat.core.springmvc.entity.MethodDetail;
import com.github.jsoncat.factory.ClassFactory;
import io.netty.handler.codec.http.HttpMethod;

import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

/**
 * Save routing-related mapping information
 *
 * @author shuang.kou
 * @createTime 2020年09月29日 13:27:00
 **/
public class RouteMethodMapper {
    public static final HttpMethod[] HTTP_METHODS = {HttpMethod.GET, HttpMethod.POST};

    // key : http method
    // value : url -> method
    private static final Map<HttpMethod, Map<String, Method>> REQUEST_METHOD_MAP = new HashMap<>(2);
    // key : http method
    // value : formatted url -> original url
    private static final Map<HttpMethod, Map<String, String>> REQUEST_URL_MAP = new HashMap<>(2);


    static {
        for (HttpMethod httpMethod : HTTP_METHODS) {
            REQUEST_METHOD_MAP.put(httpMethod, new HashMap<>(128));
            REQUEST_URL_MAP.put(httpMethod, new HashMap<>(128));
        }
    }

    public static void loadRoutes() {
        Set<Class<?>> classes = ClassFactory.CLASSES.get(RestController.class);
        for (Class<?> aClass : classes) {
            RestController restController = aClass.getAnnotation(RestController.class);
            if (null != restController) {
                Method[] methods = aClass.getDeclaredMethods();
                String baseUrl = restController.value();
                for (Method method : methods) {
                    if (method.isAnnotationPresent(GetMapping.class)) {
                        GetMapping getMapping = method.getAnnotation(GetMapping.class);
                        if (getMapping != null) {
                            mapUrlToMethod(baseUrl + getMapping.value(), method, HttpMethod.GET);
                        }
                    }
                    if (method.isAnnotationPresent(PostMapping.class)) {
                        PostMapping postMapping = method.getAnnotation(PostMapping.class);
                        if (postMapping != null) {
                            mapUrlToMethod(baseUrl + postMapping.value(), method, HttpMethod.POST);
                        }
                    }
                }
            }
        }
    }

    public static MethodDetail getMethodDetail(String requestPath, HttpMethod httpMethod) {
        MethodDetail methodDetail = new MethodDetail();
        methodDetail.build(requestPath, REQUEST_METHOD_MAP.get(httpMethod), REQUEST_URL_MAP.get(httpMethod));
        return methodDetail;
    }

    /**
     * correspond url to method
     */
    private static void mapUrlToMethod(String url, Method method, HttpMethod httpMethod) {
        String formattedUrl = formatUrl(url);
        Map<String, Method> urlToMethodMap = REQUEST_METHOD_MAP.get(httpMethod);
        Map<String, String> formattedUrlToUrlMap = REQUEST_URL_MAP.get(httpMethod);
        if (urlToMethodMap.containsKey(formattedUrl)) {
            throw new IllegalArgumentException(String.format("duplicate url: %s", url));
        }
        urlToMethodMap.put(formattedUrl, method);
        formattedUrlToUrlMap.put(formattedUrl, url);
        REQUEST_METHOD_MAP.put(httpMethod, urlToMethodMap);
        REQUEST_URL_MAP.put(httpMethod, formattedUrlToUrlMap);
    }

    /**
     * format the url
     * for example : "/user/{name}" -> "^/user/[\u4e00-\u9fa5_a-zA-Z0-9]+/?$"
     */
    private static String formatUrl(String url) {
        // replace {xxx} placeholders with regular expressions matching Chinese, English letters and numbers, and underscores
        String originPattern = url.replaceAll("(\\{\\w+})", "[\\\\u4e00-\\\\u9fa5_a-zA-Z0-9]+");
        String pattern = "^" + originPattern + "/?$";
        return pattern.replaceAll("/+", "/");
    }

}
```



### 这里是开局的初始化操作

```Java
public static final HttpMethod[] HTTP_METHODS = {HttpMethod.GET, HttpMethod.POST};

// key : http method
// value : url -> method
private static final Map<HttpMethod, Map<String, Method>> REQUEST_METHOD_MAP = new HashMap<>(2);
// key : http method
// value : formatted url -> original url
private static final Map<HttpMethod, Map<String, String>> REQUEST_URL_MAP = new HashMap<>(2);


static {
    for (HttpMethod httpMethod : HTTP_METHODS) {
        REQUEST_METHOD_MAP.put(httpMethod, new HashMap<>(128));
        REQUEST_URL_MAP.put(httpMethod, new HashMap<>(128));
    }
}
```



```
HTTP_METHODS = {HttpMethod[2]@1360} 

0 = {HttpMethod@1318} "GET"
1 = {HttpMethod@1319} "POST"
```

```
REQUEST_URL_MAP = {HashMap@1341}  size = 2

{HttpMethod@1319} "POST" -> {HashMap@1358}  size = 0
{HttpMethod@1318} "GET" -> {HashMap@1351}  size = 0
```

```
REQUEST_URL_MAP = {HashMap@1341}  size = 2
{HttpMethod@1319} "POST" -> {HashMap@1367}  size = 0
{HttpMethod@1318} "GET" -> {HashMap@1368}  size = 0
```



### 首先 ApplicationContext.run中调用的方法是   loadRoutes（）

```java
public static void loadRoutes() {
    Set<Class<?>> classes = ClassFactory.CLASSES.get(RestController.class);
    for (Class<?> aClass : classes) {
        RestController restController = aClass.getAnnotation(RestController.class);
        if (null != restController) {
            Method[] methods = aClass.getDeclaredMethods();
            String baseUrl = restController.value();
            for (Method method : methods) {
                if (method.isAnnotationPresent(GetMapping.class)) {
                    GetMapping getMapping = method.getAnnotation(GetMapping.class);
                    if (getMapping != null) {
                        mapUrlToMethod(baseUrl + getMapping.value(), method, HttpMethod.GET);
                    }
                }
                if (method.isAnnotationPresent(PostMapping.class)) {
                    PostMapping postMapping = method.getAnnotation(PostMapping.class);
                    if (postMapping != null) {
                        mapUrlToMethod(baseUrl + postMapping.value(), method, HttpMethod.POST);
                    }
                }
            }
        }
    }
}
```

下面来解析

```java
Set<Class<?>> classes = ClassFactory.CLASSES.get(RestController.class);
```



获取的是   上个方法的    类加载器

获取了RestController 类  正好六个

```
for (Class<?> aClass : classes) { ... }
```

遍历



```java
        ///**
            // * 反射获取运行时注解
            // */
RestController restController = aClass.getAnnotation(RestController.class);
```

主要目的是构建类 获取注解值   为了后续扩展



```
Method[] methods = aClass.getDeclaredMethods();
```

//获取方法（反射） 2:getDeclaredMethods(),该方法是获取本类中的所有方法，包括私有的(private、protected、默认以及public)的方法。



```
//获取写的注解值
String baseUrl = restController.value();
```



```
//遍历方法列表
for (Method method : methods) { ... }
```





```
/**
 * 获取Method中参数的所有注解
 * public Annotation[][] getParameterAnnotations();
 * isAnnotationPresent(GetMapping.class)  判断方法列表类型
 */
if (method.isAnnotationPresent(GetMapping.class)) {}
```





```
mapUrlToMethod(baseUrl + getMapping.value(), method, HttpMethod.GET);
```

向本文中的方法传递值（这里传递了 当前  /student/summy   方法   请求方式）





### mapUrlToMethod(String url, Method method, HttpMethod httpMethod)

```java
/**
 * correspond url to method
 */
private static void mapUrlToMethod(String url, Method method, HttpMethod httpMethod) {
    String formattedUrl = formatUrl(url);
    Map<String, Method> urlToMethodMap = REQUEST_METHOD_MAP.get(httpMethod);
    Map<String, String> formattedUrlToUrlMap = REQUEST_URL_MAP.get(httpMethod);
    if (urlToMethodMap.containsKey(formattedUrl)) {
        throw new IllegalArgumentException(String.format("duplicate url: %s", url));
    }
    urlToMethodMap.put(formattedUrl, method);
    formattedUrlToUrlMap.put(formattedUrl, url);
    REQUEST_METHOD_MAP.put(httpMethod, urlToMethodMap);
    REQUEST_URL_MAP.put(httpMethod, formattedUrlToUrlMap);
}
```



 String formattedUrl = formatUrl(url);  替换字符串操作

```java
private static String formatUrl(String url) {
    // replace {xxx} placeholders with regular expressions matching Chinese, English letters and numbers, and underscores
    String originPattern = url.replaceAll("(\\{\\w+})", "[\\\\u4e00-\\\\u9fa5_a-zA-Z0-9]+");
    String pattern = "^" + originPattern + "/?$";
    return pattern.replaceAll("/+", "/");
}
```

/student/summary   -> ^/student/summary/?$



获取操作

```Java
     Map<String, Method> urlToMethodMap = REQUEST_METHOD_MAP.get(httpMethod);
        Map<String, String> formattedUrlToUrlMap = REQUEST_URL_MAP.get(httpMethod);
```

获取了两个空的HashMap   类似于工厂模式



```java
if (urlToMethodMap.containsKey(formattedUrl)) {
    throw new IllegalArgumentException(String.format("duplicate url: %s", url));
}
```

判断是否存在   如果存在重复则  抛出异常



```java
urlToMethodMap.put(formattedUrl, method);  //method class com.github.demo.aop.StudentController
formattedUrlToUrlMap.put(formattedUrl, url); //url   /student/summary
REQUEST_METHOD_MAP.put(httpMethod, urlToMethodMap);
REQUEST_URL_MAP.put(httpMethod, formattedUrlToUrlMap);
```

加入对应的URL和请求地址   

![image-20211209191438354](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209191438354.png?x-oss-process=style/qingyun)



## 本类的最后作用是

![image-20211209191844115](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209191844115.png?x-oss-process=style/qingyun)

目前是加载了RestController 注解的 所有方法的 Map <Url  方法>   以及  Map<Url 请求地址>





```java
public static MethodDetail getMethodDetail(String requestPath, HttpMethod httpMethod) {
    MethodDetail methodDetail = new MethodDetail();
    methodDetail.build(requestPath, REQUEST_METHOD_MAP.get(httpMethod), REQUEST_URL_MAP.get(httpMethod));
    return methodDetail;
}
```

由  GetRequestHandler 中所调用的方法

这里会调用 MethodDetail.build方法





