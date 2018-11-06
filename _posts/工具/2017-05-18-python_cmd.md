---
layout: post
title: "python命令集合"
date: 2017-05-18
author: silly
desc: "python命令集合"
categories: ["工具"]
tags: ["python"]
permalink: "/tool/python-cmd.html"
--- 

```
pip - A tool for installing and managing Python packages
```

pip3是Python3的包安装和管理软件。

```
pip3 install -r requirement.txt
```

requirement.txt里面按行写入第三方包，`#`为注释！

导入自己的包！

```
export PYTHONPATH=/home/jinhan/pythonlib
ln -s /home/jinhan/book/dudu /home/jinhan/pythonlib/dudu

/home/jinhan/pythonlib/dudu -> /home/jinhan/book/dudu/
```