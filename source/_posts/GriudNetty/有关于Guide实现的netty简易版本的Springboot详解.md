---
title: 有关于Guide实现的netty简易版本的Springboot详解
abbrlink: 2412069045
date: 2021-12-12 15:31:59
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
## 有关于Guide实现的netty简易版本的Springboot详解



### 一 JsonCatApplication类

```java
package com.github.demo;

import com.github.jsoncat.annotation.boot.ComponentScan;
import com.github.jsoncat.annotation.boot.SpringBootApplication;
import com.github.jsoncat.core.ApplicationContext;

/**
 * @author shuang.kou
 * @createTime 2020年09月23日 17:30:00
 **/
@SpringBootApplication
@ComponentScan(value = {"com.github.demo"})
public class JsonCatApplication {

    public static void main(String[] args) {
        JsonCatApplication.run(JsonCatApplication.class, args);
    }

    public static void run(Class<?> applicationClass, String... arg) {
        ApplicationContext applicationContext = ApplicationContext.getApplicationContext();
        applicationContext.run(applicationClass);
    }
}

```

### SpringbootApplication  注解

#### 一  @SpringBootApplication

这里是 Guide 自主实现的注解  作用如下

```java
package com.github.jsoncat.annotation.boot;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@ComponentScan
public @interface SpringBootApplication {

}
```

**注 ：** SpringbootApplication  在另外一篇完成（本文所有具体内容均独立出来，防止过长）

@ComponentScan  此注解是自主实现有单独的类 故拿出来

```java
package com.github.jsoncat.annotation.boot;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
public @interface ComponentScan {
    String[] value() default {};
}
```

与 SpringBootApplication 注解相同 故不再解释

#### @ComponentScan(value = {"com.github.demo"})

此注解 是 赋值的作用  在后续的 可通过方法取出（  可看博客 自定义注解的实现实现 ）

### SpringbootApplication   方法解析

```java
JsonCatApplication.run(JsonCatApplication.class, args);
```

运行下面方法 run方法

```java
public static void run(Class<?> applicationClass, String... arg) {
    ApplicationContext applicationContext = ApplicationContext.getApplicationContext();
    applicationContext.run(applicationClass);
}
```

Run 为主要方法

```Java
ApplicationContext.getApplicationContext();
```

运行  ApplicationContext 方法 进入  ApplicationContext  去看（Guide实现的类）









## 二 ApplicationContext类

```java
package com.github.jsoncat.core;

import com.github.jsoncat.annotation.boot.ComponentScan;
import com.github.jsoncat.common.Banner;
import com.github.jsoncat.core.aop.factory.InterceptorFactory;
import com.github.jsoncat.core.boot.ApplicationRunner;
import com.github.jsoncat.core.config.Configuration;
import com.github.jsoncat.core.config.ConfigurationManager;
import com.github.jsoncat.core.ioc.BeanFactory;
import com.github.jsoncat.core.ioc.DependencyInjection;
import com.github.jsoncat.core.springmvc.factory.RouteMethodMapper;
import com.github.jsoncat.factory.ClassFactory;
import com.github.jsoncat.server.HttpServer;

import java.net.URISyntaxException;
import java.net.URL;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.LinkedHashSet;
import java.util.List;
import java.util.Objects;

/**
 * @author shuang.kou
 * @createTime 2020年09月24日 16:49:00
 **/
public final class ApplicationContext {
    private static final ApplicationContext APPLICATION_CONTEXT = new ApplicationContext();


    public void run(Class<?> applicationClass) {
        //print banner
        Banner.print();
        //analyse package
        String[] packageNames = getPackageNames(applicationClass);
        // Load classes with custom annotation
        ClassFactory.loadClass(packageNames);
        // Load routes
        RouteMethodMapper.loadRoutes();
        // Load beans managed by the ioc container
        BeanFactory.loadBeans();
        //load configuration
        loadResources(applicationClass);
        // Load interceptors
        InterceptorFactory.loadInterceptors(packageNames);
        // Traverse all the beans in the ioc container and inject instances for all @Autowired annotated attributes.
        DependencyInjection.inject(packageNames);
        // Applies bean post processors on the classes which are from ClassFactory.
        // For example, the class annotated by @Component or @RestController.
        BeanFactory.applyBeanPostProcessors();
        // Perform some callback events
        callRunners();
    }

    public static ApplicationContext getApplicationContext() {
        return APPLICATION_CONTEXT;
    }

    private static String[] getPackageNames(Class<?> applicationClass) {
        ComponentScan componentScan = applicationClass.getAnnotation(ComponentScan.class);
        return !Objects.isNull(componentScan) ? componentScan.value()
                : new String[]{applicationClass.getPackage().getName()};
    }

    private void callRunners() {
        List<ApplicationRunner> runners = new ArrayList<>(BeanFactory.getBeansOfType(ApplicationRunner.class).values());
        //The last step is to start web application
        runners.add(() -> {
            HttpServer httpServer = new HttpServer();
            httpServer.start();
        });
        for (Object runner : new LinkedHashSet<>(runners)) {
            ((ApplicationRunner) runner).run();
        }
    }


    private void loadResources(Class<?> applicationClass) {
        ClassLoader classLoader = applicationClass.getClassLoader();
        List<Path> filePaths = new ArrayList<>();
        for (String configName : Configuration.DEFAULT_CONFIG_NAMES) {
            URL url = classLoader.getResource(configName);
            if (!Objects.isNull(url)) {
                try {
                    filePaths.add(Paths.get(url.toURI()));
                } catch (URISyntaxException ignored) {
                }
            }
        }
        ConfigurationManager configurationManager = BeanFactory.getBean(ConfigurationManager.class);
        configurationManager.loadResources(filePaths);
    }
}
```

ApplicationContext 方法解析

先进入JsonCatApplication.run中调用的方法  ApplicationContext.getApplicationContext();

```java
 private static final ApplicationContext APPLICATION_CONTEXT = new ApplicationContext();

public static ApplicationContext getApplicationContext() {
    return APPLICATION_CONTEXT;
}
```

返回了一个单例类

如何运行  applicationContext.run(applicationClass);

```java
public void run(Class<?> applicationClass) {
    //print banner
    Banner.print();
    //analyse package
    String[] packageNames = getPackageNames(applicationClass);
    // Load classes with custom annotation
    ClassFactory.loadClass(packageNames);
    // Load routes
    RouteMethodMapper.loadRoutes();
    // Load beans managed by the ioc container
    BeanFactory.loadBeans();
    //load configuration
    loadResources(applicationClass);
    // Load interceptors
    InterceptorFactory.loadInterceptors(packageNames);
    // Traverse all the beans in the ioc container and inject instances for all @Autowired annotated attributes.
    DependencyInjection.inject(packageNames);
    // Applies bean post processors on the classes which are from ClassFactory.
    // For example, the class annotated by @Component or @RestController.
    BeanFactory.applyBeanPostProcessors();
    // Perform some callback events
    callRunners();
}
```

ApplicationContext.run是主要方法

Class<?> applicationClass   （传入的是单例）

```java
Banner.print();
```

引入 Banner  类（拉到一个新的md文件去理解）  （Banner  主要作用是打印标识）



```
String[] packageNames = getPackageNames(applicationClass);
```

本类中的方法

```java
private static String[] getPackageNames(Class<?> applicationClass) {
    ComponentScan componentScan = applicationClass.getAnnotation(ComponentScan.class);
    return !Objects.isNull(componentScan) ? componentScan.value()
            : new String[]{applicationClass.getPackage().getName()};
}
```



#### ClassFactory

```
ClassFactory.loadClass(packageNames);
```

引入ClassFactory 类 （拉到一个新的md文件去理解）



最终结果是加载了类，而且根据注解进行了划分

#### RouteMethodMapper

```java
 RouteMethodMapper.loadRoutes();
```

同样 我们拉到一个新的md文件去理解





#### BeanFactory

```
BeanFactory.loadBeans();
```

还是继续拉入到一个新的md文件理解





#### loadResources

```
 loadResources(applicationClass);
```

调用了

```java
private void loadResources(Class<?> applicationClass) {
    ClassLoader classLoader = applicationClass.getClassLoader();
    List<Path> filePaths = new ArrayList<>();
    for (String configName : Configuration.DEFAULT_CONFIG_NAMES) {
        URL url = classLoader.getResource(configName);
        if (!Objects.isNull(url)) {
            try {
                filePaths.add(Paths.get(url.toURI()));
            } catch (URISyntaxException ignored) {
            }
        }
    }
    ConfigurationManager configurationManager = BeanFactory.getBean(ConfigurationManager.class);
    configurationManager.loadResources(filePaths);
}
```

传入值是主类值      获取路径下的配置文件 

```java
 ClassLoader classLoader = applicationClass.getClassLoader();
    List<Path> filePaths = new ArrayList<>();
    for (String configName : Configuration.DEFAULT_CONFIG_NAMES) {
        URL url = classLoader.getResource(configName);
        if (!Objects.isNull(url)) {
            try {
                filePaths.add(Paths.get(url.toURI()));
            } catch (URISyntaxException ignored) {
            }
        }
    }
```

这里的作用作用主要是为了File 中找到配置文件的位置  （能找到是编译后的位置哦）

![image-20211209210437671](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209210437671.png?x-oss-process=style/qingyun)

```java
ConfigurationManager configurationManager = BeanFactory.getBean(ConfigurationManager.class);
configurationManager.loadResources(filePaths);
```

BeanFactory.getBean 获取之前加载的类   ConfigurationManager.class（是一个空的配置类）

调用ConfigurationManager 的 loadResources 方法

这里到  ConfigurationManager  中去解释  

主要是加载配置文件内容





### InterceptorFactory.loadInterceptors(packageNames);

InterceptorFactory 是一个单独的类  

新建一个md文件进行解析



#### DependencyInjection.inject(packageNames);

引入了一个新的类

DependencyInjection

还是单独拉去一个md文件解析

![image-20211211213315588](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211211213315588.png?x-oss-process=style/qingyun)

这个地方  他本身的实现也是有问题的，但他总体的实现就是添加了反射构造而成的AOP类，将AOP实现了反射





#### BeanFactory.applyBeanPostProcessors();

调用了类新的方法   本质是AOP类





#### callRunners();

调用了本类中的方法

```java
private void callRunners() {
    List<ApplicationRunner> runners = new ArrayList<>(BeanFactory.getBeansOfType(ApplicationRunner.class).values());
    //The last step is to start web application
    runners.add(() -> {
        HttpServer httpServer = new HttpServer();
        httpServer.start();
    });
    for (Object runner : new LinkedHashSet<>(runners)) {
        ((ApplicationRunner) runner).run();
    }
}
```

调用    BeanFactory.getBeansOfType(ApplicationRunner.class)  方法   返回了一个空的数组



HttpServer httpServer = new HttpServer();

新建一个HttpServer()

httpServer.start();

将方法装  调用方法



HttpServer.run是主要调用的方法

引出一个.md文件

