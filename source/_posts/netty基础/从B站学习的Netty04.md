---
title: 从B站学习的Netty04
abbrlink: 1321431313
date: 2021-11-29 10:05:22
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

案例要求：

1. 使用传统的 `IO` 方法传递一个大文件
2. 使用 `NIO` 零拷贝方式传递（`transferTo`）一个大文件
3. 看看两种传递方式耗时时间分别是多少

```java
package edu.IO.NewIOServer;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;

/**
 * @author qingyun
 * @version 1.0
 * @date 2021/11/27 14:28
 */
public class NewIoServer {
    public static void main(String[] args) throws IOException {
        InetSocketAddress address =  new InetSocketAddress(7001);
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        ServerSocket serverSocket = serverSocketChannel.socket();
        serverSocket.bind(address);

        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(4096);

        while (true){
            SocketChannel socketChannel = serverSocketChannel.accept();
            int readcount = 0;
            while (-1 != readcount){
                try{
                    readcount = socketChannel.read(byteBuffer);
                }catch (Exception e){
//                    e.printStackTrace();
                    break;
                }
                //倒带 position = 0 mark 作废
                byteBuffer.rewind();
            }
        }
    }

}

```

```java
package edu.IO.NewIOServer;

import java.io.FileInputStream;
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.FileChannel;
import java.nio.channels.SocketChannel;

/**
 * @author qingyun
 * @version 1.0
 * @date 2021/11/27 14:32
 */
public class NewIoClient {
    public static void main(String[] args) throws IOException {
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.connect(new InetSocketAddress("localhost",7001));
        String filename = "F:\\Customer.txt";
        FileChannel fileChannel = new FileInputStream(filename).getChannel();
        long startTime = System.currentTimeMillis();
        long transferCount = fileChannel.transferTo(0, fileChannel.size(), socketChannel);
        System.out.println("发送的总的字节数 = " + transferCount + " 耗时: " + (System.currentTimeMillis() - startTime));
        fileChannel.close();

    }
}

```

