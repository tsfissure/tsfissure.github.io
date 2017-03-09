---
title: pymssql乱码
date: 2017-01-06 10:42:13
tags: py
---

### 题外话
---
元旦居然没出去玩儿..花了一整天研究pymssql并把数据库之前MYSql更改为SQL Server,迁移真是累哭.然后迁移完发现，unicode传过去不显示或者显示编码过后的字符串.

<!-- more -->
### 正文
---
pymssql 操作数据有2种，一种是直接在代码里写sql语句然后query(),还有一种是用SSMS(或者其他工具)编写存储过程，在代码里直接调用存储过程的名字.
对于第一种，网上千篇一律的encode可以解决
然而我用的是存储过程.各种encode，charset无用.
经过无数的尝试终于得到解决.
假设我有存储过程
```sql
ALTER PROCEDURE [dbo].[register]
    @nickname   NVARCHAR(32),
    @sex        INT
AS
BEGIN
    SET NOCOUNT ON;
    ---...
    SELECT 1
END
```
首先调用存储过程有两种方式
- `callproc(procname, paramas)` #仅仅是存储过程的名字如 `cursor.callproc("register", ("tsfissure", 1))`
- `execute(procname, params)` #存储过程名字带上参数如 `cursor.execute("register @nickname=%s,sex=%d", ("tsfissure", 1))`

这里如果我们使用第一种`callproc`，你会发现不管怎么弄。如果我的nickname是中文，传到那边过后。不会存储成中文。
我们应该使用第二种，这样传过去，就可以保存成unicode了。
