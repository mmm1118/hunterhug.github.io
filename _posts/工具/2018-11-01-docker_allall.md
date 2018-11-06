---
layout: post
title: "Docker相关环境全套安装文档兼小技能"
date: 2018-11-01
author: silly
categories: ["工具"]
desc: "docker环境搭建请看这里"
tags: ["docker"]
permalink: "/tool/docker_allall.html"
---

# 这是标题

以下环境皆为ubuntu16.04，主要安装docker，docker-compose，docker仓库等。

## Docker安装

参考[官方](https://docs.docker.com/install/linux/docker-ce/ubuntu)

### A: 有源安装

```
sudo apt-get remove docker docker-engine docker.io
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo apt-get update
apt-cache madison docker-ce
sudo apt-get install docker-ce=docker-ce=18.03.0~ce-0~ubuntu
sudo docker run hello-world
```

### B: 无源安装

先下载已[编译包](https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64)。

```
wget https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce_18.06.1~ce~3-0~ubuntu_amd64.deb
sudo dpkg -i docker-ce_18.06.1~ce~3-0~ubuntu_amd64.deb
sudo docker run hello-world
```

--

在2017年的3月1号之后，Docker的版本命名开始发生变化，同时将CE版本和EE版本进行分开， 18.03表示18年3月发布。

离线安装命名前docker(docker-engine depends on libltdl7 (>= 2.4.6);)：

```
wget https://apt.dockerproject.org/repo/pool/main/d/docker-engine/docker-engine_1.12.1-0~xenial_amd64.deb
wget http://archive.ubuntu.com/ubuntu/pool/main/libt/libtool/libltdl7_2.4.6-4_amd64.deb
dpkg -i *.deb
```

## Docker-compose安装

我们可以使用docker-compose来对多个容器进行管理。

离线安装：

```
wget https://github.com/docker/compose/releases/download/1.8.1/docker-compose-`uname -s`-`uname -m`
 
mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## Docker Hub安装

VMware公司开源的企业级的Docker Registry管理项目：[harbor](https://github.com/goharbor/harbor)。

有了仓库，我们可以直接push镜像上去，然后从其他地方拉，不用借助U盘。

安装参考: 

[官方](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)，
[文章](http://blog.51cto.com/11093860/2117805)

环境依赖较新的`docker1.10+`和`docker-compose1.60+`和`python2.7`，我们选择离线安装方式：

```
wget https://storage.googleapis.com/harbor-releases/release-1.6.0/harbor-offline-installer-v1.6.0-rc3.tgz
tar xvf harbor-offline-installer-v1.6.0-rc3.tgz
cd harbor
```

编辑`docker-compose.yml`：

```
  proxy:
    image: goharbor/nginx-photon:v1.6.0
    container_name: nginx
    restart: always
    volumes:
      - ./common/config/nginx:/etc/nginx:z
    networks:
      - harbor
    ports:
      - 8888:80
      - 1443:443
      - 4443:4443
```

修改`common/templates/registry/config.yml`文件加入`8888`端口：

```
vim common/templates/registry/config.yml

auth:
  token:
    issuer: harbor-token-issuer
    realm: $public_url:8888/service/token
    rootcertbundle: /etc/registry/root.crt
    service: harbor-registry

```

编辑`harbor.cfg`：

```
hostname = 192.168.152.12
harbor_admin_password = admin
```

启动并登陆：

```
sudo su
ufw allow 8888
./prepare
docker-compose up -d
```

打开：http://192.168.152.12:8888，账号|密码:admin

## Docker配置

你可以配置某些仓库地址(第一个是阿里云加速仓库地址，第二个忽略https安全)

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ztndgg1k.mirror.aliyuncs.com"],
  "insecure-registries": ["192.168.0.88:8888"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

然后登录推送:

```
sudo docker login http://192.168.0.88:8888
sudo docker tag mysql:5.7 192.168.0.88:8888/public/mysql:5.7
docker push  192.168.0.88:8888/public/mysql:5.7
```


## Docker特定场景使用

### 离线镜像

如果不能访问外网，那么可以用save和load来保存和加载镜像

```
docker save xxx:1.0 > /root/api1.0.tar
docker  load < /root/api1.0.tar
docker images
```