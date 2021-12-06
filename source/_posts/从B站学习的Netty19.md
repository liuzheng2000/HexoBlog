---
title: 从B站学习的Netty19
abbrlink: 694762733
date: 2021-12-06 13:22:49
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
用 Netty 自己实现简单的RPC

## RPC 基本介绍

1. `RPC（Remote Procedure Call）`—远程过程调用，是一个计算机通信协议。该协议允许运行于一台计算机的程序调用另一台计算机的子程序，而程序员无需额外地为这个交互作用编程
2. 两个或多个应用程序都分布在不同的服务器上，它们之间的调用都像是本地方法调用一样(如图)

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0021.png?x-oss-process=style/qingyun)

过程：

1. 调用者(`Caller`)，调用远程API(`Remote API`)
2. 调用远程API会通过一个RPC代理(`RpcProxy`)
3. RPC代理再去调用`RpcInvoker`(这个是PRC的调用者)
4. `RpcInvoker`通过RPC连接器(`RpcConnector`)
5. RPC连接器用两台机器规定好的PRC协议(`RpcProtocol`)把数据进行编码
6. 接着RPC连接器通过RpcChannel通道发送到对方的PRC接收器(RpcAcceptor)
7. PRC接收器通过PRC协议进行解码拿到数据
8. 然后将数据传给`RpcProcessor`
9. `RpcProcessor`再传给`RpcInvoker`
10. `RpcInvoker`调用`Remote API`
11. 最后推给被调用者(Callee)

12. 常见的 `RPC` 框架有：比较知名的如阿里的 `Dubbo`、`Google` 的 `gRPC`、`Go` 语言的 `rpcx`、`Apache` 的 `thrift`，`Spring` 旗下的 `SpringCloud`。![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0022.png?x-oss-process=style/qingyun)

我们的RPC 调用流程图

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0023.png?x-oss-process=style/qingyun)

**RPC 调用流程说明**

1. 服务消费方（`client`）以本地调用方式调用服务
2. `client stub` 接收到调用后负责将方法、参数等封装成能够进行网络传输的消息体
3. `client stub` 将消息进行编码并发送到服务端
4. `server stub` 收到消息后进行解码
5. `server stub` 根据解码结果调用本地的服务
6. 本地服务执行并将结果返回给 `server stub`
7. `server stub` 将返回导入结果进行编码并发送至消费方
8. `client stub` 接收到消息并进行解码
9. 服务消费方（`client`）得到结果

小结：`RPC` 的目标就是将 `2 - 8` 这些步骤都封装起来，用户无需关心这些细节，可以像调用本地方法一样即可完成远程服务调用

## 己实现 Dubbo RPC（基于 Netty）

### 需求说明

1. `Dubbo` 底层使用了 `Netty` 作为网络通讯框架，要求用 `Netty` 实现一个简单的 `RPC` 框架
2. 模仿 `Dubbo`，消费者和提供者约定接口和协议，消费者远程调用提供者的服务，提供者返回一个字符串，消费者打印提供者返回的数据。底层网络通信使用 `Netty 4.1.20`

### 设计说明

1. 创建一个接口，定义抽象方法。用于消费者和提供者之间的约定。
2. 创建一个提供者，该类需要监听消费者的请求，并按照约定返回数据。
3. 创建一个消费者，该类需要透明的调用自己不存在的方法，内部需要使用 `Netty` 请求提供者返回数据
4. 开发的分析图

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0024.png?x-oss-process=style/qingyun)

### 代码

封装的RPC

> 可以把这块代码理解成封装的dubbo

NettyServer

```java
package com.atguigu.netty.dubborpc.netty;


import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

public class NettyServer {


    public static void startServer(String hostName, int port) {
        startServer0(hostName,port);
    }

    //编写一个方法，完成对NettyServer的初始化和启动

    private static void startServer0(String hostname, int port) {

        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();

            serverBootstrap.group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                                      @Override
                                      protected void initChannel(SocketChannel ch) throws Exception {
                                          ChannelPipeline pipeline = ch.pipeline();
                                          pipeline.addLast(new StringDecoder());
                                          pipeline.addLast(new StringEncoder());
                                          pipeline.addLast(new NettyServerHandler()); //业务处理器

                                      }
                                  }

                    );

            ChannelFuture channelFuture = serverBootstrap.bind(hostname, port).sync();
            System.out.println("服务提供方开始提供服务~~");
            channelFuture.channel().closeFuture().sync();

        }catch (Exception e) {
            e.printStackTrace();
        }
        finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }
}
```

NettyServerHandler

```java
package com.atguigu.netty.dubborpc.netty;


import com.atguigu.netty.dubborpc.customer.ClientBootstrap;
import com.atguigu.netty.dubborpc.provider.HelloServiceImpl;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

//服务器这边handler比较简单
public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("---服务端开始收到来自客户单的消息---");
        //获取客户端发送的消息，并调用服务
        System.out.println("原始消息：" + msg);

        /*
         1.客户端在调用服务器的api 时，我们需要定义一个协议，比如我们要求 每次发消息是都
         必须以某个字符串开头 "HelloService#hello#你好"
         2.Dubbo注册在Zookeeper里时，这种就是类的全路径字符串，你用IDEA的zookeeper插件
         就可以清楚地看到
         */
        if(msg.toString().startsWith(ClientBootstrap.providerName)) {

            String result = new HelloServiceImpl().hello(msg.toString().substring(msg.toString().lastIndexOf("#") + 1));
            ctx.writeAndFlush(result);
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

NettyClientHandler

```java
package com.atguigu.netty.dubborpc.netty;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.util.concurrent.Callable;

public class NettyClientHandler extends ChannelInboundHandlerAdapter implements Callable {

    private ChannelHandlerContext context;//上下文
    private String result; //返回的结果
    private String para; //客户端调用方法时，传入的参数


    //与服务器的连接创建后，就会被调用, 这个方法是第一个被调用(1)
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(" channelActive 被调用  ");
        context = ctx; //因为我们在其它方法会使用到 ctx
    }

    //收到服务器的数据后，调用方法 (4)
    //
    @Override
    public synchronized void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println(" channelRead 被调用  ");
        result = msg.toString();
        notify(); //唤醒等待的线程
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }

    //被代理对象调用, 发送数据给服务器，-> wait -> 等待被唤醒(channelRead) -> 返回结果 (3)-》5
    @Override
    public synchronized Object call() throws Exception {
        System.out.println(" call1 被调用  ");
        context.writeAndFlush(para);
        //进行wait
        wait(); //等待channelRead 方法获取到服务器的结果后，唤醒
        System.out.println(" call2 被调用  ");
        return  result; //服务方返回的结果

    }
    //(2)
    void setPara(String para) {
        System.out.println(" setPara  ");
        this.para = para;
    }
}
```

##### NettyClient

```java
package com.atguigu.netty.dubborpc.netty;


import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

import java.lang.reflect.Proxy;
import java.util.concurrent.Executor;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class NettyClient {

    //创建线程池
    private static ExecutorService executor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

    private static NettyClientHandler client;
    private int count = 0;

    //编写方法使用代理模式，获取一个代理对象

    public Object getBean(final Class<?> serivceClass, final String providerName) {

        return Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
                new Class<?>[]{serivceClass}, (proxy, method, args) -> {

                    System.out.println("(proxy, method, args) 进入...." + (++count) + " 次");
                    //{}  部分的代码，客户端每调用一次 hello, 就会进入到该代码
                    if (client == null) {
                        initClient();
                    }

                    //设置要发给服务器端的信息
                    //providerName：协议头，args[0]：就是客户端要发送给服务端的数据
                    client.setPara(providerName + args[0]);

                    //
                    return executor.submit(client).get();

                });
    }

    //初始化客户端
    private static void initClient() {
        client = new NettyClientHandler();
        //创建EventLoopGroup
        NioEventLoopGroup group = new NioEventLoopGroup();
        Bootstrap bootstrap = new Bootstrap();
        bootstrap.group(group)
                .channel(NioSocketChannel.class)
                .option(ChannelOption.TCP_NODELAY, true)
                .handler(
                        new ChannelInitializer<SocketChannel>() {
                            @Override
                            protected void initChannel(SocketChannel ch) throws Exception {
                                ChannelPipeline pipeline = ch.pipeline();
                                pipeline.addLast(new StringDecoder());
                                pipeline.addLast(new StringEncoder());
                                pipeline.addLast(client);
                            }
                        }
                );

        try {
            bootstrap.connect("127.0.0.1", 7000).sync();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

接口

```java
package com.atguigu.netty.dubborpc.publicinterface;

//这个是接口，是服务提供方和 服务消费方都需要
public interface HelloService {

    String hello(String mes);
}
```

#### 服务端(provider)

HelloServiceImpl

```java
package com.atguigu.netty.dubborpc.provider;

import com.atguigu.netty.dubborpc.publicinterface.HelloService;

public class HelloServiceImpl implements HelloService{

    private static int count = 0;
    //当有消费方调用该方法时， 就返回一个结果
    @Override
    public String hello(String mes) {
        System.out.println("收到客户端消息=" + mes);
        System.out.println();
        //根据mes 返回不同的结果
        if(mes != null) {
            return "你好客户端, 我已经收到你的消息。消息为：[" + mes + "] ，第" + (++count) + " 次 \n";
        } else {
            return "你好客户端, 我已经收到你的消息 ";
        }
    }
}
```

ServerBootstrap

```java
package com.atguigu.netty.dubborpc.provider;

import com.atguigu.netty.dubborpc.netty.NettyServer;

//ServerBootstrap 会启动一个服务提供者，就是 NettyServer
public class ServerBootstrap {
    public static void main(String[] args) {

        //代码代填..
        NettyServer.startServer("127.0.0.1", 7000);
    }
}
```

客户端(消费者)

```java
package com.atguigu.netty.dubborpc.customer;

import com.atguigu.netty.dubborpc.netty.NettyClient;
import com.atguigu.netty.dubborpc.publicinterface.HelloService;

public class ClientBootstrap {


    //这里定义协议头
    public static final String providerName = "HelloService#hello#";

    public static void main(String[] args) throws  Exception{

        //创建一个消费者
        NettyClient customer = new NettyClient();

        //创建代理对象
        HelloService service = (HelloService) customer.getBean(HelloService.class, providerName);

        for (;; ) {
            Thread.sleep(2 * 1000);
            //通过代理对象调用服务提供者的方法(服务)
            String res = service.hello("你好 dubbo~");
            System.out.println("调用的结果 res= " + res);
        }
    }
}
```

### 调用过程

1. `ClientBootstrap#main`发起调用
2. 走到下面这一行代码后

```java
HelloService service = (HelloService) customer.getBean(HelloService.class, providerName);
```

1. 调用`NettyClient#getBean`，在此方法里与服务端建立链接。

2. 于是就执行`NettyClientHandler#channelActive`

3. 接着回到`NettyClient#getBean`调用`NettyClientHandler#setPara`，调用完之后再回到`NettyClient#getBean`，用线程池提交任务

4. 因为用线程池提交了任务，就准备执行`NettyClientHandler#call`线程任务

5. 在`NettyClientHandler#call`中发送数据给服务提供者

   ```
   context.writeAndFlush(para);
   ```

   由于还没收到服务提供者的数据结果，所以wait住

   来到了服务提供者这边，从Socket通道中收到了数据，所以执行`NettyServerHandler#channelRead`，然后因为此方法中执行了

   ```java
   String result = new HelloServiceImpl().hello(msg.toString().substring(msg.toString().lastIndexOf("#") + 1));
   ```

   就去`HelloServiceImpl#hello`中执行业务逻辑，返回数据给`NettyServerHandler#channelRead`，`NettyServerHandler#channelRead`再把数据发给客户端

   `NettyClientHandler#channelRead`收到服务提供者发来的数据，唤醒之前wait的线程

   所以之前wait的线程从`NettyClientHandler#call`苏醒，返回result给`NettyClient#getBean`

   `NettyClient#getBean`get()到数据，`ClientBootstrap#main`中的此函数调用返回，得到服务端提供的数据。

   ```java
   String res = service.hello("你好 dubbo~");
   ```

   

### 效果

**ClientBootstrap打印**

```java
(proxy, method, args) 进入....1 次
 setPara  
 channelActive 被调用  
 call1 被调用  
 channelRead 被调用  
 call2 被调用  
调用的结果 res= 你好客户端, 我已经收到你的消息。消息为：[你好 dubbo~] ，第1 次 

(proxy, method, args) 进入....2 次
 setPara  
 call1 被调用  
 channelRead 被调用  
 call2 被调用  
调用的结果 res= 你好客户端, 我已经收到你的消息。消息为：[你好 dubbo~] ，第2 次 

(proxy, method, args) 进入....3 次
 setPara  
 call1 被调用  
 channelRead 被调用  
 call2 被调用  
调用的结果 res= 你好客户端, 我已经收到你的消息。消息为：[你好 dubbo~] ，第3 次 

(proxy, method, args) 进入....4 次
 setPara  
 call1 被调用  
 channelRead 被调用  
 call2 被调用  
调用的结果 res= 你好客户端, 我已经收到你的消息。消息为：[你好 dubbo~] ，第4 次 

(proxy, method, args) 进入....5 次
 setPara  
 call1 被调用  
 channelRead 被调用  
 call2 被调用  
调用的结果 res= 你好客户端, 我已经收到你的消息。消息为：[你好 dubbo~] ，第5 次 
```

**ServerBootstrap打印**

```java
服务提供方开始提供服务~~
---服务端开始收到来自客户单的消息---
原始消息：HelloService#hello#你好 dubbo~
收到客户端消息=你好 dubbo~

---服务端开始收到来自客户单的消息---
原始消息：HelloService#hello#你好 dubbo~
收到客户端消息=你好 dubbo~

---服务端开始收到来自客户单的消息---
原始消息：HelloService#hello#你好 dubbo~
收到客户端消息=你好 dubbo~

---服务端开始收到来自客户单的消息---
原始消息：HelloService#hello#你好 dubbo~
收到客户端消息=你好 dubbo~

---服务端开始收到来自客户单的消息---
原始消息：HelloService#hello#你好 dubbo~
收到客户端消息=你好 dubbo~
```