---
layout: post
title: "Golang安装和配置"
date: 2017-05-02
author: silly
desc: "Golang环境配置"
categories: ["工具"]
tags: ["Golang"]
permalink: "/tool/golang-config.html"
--- 

# Linux Golang

下载源码，解压。

下载IDE，也是解压，然后设置环境变量。

Linux版本Golang:

```
# /home/jinhan为你的主目录
mkdir /home/jinhan/go
cd /home/jinhan/go

wget https://studygolang.com/dl/golang/go1.9.2.linux-amd64.tar.gz
wget https://studygolang.com/dl/golang/go1.8.5.linux-amd64.tar.gz

tar -zxvf go1.9.2.linux-amd64.tar.gz

vim /etc/profile.d/myenv.sh

export GOROOT=/home/jinhan/go
export GOPATH=/home/jinhan/code
export GOBIN=$GOROOT/bin
export PATH=.:$PATH:/app/go/bin:$GOPATH/bin:/home/jinhan/software/Gogland-171.3780.106/bin

source /etc/profile.d/myenv.sh
```

其中`/home/jinhan/software/Gogland-171.3780.106/bin`为IDE二进制所在位置, [IDE下载](https://www.jetbrains.com/go/download/download-thanks.html?platform=linux).

因为`Golang`的版本较多, 我建议应该适当更新.

# Docker Golang

也可以将源码挂载进`docker`中进行编译, 然后在生产环境下放二进制.如:

```
# 下载源码
git clone https://github.com/hunterhug/huhu.git

# 进入目录
cd huhu

# 拉golang docker
docker pull golang:1.9

# 将源码挂载进容器, 在容器里面编译
docker run -it --rm -v $PWD:/go/src/github.com/hunterhug/huhu golang:1.9 /bin/bash
cd /go/src/github.com/hunterhug/huhu
go build
exit

# 退出容器后目录下会有一个二进制文件
./huhu
```