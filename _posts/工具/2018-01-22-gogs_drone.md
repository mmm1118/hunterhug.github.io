---
layout: post
title: "自建Git服务Gogs和CI/CD服务Drone"
date: 2018-01-17
author: silly
categories: ["工具"]
desc: "建立自己公司内部的代码仓库服务, 类似Github, 并支持Drone持续集成和持续部署"
tags: ["git","drone"]
permalink: "/tool/gogs-drone.html"
---

为了让开发和运维更方便, 可以使用很多工具来简化工作. 

我的需求是: 为了测试`Drone`对`docker部署`方式的`CD/CI`支持, 所以建了一个git服务, 类似于Github, 然后互相绑钩子, 使得每次写好代码, 推上仓库后自动构建, 并且自动部署.

# 一.搭建Gogs服务

我的操作系统是`64位ubuntu16.04`, 我这里只介绍二进制部署

请先安装好`Mysql`和`Nginx`和`Git`:

```sh
# 安装必要软件
apt update
apt install git
apt install mysql-server
apt install nginx

# 建一个数据库
mysql -uroot -p
mysql> DROP DATABASE IF EXISTS gogs;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> CREATE DATABASE IF NOT EXISTS gogs CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.00 sec)

mysql>

```

然后从[Gogs项目地址](https://github.com/gogits/gogs)下载二进制.

我们以root权限先建一个新的Linux用户:

```sh
# 加一个用户
useradd -c "git service" -m git
passwd git

# 加入sudo列表
chmod u+w /etc/sudoers
vim /etc/sudoers

# 增加以下内容
git ALL=(ALL:ALL) ALL
```

## 二进制基本安装

您可参考[官方](https://gogs.io/docs/installation/install_from_binary), 直接下载二进制:

```sh
# 切换用户
su git
cd

# 下载二进制
wget https://github.com/gogits/gogs/releases/download/v0.11.34/linux_amd64.tar.gz
tar -zxvf linux_amd64.tar.gz

# 跑程序
cd gogs
./gogs web
```

请打开`127.0.0.1:3000`, 我们可以在真实服务器用Nginx反向代理到一个域名中!

配置中:

1. `运行系统用户`:改为git(该用户必须具有对仓库根目录和运行 Gogs 的操作权限)
2. `数据库名称`: 请执行上述`CREATE DATABASE IF NOT EXISTS gogs CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;`
3. `内置SSH服务器`必须勾选!!!且将端口:22改成2222, 必须, 不然和你登录的22端口可能冲突
4. `管理员帐号设置`务必要填!!
5. 请将`localhost`全部换为内网IP(172.16.13.90), 可使用`ip addr`获取自己机器的ip

如果忘了设置管理员, 可以:

```
./gogs admin create-user --name xxxx --password 123123 --email gdccmcm@xx.com
```

## 配置使用

创建成功后, 我们可以登录, 然后创建一个仓库, 根据提示:

```sh
touch README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin http://172.16.13.90:3000/xxxx/test.git
git push -u origin master
```

我们发现成功推了仓库.

`SSH`方式也是可以的(我们`ssh-keygen`,然后将公钥复制进授权里):

```sh
touch README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin ssh://git@172.16.13.90:2222/xxxx/test.git
git push -u origin master
```

重新配置可以如下:

我们停掉`gogs`, 然后在同级文件夹编辑`custom/conf/app.ini`文件, 默认内容请见[这里](https://github.com/gogits/gogs/blob/master/conf/app.ini), 解释请见(这里)[https://gogs.io/docs/advanced/configuration_cheat_sheet]:

```ini
vim custom/conf/app.ini

; APP名字
APP_NAME = MYGIT
; 运行的用户
RUN_USER = git

[repository]
; 仓库保存的地址
ROOT = /home/git/gogs-repositories

[database]
; MYSQL数据库配置
DB_TYPE = mysql
HOST = 127.0.0.1:3306
NAME = gogs
USER = root
PASSWD = 123123

[server]
DOMAIN           = 172.16.13.90
HTTP_PORT        = 3000
ROOT_URL         = http://172.16.13.90:3000/
DISABLE_SSH      = false
SSH_PORT         = 2222
START_SSH_SERVER = true
OFFLINE_MODE     = false
```

## 守护进程使用

我们以更方便的方式启动:

```sh
ls scripts/systemd/gogs.service -al

# 加入系统
sudo cp scripts/systemd/gogs.service /etc/systemd/system/

# 在任何地方查看状态
sudo service gogs status

# 启动
sudo service gogs start

# 停止
sudo service gogs stop
```

# 二.搭建Drone

编辑`docker-compose.yaml`, 只需更改`http://172.16.13.90:3000`:

```
version: '2'
services:
  # 服务器主端
  drone-server:
    container_name: "mydrone-server"
    image: drone/drone:latest
    # 端口开放8080
    ports:
      - 8080:8000
      - 9000
    volumes:
      - /var/lib/drone:/var/lib/drone/
    restart: always
    environment:
      - DRONE_OPEN=true
      - DRONE_HOST=http://127.0.0.1
      # Gogs必备
      - DRONE_GOGS=true
      - DRONE_GOGS_URL=http://172.16.13.90:3000
      - DRONE_SECRET=xxxigogo

  # 从端
  drone-agent:
    container_name: "mydrone-agent"
    image: drone/agent:latest

    command: agent
    restart: always
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_SERVER=drone-server:9000
      - DRONE_SECRET=xxxigogo
```

启动:

```
docker-compose up -d
```

我们开放了8080端口, 打开`127.0.0.1:8080`登录使用gogs帐号!

# 三.基础实践

我们先登录Gogs:`http://172.16.13.90:3000`, 建立一个仓库`testmydrone`.


先提交:

```
touch README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin ssh://git@172.16.13.90:2222/xxxx/testmydrone.git
git push -u origin master
```

然后登录`http://127.0.0.1:8080`勾选项目!

![](/picture/public/drone1.png)

勾选完之后, 我们在我们的仓库新建一个文件`.drone.yml`

```
pipeline:
  backend:
    image: golang
    commands:
      - go build
```

还有一个源码`main.go`:

```go
package main

func main(){
}
```

提交:

```sh
git add --all
git commit -m "push"
git push
```

![](/picture/public/drone2.png)

成功!!!

下面部分为`drone`的具体使用


# 四.具体使用

待写