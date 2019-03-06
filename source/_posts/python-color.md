---
title: python控制台输出控制字体颜色
date: 2019-03-06 11:46:53
tags: python
---

写工具时需要把某些信息高亮给他们看，
于是收集了几种"控制python输出颜色"的方式

<!-- more -->

---

## \033/\x1b

这是网上搜的第一种，也是我收集的原因，因为这种失败了，在我的电脑上不能控制输出颜色，因为这是python3才有的，我用的python2.
比如:
```python
#coding=utf-8

print '\033[1;30;41m', 'hello world!'
```
应该输出红色的`hello world!`,但是我电脑上会是`[1;30;41m hello world!`

同时还延伸至其他库如`colorama`,`sty`等，我这里均以失败告终

---
## ctypes..SetConsoleTextAttribute

这种比较方便，调用ctypes直接操作控制台.
```python
#coding=utf-8

import ctypes

STD_OUTPUT_HANDLE= -11

FOREGROUND_RED = 0x0c # red.
FOREGROUND_DARKWHITE = 0x07 # dark white.

std_out_handle = ctypes.windll.kernel32.GetStdHandle(STD_OUTPUT_HANDLE)
def set_color(color, handle=std_out_handle):
    bool = ctypes.windll.kernel32.SetConsoleTextAttribute(handle, color)
    return bool

print u'普通颜色文字'
set_color(FOREGROUND_RED)
print u'红色文字'
set_color(FOREGROUND_DARKWHITE)
print u'又是普通文字'
```
结果是想要的样子.

---

## logging && coloredlogs

coloredlogs是一个专为logging准备的库，日志在控制台输出的时候，可以有不同颜色区分.
简单用法如下
```python
#coding=utf-8

import logging
import coloredlogs

coloredlogs.install(level = 'DEBUG')
logger = logging.getLogger(__name__)

logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.fatal('fatal message')
```
利用coloredlogs的install方法，把level以上等级的，用他自带的颜色配置输出，需要注意的是，install之后，logger.setLevel会失效，而且level等级以下的，将不会显示在控制台.

---
### 最后

第一种最方便，如果是python3，适用于小脚本/工具。
第二种兼容，python2和3都可以用，适用于小脚本/工具。
第三种适合于工程中的使用，工程中都是用logging来写日志的。