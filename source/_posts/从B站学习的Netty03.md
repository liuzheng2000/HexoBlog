---
title: 从B站学习的Netty03
abbrlink: 3500663986
date: 2021-11-29 10:04:08
tags:
  - 转载
  - netty
  - 风祈的时光录
categories: 
  - 转载 
  - netty
copyright: false
---
 转载: https://imlql.cn/
## NIO与零拷贝

> 1、尚硅谷这里的零拷贝感觉讲的感觉有点问题，但是为了笔记的完整性，任然保留了这里的笔记。不过笔者考虑再写一篇零拷贝。
>
> 2、而且这里课件的图也看不太清
>
> 3、读者可以将我写的零拷贝和尚硅谷这里讲的零拷贝对照着看，取长补短

### 零拷贝基本介绍

1. 零拷贝是网络编程的关键，很多性能优化都离不开。
2. 在 `Java` 程序中，常用的零拷贝有 `mmap`（内存映射）和 `sendFile`。那么，他们在 `OS` 里，到底是怎么样的一个的设计？我们分析 `mmap` 和 `sendFile` 这两个零拷贝
3. 另外我们看下 `NIO` 中如何使用零拷贝

### 传统 IO 数据读写

`Java` 传统 `IO` 和网络编程的一段代码

```java
File file = new File("test.txt");
RandomAccessFile raf = new RandomAccessFile(file, "rw");

byte[] arr = new byte[(int) file.length()];
raf.read(arr);

Socket socket = new ServerSocket(8080).accept();
socket.getOutputStream().write(arr);
```

### 传统 IO 模型

![img](http://typa.qingyun.run/img/0024.png)

**DMA**：`direct memory access` 直接内存拷贝（不使用 `CPU`）

### mmap 优化

1. `mmap` 通过内存映射，将文件映射到内核缓冲区，同时，用户空间可以共享内核空间的数据。这样，在进行网络传输时，就可以减少内核空间到用户空间的拷贝次数。如下图
2. `mmap` 示意图

![img](http://typa.qingyun.run/img/0025.png)

### sendFile 优化

1. `Linux2.1` 版本提供了 `sendFile` 函数，其基本原理如下：数据根本不经过用户态，直接从内核缓冲区进入到 `SocketBuffer`，同时，由于和用户态完全无关，就减少了一次上下文切换
2. 示意图和小结

![img](http://typa.qingyun.run/img/0026.png?x-oss-process=style/qingyun)

1. 提示：零拷贝从操作系统角度，是没有 `cpu` 拷贝
2. `Linux在2.4` 版本中，做了一些修改，避免了从内核缓冲区拷贝到 `Socketbuffer` 的操作，直接拷贝到协议栈，从而再一次减少了数据拷贝。具体如下图和小结：

![img](http://typa.qingyun.run/img/0027.png)

1. 这里其实有一次 `cpu` 拷贝 `kernel buffer` -> `socket buffer` 但是，拷贝的信息很少，比如 `lenght`、`offset` 消耗低，可以忽略

### 零拷贝的再次理解

1. 我们说零拷贝，是从操作系统的角度来说的。因为内核缓冲区之间，没有数据是重复的（只有 `kernel buffer` 有一份数据）。
2. 零拷贝不仅仅带来更少的数据复制，还能带来其他的性能优势，例如更少的上下文切换，更少的 `CPU` 缓存伪共享以及无 `CPU` 校验和计算。

### mmap 和 sendFile 的区别

1. `mmap` 适合小数据量读写，`sendFile` 适合大文件传输。
2. `mmap` 需要 `4` 次上下文切换，`3` 次数据拷贝；`sendFile` 需要 `3` 次上下文切换，最少 `2` 次数据拷贝。
3. `sendFile` 可以利用 `DMA` 方式，减少 `CPU` 拷贝，`mmap` 则不能（必须从内核拷贝到 `Socket`缓冲区）







