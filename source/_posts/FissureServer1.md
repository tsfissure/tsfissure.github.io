---
title: FissureServer(一)
date: 2016-05-14 22:02:36
tags: GameServer
---

## 源码: [github](https://github.com/tsfissure/FissureServer)

### 内容决定
- FisureHost
- LogSystem
- Config

<!-- more -->
### FisureHost

&nbsp;&nbsp;&nbsp;&nbsp;这是一个掌管整个服务端底层生命周期的Host.管理网络服务，逻辑脚本的编译，数据库服务等。主要有开始和结束两个操作，是整个引擎的开始点.

- 开启Host:
  - 编译逻辑脚本，创建运行时
  - 创建[网络服务](#)
  - 启动运行时
  - 启动网络服务

- 停止Host:
  - 停止[网络服务](#).
  - 停止运行时

### 日志系统
&nbsp;&nbsp;&nbsp;&nbsp;debug最有效的工具就是日志系统，所以写工程前最有必要做的热身准备就是日志系统了，这里直接用了[NLog.dll](http://nlog-project.org/).虽然很古老了，但使用挺方便的.
它可以根据配置不同输出日志输出到不同目录，并可控制要不要在控制台输出.

### App.config
配置文件目前分为两类，一类就普通的键值对儿，网络传输限制长度等，另一类为Mysql数据库的连接,例子如下

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
	<startup>
		<supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
	</startup>

	<appSettings>
    <!--服务器基础-->
    <add key="Server.Id" value="1" />
    <add key="Server.Port" value="6001" />
    
    <!--网络设定-->
    <add key="Network.MaxConnectionCount" value="20000"/>
    <add key="Network.MaxMessageSize" value="102400"/>
    <add key="Network.MessageBufferSize" value="8192"/>
    
  </appSettings>
  <connectionStrings>
    <!--MySQL数据库配置-->
    <add name="FisureLog" providerName="MySqlDataProvider" connectionString="Data Source=localhost;Port=3306;Database=fisureLogs;Uid=game_user;Pwd=123;" />
  </connectionStrings>
</configuration>
```
因为appSettings的值有不同类型，所以得为不同类型写一个专门的读取接口.见fisure.Common.Utils.ConfigAppSettingUtil.cs;

