---
title: FissureServer(四)
date: 2016-05-27 22:12:31
tags: GameServer
---

## Mysql
## Timer

### Mysql

着手写了Mysql存储数据.
这里利用了我们自定义的特性(Attribute)，也就是用在class field上面的[]，比如[obsote]表示过期。
如果要写入数据，我们得自定义种特性[DataEntityTable]表示这个结构要存入数据库。里面的字段用[DataEntityField]表示要存入的字段
在编译的时候，我们会把编译好的Assembly利用反射，找到带有[DataEntityTable]特性的类，和标有[DataEntityField]的字段，进行创建(修改)表.
这里还利用了一个名叫`MySql.Data.dll`的dll。主要用来执行sql语句，

未完待续...