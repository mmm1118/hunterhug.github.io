---
layout: post
title: "mesos安装"
date: 2017-05-02
author: silly
desc: "mesos安装"
categories: ["工具"]
tags: ["mesos"]
permalink: "/tool/mesos-install.html"
--- 

[mesos入门介绍](http://blog.csdn.net/lsshlsw/article/details/47086869)

# docker-compose方式安装

安装docker-compose，然后新建文件夹，新建文件`docker-compose.yml`

```
zk:
    image: mesosphere/mesos:1.2.0
    command: /usr/share/zookeeper/bin/zkServer.sh start-foreground
master:
    image: mesosphere/mesos-master:1.2.0
    environment:
        - MESOS_ZK=zk://zk:2181/mesos
        - MESOS_LOG_DIR=/var/log/mesos
        - MESOS_QUORUM=1
        - MESOS_WORK_DIR=/var/lib/mesos
    links:
        - "zk:zk"
    ports:
        - "5050:5050"
    volumes:
        - ./target:/tmp/bin
slave:
    image: mesosphere/mesos-slave:1.2.0
    environment:
        - MESOS_MASTER=zk://zk:2181/mesos
        - MESOS_LOG_DIR=/var/log/mesos
        - MESOS_LOGGING_LEVEL=INFO
        - MESOS_LAUNCHER=posix
        - MESOS_WORK_DIR=/var/lib/mesos
    links:
        - "zk:zk"
        - "master:master"
    volumes:
        - ./target:/tmp/bin
```

上面volumes可修改

运行这些镜像在后台`-d`，帮助查看：`man docker-compose`

```
docker-compose up -d
```

在mesos运行框架：`docker exec  montecarloarea_master_1 bash -c "/tmp/bin/binary arg1 arg2"`

关闭则

```
docker-compose stop
```

# docker方式安装(有点问题)

以下为ubuntu操作系统，参考：http://www.open-open.com/lib/view/open1421195966031.html

配置本机IP：127.0.0.1
```
jinhan@jinhan-chen-110:~$ HOST_IP=127.0.0.1
jinhan@jinhan-chen-110:~$ echo $HOST_IP
127.0.0.1
```

## zookeeper

1.手动build镜像安装zookeeper

https://github.com/jplock/docker-zookeeper

```
FROM openjdk:8-jre-alpine
MAINTAINER Justin Plock <justin@plock.net>

ARG MIRROR=http://apache.mirrors.pair.com
ARG VERSION=3.4.10

LABEL name="zookeeper" version=$VERSION

RUN apk add --no-cache wget bash \
    && mkdir -p /opt/zookeeper \
    && wget -q -O - $MIRROR/zookeeper/zookeeper-$VERSION/zookeeper-$VERSION.tar.gz \
      | tar -xzC /opt/zookeeper --strip-components=1 \
    && cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg \
    && mkdir -p /tmp/zookeeper

EXPOSE 2181 2888 3888

WORKDIR /opt/zookeeper

VOLUME ["/opt/zookeeper/conf", "/tmp/zookeeper"]

ENTRYPOINT ["/opt/zookeeper/bin/zkServer.sh"]
CMD ["start-foreground"]
```

构建镜像
```
sudo docker build -t jinhan/zookeeper:3.4.10 .
```

2.使用别人做好的zookeeper镜像

```
sudo docker pull garland/zookeeper
```

3.启动

```
sudo docker run -d \
-p 2181:2181 \
-p 2888:2888 \
-p 3888:3888 \
garland/zookeeper


# 进入容器测试zk
sudo docker exec -i -t  18d71ccb9b0b /bin/bash

./zkCli.sh -server 127.0.0.1:2181
ls /

```

## mesos-master

1.拉取镜像

```
sudo docker pull garland/mesosphere-docker-mesos-master
```

2.跑master

```
sudo docker run --net="host" \
-p 5050:5050 \
-e "MESOS_HOSTNAME=${HOST_IP}" \
-e "MESOS_IP=${HOST_IP}" \
-e "MESOS_ZK=zk://${HOST_IP}:2181/mesos" \
-e "MESOS_PORT=5050" \
-e "MESOS_LOG_DIR=/var/log/mesos" \
-e "MESOS_QUORUM=1" \
-e "MESOS_REGISTRY=in_memory" \
-e "MESOS_WORK_DIR=/var/lib/mesos" \
-d \
garland/mesosphere-docker-mesos-master
```

3.跑slave

```
sudo docker run --net="host" -d \
--name mesos_slave_1 \
--entrypoint="mesos-slave" \
-e "MESOS_MASTER=zk://${HOST_IP}:2181/mesos" \
-e "MESOS_LOG_DIR=/var/log/mesos" \
-e "MESOS_LOGGING_LEVEL=INFO" \
garland/mesosphere-docker-mesos-master
```

## marathon

1.拉取镜像

```
sudo docker pull garland/mesosphere-docker-marathon
```

2.跑

不加host会连不上zk

```
指定方法：--net="host"
这种创建出来的容器，可以看到host上所有的网络设备。
容器中，对这些设备（比如DUBS）有全部的访问权限。因此docker提示我们，这种方式是不安全的。
```

```
sudo docker run --net="host" -d -p 8080:8080 \
garland/mesosphere-docker-marathon --master zk://${HOST_IP}:2181/mesos --zk zk://${HOST_IP}:2181/marathon
```

## Web界面

1. Mesos界面

```
http://${HOST_IP}:5050
```

2. Marathon界面

```
http://${HOST_IP}:8080
```

新建APP ：`echo "1" >> /tmp/output.txt`


进容器:

```
sudo docker exec -it mesos_slave_1 /bin/bash
tail -f /tmp/output.txt
```

# 非docker安装

UBUNTU16.04，下载好源码！并且要以root用户执行！

```
# Update the packages.
$ sudo apt-get update

# Install a few utility tools.
$ sudo apt-get install -y tar wget git

# Install the latest OpenJDK.
$ sudo apt-get install -y openjdk-8-jdk

# Install autotools (Only necessary if building from git repository).
$ sudo apt-get install -y autoconf libtool

# Install other Mesos dependencies.
$ sudo apt-get -y install build-essential python-dev python-virtualenv libcurl4-nss-dev libsasl2-dev libsasl2-modules maven libapr1-dev libsvn-dev zlib1g-dev

# Change working directory.
$ cd mesos

# Bootstrap (Only required if building from git repository).
$ ./bootstrap

# Configure and build.
$ mkdir build
$ cd build
$ ../configure
$ make

# Run test suite.  跑例子必须！！！会生成一些东西。
$ make check

# Install (Optional).
$ make install

# Change into build directory.
$ cd build

# Start Mesos master (ensure work directory exists and has proper permissions).
$ ./bin/mesos-master.sh --ip=127.0.0.1 --work_dir=/var/lib/mesos

# Start Mesos agent (ensure work directory exists and has proper permissions).
$ ./bin/mesos-agent.sh --master=127.0.0.1:5050 --work_dir=/var/lib/mesos

# Visit the Mesos web page.
$ http://127.0.0.1:5050

# Run C++ framework (exits after successfully running some tasks).
$ ./src/test-framework --master=127.0.0.1:5050

# Run Java framework (exits after successfully running some tasks).
$ ./src/examples/java/test-framework 127.0.0.1:5050

# Run Python framework (exits after successfully running some tasks).
$ ./src/examples/python/test-framework 127.0.0.1:5050
```
