---
title: cpp+swig=>py|lua
date: 2016-12-15 18:12:02
tags: [cpp,py,lua]
---

把cpp打包给其他语言用的方式有很多。这两天写了个给py和lua用。 记录一下。

<!-- more -->

## swig
---

swig算是一个第三方SDK。把cpp打包成库把接口暴露给其他语言使用。可以到[官网下载](http://www.swig.org/download.html)。后面几个版本似乎没有现成的exe了。我懒得再编。下载了swigwin2.0.9的。
他的优势在于简便，步骤简单，而且你可以只管写C++。不用在意其他语言。比如我要打包给python使用，如果使用其他工具，你会在写的过程中不限次乃至PyObject，extern之类的东西，但这个不需要。
- 安装
  我下的2.0.9，自带一个exe.所以我直接把解压路径添加到path就ok了。如果你用了后面的版本，你得需要按照[官网](http://www.swig.org/Doc3.0/Windows.html#Windows_swig_exe)介绍那样撸一波生成exe.

## 打包到python和lua

#### cpp code

topy.h
```cpp
/* File Name: topy.h */

class Topy{
public:
    void say();
    int convert(int k);
};
```
topy.cpp
```cpp
/* File Name: topy.cpp */

#include "topy.h"
#include <iostream>

void Topy::say(){
    std::cout << "hello swig..." << std::endl;
}

int Topy::convert(int k){
    return k + 100;
}
```

### To Python
---

打包给ython用我觉得是最简单的。swig简直就是给python量身打造,不过也不是了完美的，因为打包后得两个文件一起使用，如果用其他SDK,可以只有一个PYD文件

1, 编写topy.i
```python
/* File Name: topy.i */

%module topy
%{
#include "topy.h"
%}

%include "topy.h"
```
2,  打开命令行
```python
> swig -c++ -python topy.i
```
这时候会新生成两个文件: `topy.py`,`topy_wrap.cxx`
PS: 如果是C。不是C++。可以去掉`-c++`

3, 编写setup
利用python的distutils打包

```python
#coding=utf-8
# File Name: setup.py

from distutils.core import setup
from distutils.extension import Extension

setup(
        name = "ToPython",
        ext_modules=[
                Extension("_topy", ["topy_wrap.cxx", "topy.cpp"])
            ]
        )

```

PS: Extension那里名字必须是前面生成的xx.py的名字前面加下划线`_xx`.后面把需要用到的cpp和前面生成的cxx文件包含进来

4, 生成
打开命令行生成_topy.pyd
```cpp
> python setup.py build_ext --inplace
```
PS: 关于这里的参数，网上有很多资料我使用的build_ext --inplace生成在当前目录
PPS: 不要用install。这个会把生成的两个文件安装到python的site-packages下面去

5, 把topy.py 和_topy.pyd两份文件放一块就可正常使用了
```python
>>> import topy
>>> a = topy.Topy()
>>> a.say()
hello swig...
>>> a.convert(23)
123
>>>
```

#### 问题
有天我服务器重装了一下.把生成好的py和pyd拷贝过去使用。发现找不到模块，开始以为是python出了问题，试了各种方式换了版本，都发现还是不行，有点方。后来忘了在哪里搜到电脑缺msvcpXX.dll会有各种异常，于是把我电脑上的msvcp120.dll,msvcr120.dll拷贝过去。成功执行...吓坏了。有点疑问，生成pyd.为什么和这俩库有关系.报错是找不到module...


### To Lua
---

懒得再写新代码了。就用刚才的topy.h,topy.cpp吧
1，命令行 
```cpp
> swig -c++ -lua topy.i
```
这里只会生成一个topy_wrap.cxx文件

2, 用vs创建空工程
创建新的空工程
把你编写的所有.h和.cpp还有刚才生成的cxx文件加进来
修改项目属性为生成dll.c/c++附加包含目录为$luaroot$\include.链接输入的附加库目录$luaroot$\lib;附加依赖项$luaroot$\lua51.lib
其中$luaroot$表示lua安装目录

3，编译生成
点生成。可生成<项目名称>.dll
修改名为topy.dll即可使用

