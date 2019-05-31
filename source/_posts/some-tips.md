---
title: some_tips
date: 2019-02-20 00:01:26
tags: [技巧,py]
---


有时候小问题，网上搜了多种解决方案行不通，最后才搜到一个可行方案。
怕忘了。记录一下。

或者是一些有趣小知识，记录一下。

<!-- more -->

### 已取消网页导航

有好几次出现在uc上打开一个网址提示"已取消见面导航"。但是换chrome或者firefox都正常。
网上搜好些方案都是要么无效，要么很复杂(修改注册表，执行啥啥命令啥的)。
最后在一个[csdn](https://blog.csdn.net/daman1985/article/details/51731155)上找到了解决方案
> 工具->代理设置->局域网设置: 自动配置时选择“自动检测设置”，去掉“使用自动配置脚本”选项


### python gbk

python读取中文文件时使用`gbk`或者`gb18030`,格式如`codecs.open(fn, encoding='gbk')`，不要用`gb2312`
收集的字符`gb18030>gbk>gb2312`,比如涅槃的槃字。gb2312是读不出来的。
我不会说因为这个，给他们生成配置时出过问题。。。

### python shell启动/UAC

利用python启动文件方式有很多，能够控制管理员确认的方式有如下两种：
```python
win32api.ShellExecute(None, u'runas', sys.executable, __file__, None, 1)
ctypes.windll.shell32.ShellExecuteW(None, u'runas', sys.executable, __file__, None, 1)
```
但是谨记，如果路径包含非英文，只能用上面种方式。

### python打开文件夹
试了两种：
```python
os.system("explorer E:\xx\yy")
os.startfile("E:\xx\yy")
```
对于上面一种可以顺利执行但是只能是左\不能是右/。而且如果gui程序，还会弹一个命令行窗口，逼死强迫症。下面一种很完美...

