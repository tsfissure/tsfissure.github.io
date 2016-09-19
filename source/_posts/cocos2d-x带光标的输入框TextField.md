title: cocos2dx-TextField向左推进光标跟随
date: 2015-12-04 23:49:28
tags: [cocos2d-x,TextField]
---

在什么网站上啊，游戏登录界面啊什么的，你输入的账号如果超过了那个框框的宽度，有些就会只显示后面输入的部分前面的隐藏了，同时光标也是接在后面的.这个看起来似乎很直接很简单的功能，对于cocos2dx的新手来说，我想也不会那么的顺利.
PS: 本人用的是**Cocos2d-x 3.6**版本，可以使用*csb*.
<!--more-->

- 功能：实现输入框的输入，向左推进，光标跟随(当前只适合iOS版本，且不能中间删除，只能结尾删除).
- 实现：
	- [创建csb](#csb)
  	- [定义场景和需要的成员变量](#var)
  	- [初始化csb和成员变量](#init)
  	- 对事件监听进行处理
	  	- [打开输入法](#attach)
	  	- [输入字符](#insert)
	  	- [删除字符](#del)
	  	- [关闭输入法](#dettach)
	- [其他](#other)

## 具体内容

### <a name = "csb"/> 创建csb
- 创建一个csb,包含一个横向输入框*TextField*控制成只能存一行文字，名叫`textField`.
- *TextField*下面包含一个细长细长的小图片*ImageView*用作光标`cursor`,*TextField*没有自带光标~~2333~~.
### <a name="var"/> 变量
- `static const int sciBufferLength = 256`: 最多256个文字
- `TextField* 	m_pkField`: 目标输入框
- `Text*			m_pkFontInfo`: 输入框的字体信息(名字名称，大小)
- `ImageView* 	m_pkCursor`: 光标
- `RepeatForever* m_pkCursorAction`: 光标的动画(一闪一闪的)
- `std::string	m_kFinalStr`: 最终的字符串
- `std::string 	m_kShowStr`: 显示在输入框中的字符串
- `float 		m_afUnitWidth[sciBufferLength]`: 输入的字符的单个宽度
- `float			m_fShowStrWidth`:显示的字符串的总宽度
- `int			m_iShowLeftIndx,m_iShowRightIndx`:显示的字符串的两个索引[L,R),显示的字符串的总宽度就是这个半开半闭区间的和. 

### <a name = "init"/> 初始化
这里主要是初始化`m_pkFontInfo`和`m_pkCursorAction`。
- 利用输入框中的字体信息`create`一个*Text\**出来，用于后面的[输入字符串](#insert)处理单个字符.
- `create`一个无限重复的动作，用于输入状态时光标闪动动画
- PS:记得`create`之后要`retain`一下，不然下一帧就成野指针了.

PS: 初始记得把光标隐藏，因为开始不是输入状态.
```c
m_pkFontInfo = Text::create("", m_pkField->getFontName(), m_pkField->getFontSize());
CC_SAFE_REATAIN(m_pkFontInfo);
```

### <a name = "attach"/>打开输入法
这一步，大概正常情况下，只需要让光标进行闪烁就ok了吧.`m_pkCursor->runAction(m_pkCursorAction)`(PS:注意判空);
### <a name = "insert"/>输入
这一步是关键，处理顺序大概如下：
- 循环读取每一个字符(英语字母或者一个中文)
- 判断是否合法(包括非法字符和是否超过最大限制)，如果不合法，停止
- `m_pkFontInfo->setString()`,把单个字符`set`进去，用`getVirtualRenderSize().width`计算宽度`fWidth` ，加到`m_fShowStrWidth`和`m_afUnitWidth`
- 将字符串加到`m_kFinalStr`.
- 循环完后，去掉前面超出的部分(应该在输入框中显示的部分)，同时维护显示的字符串的宽度。
- 获取`[m_iShowLeftIndx,m_iShowRightIndx)`的子字符串,`setString()`到`TextField`，设置光标位置.

### <a name = "del"/> 删除
这一步也比较关键,不过不用详细描述了，跟[输入](#insert)差不多，它是把输入的字符一个个加到`m_kFinalStr`，这一步则是调用一次，我就删除一个，维护光标位置和重新计算显示的串`m_kShowStr`即可，除非`m_kFinalStr`为空.
### <a name = "dettach"/>关闭输入法
这一步就是后续处理了，正常情况下需要让光标停止动作并隐藏，
### <a name = "other"/> 其他
- 记得在缩主类的析构函数中`m_pkField->didNotSelectSelf()`，为了避免在键盘弹出的时候，游戏跳转到了另外一个界面，这时候键盘就消不下去了。别问我为什么要提示，因为我遇到过...
- 过滤非法字符和缩放什么的~，计算的时候记得带进去~
- 调用系统的输入框也是kuo以的~不用我这么麻烦
- 真·源码在[下篇文章]\(待完成)中和[github]\(待完成)中出现.