---
layout: post
title: "Docker-compose:多个Docker容器管理"
date: 2017-07-07
author: silly
categories: ["工具"]
desc: "Docker-compose 多个Docker容器管理:以MYSQL和Wordpress为例"
tags: ["docker"]
permalink: "/tool/docker-compose.html"
--- 

最新的应用, 可以参考: [https://github.com/hunterhug/GoSpider-docker](https://github.com/hunterhug/GoSpider-docker)

Docker-compose 多个Docker容器管理:以MYSQL和Wordpress为例

环境：Ubuntu

```
jinhan@jinhan-chen-110:~$ uname -a
Linux jinhan-chen-110 4.4.0-83-generic #106-Ubuntu SMP Mon Jun 26 17:54:43 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
jinhan@jinhan-chen-110:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.2 LTS
Release:	16.04
Codename:	xenial
```

先安装好docker:参考：[Ubuntu docker安装全套](https://www.lenggirl.com/tool/docker-ubuntu-install.html)

# 一. 安装

```
curl -L https://github.com/docker/compose/releases/download/1.8.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

# 二. 拉镜像

```
docker pull mysql
docker pull wordpress
```

# 三. 编辑docker-compose.yaml

```
version: '2'
services:
    web: 
      image: wordpress:latest
      depends_on: 
        - db
      links: 
        - db
      ports: 
        - "8002:80"
      environment:
        WORDPRESS_DB_HOST: db:3306
        WORDPRESS_DB_PASSWORD: 123456
      volumes:
        - /home/jinhan/wordpress:/var/www/html

    db: 
      image: mysql
      ports: 
        - "8003:3306"
      environment: 
        - MYSQL_ROOT_PASSWORD=123456
      volumes:
        - /home/jinhan/mysql/data:/var/lib/mysql
        - /home/jinhan/mysql/conf:/etc/mysql/conf.d
```

说明：

```
version: '2'
services:
    web:    // 服务名
      image: wordpress:latest  // 镜像名
      depends_on:   // 依赖的服务名，即是必须在这个服务启动后再启动
        - db
      links:   // 链接到的服务，即是对于这个服务洪的容器，网络是透明的，可以直接使用其内部端口访问
        - db
      ports:  // 端口主机映射，在外面可以用8002访问到网站
        - "8002:80"
      environment: // 环境变量
        WORDPRESS_DB_HOST: db:3306  // 数据库地址，服务名和端口，因为上面已经links，所以自动会寻址
        WORDPRESS_DB_PASSWORD: 123456 // 数据库密码
      volumes:
        - /home/jinhan/wordpress:/var/www/html  // 挂载卷，拉镜像会把wordpress下载在这里，我们把它挂载在本地，这样我们修改本地文件即可

    db: 
      image: mysql
      ports: 
        - "8003:3306"
      environment: 
        - MYSQL_ROOT_PASSWORD=123456
      volumes:
        - /home/jinhan/mysql/data:/var/lib/mysql  // 数据库数据，挂在本地
        - /home/jinhan/mysql/conf:/etc/mysql/conf.d // 数据库配置，我们要自己放
```

# 四. 挂载卷

我们用MYSQL和Wordpress来做试验

先配置mysql config，加大连接数，因为连接数可能会爆:

```
mkdir -p /home/jinhan/mysql/conf
vim /home/jinhan/mysql/conf/my.cnf
```

```
[mysqld]
max_connections = 15000
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 1000
skip-name-resolve
```

我们已经把所有的卷都挂在本地，这样你每次启动的数据都不会消失！

# 五. 启动
```
docker-compose up
```

浏览器访问：http://127.0.0.1:8002/
数据库访问：root:123456 端口8003

如果
```
show variables like '%max_connect%';
```

显示15000,配置成功！

# 六. 查看
```
docker-compose ps
```

# 七. 后台启动
```
docker-compose up -d
```

# 八. 查看日志
```
docker-compose logs -f
```

# 九. 删除容器
```
docker-compose rm
```

# 十. 寻求帮助
```
docker-compose -h
```

