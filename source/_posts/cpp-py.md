---
title: cpp python通信初探
date: 2016-8-18 15:47:42
tags: [cpp, py]
---

## 初衷

&nbsp;&nbsp;&nbsp;&nbsp;游戏服务端比较流行的组合是`cpp+lua`，似乎是从云风dalao的skynet开始的，不过相对来说我更喜欢`python`，而且现在工作也用的是`python`，所以毫无疑问，用`python`必须的.作为将来拥有自己的服务端计划的一环，先初探一下他们之间的通信

## 工具环境
- visual studio 2013
- Python 2.7.12

<!-- more-->
___

### 创建工程添加依赖

- 如果需要在Debug环境下运行程序，得先对Python做一些处理
  - 把`%PythonRoot%/libs/Python27.dll` 拷贝一份改名为`Python27_d.dll`，为Debug所用
  - 修改`%PythonRoot%/Libs/Pyconfig.h`,搜索`#define Py_DEBUG`注释掉此行,因为Debug模式下有些宏在`python27.dll`里面是没有的

- 用vs创建一个空程，进入工程属性
  - 如果你的`python`是32位，无视这步; 点击配置管理器,活动解决方案平台/新建 x64平台，用于跑64位`python`:
  ![build x64 platform](/img/cpx64pf.png)
  - CC++/常规/附加包含目录 -> '%PythonRoot%/include/' 把python安装目录下的include包含进去
  ![add py include](/img/cppyinc.png)
  - 链接器/常规/附加库目录 -> `%PythonRoot%/libs/` 把python安装目录下的libs库包含进去
  ![link pylibs](/img/cplink.png)
  - 创建main.cpp

### 入门代码
先写一份first.py代码，我放在与main.cpp相同目录下
```python
#coding=utf-8
# first.py

def getName():
    return 'tsfissure'

def add(a, b):
    return a + b
```
main.cpp中如下代码
```cpp
#include <iostream>
#include <Python.h>

inline void strFunc() {
  auto pyModule = PyImport_ImportModule("first");  //PyObject* //import first
  if (pyModule) { //import success
    auto pyFunc = PyObject_GetAttrString(pyModule, "getName"); //PyObject* // first.getName()
    if (pyFunc) {
      auto v = PyObject_CallObject(pyFunc, nullptr);  //PyObject //call function
      std::cout << PyString_AsString(v) << std::endl;
      Py_DECREF(pyFunc);//free pointer
    }
    Py_DECREF(pyModule);  //free pointer
  }
}

inline void intFunc() {
  auto pyModule = PyImport_ImportModule("first");
  if (pyModule) {
    auto pyFunc = PyObject_GetAttrString(pyModule, "add");
    auto pyParam = PyTuple_New(2);  //PyObject* //param of first.add
    if (pyFunc && pyParam) {
      PyTuple_SetItem(pyParam, 0, PyLong_FromLongLong(123456789123LL)); //set tuple value, pylong from long long
      PyTuple_SetItem(pyParam, 1, PyInt_FromSize_t(1)); 
      auto v = PyObject_CallObject(pyFunc, pyParam);
      std::cout << PyLong_AsLongLong(v) << std::endl;
      Py_DECREF(pyFunc);
      Py_DECREF(pyParam);
    }
    Py_DECREF(pyModule);
  }
}


int main(int argc, char**argv) {
  Py_Initialize();
  if (!Py_IsInitialized()) {
    std::cout << "Python Initialize error" << std::endl;
  } else {
    PyRun_SimpleString("print 'Hello Cpp Python!'"); // run a simple commond
    strFunc();
    intFunc();
    Py_Finalize();
  }
  std::system("pause");
  return 0;
}

```
结果是：
```html
Hello Cpp Python!
123456789124
请按任意键继续. . .
```
在first.py中定义了两个函数，一个是传入字符串返回一个字符串，一个是传入ab返回a+b。
main.cpp中先Py_initialize()如果成功了，先调用一个简单串，相当于在py环境下输入这句话，然后分别调用了py中的两个函数

有了注释我想应该很容易看懂了
- 包含`Python.h`头文件
- 从`Py_Initialze()`开始，到`Py_Finalized()`结束
- 一切对象都是`PyObject*`   //我用了`auto`
- 空参数时传`null`

over.入门还是挺简单的.


## python 调用cpp

