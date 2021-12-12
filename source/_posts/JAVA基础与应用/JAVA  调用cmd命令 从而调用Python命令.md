---
title: JAVA调用CMD
tags:
  - JAVA
categories: JAVA
description: JAVA调用Python
abbrlink: 3445041162
date: 2021-11-26 16:48:35
---


## JAVA  调用cmd命令 从而调用Python命令



### CMD的命令

```cmd
cmd /c cd /f F:\JS\CuteR\CuteR-master\CuteR-master && CuteR -c 10 -e H -o sample_output.png -v 10 sample_input.png http://www.chinuno.com
```



### 本质是通过传参数实现调用python命令来实现效果

```java
package edu.sdut.CMD;

import java.io.IOException;

/**
 * 调用CMD命令
 * @author qingyun
 * @version 1.0
 * @date 2021/9/29 16:49
 */
public class CallCmd {


    /**
     * 异步方式调用CMD生成二维码
     * @param InputPath  输入的图片地址
     * @param OutPath    输出的图片地址
     * @param content    图片二维码的内容
     */
    public static void Cmd(String InputPath , String OutPath ,String content) throws IOException {
        Runtime.getRuntime().exec("cmd /c cd  F:\\JS\\CuteR\\CuteR-master\\CuteR-master && CuteR -c 10 -e H -o "+OutPath+" -v 10 "+InputPath+" "+content);
    }


    public static void main(String[] args) throws IOException {
//            Cmd(null,null);
    }
}

```

### 下面的写法可通过控制线程来判断是否完成任务，达到同步执行线程的效果

```java
package edu.sdut.CMD;

import java.io.IOException;

/**
 * 调用CMD命令
 * @author qingyun
 * @version 1.0
 * @date 2021/9/29 16:49
 */
public class CallCmd {


    /**
     * 通过控制子线程判断是否执行完毕
     * 异步方式调用CMD生成二维码
     * @param InputPath  输入的图片地址
     * @param OutPath    输出的图片地址
     * @param content    图片二维码的内容
     */
    public static Integer Cmd(String InputPath , String OutPath ,String content) throws IOException, InterruptedException {
        // Process可以控制该子进程的执行或获取该子进程的信息
        Process process;
        process = Runtime.getRuntime().exec("cmd /c cd  F:\\JS\\CuteR\\CuteR-master\\CuteR-master && CuteR -c 10 -e H -o "+OutPath+" -v 10 "+InputPath+" "+content);
        int i = process.waitFor();
        return i;
    }


    public static void main(String[] args) throws IOException, InterruptedException {
//            Cmd(null,null);
        Integer cmd = Cmd("", "", "");
        System.out.println(cmd);
    }
}

```

