---
title: 可怕浮点の玩家少了一个金币
date: 2016-10-18 19:47:42
tags: [禁忌, 杂谈]
---


对浮点数抱有恐惧感，大概是刚~~出生~~学会用浮点数的时候。
一个原因是因为效率，浮点数乘除法都比整数慢，所以在能用整数的情况下，我会尽量用整数，不过这原因似乎也不是很重要，因为有了第二个原因:误差，

> 人类终于回想起那一天，被浮点数误差所支配的恐惧

在学校的时候，就很怕ZJU的题目，因为每次总会有不愿看见的浮点数题，总会要求误差不超过0.000001(1e-6)，然后你可能在哪个地方先除再乘了，在哪个地方多了个不必要的运算，这误差就出来了。

工作虽然也一直担心屏幕坐标取得不对啊，运算太慢导致太卡啊啥的，不过也一直没有出现什么问题。然而今天，玩家少了一个金币

<!-- more -->

---

同事A：“高哥，出了个问题，玩家少了一个金币”
同事B：“......”
A：“因为运算出错了，50000 * 1.15 不等于57500”
B&I：“......”

我一直知道python的大数据很叼，但浮点数会容易有误差，可是我一直没遇到过，也没在意，直到至今，更加坚定了我的想法，**尽量不要用浮点数**.  最后同事A给他的运算加了个ceil解决了.

于是我自己试了一下，越发觉得神奇。。。
```python
>>> 50000 * 1.15
57499.99999999999
>>> print 50000 * 1.15
57500.0
>>> 50000 * 115 / 100
57500
>>>
```
我也不懂为什么加了print，就能正确运算了，大概，是因为要保存前X位，进行了四舍五入吧。

---
还是那句话，为了避免这种注意不到的小坑，各种后患，尽量用整数表达。

