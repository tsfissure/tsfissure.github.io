---
title: go_protobuf
date: 2018-07-09 14:38:20
tags: go
---


### go使用protobuf初接触

记录一下从零开始在go中使用protobuf通信.

<!-- more -->


## 1 下载protoc.exe

> https://github.com/google/protobuf/releases

我下载的是protoc-xxx-win32.zip,在windows上这个直接包含exe，免去编译。
同时把exe拷贝到$GOPATH/bin目录下，因为后面gofaster也在这目录，所以放一起,执行命令时能找到.。把目录加到环境变量中.

## 安装protobuf库

> go get github.com/golang/protobuf/proto

执行后就会在$GOPATH/pkg/和$GOPATH/src下面有相相应代码和.a库文件

## gogoprotobuf 插件

> go get github.com/gogo/protobuf/protoc-gen-gofast

执行后还需要install一下生成一堆exe .我是用的LiteIDE install all的，也比较方便

## 测试

此时随便找个文件夹建个.proto文件即可以
```proto
syntax = "proto3";
package example;


message UserInfo{
    string name = 1;
    int32 age = 2;
}

```
第一行网上说是必须指定是proto2还是proto3.后面就随意了

> protoc --gofast_out=. example.proto

顺利执行的话会生成example.pb.go文件
内容太多，就不贴了。

最后是正式使用时这些.proto文件和.pb.go文件的位置问题,下一节会再来。


## 使用

所有基本环境已经搭建完成,接下来正式使用:
在$GOPATH/src下面建目录protocol表示协议相关的文件.
protocol/proto放*.proto文件,生成的.pb.go文件放在protocol/pbgo/目录下面
注意这里.proto文件的package应写为pbgo，与生成的目录名相同.方便后面install和import
> 
src
 └─protocol
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├─pbgo
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└─proto
> 

所以需要在src/protocol/目录下建个脚本生成所有.pb.go文件,关键语句为`protoc --gofast_out=./../pbgo xxx.proto`.批量生成代码如下(bat文件)
```bat
cd %~dp0
@echo off

for %%i in (proto/*.proto) do (
    echo %%i
    protoc -I=proto --gofast_out=pbgo proto/%%i
)

pause

```
这样就会在/protocol/pbgo/目录下生成对应的pb.go文件
然后在/protocol/pbgo目录下go install。这样就会生成$GOPATH/pkg/protocol/pbgo.a库文件。(其实这一步也可以写在上面的.bat批处理里面)
此时在src的其他项目中我们就可以`import "protocol/pbgo"`把协议包包含进来使用

## 实践
根据上面的操作后，新建客户端和服务端代码通信试试：
我实践用例是客户端把今日日期和天气传给服务端。
所以proto:
```proto
syntax = "proto3";
package pbgo;

message TodayInfo {
    int32 year = 1;
    int32 month = 2;
    int32 day = 3;
    string weather = 4;
}
```
Server.go:
```go
package main

import (
    "fmt"
    "io"
    "net"
    "os"
    "protocol/pbgo"

    "github.com/gogo/protobuf/proto"
)

func onAccept(conn net.Conn) {
    defer conn.Close()
    var msg []byte = make([]byte, 2048)
    for {
        n, err := conn.Read(msg)
        if err != nil {
            if err == io.EOF {
                break
            }
            fmt.Println("Read Error", err.Error())
            break
        }
        data := msg[:n]
        todayInfo := &pbgo.TodayInfo{}
        err = proto.Unmarshal(data, todayInfo)
        if err != nil {
            fmt.Println("Unmarchal Error", err.Error())
            break
        }
        fmt.Printf("Year[%d] Month[%d] Day[%d] Weather[%s]\n",
            todayInfo.GetYear(), todayInfo.GetMonth(),
            todayInfo.GetDay(), todayInfo.GetWeather())
    }
}

func main() {
    listen, err := net.Listen("tcp", ":2333")
    if err != nil {
        fmt.Println(err.Error())
        os.Exit(1)
    }
    defer listen.Close()
    fmt.Println("Listen Success")
    for {
        conn, err := listen.Accept()
        if err != nil {
            fmt.Println("Aceept Error:", err.Error())
            break
        }
        fmt.Println("Accept:", conn.LocalAddr().String(), conn.RemoteAddr().String())
        go onAccept(conn)
    }
}

```
Client.go:

```go
package main

import (
    "fmt"
    "net"
    "os"
    "protocol/pbgo"
    "sync"
    "time"

    "github.com/gogo/protobuf/proto"
)

func onConnect(conn net.Conn, mux *sync.WaitGroup) {
    defer mux.Done()
    defer conn.Close()

    todayInfo := &pbgo.TodayInfo{
        Year:    2018,
        Month:   7,
        Day:     9,
        Weather: "非常热",
    }
    data, err := proto.Marshal(todayInfo)
    if err != nil {
        fmt.Println("Send Error:", err.Error())
        return
    }
    conn.Write(data)
    time.Sleep(1 * time.Second)
}

func main() {
    conn, err := net.Dial("tcp", "127.0.0.1:2333")
    if err != nil {
        fmt.Println("Connect Error:", err.Error())
        os.Exit(1)
    }
    fmt.Println("Connect Success")
    mux := sync.WaitGroup{}
    mux.Add(1)
    go onConnect(conn, &mux)
    mux.Wait()
}

```
分别install后，先打开server.exe再打开client.exe
server端结果显示如下：
```cpp
Listen Success
Accept: 127.0.0.1:2333 127.0.0.1:56355
Year[2018] Month[7] Day[9] Weather[非常热]
```

到此。便可告一段落了.


