---
title: DependencyInjection
abbrlink: 2216014141
date: 2021-12-12 15:43:48
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
### DependencyInjection



```java
package com.github.jsoncat.core.ioc;

import lombok.extern.slf4j.Slf4j;

import java.util.Map;

/**
 * @author shuang.kou & tom
 * @createTime 2020年09月30日 07:51:00
 **/
@Slf4j
public class DependencyInjection {


    /**
     * 遍历ioc容器所有bean的属性, 为所有带@Autowired/@Value注解的属性注入实例
     */
    public static void inject(String[] packageNames) {
        AutowiredBeanInitialization autowiredBeanInitialization = new AutowiredBeanInitialization(packageNames);
        Map<String, Object> beans = BeanFactory.BEANS;
        if (beans.size() > 0) {
            BeanFactory.BEANS.values().forEach(autowiredBeanInitialization::initialize);
        }
    }

}
```

这里也牵扯到了一个类  AutowiredBeanInitialization  up自己写的一个类

而且比较重要，故引入另一个md文件进行解析







```
Map<String, Object> beans = BeanFactory.BEANS;
```

这里是获取了之前加载的类   那几个类  和配置文件类等等





```java
if (beans.size() > 0) {
    BeanFactory.BEANS.values().forEach(autowiredBeanInitialization::initialize);
}
```

这地方是把所有的类都拿了出来   一个一个调用  autowiredBeanInitialization.initialize()初始化方法

