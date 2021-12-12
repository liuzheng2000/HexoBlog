---
title: ConfigurationFactory
abbrlink: 560183152
date: 2021-12-12 15:45:21
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
### ConfigurationFactory

```
package com.github.jsoncat.core.config;

public class ConfigurationFactory {

    public static Configuration getConfig() {
        return SingleConfigurationHolder.INSTANCE;
    }

    private static class SingleConfigurationHolder {

        private static final Configuration INSTANCE = buildConfiguration();

        private static Configuration buildConfiguration() {
            return new DefaultConfiguration();
        }
    }
}
```

调用 getConfig（） 

返回

```java
private static final Configuration INSTANCE = buildConfiguration();
private static Configuration buildConfiguration() {
      return new DefaultConfiguration();
      }
```

故需要引入  DefaultConfiguration

```java
package com.github.jsoncat.core.config;

public class DefaultConfiguration extends AbstractConfiguration {

}
```



DefaultConfiguration   继承  AbstractConfiguration





```java
package com.github.jsoncat.core.config;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class AbstractConfiguration implements Configuration {

    private static final Map<String, String> CONFIGURATION_CACHE = new ConcurrentHashMap<>(64);

    @Override
    public int getInt(String id) {
        String result = CONFIGURATION_CACHE.get(id);
        return Integer.parseInt(result);
    }

    @Override
    public String getString(String id) {
        return CONFIGURATION_CACHE.get(id);
    }

    @Override
    public Boolean getBoolean(String id) {
        String result = CONFIGURATION_CACHE.get(id);
        return Boolean.parseBoolean(result);
    }

    @Override
    public void put(String id, String content) {
        CONFIGURATION_CACHE.put(id, content);
    }

    @Override
    public void putAll(Map<String, String> maps) {
        CONFIGURATION_CACHE.putAll(maps);
    }

}
```



```java
AbstractConfiguration implements Configuration  // 实现接口  Configuration
```



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