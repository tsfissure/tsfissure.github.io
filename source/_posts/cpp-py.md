---
title: cpp python通信初探
date: 2016-8-18 15:47:42
tags: cpp-py
---

## 初衷

&nbsp;&nbsp;&nbsp;&nbsp;游戏服务端比较流行的组合是`cpp+lua`，似乎是从云风dalao的skynet开始的，不过相对来说我更喜欢`python`，而且现在工作也用的是`python`，所以毫无疑问，用`python`必须的.作为将来拥有自己的服务端计划的一环，先初探一下他们之间的通信

<!-- more-->

## 工具环境
- visual studio 2015
- Python 2.7.6

### 

### 创建工程添加依赖

- 如果需要在Debug环境下运行程序，得先对Python做一些处理
  - 把`%PythonRoot%/libs/Python27.dll` 拷贝一份改名为`Python27_d.dll`，为Debug所用
  - 修改`%PythonRoot%/Libs/Pyconfig.h`,搜索`#define Py_DEBUG`注释掉此行,因为Debug模式下有些宏在`python27.dll`里面是没有的

- 用vs创建一个空程，进入工程属性
  - CC++/input/
  - d
  - df

### 入门代码
直接从代码开始看
```cpp
```

- 包含`Python.h`头文件
- 从`Py_Initialze()`开始，到`Py_Finalized()`结束
- 空参数时传`NULL`

over.入门还是挺简单的.


## python 调用cpp

