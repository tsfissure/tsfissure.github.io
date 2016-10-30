---
title: <del>装饰器继承</del>__call__
date: 2016-10-30 15:47:42
tags: py
---

## 初衷
最近在计划着用Django写个web出来玩玩儿，在用户验证这里用装饰器简单粗暴，细分有登录验证，管理员验证。发现两个装饰器的代码仅有一丢丢的差别，第一反应是就这样吧，第二反应是如果过两天我加了验证怎么办，比如VIP,SVIP,特定渠道验证等，感觉也只有一行代码的差别，于是我想着把这抽象出来，搜了搜发现`class`有一个有趣的东西`__call__`.

<!-- more -->

## __call__
class本身不具备运算能力，但加上call函数，可以让它成为一个函数那样调用，相当于c++中的`bool operator()(T const&)`重载运算符;
```python
#coding=utf-8

class Foo:

    def __call__(self, *args, **kwargs):
        print u'Foo.__call__', args, kwargs

f = Foo()   # new a object Foo
f(1, 2, 3, four = 4) # Equal to `Foo()(1, 2, 3, four = 4)`

"""
result:
Foo.__call__ (1, 2, 3) {'four': 4}
"""
```
上面这样一个类，就具有了运算能力，我们可在call函数里面做相应操作


##　装饰器继承

知道了上面call的妙用之后，就可以来实现一个可继承的装饰器了，把各个装饰器的**相同代码**放在call下，重写个成员函数放不同代码，即可。

```python
#coding=utf-8

class Base:

    def __init__(self, foo):
        self.foo = foo

    def __call__(self, *args, **kwargs):
        if self.verify(*args, **kwargs):
            print self.foo.__name__, u'verify success'
            return self.foo(*args, **kwargs) #The 'return' not necessary aways, is's ok.
        else:
            print self.foo.__name__, u'verify failed, doc:', self.foo.__doc__, u'param:', args, kwargs


    def verify(self, *args, **kwargs):
        raise 'Base::verify Not Implement'

class SubClass1(Base):
    def verify(self, *args, **kwargs):
        return args[0] > 5

class SubClass2(Base):
    def verify(self, *args, **kwargs):
        return args[0] < 5


@SubClass1
def first(a, b):
    '''first function, a + b'''
    print u'a + b =', a + b

@SubClass2
def second(a, b):
    '''second function, a * b'''
    print u'a * b =', a * b


first(10, 20)
first(2, 12)
second(3, 5)
second(10, 15)

"""
result:
first verify success
a + b = 30
first verify failed, doc: first function, a + b param: (2, 12) {}
second verify success
a * b = 15
second verify failed, doc: second function, a * b param: (10, 15) {}
"""
```
上面就是一个简单装饰器继承的例子。
更有趣的一点是，这还省去了一个小麻烦，普通装饰器需要用`functools.wraps`来获取原函数的各种信息，然而上面这样写，可以直接获得原函数的所有信息如`__name__`,`__doc__`等
