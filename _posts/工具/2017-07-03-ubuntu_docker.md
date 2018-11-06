---
layout: post
title: "Ubuntu docker安装全套"
date: 2017-07-03
author: silly
categories: ["工具"]
desc: "Ubuntu16.04安装docker并开启远程访问"
tags: ["docker"]
permalink: "/tool/docker-ubuntu-install.html"
--- 

# 一. 环境说明

```
jinhan@jinhan-chen-110:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.2 LTS
Release:	16.04
Codename:	xenial
jinhan@jinhan-chen-110:~$ uname -a
Linux jinhan-chen-110 4.4.0-81-generic #104-Ubuntu SMP Wed Jun 14 08:17:06 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

# 二. 安装

参考: [https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)

```
sudo apt-get remove docker docker-engine docker.io
sudo apt-get update
sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce
sudo docker run hello-world
```

# 三. 开启远程访问

```
jinhan@jinhan-chen-110:~$ sudo vi /etc/default/docker

# Docker Upstart and SysVinit configuration file
DOCKER_OPTS='-H 0.0.0.0:6000 -H unix:///var/run/docker.sock'

jinhan@jinhan-chen-110:~$ vim /etc/default/docker 
jinhan@jinhan-chen-110:~$ ps -ef|grep  docker
root     11442     1  0 10:51 ?        00:00:02 /usr/bin/dockerd -H fd://
root     11451 11442  0 10:51 ?        00:00:01 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc
jinhan   13116 12039  0 11:06 pts/22   00:00:00 grep --color=auto docker
```

这时我们发现设置并没有生效，这是因为`/etc/default/docker`文件是为upstart和SysVInit准备的（正如文件第一行注释所言），而使用service命令时并不会读取它，因此我们还需要做如下更改：

```
jinhan@jinhan-chen-110:~$ sudo vim /lib/systemd/system/docker.service

#ExecStart=/usr/bin/dockerd -H fd://
#改为下面
ExecStart=/usr/bin/dockerd -H fd:// $DOCKER_OPTS
EnvironmentFile=/etc/default/docker

jinhan@jinhan-chen-110:~$ sudo systemctl daemon-reload
jinhan@jinhan-chen-110:~$ sudo systemctl restart docker

jinhan@jinhan-chen-110:~$ ps -ef|grep docker
root     15321     1  0 11:25 ?        00:00:00 /usr/bin/dockerd -H fd:// -H 0.0.0.0:6000 -H unix:///var/run/docker.sock
root     15330 15321  0 11:25 ?        00:00:00 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc
jinhan   15757 12039  0 11:28 pts/22   00:00:00 grep --color=auto docker
```


# 四. 其他配置

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ztndgg1k.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

你可以配置某些仓库地址(第一个是阿里云加速仓库地址,第二个忽略https安全):

```
{
  "registry-mirrors": ["https://ztndgg1k.mirror.aliyuncs.com"],
  "insecure-registries": ["http://xx.dd.cc"]
}
```

然后登录:

```
sudo login http://xx.dd.cc
```

以下是打包并推送:

```
docker build -t mymmymy .
docker tag mymmymy xx.dd.cc/ddd
docker push  xx.dd.cc/ddd
```
