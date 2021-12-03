---
title: 从B站学习的Netty01
abbrlink: 1051297182
date: 2021-11-29 09:42:25
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

### Selector（选择器）

### 基本介绍

1. `Java` 的 `NIO`，用非阻塞的 `IO` 方式。可以用一个线程，处理多个的客户端连接，就会使用到 `Selector`（选择器）。
2. `Selector` 能够检测多个注册的通道上是否有事件发生（注意：多个 `Channel` 以事件的方式可以注册到同一个 `Selector`），如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求。
3. 只有在连接/通道真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程。
4. 避免了多线程之间的上下文切换导致的开销。

### Selector 示意图和特点说明

![img](http://typa.qingyun.run/img/0017.png)

说明如下：

1. `Netty` 的 `IO` 线程 `NioEventLoop` 聚合了 `Selector`（选择器，也叫多路复用器），可以同时并发处理成百上千个客户端连接。
2. 当线程从某客户端 `Socket` 通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。
3. 线程通常将非阻塞 `IO` 的空闲时间用于在其他通道上执行 `IO` 操作，所以单独的线程可以管理多个输入和输出通道。
4. 由于读写操作都是非阻塞的，这就可以充分提升 `IO` 线程的运行效率，避免由于频繁 `I/O` 阻塞导致的线程挂起。
5. 一个 `I/O` 线程可以并发处理 `N` 个客户端连接和读写操作，这从根本上解决了传统同步阻塞 `I/O` 一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升。

### Selector 类相关方法

![img](http://typa.qingyun.run/img/0018.png)

### 注意事项

1. `NIO` 中的 `ServerSocketChannel` 功能类似 `ServerSocket`、`SocketChannel` 功能类似 `Socket`。

2. ```
   Selector
   ```

    相关方法说明

   - `selector.select();` //阻塞
   - `selector.select(1000);` //阻塞 1000 毫秒，在 1000 毫秒后返回
   - `selector.wakeup();` //唤醒 selector
   - `selector.selectNow();` //不阻塞，立马返还

## NIO 非阻塞网络编程原理分析图

`NIO` 非阻塞网络编程相关的（`Selector`、`SelectionKey`、`ServerScoketChannel` 和 `SocketChannel`）关系梳理图

![img](http://typa.qingyun.run/img/0019.png)

对上图的说明：

1. 当客户端连接时，会通过 `ServerSocketChannel` 得到 `SocketChannel`。
2. `Selector` 进行监听 `select` 方法，返回有事件发生的通道的个数。
3. 将 `socketChannel` 注册到 `Selector` 上，`register(Selector sel, int ops)`，一个 `Selector` 上可以注册多个 `SocketChannel`。
4. 注册后返回一个 `SelectionKey`，会和该 `Selector` 关联（集合）。
5. 进一步得到各个 `SelectionKey`（有事件发生）。
6. 在通过 `SelectionKey` 反向获取 `SocketChannel`，方法 `channel()`。
7. 可以通过得到的 `channel`，完成业务处理。

## NIO 非阻塞网络编程快速入门

案例：

1. 编写一个 `NIO` 入门案例，实现服务器端和客户端之间的数据简单通讯（非阻塞）
2. 目的：理解 `NIO` 非阻塞网络编程机制

```java
package edu.IO.query;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

/**
 * @author qingyun
 * @version 1.0
 * @date 2021/11/22 20:51
 */
public class NIOServer {
    public static void main(String[] args) throws Exception{

        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        Selector selector = Selector.open();

        serverSocketChannel.socket().bind(new InetSocketAddress(6666));

        serverSocketChannel.configureBlocking(false);

        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        while (true){
            if (selector.select(1000) == 0){
                System.out.println("服务器等待了1秒钟，无连接");
                continue;
            }

            Set<SelectionKey> selectionKeys = selector.selectedKeys();

            Iterator<SelectionKey> keyiterator = selectionKeys.iterator();

            while (keyiterator.hasNext()){
                SelectionKey key = keyiterator.next();
                if (key.isAcceptable()){
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    socketChannel.register(selector,SelectionKey.OP_READ, ByteBuffer.allocate(1024));
                }
                if (key.isConnectable()){
                    SocketChannel channel = (SocketChannel) key.channel();
                    ByteBuffer buffer = (ByteBuffer) key.attachment();
                    channel.read(buffer);
                    System.out.println("form 客户端" + new String(buffer.array()));
                }
                keyiterator.remove();
            }


        }
    }
}

```

> pos1：
>
> 1、对操作系统有一定了解的同学，就会大概知道这里监听的是一个Accept通道。这个通道的
> 作用就是监听，实际建立连接了还会有一个通道。
> 2、简单说一下为什么。因为客户端发请求的时候，服务器这边是肯定要先有一个监听通道，
> 监听某个端口是否有客户端要建立链接，如果有客户端想要建立链接，那么会再创建一个和
> 客户端真正通信的通道。
> 3、如果有其它客户端还想要建立链接，这个Accept监听端口监听到了，就会再创建几个真正
> 的通信通道。
> 4、也就是Server的一个端口可以建立多个TCP连接，因为IP层协议通过
> 目标地址+端口+源地址+源端口四个信息识别一个上下文
>
> **顺便插一句嘴：因为学netty的过程中，发现计算机网络和操作系统蛮重要的，所以接下来会写几篇这方面的文章**

```java
package edu.IO.query;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

/**
 * @author qingyun
 * @version 1.0
 * @date 2021/11/22 21:08
 */
public class NIOClient {
    public static void main(String[] args) throws Exception{

        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.configureBlocking(false);

        InetSocketAddress inetSocketAddress = new InetSocketAddress(6666);

        if (!socketChannel.connect(inetSocketAddress)){
            while (!socketChannel.finishConnect()){
                System.out.println("连接服务器需要时间，客户端不会阻塞，可以做其他工作");
            }
        }
        String str = "hello ,lingYun";
        ByteBuffer byteBuffer = ByteBuffer.wrap(str.getBytes());
        socketChannel.write(byteBuffer);
        byteBuffer.clear();
        socketChannel.read(byteBuffer);

        System.in.read();
    }
}

```

### SelectionKey

1. ```
   SelectionKey
   ```

   ，表示 

   ```
   Selector
   ```

    和网络通道的注册关系，共四种：

   - `int OP_ACCEPT`：有新的网络连接可以 `accept`，值为 `16`
   - `int OP_CONNECT`：代表连接已经建立，值为 `8`
   - `int OP_READ`：代表读操作，值为 `1`
   - `int OP_WRITE`：代表写操作，值为 `4`

源码中：

```java
public static final int OP_READ = 1 << 0;
public static final int OP_WRITE = 1 << 2;
public static final int OP_CONNECT = 1 << 3;
public static final int OP_ACCEPT = 1 << 4;
```

`SelectionKey` 相关方法

![img](http://typa.qingyun.run/img/0020.png)

### ServerSocketChannel

1. `ServerSocketChannel` 在服务器端监听新的客户端 `Socket` 连接，负责监听，不负责实际的读写操作
2. 相关方法如下

![img](http://typa.qingyun.run/img/0021.png)

### SocketChannel

1. `SocketChannel`，网络 `IO` 通道，**具体负责进行读写操作**。`NIO` 把缓冲区的数据写入通道，或者把通道里的数据读到缓冲区。
2. 相关方法如下

![img](http://typa.qingyun.run/img/0022.png)

