---
title: 噩梦般的半周
date: 2017-07-28 17:31:54
tags: [杂谈,lua]
---


本来是想回忆一下这两周的情况.很惨，记忆也有些不清了，上周的比较深刻，就这周本来也让人很苦恼呢.懒得写了.


### 7.14(周五)

---

前些天他们在大改协议相关的东西，我这边下午程序crash了一次,跪在莫名奇妙的地方,我以为是他们没提完的原因,晚上他们提交，我更新代码，顺利跑N久(约一小时吧).于是提交svn.并部署到内网外开启压测.内心念了一句，明天不用加班工作完美完成(~~flag已高高立起~~)...

<!-- more -->

### 7.17(周一)

---

早上一来同事A便说，周六加班的时候crash了，很惨，把你后台的配置关了也不行.压测只500人，开一会儿便报错提示栈不够用.其中某个地方还有死循环.
于是我开始review代码和重现(其实这时候我还很觉得不是我这边的问题，这版本改动那么大，我这边逻辑很简单，不会用到太多栈)。
我开着50到200人的机器人压测，发现挺顺利的!没错，一直不crash，这就让我很迷了.我要求同事A把工程rebuild再试试.神奇的事情是，他那边随便跑一下，就报错说栈不够用，而且不够的点，非常随机。
没错我们遇到了传说中的**random error**.
他也查得很累，到了下午两边都没啥进展.脸色略显难看...
晚上大约五点过，我把我的思路告诉他，我们一起review我的代码.逻辑很清晰，无递归，变量不足以导致崩溃.找了一会儿，发现我的确有个地方没有delete.在向rsyslog发数据的时候，一直在new，在线程里面发送完应该delete但我忘了.
晚上吃完饭，我把bug修掉，再跑，我这边仍然很顺利.于是提交svn，他更新，跑了一会儿，好像也没崩溃.于是下班时部署内外网开启压测。开心地下了班..
(yeah~,这时候天真的我以为万事大吉了)

### 7.18(周二)

---

早上一来同事A说，崩了，依然random error.我的内心开始惊恐，但我心里是认为问题出了他们那里.我昨天那bug已经修了，他们改了那么多.
日子如同周一一样，我这边依然跑得很顺利，他那边依然re。
两边脸色比昨天更难看。
于是几个server端的同事都开始review自己的功能.一起讨论.终无果。
晚上的时候找到了那个死循环的地方,是另一个同事写的商店还是啥的，随机物品，表没配好导致的.加了保护解决了死循环.
但这时候的问题表现仍然是：开着50个机器人跑，没一会儿就会random error.这时候不是栈不够了，而是各种指针为空.各种访问无权限。
直到晚上不开心的下了班....

问题出现了两天，server感觉已经不稳定了近一周，心里压力突然大了起来，因为他们都认为问题出在我这里.我不怕被开除，但怕同事认为是个坑。

### 7.19(周三)

---

问题拖了两天，安排的活也停滞了两天没做，同事A决定使用终极手段：版本回退。

首先把版本回退到我提交之前.结果是没崩溃，死循环在.~~我的内心已经开始哭泣~~
然后一个版本一个版本回，然后把和上一个版本对比的修改注释掉(这样改动比较好控，不是一次回退多个版本)。
直到下午四五点，回退10多个版本后。
不崩溃了。
发现注释的代码是(去掉了多余的部分，贴了关键部分)：
```lua
void ScriptLua::OnItemChange(OGSLib::DomainPlay* player, OGSLib::SubItem* item, OGSLib::ItemChangeType optype, int src)
{
    int result_catch = 0;
    const char* function = "gm.rsys.OnOpItem";
    try
    {
        luaL_findfunction(m_l, LUA_GLOBALSINDEX, function);
        if (lua_isfunction(m_l, -1))
        {
            ...
            result_catch = report(m_l, lua_pcall(m_l, 4, 0, 0), function);
        }
        lua_pop(m_l, 1);
    }
    catch (...)
    {
    }
    ...
}
```
他们的内心惊喜无比，我的内心崩溃无比。问题还是出在了我这里，原来我是个坑。
可是，这代码有啥问题？
我说这函数是为了在玩家的道具变化时记到rsyslog。在C++层调用lua函数在那边组包发送.调用的代码是在前面复制的模板改的.逻辑没问题，参数也没问题。要是report出错我也是能catch到的.
几个人看着这段代码大眼瞪小眼。
找了前面几个函数对比了一下。
发现。有的函数那个lua_pop(m_l,1)，是用else括起来的，有的没有，很是迷，如下：
```lua
if (lua_isfunction(m_l, -1))
{
    ...
}else
{
    lua_pop(m_l, 1);
}
```
然后我把注释取消，加上else.
嗯是的没错，没有崩溃...终于定位了问题就出现在这里.

经过多个函数对比，我们找到规律：C++调用lua的时候，如果有返回值，才需要直接pop，如果没有就要加else.(这是前人写的框架，已离职，只有我们几个自己探索了)

我们的脸上终于泛起一丝正常的光.

晚上我请他们吃饭.算作我把他们坑得这么惨的赔偿吧.(~~嗯不对，应该是我被lua坑得这么惨的代价2333~~)，嗯当然他们希望来顿大的,可是还要赶着回来修bug呢,大的留着下次吧.

### 7.20(周四)

---

早上醒来首先想到的是，昨夜有没有崩溃呢(瑟瑟发抖.jpg)...

其实事情并没有完...
