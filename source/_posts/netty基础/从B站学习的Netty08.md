---
title: 从B站学习的Netty08
abbrlink: 1198865722
date: 2021-11-30 13:01:55
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
 ## Netty 模型

> 讲解netty的时候采用的是先写代码体验一下，再细讲里面的原理。前面看不懂的可以先不用纠结，先往后面看，后面基本都会讲清楚

### 工作原理示意图1 - 简单版

<p><code>Netty</code> 主要基于主从 <code>Reactors</code> 多线程模型（如图）做了一定的改进，其中主从 <code>Reactor</code> 多线程模型有多个 <code>Reactor</code></p>

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0010.png?x-oss-process=style/qingyun)

**对上图说明**

1. `BossGroup` 线程维护 `Selector`，只关注 `Accecpt` 
2. 当接收到 `Accept` 事件，获取到对应的 `SocketChannel`，封装成 `NIOScoketChannel` 并注册到 `Worker` 线程（事件循环），并进行维护
3. 当 `Worker` 线程监听到 `Selector` 中通道发生自己感兴趣的事件后，就进行处理（就由 `handler`），注意 `handler` 已经加入到通道

### 工作原理示意图2 - 进阶版

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0011.png?x-oss-process=style/qingyun)

`BossGroup`有点像主`Reactor` 可以有多个，`WorkerGroup`则像`SubReactor`一样可以有多个。

## 工作原理示意图3 - 详细版

![img](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/0012.png?x-oss-process=style/qingyun)

<ol>
<li><code>Netty</code> 抽象出两组线程池 ，<code>BossGroup</code> 专门负责接收客户端的连接，<code>WorkerGroup</code> 专门负责网络的读写</li>
<li><code>BossGroup</code> 和 <code>WorkerGroup</code> 类型都是 <code>NioEventLoopGroup</code> </li>
<li><code>NioEventLoopGroup</code> 相当于一个事件循环组，这个组中含有多个事件循环，每一个事件循环是 <code>NioEventLoop</code> </li>
<li><code>NioEventLoop</code> 表示一个不断循环的执行处理任务的线程，每个 <code>NioEventLoop</code> 都有一个 <code>Selector</code>，用于监听绑定在其上的 <code>socket</code> 的网络通讯</li>
<li><code>NioEventLoopGroup</code> 可以有多个线程，即可以含有多个 <code>NioEventLoop</code> </li>
<li>每个 <code>BossGroup</code>下面的<code>NioEventLoop</code> 循环执行的步骤有 <code>3</code> 步<ul>
<li>轮询 <code>accept</code> 事件</li>
<li>处理 <code>accept</code> 事件，与 <code>client</code> 建立连接，生成 <code>NioScocketChannel</code>，并将其注册到某个 <code>workerGroup</code> <code>NIOEventLoop</code> 上的 <code>Selector</code></li>
<li>继续处理任务队列的任务，即 <code>runAllTasks</code></li>
</ul>
</li>
<li>每个 <code>WorkerGroup</code> <code>NIOEventLoop</code> 循环执行的步骤<ul>
<li>轮询 <code>read</code>，<code>write</code> 事件</li>
<li>处理 <code>I/O</code> 事件，即 <code>read</code>，<code>write</code> 事件，在对应 <code>NioScocketChannel</code> 处理</li>
<li>处理任务队列的任务，即 <code>runAllTasks</code></li>
</ul>
</li>
<li>每个 <code>Worker</code> <code>NIOEventLoop</code> 处理业务时，会使用 <code>pipeline</code>（管道），<code>pipeline</code> 中包含了 <code>channel（通道）</code>，即通过 <code>pipeline</code> 可以获取到对应通道，管道中维护了很多的处理器。（这个点目前只是简单的讲，后面重点说）</li>
</ol>

