---
title: Auto-append the MySQL MERGE ENGINEs UNION
date: 2017-06-20 16:36:37
tags: mysql
---


## 如何在创建分表时自动维护主表的union??

---
 
### ENGINE=MERGE
MySQL提供一种分表机制是创建一个engine=merge的主表.子表为engine=MyISAM,然后主表属性union=(...)来*告诉*它子表有哪些.
例如
```sql
create table t1(
    id int
)engine = MyISAM;

create table t2 like t1;

create table t_merge(
    id int
)engine = merge insert_method=last union=(t1,t2);
```
这样就能创建一个名为t_merge的主表，t1,t2为子表.在子表插入数据的时候，直接查询主表可得到子表的数据.
其他属性网上的资料非常多，官网也介绍得特别好.
重点是这个**union=(xxx)**,每创建一个子表就需要去维护一次.

<!-- more -->

据[官方文档](https://dev.mysql.com/doc/refman/5.7/en/merge-storage-engine.html)介绍：

> To remap a MERGE table to a different collection of MyISAM tables, you can use one of the following methods:

> - DROP the MERGE table and re-create it.

> - Use ALTER TABLE tbl_name UNION=(...) to change the list of underlying tables.

> It is also possible to use ALTER TABLE ... UNION=() (that is, with an empty UNION clause) to remove all of the underlying tables. However, in this case, the table is effectively empty and inserts fail because there is no underlying table to take new rows. Such a table might be useful as a template for creating new MERGE tables with CREATE TABLE ... LIKE.



也就是说，你在创建子表的时候。1，删除主表重新创建;2，alter table table_name union=(sub_table_list);两个办法可以修改主表的union.
可是啊。我们的表那么多(~~其实也就二三十张~~)，创建子表那么频繁(~~一个月一次~~)。肯定不能是人工维护的.

google了半天，似乎也没有搜到啥方便的方案.
于是自己想了个比较笨拙的方法.

### 一个暂时可用的方案.

对于需要建分表的数据.表名我们一般都会按照时间来定，比如我有一个玩家注册表`register`(<span style="background:#333333">~~玩家注册你也要分表?你一个月要注册几亿玩家吗?~~</span>),那么我的子表名一般会定义成`register1706`，`register1707`表示2017年6月和7月份注册的玩家.这里好像在[前面](/2017/06/19/mysql-dynamic-table#ctd)说过

所以我的方案可以概括成:
1. 把后缀存起来
2. 创建表时，把后缀连接成字符串
3. 动态执行sql语句.

#### 创建一个存后缀的表.

```sql
drop table if exists table_suffix;
create table table_suffix(
    suffix smallint not null unique
);
```
存的数据是这样的:
```sql
mysql> select * from table_suffix;
+--------+
| suffix |
+--------+
| 1706   |
| 1707   |
| 1708   |
+--------+
3 rows in set (0.00 sec)
```

#### 把后缀连接成字符串

先看一下正常修改语句的形式是`alter table register union=(register1706,register1707,register1708)`。我们需要组成的字符串,就是括号里面的那句.
为了通用,我们可以创建一个function，传入主表名，返回字符串.

```sql
CREATE FUNCTION `f_concat_union`(ftablename text) RETURNS text CHARSET utf8
BEGIN
    declare vrlt text default '';
    declare vdone int default 0;
    declare vsfx int default 0;
    declare vcur cursor for select suffix from table_suffix;
    declare continue handler for sqlstate '02000' set vdone = 1;
    
    open vcur;
    while vdone=0 do
        fetch vcur into vsfx;
        if 0 = vdone then
            if vrlt != '' then
                set vrlt=concat(vrlt,',');
            end if;
            set vrlt=concat(vrlt,ftablename,vsfx);
        end if;
    end while;
    close vcur;
    
    RETURN vrlt;
END
```
打开游标循环后缀，然后组成字符串返回。例子如下:

```sql
mysql> select f_concat_union('payment');
+-------------------------------------+
| f_concat_union('payment')           |
+-------------------------------------+
| payment1706,payment1707,payment1708 |
+-------------------------------------+
1 row in set (0.00 sec)
```

#### 动态执行sql语句

已经拿到括号内的字符串了，所以这时候我们直接像前面说动态创建表那样执行sql语句就ok了.
因为是在创建子表的时候要执行的，所以创建存储过程来完成一整套流程.

```sql
CREATE PROCEDURE `p_backup_table`(IN `ptablename` text,IN `pyearmonth` smallint)
BEGIN
    set @q0 = concat('drop table if exists ',ptablename,pyearmonth);
    set @q1 = concat('create table ',ptablename,pyearmonth,' like ',ptablename);
    set @q2 = concat('alter table ',ptablename,pyearmonth,' engine=MyISAM');
    set @concat_union = f_concat_union(ptablename);
    set @q3 = concat('alter table ', ptablename,' union=(',@concat_union,')');
    
    prepare stmt from @q0;
    execute stmt;
    drop prepare stmt;

    prepare stmt from @q1;
    execute stmt;
    drop prepare stmt;

    prepare stmt from @q2;
    execute stmt;
    drop prepare stmt;
    
    prepare stmt from @q3;
    execute stmt;
    drop prepare stmt;
END
```
传入的参数是主表名和年月(后缀)
整个流程是，删表如果存在，创建表结构和主表一样，修改子表engine=MyISAM，修改主表的union.关键的是最后一步，不再需要我们人工维护主表的union.
PS：记得在调用前要先把后缀insert到table_suffix表中.

刚才payment建表的结果和payment表的DDL:
```sql
mysql> show tables;
+----------------------+
| Tables_in_testmerge  |
+----------------------+
| payment              |
| payment1706          |
| payment1707          |
| payment1708          |
| table_suffix         |
+----------------------+
5 rows in set (0.00 sec)

mysql> show create table payment;
+---------+---------------------------------------------------------------------------------------------------+
| Table   | Create Table                                                                                      |
+---------+---------------------------------------------------------------------------------------------------+
| payment | CREATE TABLE `payment` (
  `id` int(11) NOT NULL,
  `val` int(11) NOT NULL
) ENGINE=MRG_MyISAM DEFAULT CHARSET=utf8 INSERT_METHOD=LAST UNION=(`payment1706`,`payment1707`,`payment1708`) |
+---------+---------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

可以看出主表payment的union已正常更新.

---
### 题外
到此动态建表的操作已经差不多了，还剩下一个问题就是在正常运营后，我们会删表(数据太多了当然得删啦)，这时候还要来维护一下这个union(不然表结构就GG了)。思路大概就是在f_concat_union里面入手了，判断一下.在删除表的时候不要直接删(本来就不应该这样),走删表正常流程，再加个上面的@q3就差不多了.后面再说吧:)