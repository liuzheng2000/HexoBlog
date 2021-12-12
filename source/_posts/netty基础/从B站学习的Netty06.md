---
title: 从B站学习的Netty06
abbrlink: 2697793597
date: 2021-11-29 10:07:39
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
# Netty

## 原生 NIO 存在的问题

1. `NIO` 的类库和 `API` 繁杂，使用麻烦：需要熟练掌握 `Selector`、`ServerSocketChannel`、`SocketChannel`、`ByteBuffer`等。
2. 需要具备其他的额外技能：要熟悉 `Java` 多线程编程，因为 `NIO` 编程涉及到 `Reactor` 模式，你必须对多线程和网络编程非常熟悉，才能编写出高质量的 `NIO` 程序。
3. 开发工作量和难度都非常大：例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常流的处理等等。
4. `JDK NIO` 的 `Bug`：例如臭名昭著的 `Epoll Bug`，它会导致 `Selector` 空轮询，最终导致 `CPU100%`。直到 `JDK1.7` 版本该问题仍旧存在，没有被根本解决。

## Netty 官网说明

官网：https://netty.io/

Netty is an asynchronous event-driven network application framework  for rapid development of maintainable high performance protocol servers  & clients.

![img](http://typa.qingyun.run/img/0001.png?x-oss-process=style/qingyun)

## Netty 的优点

`Netty` 对 `JDK` 自带的 `NIO` 的 `API` 进行了封装，解决了上述问题。

1. 设计优雅：适用于各种传输类型的统一 `API` 阻塞和非阻塞 `Socket`；基于灵活且可扩展的事件模型，可以清晰地分离关注点；高度可定制的线程模型-单线程，一个或多个线程池。
2. 使用方便：详细记录的 `Javadoc`，用户指南和示例；没有其他依赖项，`JDK5（Netty3.x）`或 `6（Netty4.x）`就足够了。
3. 高性能、吞吐量更高：延迟更低；减少资源消耗；最小化不必要的内存复制。
4. 安全：完整的 `SSL/TLS` 和 `StartTLS` 支持。
5. 社区活跃、不断更新：社区活跃，版本迭代周期短，发现的 `Bug` 可以被及时修复，同时，更多的新功能会被加入。

## Netty 版本说明

1. `Netty` 版本分为 `Netty 3.x` 和 `Netty 4.x`、`Netty 5.x`
2. 因为 `Netty 5` 出现重大 `bug`，已经被官网废弃了，目前推荐使用的是 `Netty 4.x`的稳定版本
3. 目前在官网可下载的版本 `Netty 3.x`、`Netty 4.0.x` 和 `Netty 4.1.x`
4. 在本套课程中，我们讲解 `Netty4.1.x` 版本
5. `Netty` 下载地址：https://bintray.com/netty/downloads/netty/

# Netty 高性能架构设计

## 线程模型基本介绍

1. 不同的线程模式，对程序的性能有很大影响，为了搞清 `Netty` 线程模式，我们来系统的讲解下各个线程模式，最后看看 `Netty` 线程模型有什么优越性。
2. 目前存在的线程模型有：传统阻塞 `I/O` 服务模型 和`Reactor` 模式
3. 根据 Reactor的数量和处理资源池线程的数量不同，有 3种典型的实现
   - 单 `Reactor` 单线程；
   - 单 `Reactor`多线程；
   - 主从 `Reactor`多线程
4. `Netty` 线程模式（`Netty` 主要基于主从 `Reactor` 多线程模型做了一定的改进，其中主从 `Reactor` 多线程模型有多个 `Reactor`）

## 传统阻塞 I/O 服务模型

### 工作原理图

1. 黄色的框表示对象，蓝色的框表示线程
2. 白色的框表示方法（`API`）

### 模型特点

1. 采用阻塞 `IO` 模式获取输入的数据
2. 每个连接都需要独立的线程完成数据的输入，业务处理，数据返回

### 问题分析

1. 当并发数很大，就会创建大量的线程，占用很大系统资源
2. 连接创建后，如果当前线程暂时没有数据可读，该线程会阻塞在 Handler对象中的`read` 操作，导致上面的处理线程资源浪费

[![img](http://typa.qingyun.run/img/0002.png?x-oss-process=style/qingyun)](https://unpkg.zhimg.com/youthlql@1.0.0/netty/introduction/chapter_002/0002.png)

## Reactor 模式

### 针对传统阻塞 I/O 服务模型的 2 个缺点，解决方案：

基于 `I/O` 复用模型：多个连接共用一个阻塞对象`ServiceHandler`，应用程序只需要在一个阻塞对象等待，无需阻塞等待所有连接。当某个连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理。

 `Reactor` 在不同书中的叫法：

1. 反应器模式 
2. 分发者模式（Dispatcher）
3. 通知者模式（notifier）
4. 基于线程池复用线程资源：不必再为每个连接创建线程，将连接完成后的业务处理任务分配给线程进行处理，一个线程可以处理多个连接的业务。（解决了当并发数很大时，会创建大量线程，占用很大系统资源）
5. 基于 `I/O` 复用模型：多个客户端进行连接，先把连接请求给`ServiceHandler`。多个连接共用一个阻塞对象`ServiceHandler`。假设，当C1连接没有数据要处理时，C1客户端只需要阻塞于`ServiceHandler`，C1之前的处理线程便可以处理其他有数据的连接，不会造成线程资源的浪费。当C1连接再次有数据时，`ServiceHandler`根据线程池的空闲状态，将请求分发给空闲的线程来处理C1连接的任务。（解决了线程资源浪费的那个问题）
6. ![img](http://typa.qingyun.run/img/0003.png?x-oss-process=style/qingyun)

I/O 复用结合线程池，就是 Reactor 模式基本设计思想，如图

![img](http://typa.qingyun.run/img/0004.png?x-oss-process=style/qingyun)

对上图说明：

1. `Reactor` 模式，通过一个或多个输入同时传递给服务处理器（ServiceHandler）的模式（基于事件驱动）
2. 服务器端程序处理传入的多个请求,并将它们同步分派到相应的处理线程，因此 `Reactor` 模式也叫 `Dispatcher` 模式
3. `Reactor` 模式使用 `IO` 复用监听事件，收到事件后，分发给某个线程（进程），这点就是网络服务器高并发处理关键

> 原先有多个Handler阻塞，现在只用一个ServiceHandler阻塞

#### Reactor 模式中核心组成

1. `Reactor（也就是那个ServiceHandler）`：`Reactor` 在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理线程来对 `IO` 事件做出反应。它就像公司的电话接线员，它接听来自客户的电话并将线路转移到适当的联系人；
2. `Handlers（处理线程EventHandler）`：处理线程执行 `I/O` 事件要完成的实际事件，类似于客户想要与之交谈的公司中的实际官员。`Reactor` 通过调度适当的处理线程来响应 `I/O` 事件，处理程序执行非阻塞操作。

#### Reactor 模式分类

根据 `Reactor` 的数量和处理资源池线程的数量不同，有 `3` 种典型的实现

1. 单 `Reactor` 单线程
2. 单 `Reactor` 多线程
3. 主从 `Reactor` 多线程

#### 单 Reactor 单线程

原理图，并使用 `NIO` 群聊系统验证![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0005.png?x-oss-process=style/qingyun)

#### 方案说明

1. `Select` 是前面 `I/O` 复用模型介绍的标准网络编程 `API`，可以实现应用程序通过一个阻塞对象监听多路连接请求
2. `Reactor` 对象通过 `Select` 监控客户端请求事件，收到事件后通过 `Dispatch` 进行分发
3. 如果是建立连接请求事件，则由 `Acceptor` 通过 `Accept` 处理连接请求，然后创建一个 `Handler` 对象处理连接完成后的后续业务处理
4. 如果不是建立连接事件，则 `Reactor` 会分发调用连接对应的 `Handler` 来响应
5. `Handler` 会完成 `Read` → 业务处理 → `Send` 的完整业务流程

结合实例：服务器端用一个线程通过多路复用搞定所有的 `IO` 操作（包括连接，读、写等），编码简单，清晰明了，但是如果客户端连接数量较多，将无法支撑，前面的 `NIO` 案例就属于这种模型。

### 方案优缺点分析

1. 优点：模型简单，没有多线程、进程通信、竞争的问题，全部都在一个线程中完成
2. 缺点：性能问题，只有一个线程，无法完全发挥多核 `CPU` 的性能。`Handler`在处理某个连接上的业务时，整个进程无法处理其他连接事件，很容易导致性能瓶颈
3. 缺点：可靠性问题，线程意外终止，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障
4. 使用场景：客户端的数量有限，业务处理非常快速，比如 `Redis` 在业务处理的时间复杂度 `O(1)` 的情况

#### 单 Reactor 多线程

方案说明

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0006.png?x-oss-process=style/qingyun)

1. `Reactor` 对象通过 `Select` 监控客户端请求事件，收到事件后，通过 `Dispatch` 进行分发
2. 如果是建立连接请求，则由 `Acceptor` 通过 `accept` 处理连接请求，然后创建一个 `Handler` 对象处理完成连接后的各种事件
3. 如果不是连接请求，则由 `Reactor` 分发调用连接对应的 `handler` 来处理（也就是说连接已经建立，后续客户端再来请求，那基本就是数据请求了，直接调用之前为这个连接创建好的handler来处理）
4. `handler` 只负责响应事件，不做具体的业务处理（这样不会使handler阻塞太久），通过 `read` 读取数据后，会分发给后面的 `worker` 线程池的某个线程处理业务。【业务处理是最费时的，所以将业务处理交给线程池去执行】
5. `worker` 线程池会分配独立线程完成真正的业务，并将结果返回给 `handler`
6. `handler` 收到响应后，通过 `send` 将结果返回给 `client`

### 方案优缺点分析

1. 优点：可以充分的利用多核 `cpu` 的处理能力
2. 缺点：多线程数据共享和访问比较复杂。`Reactor` 承担所有的事件的监听和响应，它是单线程运行，在高并发场景容易出现性能瓶颈。也就是说`Reactor`主线程承担了过多的事

主从 Reactor 多线程

### 工作原理图

针对单 `Reactor` 多线程模型中，`Reactor` 在单线程中运行，高并发场景下容易成为性能瓶颈，可以让 `Reactor` 在多线程中运行

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0007.png?x-oss-process=style/qingyun)

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0008.jpg?x-oss-process=style/qingyun)

> SubReactor是可以有多个的，如果只有一个SubReactor的话那和`单 Reactor 多线程`就没什么区别了。

1. `Reactor` 主线程 `MainReactor` 对象通过 `select` 监听连接事件，收到事件后，通过 `Acceptor` 处理连接事件
2. 当 `Acceptor` 处理连接事件后，`MainReactor` 将连接分配给 `SubReactor`
3. `subreactor` 将连接加入到连接队列进行监听，并创建 `handler` 进行各种事件处理
4. 当有新事件发生时，`subreactor` 就会调用对应的 `handler` 处理
5. `handler` 通过 `read` 读取数据，分发给后面的 `worker` 线程处理
6. `worker` 线程池分配独立的 `worker` 线程进行业务处理，并返回结果
7. `handler` 收到响应的结果后，再通过 `send` 将结果返回给 `client` 
8. `Reactor` 主线程可以对应多个 `Reactor` 子线程，即 `MainRecator` 可以关联多个 `SubReactor`

Scalable IO in Java 对 Multiple Reactors 的原理图解

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0009.jpg?x-oss-process=style/qingyun)

### 方案优缺点说明

1. 优点：父线程与子线程的数据交互简单职责明确，父线程只需要接收新连接，子线程完成后续的业务处理。
2. 优点：父线程与子线程的数据交互简单，`Reactor` 主线程只需要把新连接传给子线程，子线程无需返回数据。
3. 缺点：编程复杂度较高

## Reactor 模式小结

### 3 种模式用生活案例来理解

1. 单 `Reactor` 单线程，前台接待员和服务员是同一个人，全程为顾客服
2. 单 `Reactor` 多线程，`1` 个前台接待员，多个服务员，接待员只负责接待
3. 主从 `Reactor` 多线程，多个前台接待员，多个服务生

### Reactor 模式具有如下的优点

1. 响应快，不必为单个同步时间所阻塞，虽然 `Reactor` 本身依然是同步的（比如你第一个SubReactor阻塞了，我可以调下一个 SubReactor为客户端服务）
2. 可以最大程度的避免复杂的多线程及同步问题，并且避免了多线程/进程的切换开销
3. 扩展性好，可以方便的通过增加 `Reactor` 实例个数来充分利用 `CPU` 资源
4. 复用性好，`Reactor` 模型本身与具体事件处理逻辑无关，具有很高的复用性

