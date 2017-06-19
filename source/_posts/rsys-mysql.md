---
title: rsyslog远程日志落实到mysql
date: 2017-06-16 20:02:36
tags: [rsyslog,mysql]
---


在v8版本的rsyslog中，把远程接收到的日志落实到mysql的一种合理配置方式及流程.
<!-- more -->

在rsyslog和mysql都安装完成的情况下。

---
### 建立相应数据库和专用账号
```
# mysql -uroot -p
mysql> create database TSRsyslog;
mysql> grant all on TSRsyslog.* to `rsyslog`@`localhost` identified by '123456';
mysql> flush privileges;
```
第一行代码是进入到mysql环境下.然后依次建立数据库TSRsyslog，创建账号`rsyslog`@`localhost`密码123456同时开放TSRsyslog的所有权限

---
### 创建日志处理所用的存储过程
例子:
```cpp
CREATE DEFINER=`rsyslog`@`localhost` PROCEDURE `p_log_factory`(IN `ptime` timestamp,IN `pmsg` text)
BEGIN
    declare vlogtype int default 0;
    insert systemevents(rtime, message) values(ptime, pmsg);
END
```

---
### 安装rsyslog的ommysql模块
```cpp
yum install rsyslog-mysql
```

---
### rsyslog配置
最重要的一步，配置rsyslog
首先在`/etc/rsyslog.d/`下面建立mysql专用的配置文件

```cpp
# file:tsmysql.conf

module(load="ommysql.so")

template (name="tsmysqltpl" type = "string" option.sql="on" string="call p_log_factory('%timereported:::date-mysql%','%msg:2:$%')")
        
ruleset(name="tslogrule"){
        if prifilt("local1.*") then {
                :ommysql:localhost,TSRsyslog,rsyslog,123456;tsmysqltpl
                stop
        }
}
```
就三步，首先加载ommysql模块，然后建立写入数据库的模板，最后写收到日志时的规则
规则里我只处理是local1.*的日志，stop表示只到mysql，后面就不做其他处理了(如果没有stop，而在其他文件又对local1.*创建了action，则会继续处理)
`:ommysql:localhost,TSRsyslog,rsyslog,123456;tsmysqltpl`，这里依次是:模块，地址(localhost),数据库(TSRsyslog),账号(rsyslog),密码(123456),处理方式模板(tsmysqltpl)
tsmysqltpl就是上面定义的模板,名字为(tsmysqltpl)，option.sql记得要等于on,后面string我就是调用的刚才的存储过程.当然可以直接insert,update啥的，不过，用存储过程比较好维护.

最后在主conf下开启udp接收远程日志并使用上面所写的规则
在`/etc/rsyslog.conf`文件中有两行代码把它取消注释并修改为新的配置(在前面添加了一行,input添加属性)
```cpp
#module(load="imudp") # needs to be done just once
#input(type="imudp" port="514")


# 修改为:
$IncludeConfig /etc/rsyslog.d/*.conf
module(load="imudp") # needs to be done just once
input(type="imudp" port="514" ruleset="tslogrule")
```
新加的那行原本配置里面也有，不过是在下面，但是我们要使用tslogrule规则。必须在前面include进来.
input表示从514端口收到的udp消息，使用我们的tslogrule规则，就是前面所写的tsmysql.conf配置里面的ruleset。

---
### 重启服务
```cpp
# service rsyslog restart
```

---
### 随意
经过如上操作，就可以完成从远程接收日志(514端口udp数据)，存到本机mysql通过TSRsyslog.p_log_factory存储过程.
配置尽量独立，不要应用的都去主配置里面修改，难于维护。
对数据库的处理语句尽量使用存储过程，因为我们可以在rsyslog运行的过程中修改存储过程，如果写在配置中了，每一次修改，都需要重启rsyslog服务。这是非常不友好的。
