---
title: CentOS上Rsyslog升级
date: 2017-06-014 20:09:57
tags: rsyslog
---

为什么会有这么low的一话题.
因为，我想吐槽一下各搜索的结果和，官方文档，顺便让新手少走弯路.

---
<!-- more -->

我所用的系统版本是CentOS 6.5,直接用`yum update`或者`yum install`命令处理rsyslog,最高版本也只能找到5.8.x的。
rsyslog的官方文档不愧是被无数用户吐槽的文档，很！难懂..是的没错我也没有找到升级的方案在里面
所以我得去找其他方式升级一下.

在bing和baidu中输入关键字`upgrade rsyslog centos`，你会发现，翻几页，也找不到一个可行方案~~(可能有麻烦的我嫌麻烦跳过了)~~.
最后才在google中输入关键字，第二个链接，就帮我解决了问题.

附上链接:[传送门](https://gist.github.com/baskaran-md/7eed2bf213b0e857960e)

```bash
yum install -y wget
wget http://rpms.adiscon.com/v8-stable/rsyslog.repo
mv rsyslog.repo /etc/yum.repos.d/rsyslog.repo
yum info rsyslog --skip-broken
yum install -y rsyslog
rsyslogd -version
```

上面几行代码的问题，我在另外两个搜索引擎中找了几页也没找到.
如果你已经装了wget，第一行还可以跳过，#4#6行也只是查看一下信息.所以重要的信息也就只有那个http链接了。

---

但是.
也并不是那么顺利的.
因为我原来就有安装rsyslog 5.8.x的，所以我们还需要做一些简单的处理。
新安装的rsyslog会新生成一个`/etc/rsyslog.conf.newrpm`文件，我们需要把后缀`.newrpm`去掉，所以原来的`rsyslog.conf`我们必须得删掉。
因为版本差异太大，老的配置语法似乎不怎么支持了，所以我们也得删(重)掉(写)所有自己以前的`/etc/rsyslog.d/*.conf`配置文件
然后`service rsyslog start` 成功运行.如果上面操作没有，可能会出现`segmentfault error`
