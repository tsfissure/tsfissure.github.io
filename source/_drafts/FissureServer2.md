---
title: FissureServer(二)
date: 2016-05-15 22:42:24
tags: GameServer
---

## 源码: [github](https://github.com/tsfissure/FissureServer)

### NetService

网络相关总的来说，被我分成了以下几类
  - Message: 传输消息协议
  - Connection: 单个连接
  - SocketListener: 专门监听客户端的连接
  - SocketService: 服务，消息行为逻辑处理一般都放在里面
  - SocketHost: 掌管整网络服务的生命周期,Listener的开关控制，上面各模块中转站
  
<!-- more -->
## 简介
整个网络服务差不多都归纳在这几个模块中，流程大概是：
SocketHost初始时创建SocketListener，SocketService，
SocketHost启动(Start())时启动(异步)监听，当有客户端连接时，SocketListener通知SocketHost接收到连接，然后SocketHost注册Connection，触发SocketService的新连接事件(如果连接时需要做些什么，放在这里).
当Connection发现Socket接收到客户端发来的消息，通知SocketHost有消息收到，SocketHost对消息进行解析成特定的Message，然后把消息中转到Service进行逻辑处理.
特别地，如果是服务端广播的话，也相当于是这个Connection收到消息，去Service进行逻辑处理，然后按照正常流程返回给客户端.
  
### SocketHost

因为一开始就要固定消息协议Message， 而且要整个底层统一用同一种，所以我们把SocketHost和SocketService都写成泛型，在创建Host的时候，就确定Message.
SocketService在SocketHost初始的时候创建，这样就保证了用同一种Message。

有个成员变量:`public readonly SocketAsyncEventArgsPool SaePool;`这个是用来反复利用SocketAsyncEventArgs(简称sae)的，不是必须e r ，因为有不断的异步连接与断开，会一直有它的释放的申请，于是这个是用一个栈来维护整个过程中的sae,栈的大小为10000，这个可改.

成员方法一般就是为了中转了。Listener到Service,Connection到Service等.逻辑都中转到Service里面去便于维护.

### SocketListener

Listener的任务非常简单，就是异步监听，接收到连接时，触发连接事件，然后继续监听..

```
public void AcceptSocketAsync() {
      if (null == ListenSocket) return;
      bool completed = true;

      try {
        completed = ListenSocket.AcceptAsync(_saea);
      } catch (Exception e) {
        TraceLog.WriteError("ListenSocket.AscceptAsync Error: Ex:{0}", e.StackTrace);
      }
      if (!completed) ThreadPool.QueueUserWorkItem(_ => AcceptCompleted(this, _saea));
    }
```
在这段代码可能难理解，就是completed如果返回的是false的话，是不会触发完成事件的，所以需要我们手动触发,MSDN上说当I/O操作挂起时，返回True,如果是同步完成了，则返回false.


### Connection & Message & SocketService

因为不同游戏可能对Connection的需求不同，所以这里也定义了一个Interface,把框架必须要有的放在了里面，继承于它的Connection都可以与SocketHost通信.
Message和SocketService同理，多的是，Service是要对具体的Message处理，所以Service是泛型比较好，可以适应不同的Message.


想到什么再来补上吧...