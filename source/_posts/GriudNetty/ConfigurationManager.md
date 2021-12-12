---
title: ConfigurationManager
abbrlink: 544312368
date: 2021-12-12 15:44:33
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---


### ConfigurationManager

```java
package com.github.jsoncat.core.config;

import com.github.jsoncat.core.config.resource.ResourceLoader;
import com.github.jsoncat.core.config.resource.property.PropertiesResourceLoader;
import com.github.jsoncat.core.config.resource.yaml.YamlResourceLoader;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;
import java.nio.file.Path;
import java.util.List;

@Slf4j
public class ConfigurationManager implements Configuration {

    private static final String PROPERTIES_FILE_EXTENSION = ".properties";

    private static final String YAML_FILE_EXTENSION = ".yaml";

    private static final String YML_FILE_EXTENSION = ".yml";


    private final Configuration configuration;

    public ConfigurationManager(Configuration configuration) {
        this.configuration = configuration;
    }

    @Override
    public int getInt(String id) {
        return configuration.getInt(id);
    }

    @Override
    public String getString(String id) {
        return configuration.getString(id);
    }

    @Override
    public Boolean getBoolean(String id) {
        return configuration.getBoolean(id);
    }

    @Override
    public void loadResources(List<Path> resourcePaths) {
        try {
            for (Path resourcePath : resourcePaths) {
                String fileName = resourcePath.getFileName().toString();
                if (fileName.endsWith(PROPERTIES_FILE_EXTENSION)) {
                    ResourceLoader resourceLoader = new PropertiesResourceLoader();
                    configuration.putAll(resourceLoader.loadResource(resourcePath));
                } else if (fileName.endsWith(YML_FILE_EXTENSION) || fileName.endsWith(YAML_FILE_EXTENSION)) {
                    ResourceLoader resourceLoader = new YamlResourceLoader();
                    configuration.putAll(resourceLoader.loadResource(resourcePath));
                }
            }
        } catch (IOException ex) {
            log.error("not load resources.");
            System.exit(-1);
        }
    }
}
```





```java
new ConfigurationManager(ConfigurationFactory.getConfig())
```



调用了有参构造函数

```java
public ConfigurationManager(Configuration configuration) {
    this.configuration = configuration;
}
```

此函数的传值是一个新的类         Configuration  接口  









ConfigurationManager 实现了 Configuration

```java
package com.github.jsoncat.core.config;

import java.nio.file.Path;
import java.util.List;
import java.util.Map;

public interface Configuration {

    String[] DEFAULT_CONFIG_NAMES = {"application.properties", "application.yaml"};

    int getInt(String id);

    String getString(String id);

    Boolean getBoolean(String id);

    default void put(String id, String content) {
    }

    default void putAll(Map<String, String> maps) {
    }

    default void loadResources(List<Path> resourcePaths) {
    }
}
```





### configurationManager.loadResources(filePaths);

调用的方法

```java
@Override
public void loadResources(List<Path> resourcePaths) {
    try {
        for (Path resourcePath : resourcePaths) {
            String fileName = resourcePath.getFileName().toString();
            if (fileName.endsWith(PROPERTIES_FILE_EXTENSION)) {
                ResourceLoader resourceLoader = new PropertiesResourceLoader();
                configuration.putAll(resourceLoader.loadResource(resourcePath));
            } else if (fileName.endsWith(YML_FILE_EXTENSION) || fileName.endsWith(YAML_FILE_EXTENSION)) {
                ResourceLoader resourceLoader = new YamlResourceLoader();
                configuration.putAll(resourceLoader.loadResource(resourcePath));
            }
        }
    } catch (IOException ex) {
        log.error("not load resources.");
        System.exit(-1);
    }
}
```

 String fileName = resourcePath.getFileName().toString();  获取路径名

```
 if (fileName.endsWith(PROPERTIES_FILE_EXTENSION)) {}
```

判断是否按照这个结尾

```
 ResourceLoader resourceLoader = new PropertiesResourceLoader();  
```

引入了一个新的类  ResourceLoader



###  **configuration.putAll(resourceLoader.loadResource(resourcePath));**

这里是将解析的配置文件中的内容都放入了 configuration.putAll 中  也就是 DefaultConfiguration类

![image-20211209213809732](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209213809732.png?x-oss-process=style/qingyun)

![image-20211209213731692](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209213733713.png?x-oss-process=style/qingyun)

这是放入的两个配置文件

![image-20211209213917802](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209213917802.png?x-oss-process=style/qingyun)

一共读取的五个配置信息



