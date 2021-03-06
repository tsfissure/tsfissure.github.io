---
title: 鸡哥脚步始末
date: 2018-05-18 15:43:22
tags: [杂谈, py]
---

## 始
___

时不时的又会听说阴阳师有人用按键精灵被封号了，用连点器也封号了之类的。本来没啥感觉~~(毕竟本大佬全身都是肝不需要辅助工具)~~.
在前几天帮朋友写了个脚本copy excel之后，有个寮友说了一句让我写个脚本来给yys砸百鬼刷魂十吧。
我突然来了兴致决定试一试，写一个能自动刷魂十，又能避开官方外挂检测的脚本.

<!-- more -->

___

### 大体思路决定

1. 避开检测到外挂，那必须点击的时间和坐标一定要随机
2. 要方便启动和关闭，那就得开两条线程一边控制鼠标点击，一边检测键盘事件。
3. 要尽量方便用户操作，不接受按键精灵那样要先自己手动过一遍再保存成脚本运行。

控制鼠标和监听键盘都有现成的库(`pykeyboard` and `pymouse`)，也比较容易。
其中难点在于坐标的获取。只要把第一点解决好，第三点也就可以无视了。
为了获取开始战斗的按钮。考虑了几个版本。

#### version.1

在游戏启动后，通过截图方式获取按钮在屏幕左上角算过来的坐标，写到配置文件里面。然后启动脚本。
经测试，成功。但是。每次启动都需要填配置不是很操蛋的事情吗，而且除了开始按钮还有接任务按钮等，这些要配置，是会哭的。pass

#### version.2

为了不填配置。我想了另外一个方案：
先把开始按钮截图保存下来`img1`。
游戏启动到开始战斗界面，然后启动脚本，脚本启动时截图整个屏幕，然后通过图像查找目标图片(`img1`)在整个屏幕图片中的坐标。查找不到就告之启动失败。
为此我还发现一个网易的开源库[`aircv`](https://github.com/NetEaseGame/aircv)。是真的强大！(以后要用图片查找，可以用这库
经测试，成功。但是。游戏窗口是可以调节的！没有办法根据不同大小的窗口来获取对应图片大小的坐标!!! pass


#### version.3

发现目前最大难题还是坐标的获取。思索良久，在感觉要以失败告终愧对寮友的时候，突然脑袋亮灯:
不管界面大小，按钮的坐标相对游戏的窗口来说，是按一定比例位置显示的。所以我们只要知道了游戏窗口的位置。所有按钮的位置都可以通过百分比算出来。
完美，这样就只需要在启动脚本的时候寻找窗口的位置就好了。而且可以随时中断重启脚本。
寻找游戏窗口的位置，这个比较容易，可以通过`win32gui`库里面的函数完成。核心代码如下
```python
from win32gui import *

def listWindows():
    dialogs = []
    tmp = '\xd2\xf5\xd1\xf4\xca\xa6'  #阴阳师
    def callback(hwnd, mouse):
        if IsWindow(hwnd) and IsWindowEnabled(hwnd) and IsWindowVisible(hwnd):
            title = GetWindowText(hwnd)
            if tmp in title:
                dialogs.append(GetWindowRect(hwnd))
    EnumWindows(callback, 0)
    return dialogs
```
很简单的几行就解决了所有麻烦事。简单概括就是，遍历所有进程的窗口。找到标题含有阴阳师字段的窗口存下来(存的是左上和右下的坐标).然后在这范围内计算对应按钮的坐标，就odk了。

___

## 末

花了几天完美写了个脚本，发给群里被寮友们称为鸡哥脚步，甚是开心.
整个过程还是学到不少东西，比如更熟悉使用鼠标键盘库，学知道了网易的那个aircv开源库。
一开始遇到启动脚本点击事件无响应的问题，纠结了半天，最后解决方案是以管理员模式运行脚本.猜应该是阴阳师游戏的原因吧，比较特殊，其他软件没有此问题!
已经跑了多天，没有被封，我想网易检测外挂的方式应该就是连点器(点击时间间隔相同)和按钮精灵(点击坐标有序)吧

(终)

