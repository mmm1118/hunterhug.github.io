---
layout: post
title: "第N次Ubuntu安装记录"
date: 2017-07-15
author: nil
desc: "只是一个幼稚的记录..."
categories: ["杂文"]
tags: ["杂文","ubuntu"]
permalink: "/cao/ubuntu-record.html"
---

更新, 我在2018-01-07日又宣布重抱Windows!

今天是一个愉快的日子，阳光明媚，2017.7.15,深圳特区风景不错！

看着自己的Win10，之前系统怀了就换成Win10了，现在觉得真是不好用，docker都弄不了，好多工具也好难用，所以这是第N次装Ubuntu了。这个N好多年前了，大一时装了第一次Ubuntu,那时好像还是12的版本，五年过去了，人变丑了，可操作系统乌干达好像没变丑。。。

先下载一个镜像，克隆在U盘中，然后安装，我是选择自己分区，划了250M给/boot，8000M给交换空间，其他全部给/
然后按提示安装，请不要作死使用安装第三方软件，flash的官网中国大区访问受限。

Ubuntu版本号：

```
xxx@xxxxx-Inspiron-3521:~$ uname -a
Linux xxx@xxxxx-Inspiron-3521 4.8.0-36-generic #36~16.04.1-Ubuntu SMP Sun Feb 5 09:39:57 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
xxx@xxxxx-Inspiron-3521:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.2 LTS
Release:	16.04
Codename:	xenial
```

装好后我嫌弃自带界面unity真难看，改成：

```
sudo apt-get  update
sudo  apt-get install  ubuntu-mate-core
sudo  apt-get install  ubuntu-mate-desktop
```

我运气差，遇见

```
E: 无法获得锁 /var/lib/dpkg/lock - open (11: 资源暂时不可用)
E: 无法锁定管理目录(/var/lib/dpkg/)，是否有其他进程正占用它？
```

我们强制解锁！！

```
sudo rm /var/lib/apt/lists/lock
sudo rm /var/lib/dpkg/lock
```

然后因为win10残留了几个ntfs盘，挂载时提示错误啥的，请看清楚盘号，然后

```
sudo ntfsfix /dev/sda6
```


这是第一篇无聊的记录！总要记录点什么！