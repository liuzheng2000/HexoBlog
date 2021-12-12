---
title: JsonCatApplication中的Banner
abbrlink: 4098930299
date: 2021-12-12 15:39:54
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
### JsonCatApplication   中的 Banner 类

```Java
package com.github.jsoncat.common;

import java.io.IOException;
import java.net.URISyntaxException;
import java.net.URL;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * @author shuang.kou
 * @createTime 2020年09月22日 18:07:00
 **/
public class Banner {
    // banner made by https://www.bootschool.net/ascii
    public static final String DEFAULT_BANNER_NAME = "default-banner.txt";
    public static final String CUSTOM_BANNER_NAME = "banner.txt";

    public static void print() {
        URL url = Thread.currentThread().getContextClassLoader().getResource(CUSTOM_BANNER_NAME);
        if (url != null) {
            try {
                Path path = Paths.get(url.toURI());
                Files.lines(path).forEach(System.out::println);
            } catch (URISyntaxException | IOException e) { }
        } else {
            url = Thread.currentThread().getContextClassLoader().getResource(DEFAULT_BANNER_NAME);
            try {
                Path path = Paths.get(url.toURI());
                Files.lines(path).forEach(System.out::println);
            } catch (URISyntaxException | IOException e) { }
        }
    }
}
```

```
default-banner.txt   
banner.txt
```

这两个都是加载的启动标志   .txt文件





```java
public static void print() {
    URL url = Thread.currentThread().getContextClassLoader().getResource(CUSTOM_BANNER_NAME);
    if (url != null) {
        try {
            Path path = Paths.get(url.toURI());
            Files.lines(path).forEach(System.out::println);
        } catch (URISyntaxException | IOException e) { }
    } else {
        url = Thread.currentThread().getContextClassLoader().getResource(DEFAULT_BANNER_NAME);
        try {
            Path path = Paths.get(url.toURI());
            Files.lines(path).forEach(System.out::println);
        } catch (URISyntaxException | IOException e) { }
    }
}
```

使用的主要方法

```
Thread.currentThread().getContextClassLoader().getResource(CUSTOM_BANNER_NAME);
```

获取资源的相对文件地址



```
Path path = Paths.get(url.toURI());
Files.lines(path).forEach(System.out::println);
```

打印文件中的每一行

