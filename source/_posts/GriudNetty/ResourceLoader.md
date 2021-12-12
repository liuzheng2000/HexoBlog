---
title: ResourceLoader
abbrlink: 3558349110
date: 2021-12-12 15:36:49
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
引入  读取配置文件的新的 机制

接口



### ResourceLoader

```
package com.github.jsoncat.core.config.resource;

import java.io.IOException;
import java.nio.file.Path;
import java.util.Map;

public interface ResourceLoader {

    Map<String, String> loadResource(Path path) throws IOException;
}
```



次一级实现类

### AbstractResourceLoader

```java
package com.github.jsoncat.core.config.resource;

import java.io.IOException;
import java.nio.file.Path;
import java.util.Map;

public abstract class AbstractResourceLoader implements ResourceLoader {

    @Override
    public Map<String, String> loadResource(Path path) throws IOException {
        return loadResources(path);
    }

    protected abstract Map<String, String> loadResources(Path path) throws IOException;
}
```



### PropertiesResourceLoader

```java
package com.github.jsoncat.core.config.resource.property;

import com.github.jsoncat.core.config.resource.AbstractResourceLoader;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

// 读取  按照  application.properties 的配置文件内容
public class PropertiesResourceLoader extends AbstractResourceLoader {

    @Override
    protected Map<String, String> loadResources(Path path) throws IOException {
        Properties properties = new Properties();
        try (InputStream stream = Files.newInputStream(path); Reader reader = new InputStreamReader(stream)) {
            properties.load(reader);
        }
        Map<String, String> resource = new HashMap<>(properties.size());
        for (Map.Entry<Object, Object> entry : properties.entrySet()) {
            resource.put(entry.getKey().toString(), entry.getValue().toString());
        }
        return resource;
    }
}
```



### YamlResourceLoader

```java
package com.github.jsoncat.core.config.resource.yaml;

import com.github.jsoncat.core.config.resource.AbstractResourceLoader;
import org.yaml.snakeyaml.Yaml;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.Reader;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.LinkedHashMap;
import java.util.Map;

public class YamlResourceLoader extends AbstractResourceLoader {

    @Override
    protected Map<String, String> loadResources(Path path) throws IOException {

        Map<String, String> result = new LinkedHashMap<>();
        InputStream stream = null;
        Reader reader = null;
        try {
            Yaml yaml = new Yaml();
            stream = Files.newInputStream(path);
            reader = new InputStreamReader(stream);
            Map<String, Object> map = asMap(yaml.load(reader));
            buildFlattenedMap(result, map, null);
        } finally {
            if (stream != null) {
                stream.close();
            }
            if (reader != null) {
                reader.close();
            }
        }
        return result;
    }

    private void buildFlattenedMap(Map<String, String> result, Map<String, Object> source, String path) {
        source.forEach((key, value) -> {
            if (path != null && !path.isEmpty()) {
                key = path + '.' + key;
            }
            if (value instanceof String) {
                result.put(key, String.valueOf(value));
            } else if (value instanceof Map) {
                Map<String, Object> map = (Map<String, Object>) value;
                buildFlattenedMap(result, map, key);
            } else {
                result.put(key, (value != null ? String.valueOf(value) : ""));
            }
        });
    }


    private Map<String, Object> asMap(Object object) {
        Map<String, Object> result = new LinkedHashMap<>();

        Map<Object, Object> map = (Map<Object, Object>) object;
        map.forEach((key, value) -> {
            if (value instanceof Map) {
                value = asMap(value);
            }
            result.put(key.toString(), value);
        });
        return result;
    }
}
```











#### 具体实现   configuration.putAll(resourceLoader.loadResource(resourcePath));

```java
public class PropertiesResourceLoader extends AbstractResourceLoader {

    @Override
    protected Map<String, String> loadResources(Path path) throws IOException {
        //新建一个配置文件类
        Properties properties = new Properties();
        //读取流的内容
        try (InputStream stream = Files.newInputStream(path); Reader reader = new InputStreamReader(stream)) {
            properties.load(reader);
        }
        Map<String, String> resource = new HashMap<>(properties.size());
        // 放入Map
        for (Map.Entry<Object, Object> entry : properties.entrySet()) {
            resource.put(entry.getKey().toString(), entry.getValue().toString());
        }
        //返回
        return resource;
    }
}
```

#### 具体实现  ResourceLoader resourceLoader = new YamlResourceLoader();

#### configuration.putAll(resourceLoader.loadResource(resourcePath));

```java
protected Map<String, String> loadResources(Path path) throws IOException {

    Map<String, String> result = new LinkedHashMap<>();
    InputStream stream = null;
    Reader reader = null;
    try {
        Yaml yaml = new Yaml();
        stream = Files.newInputStream(path);
        reader = new InputStreamReader(stream);
        Map<String, Object> map = asMap(yaml.load(reader));
        buildFlattenedMap(result, map, null);
    } finally {
        if (stream != null) {
            stream.close();
        }
        if (reader != null) {
            reader.close();
        }
    }
    return result;
}
```

调用的方法如上

![image-20211209213113426](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211209213113426.png?x-oss-process=style/qingyun)

这里的主要作用是返回了配置文件的内容  结果如上   且改为了Map格式

