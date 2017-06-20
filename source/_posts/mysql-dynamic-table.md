---
title: mysql事件event定时分表
date: 2017-06-19 20:12:58
tags: mysql
---


对于日志这种作用的数据库，如果历史记录不太重要，可以选择定期删除日志，比如现在在玩儿的阴阳师的突破记录只保留两天数据，近两年很火的棋牌类游戏，回放也只支持两三天的.但如果数据比较重要，比如玩家充值记录，消耗记录等，那就必须要考虑数据量大了后的影响了，比如查询插入操作太慢带来的负面影响之类的.一种优化方式就是分表备份.现在手上的活就有这需求，所以我决定按月分表.
开始想着在mysql收到数据时进行check一下是否需要分表，但是这数据量，担心扛不住而且也做了无数的无用功，最重要的是，rsyslog调用存储过程，并!不能在里面有诸如`create table`之类的语句，会返回给你一个`XXXXcan't return a result set in the given context`错误.
于是想着在mysql自身上去找解决方案，最后决定用event来解决这个问题。
<!-- more -->

---

### event定义
事件可以说是数据库独有的定时器，到点执行一次,可以随时增删改，但不能由其他程序调用(触发).
在创建时你可以创建成一次性或者重复性事件。
启用事件功能，需要设置开关。可以用mysql进入到mysql环境下执行`set global event_scheduler=1`.但这只是一次的，mysql重启后，就不行了，所以我们用第二种方式，在mysql的配置文件`/etc/my.cnf`中添加`event_scheduler=1`语句在`[mysqld]`下。这样就可以在重启mysql的时候也开启事件功能.

### 思路
写一个存储过程专门去check分表.如果当前时间应该所存在在的的表是否存在，如果不存在，就备份一下并清空现在正使用存放数据的表，比如有个recharge表，现在时间是2017-06-19，那么应该存在一个表名叫`recharge1706`的表.
创建一个事件，每天去调用一次


### 存储过程

预备知识1:获取当前时间的年月
```sql
set @vNow = now();
set @v1 = month(@vNow);
set @v2 = year(@vNow);
set @v3 = extract(year_month from @vNow);
set @v4 = date_format(@vNow, "%y%m");
select @v1,@v2,@v3,@v4

#----result:
6   2017    201706  1706
```
这是部分对时间的有趣处理结果.
可以看出，@v4是我们想要的结果.

预备知识2:动态创建表
```sql
set @q = concat('create table if not exists `recharge', 1706,'` (id int not null)');
prepare stmt from @q;
execute stmt;
drop prepare stmt;
```
这是创建一个表的语句例子，先把整个语句连接成字符串，然后变成sql语句，然后执行，清除。得到的结果是多了一个名为`recharge1706`的表。

所以最后总的存储过程如下：

```sql
CREATE DEFINER=`rsyslog`@`%` PROCEDURE `p_tst_check_table`()
BEGIN
    declare vYearMonth int default date_format(now(), "%y%m");
    declare vcnt int default 0;
    select vcnt = count(*) from `information_schema`.`tables` where table_name = concat('recharge', vYearMonth);
    if vcnt = 0 then
        set @q = concat('create table `recharge', vYearMonth,'` select * from recharge');
        prepare stmt from @q;
        execute stmt;
        drop prepare stmt;
        truncate table recharge;
    end if;
END
```
先从`information_schema.tables`中统计一下有多少条名为xxx的表，如果为0条，说明需要新创建，然后就是创建表recharge1706并把recharge表中的数据全部备份过来，清除recharge表的数据.

### 创建事件
创建事件的语法也非常简单，网上有各种姿势。这里创建一个符合我们的事件，每天check一次(调用上面的存储过程`p_tst_check_table`)，无限循环
```sql
create event if not exists event_check_table
on schedule every 1 day
do call p_tst_check_table()
```
这时候就可以每天调用一次存储过程了，可以用`show events`查询语句查看已建立的事件.

### 题外
这只是个例子，细心一点会发现创建表备份不应该利用当前时间的年月，应该备份的是上个月的。
事件的触发是在创建的那个点的每一天，不是每一天的0点。
这些就在具体的逻辑中去细化吧.