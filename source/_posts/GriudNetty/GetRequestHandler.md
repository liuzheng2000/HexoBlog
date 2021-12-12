---
title: GetRequestHandler
abbrlink: 2308705898
date: 2021-12-12 15:52:02
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
### GetRequestHandler

```java
package com.github.jsoncat.core.springmvc.handler;

import com.github.jsoncat.core.ioc.BeanFactory;
import com.github.jsoncat.core.ioc.BeanHelper;
import com.github.jsoncat.core.springmvc.entity.MethodDetail;
import com.github.jsoncat.core.springmvc.factory.FullHttpResponseFactory;
import com.github.jsoncat.core.springmvc.factory.ParameterResolverFactory;
import com.github.jsoncat.core.springmvc.factory.RouteMethodMapper;
import com.github.jsoncat.core.springmvc.resolver.ParameterResolver;
import com.github.jsoncat.core.springmvc.util.UrlUtil;
import io.netty.handler.codec.http.FullHttpRequest;
import io.netty.handler.codec.http.FullHttpResponse;
import io.netty.handler.codec.http.HttpMethod;
import io.netty.handler.codec.http.QueryStringDecoder;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.codec.CharEncoding;
import org.apache.commons.codec.Charsets;

import java.lang.reflect.Method;
import java.lang.reflect.Parameter;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Handle get request
 *
 * @author shuang.kou
 * @createTime 2020年09月24日 13:33:00
 **/
@Slf4j
public class GetRequestHandler implements RequestHandler {
    @Override
    public FullHttpResponse handle(FullHttpRequest fullHttpRequest) {
        String requestUri = fullHttpRequest.uri();
        Map<String, String> queryParameterMappings = getQueryParams(requestUri);
        // get http request path，such as "/user"
        String requestPath = UrlUtil.getRequestPath(requestUri);
        // get target method
        MethodDetail methodDetail = RouteMethodMapper.getMethodDetail(requestPath, HttpMethod.GET);
        methodDetail.setQueryParameterMappings(queryParameterMappings);
        Method targetMethod = methodDetail.getMethod();
        if (targetMethod == null) {
            return null;
        }
        log.info("requestPath -> target method [{}]", targetMethod.getName());
        Parameter[] targetMethodParameters = targetMethod.getParameters();
        // target method parameters.
        // notice! you should convert it to array when pass into the executeMethod method
        List<Object> targetMethodParams = new ArrayList<>();
        for (Parameter parameter : targetMethodParameters) {
            ParameterResolver parameterResolver = ParameterResolverFactory.get(parameter);
            if (parameterResolver != null) {
                Object param = parameterResolver.resolve(methodDetail, parameter);
                targetMethodParams.add(param);
            }
        }
        String beanName = BeanHelper.getBeanName(methodDetail.getMethod().getDeclaringClass());
        Object targetObject = BeanFactory.BEANS.get(beanName);
        return FullHttpResponseFactory.getSuccessResponse(targetMethod, targetMethodParams, targetObject);
    }

    /**
     * get the parameters of uri
     */
    private Map<String, String> getQueryParams(String uri) {
        QueryStringDecoder queryDecoder = new QueryStringDecoder(uri, Charsets.toCharset(CharEncoding.UTF_8));
        Map<String, List<String>> parameters = queryDecoder.parameters();
        Map<String, String> queryParams = new HashMap<>();
        for (Map.Entry<String, List<String>> attr : parameters.entrySet()) {
            for (String attrVal : attr.getValue()) {
                queryParams.put(attr.getKey(), attrVal);
            }
        }
        return queryParams;
    }
}
```





```java
String requestUri = fullHttpRequest.uri();
Map<String, String> queryParameterMappings = getQueryParams(requestUri);
```

会调用这里的方法**getQueryParams**



```java
private Map<String, String> getQueryParams(String uri) {
    QueryStringDecoder queryDecoder = new QueryStringDecoder(uri, Charsets.toCharset(CharEncoding.UTF_8));
    Map<String, List<String>> parameters = queryDecoder.parameters();
    Map<String, String> queryParams = new HashMap<>();
    for (Map.Entry<String, List<String>> attr : parameters.entrySet()) {
        for (String attrVal : attr.getValue()) {
            queryParams.put(attr.getKey(), attrVal);
        }
    }
    return queryParams;
}
```

得到参数Url

![image-20211212094935975](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211212094935975.png?x-oss-process=style/qingyun)





```java
MethodDetail methodDetail = RouteMethodMapper.getMethodDetail(requestPath, HttpMethod.GET);
```

进入的方法





```java
methodDetail.setQueryParameterMappings(queryParameterMappings);
Method targetMethod = methodDetail.getMethod();
if (targetMethod == null) {
    return null;
}
log.info("requestPath -> target method [{}]", targetMethod.getName());
```

这里是获取到了目标方法



```java
Parameter[] targetMethodParameters = targetMethod.getParameters();
```

获取到可执行参数









```java
for (Parameter parameter : targetMethodParameters) {
    ParameterResolver parameterResolver = ParameterResolverFactory.get(parameter);
    if (parameterResolver != null) {
        Object param = parameterResolver.resolve(methodDetail, parameter);
        targetMethodParams.add(param);
    }
}
```

进行了参数匹配

```java
ParameterResolver parameterResolver = ParameterResolverFactory.get(parameter);
```

判断了参数类型

```java
package com.github.jsoncat.core.springmvc.factory;

import com.github.jsoncat.annotation.springmvc.PathVariable;
import com.github.jsoncat.annotation.springmvc.RequestBody;
import com.github.jsoncat.annotation.springmvc.RequestParam;
import com.github.jsoncat.core.springmvc.resolver.ParameterResolver;
import com.github.jsoncat.core.springmvc.resolver.PathVariableParameterResolver;
import com.github.jsoncat.core.springmvc.resolver.RequestBodyParameterResolver;
import com.github.jsoncat.core.springmvc.resolver.RequestParamParameterResolver;

import java.lang.reflect.Parameter;

/**
 * @author shuang.kou
 * @createTime 2020年09月28日 10:39:00
 **/
public class ParameterResolverFactory {

    public static ParameterResolver get(Parameter parameter) {
        if (parameter.isAnnotationPresent(RequestParam.class)) {
            return new RequestParamParameterResolver();
        }
        if (parameter.isAnnotationPresent(PathVariable.class)) {
            return new PathVariableParameterResolver();
        }
        if (parameter.isAnnotationPresent(RequestBody.class)) {
            return new RequestBodyParameterResolver();
        }
        return null;
    }
}
```

具体的就不说了





```java
if (parameterResolver != null) {
    Object param = parameterResolver.resolve(methodDetail, parameter);
    targetMethodParams.add(param);
}
```

调用了ParameterResolver.resolve方法

```java
package com.github.jsoncat.core.springmvc.resolver;

import com.github.jsoncat.annotation.springmvc.PathVariable;
import com.github.jsoncat.common.util.ObjectUtil;
import com.github.jsoncat.core.springmvc.entity.MethodDetail;

import java.lang.reflect.Parameter;
import java.util.Map;

/**
 * process @PathVariable annotation
 *
 * @author shuang.kou
 * @createTime 2020年09月27日 20:58:00
 **/
public class PathVariableParameterResolver implements ParameterResolver {
    @Override
    public Object resolve(MethodDetail methodDetail, Parameter parameter) {
        PathVariable pathVariable = parameter.getDeclaredAnnotation(PathVariable.class);
        String requestParameter = pathVariable.value();
        Map<String, String> urlParameterMappings = methodDetail.getUrlParameterMappings();
        String requestParameterValue = urlParameterMappings.get(requestParameter);
        return ObjectUtil.convert(parameter.getType(), requestParameterValue); //就是返回了相应类型
    }
}
```

因为类比较简单，故直接拿出

parameter.getDeclaredAnnotation(PathVariable.class);  返回直接存在于此元素上的所有注解

![image-20211212102424168](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211212102424168.png?x-oss-process=style/qingyun)

这里总体来说 是添加了方法字符串Map







```java
String beanName = BeanHelper.getBeanName(methodDetail.getMethod().getDeclaringClass());
```

获取方法的  类  -> 取出来类的名字





```java
Object targetObject = BeanFactory.BEANS.get(beanName);
```

调出来相应的实例类



```java
return FullHttpResponseFactory.getSuccessResponse(targetMethod, targetMethodParams, targetObject);
```

牵扯到了一个新的类  FullHttpResponseFactory



