---
title: Java使用Redis删除指定前缀Key
tags:
  - JAVA
  - 转载
categories:
  - JAVA
  - 转载
abbrlink: 3850070033
date: 2022-03-02 09:38:54
---
转载：https://gitee.com/sdutitlab/LearningNotes/blob/master/Java/Java%E4%BD%BF%E7%94%A8Redis%E5%88%A0%E9%99%A4%E6%8C%87%E5%AE%9A%E5%89%8D%E7%BC%80Key/README.md#

前言
--

最近很多模块使用了Redis进行数据的缓存，然后遇到一个问题就是删除缓存，有的键是这样的方式进行存储的


我们能发现，它们都是有特定的前缀的，如果我们需要根据指定前缀删除的话，因为redis没有提供根据前缀来删除key的方法

但是提供了另外一个方法，就是根据模糊查询出符合条件的key，然后在调用delete方法删除，具体代码为

    // 获取Redis中特定前缀
    Set<String> keys = stringRedisTemplate.keys("BLOG_SORT_BY_MONTH:"  + "*");
    
    // 删除
    stringRedisTemplate.delete(keys);

> 需要注意的是：keys的操作会导致数据库暂时被锁住，其他的请求都会被堵塞；业务量大的时候会出问题