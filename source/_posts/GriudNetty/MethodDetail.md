---
title: MethodDetail
abbrlink: 47795634
date: 2021-12-12 15:37:35
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---


### MethodDetail

```java
package com.github.jsoncat.core.springmvc.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;
import java.util.regex.Pattern;

/**
 * @author shuang.kou
 * @createTime 2020年09月28日 09:42:00
 **/
@Data
@AllArgsConstructor
@NoArgsConstructor
public class MethodDetail {
    // target method
    private Method method;
    // url parameter mapping
    private Map<String, String> urlParameterMappings;
    // url query parameter mapping
    private Map<String, String> queryParameterMappings;
    // json type http post request data
    private String json;

    public void build(String requestPath, Map<String, Method> requestMappings, Map<String, String> urlMappings) {
        requestMappings.forEach((key, value) -> {
            Pattern pattern = Pattern.compile(key);
            boolean b = pattern.matcher(requestPath).find();
            if (b) {
                this.setMethod(value);
                String url = urlMappings.get(key);
                Map<String, String> urlParameterMappings = getUrlParameterMappings(requestPath, url);
                this.setUrlParameterMappings(urlParameterMappings);
            }
        });
    }

    /**
     * Match the request path parameter to the URL parameter
     * <p>
     * eg: requestPath="/user/1" url="/user/{id}"
     * this method will return:{"id" -> "1","user" -> "user"}
     * </p>
     */
    private Map<String, String> getUrlParameterMappings(String requestPath, String url) {
        String[] requestParams = requestPath.split("/");
        String[] urlParams = url.split("/");
        Map<String, String> urlParameterMappings = new HashMap<>();
        for (int i = 1; i < urlParams.length; i++) {
            urlParameterMappings.put(urlParams[i].replace("{", "").replace("}", ""), requestParams[i]);
        }
        return urlParameterMappings;
    }

}
```

![image-20211212095651615](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211212095651615.png?x-oss-process=style/qingyun)

调用了方法



```java
public void build(String requestPath, Map<String, Method> requestMappings, Map<String, String> urlMappings) {
    requestMappings.forEach((key, value) -> {
        Pattern pattern = Pattern.compile(key);
        boolean b = pattern.matcher(requestPath).find();
        if (b) {
            this.setMethod(value);
            String url = urlMappings.get(key);
            Map<String, String> urlParameterMappings = getUrlParameterMappings(requestPath, url);
            this.setUrlParameterMappings(urlParameterMappings);
        }
    });
}
```

```java
/**
 * Match the request path parameter to the URL parameter
 * <p>
 * eg: requestPath="/user/1" url="/user/{id}"
 * this method will return:{"id" -> "1","user" -> "user"}
 * </p>
 */
private Map<String, String> getUrlParameterMappings(String requestPath, String url) {
    String[] requestParams = requestPath.split("/");
    String[] urlParams = url.split("/");
    Map<String, String> urlParameterMappings = new HashMap<>();
    for (int i = 1; i < urlParams.length; i++) {
        urlParameterMappings.put(urlParams[i].replace("{", "").replace("}", ""), requestParams[i]);
    }
    return urlParameterMappings;
}
```



这两个方法一起作用的

这里通过方法匹配 匹配到了相应的方法请求



![image-20211212100050177](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211212100050177.png?x-oss-process=style/qingyun)

