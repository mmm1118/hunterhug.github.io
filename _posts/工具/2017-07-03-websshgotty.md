---
layout: post
title: "Web SSH远程连接利器:gotty"
date: 2017-07-03
author: silly
categories: ["工具"]
desc: "从Web端ssh连接到机器或者机器内的容器"
tags: ["docker"]
permalink: "/tool/gotty.html"
--- 

这个东东能让你使用浏览器连接你远程的机器!

# 一. 环境准备

下载[https://github.com/yudai/gotty](https://github.com/yudai/gotty)

请先配置好Golang环境,然后

```
go get github.com/yudai/gotty
```

否则请下载二进制文件

# 二. ssh到机器

在机器中命令执行

```
./gotty --port 9090 -c admin:12345 -w bash
```

打开浏览器`http://127.0.0.1:9090/`,输入帐号:admin,密码:12345

![](/picture/public/gotty.png)

# 三. ssh到容器内

新建Dockerfile,在同一目录放入gotty二进制文件

```
FROM golang:1.8
ADD ./gotty /root/debug/gotty
RUN chmod 755 /root/debug/gotty
```

build 镜像

```
docker build -t gotty .
```

运行该容器,暴露端口,并在容器内执行命令

```
docker run --rm -it -p 9090:9090 gotty /bin/bash
/root/debug/gotty --port 9090 -c admin:12345 -w bash
```