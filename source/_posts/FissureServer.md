---
title: FissureServer
date: 2016-05-08 20:41:43
tags: GameServer
---

## 简介
公司的服务器框架原型来自于[SCUTGAME](http://www.scutgame.com/),最近在解读中，顺便就造一个自己的出来吧.力求更简洁，更易懂，效率不失,扩展性不失.

我的属性:
- 语言：c#
- 平台: Windows
- 数据库: Mysql(存日志),SSDB(存数据)
- 网络传输协议: tcp
- 源码: [In Github](https://github.com/tsfissure/FissureServer)

<!-- more -->

## 初识SharkServer.Common

这应该算是一个比较体积小却功能还OK的gameserver，能处理客户端的请求响应，和广播客户端数据，还能在服务器之间完成跨服处理,均衡负载等。
它对数据库的处理可能不够优秀，如果数据库崩了，没有备用的用，整个服务器就跪了，也就是，它的数据，只存在了一个地方。


//TODO LIST:
- [x] Host: 最外层的主机，控制整个服务器的生命周期
- [x] NetService: 网络服务
- [ ] Runtime: 运行时
- [x] 日志系统
- [ ] 定时器系统
- [ ] 数据库
- [ ] ...




