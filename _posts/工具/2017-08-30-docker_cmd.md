---
layout: post
title: "Dockerfile中CMD与ENTRYPOINT的区别 "
date: 2017-08-30
author: silly
categories: ["工具"]
desc: "Dockerfile中CMD与ENTRYPOINT的区别 "
tags: ["linux","docker"]
permalink: "/tool/docker-cmd.html"
--- 

开发和使用`docker`久了后,有些体会:

1. ENTRYPOINT，表示镜像在初始化时需要执行的命令，不可被重写覆盖，CMD参数可以接在其后面,所以那些可以变化的参数都写在CMD.
2. CMD，表示镜像运行默认参数，可被重写覆盖, docker run是如果后面有参数,呵呵.
3. ENTRYPOINT/CMD都只能在文件中存在一次，并且最后一个生效, 多个存在，只有最后一个生效，其它无效！CMD 可以补充,见 1
4. 需要初始化运行多个命令，彼此之间可以使用 && 隔开，但最后一个须要为无限运行的命令，不然容器运行后就退出了
