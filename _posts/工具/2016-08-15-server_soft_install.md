---
layout: post
title: "大数据开发常用软件安装"
date: 2016-08-15
author: silly
categories: ["工具"]
desc: "数据开发常用环境部署,centos系统安装mongodb,redis,mysql,java,kafka,zookeeper,storm,nginx等"
tags: ["大数据","Centos","Java","Storm","Nginx","Mysql","Kafka","Mongodb","Redis","Zookeeper"]
permalink: "/tool/server-sofeware-install.html"
---

更新:

20171207 这是我毕业后第一份工作, 数据开发中需要使用的到的软件安装步骤, 现在可能已经过时了, 请查看不同软件的最新官网文档.

正文:

# 一.安装mongodb

1. 官网下载包[mongodb](https://www.mongodb.com/download-center?jmp=nav)
2. 配置文件config.conf，需绑定内网IP且开启web监控
 
```
	bind_ip=172.16.0.10,127.0.0.1
	rest=true
	dbpath=/data/db/mongodb/27017
	port=27017
	logpath=/data/logs/db/mongodb/mongo.log
	logappend=true
	fork = true
```

3. 运行服务器端

```
[root@clicki-v4 app]# /data/app/mongodb/bin/mongod --config /data/app/mongodb/config.conf 
```

4. 运行客户端

```
[root@clicki-v4 app]# /data/app/mongodb/bin/mongo -port 27017 -host 127.0.0.1
show dbs
```

# 二.安装redis

1. 官网下载安装包[redis](http://redis.io/download)
2. 后台安装

```
    silly@silly-ThinkPad-T420s ~/redis-3.2.0 $ sudo make install
    [sudo] password for silly: 
    cd src && make install
    make[1]: Entering directory `/home/silly/redis-3.2.0/src'
    Success!
    Starting Redis server...
    Installation successful!
```

3. 后台启动

```
    /etc/init.d/redis_6379 start
    silly@silly-ThinkPad-T420s ~/redis-3.2.0 $ redis-cli 
    127.0.0.1:6379> set s "ss"
    ok
    127.0.0.1:6379> get s
    "ss"
    127.0.0.1:6379> 
```

4. 非后台启动

```
    % cd src
    % ./redis-server /path/to/redis.conf
```


5. redis坑

```
#redis.conf
bind 172.16.0.10
#绑定IP
port 6379
#端口
daemonize yes
#后台运行
logfile "/data/logs/db/redis/redis-6379.log"
#日志文件
dir /data/db/redis/6379
#数据保存目录
```

6. 集群

```
src/redis-server cluster/7000.conf
src/redis-server cluster/7001.conf
src/redis-server cluster/7002.conf
src/redis-server cluster/7003.conf
src/redis-server cluster/7004.conf
src/redis-server cluster/7005.conf

7000.conf

port 7000
bind 192.168.28.158
daemonize yes
pidfile ./redis_7000.pid
cluster-enabled yes
cluster-config-file nodes_7000.conf
cluster-node-timeout 15000
appendonly yes


redis-trib.rb  create  --replicas  1 192.168.28.158:7000 192.168.28.158:7001  192.168.28.158:7002 192.168.28.158:7003  192.168.28.158:7004  192.168.28.158:7005
```

# 三.安装MAVEN

首先从官网上 [http://maven.apache.org/](http://maven.apache.org/) 下载最新版Maven。

1. 执行 tar -zxvf apache-maven-x.x.x-bin.tar.gz 命令解压文件
2. 解压后会生成apache-maven-3.0.4目录，删除apache-maven-3.0.4-bin.tar.gz压缩包文件
3. 执行 ln -s apache-maven-x.x.x maven（为Maven做一个软链接，方便以后升级）
4. 执行 vi /etc/profile.d/myenv.sh 文件，插入如下内容

```
export M2_HOME=/data/app/maven
export MAVEN_HOME=/data/app/maven
export PATH=$PATH:$GOBIN:$JAVA_HOME/bin:$M2_HOME/bin
```

5. 保存并退出VI编辑器，执行 source /etc/profile 命令使改动生效
6. 执行 mvn -v 命令，如出现如下内容表示安装配置成功


# 四.安装Mysql

CentOS7的yum源中默认是没有mysql的。为了解决这个问题，我们要先下载mysql的repo源。不直接下载是因为很难找到合适的安装包。

1. 下载mysql的repo源

```
$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

2. 安装mysql-community-release-el7-5.noarch.rpm包

```
$ sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

安装这个包后，会获得两个mysql的yum repo源：

```
/etc/yum.repos.d/mysql-community.repo
/etc/yum.repos.d/mysql-community-source.repo。
```

3. 安装mysql

```
$ sudo yum install mysql-server
```

根据步骤安装就可以了，不过安装完成后，没有密码，需要重置密码。

4. 重置密码

重置密码前，首先要登录

```
$ mysql -u root
```

登录时有可能报这样的错：ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘ (2)，原因是/var/lib/mysql的访问权限问题。下面的命令把/var/lib/mysql的拥有者改为当前用户：

```
$ sudo chown -R openscanner:openscanner /var/lib/mysql
```

然后，重启服务：

```
$ service mysqld restart
```

接下来登录重置密码：

```
$ mysql -u root
mysql > use mysql;
mysql > update user set password=password(‘123456‘) where user=‘root‘;
mysql > exit;
```

5. 开放3306端口

```
$ sudo vim /etc/sysconfig/iptables
```

添加以下内容：

```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
```

保存后重启防火墙：

```
$ sudo service iptables restart
```

这样从其它客户机也可以连接上mysql服务了。

# 五.安装Cassandra

1. 下载安装包[cassandra](http://cassandra.apache.org)

2. 运行

```
[root@clicki-v4 apache-cassandra-2.2.6]# bin/cassandra
[root@clicki-v4 apache-cassandra-2.2.6]# bin/cqlsh 192.168.11.74
cqlsh> 
```

3. 配置

```
# vim conf/cassandra.yaml 

data_file_directories:
     - /data/db/cassandra
commitlog_directory: /data/logs/db/cassandra
saved_caches_directory: /data/db/cassandra/saved_caches
seed_provider:
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          # seeds is actually a comma-delimited list of addresses.
          # Ex: "<ip1>,<ip2>,<ip3>"
          - seeds: "172.16.0.10"
listen_interface: eth0
native_transport_port: 9042
rpc_interface: eth0
```

4. 用法

```
cqlsh:clicki_v4> desc clicki_v4.  //这样很好!!
app_visitor     sdk_visitor     visitor         visitor_reload
cqlsh:clicki_v4> SELECT * from visito
```

5. 资源
[http://www.ibm.com/developerworks/cn/opensource/os-cn-cassandra/](http://www.ibm.com/developerworks/cn/opensource/os-cn-cassandra/)


# 六.安装zookeeper

1. 官网下载包[zookeeper](https://zookeeper.apache.org/releases.html)
2. 配置文件conf/zoo.cfg

```
    # 心跳时间
    tickTime=2000
    # 初始化最长时间，两台机器沟通
    initLimit=10
    # 请求-应答最长时间，数据同步
    syncLimit=5
    # 端口
    clientPort=2181
    #集群，IP:数据交换端口：选举领导端口，需在数据文件夹下新建myid，写入标志1/2/3
	server.1=172.16.5.12:2888:3888
	server.2=172.16.5.12:2889:3889
	server.3=172.16.5.12:2890:3890
	dataDir=/data/dc/zookeeper
	dataLogDir=/data/logs/dc/zookeeper
```

3. 运行

单机模式

```
[root@storm2 zookeeper] $ bin/zkServer.sh start
    ZooKeeper JMX enabled by default
    Using config: /home/silly/zookeeper/1/bin/../conf/zoo.cfg
    Starting zookeeper ... STARTED
[root@storm2 zookeeper] $ bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /home/silly/zookeeper/1/bin/../conf/zoo.cfg
    Mode: standalone
```

集群模式

```
[root@storm2 zookeeper]# bin/zkServer.sh status
	ZooKeeper JMX enabled by default
	Using config: /data/app/zookeeper/bin/../conf/zoo.cfg
	Mode: leader
```

操作

```
创建 testnode节点，关联字符串"zz"         create /zk/testnode "zz"
查看节点内容  get /zk/testnode
设置节点内容  set /zk/testnode abc
删除节点      delete /zk/testnode 
```

```
[root@storm2 zookeeper]# bin/zkCli.sh -server 192.168.11.73:2182
Connecting to 192.168.11.73:2182
[zk: 192.168.11.73:2182(CONNECTED) 0] HELP
```

4. 坑
CentOS虚拟机下，IP设置应使用内网IP,因为该机器不知道自己的外网IP,但是连接时可用外网IP

# 七.安装kafka

1. 官网下载安装包[kafka]()http://kafka.apache.org/downloads.html)
2. 源码安装

```bahs
    silly@silly-ThinkPad-T420s ~ $ cd kafka-0.10.0.0-src/
    silly@silly-ThinkPad-T420s ~/kafka-0.10.0.0-src $ gradle
```

3. 运行

```
#启动
bin/kafka-server-start.sh  config/server.properties

#创建Topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

#列出Topic
bin/kafka-topics.sh --list --zookeeper localhost:2181

#生产
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

#消费
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```

 坑好大，安装 二进制kafka_2.9.1-0.8.2.2

4. 配置文件server.properties

```
port=9092
advertised.host.name=192.168.11.73
log.dirs=/data/dc/kafka1
zookeeper.connect=172.16.5.12:2181,172.16.5.12:2182,172.16.5.12:2183
```

5. 坑
网卡问题：
advertised.host.name=192.168.11.73


# 八.安装JStorm
1. 下载 
[https://github.com/alibaba/jstorm/wiki/%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85](https://github.com/alibaba/jstorm/wiki/%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85)

2. 环境配置

```
#/etc/profile.d/myenv.sh
export JSTORM_HOME=/data/app/jstorm-2.1.1
export PATH=$PATH:$JSTORM_HOME/bin:$JAVA_HOME/bin
```
 
3. 配置文件：

```
 #$JSTORM_HOME/conf/storm.yaml
 storm.zookeeper.servers:
     - "172.16.5.12"
 storm.zookeeper.root: "/jstorm"
```

4. 运行WEBUI

```
#在提交jar的节点上执行:
mkdir ~/.jstorm
cp -f $JSTORM_HOME/conf/storm.yaml ~/.jstorm

#下载tomcat 7.x （以apache-tomcat-7.0.37 为例）
tar -xzf apache-tomcat-7.0.37.tar.gz
cd apache-tomcat-7.0.37
cd webapps
cp $JSTORM_HOME/jstorm-ui-0.9.6.3.war ./
mv ROOT ROOT.old
ln -s jstorm-ui-0.9.6.3 ROOT
          
#这个地方可能变化，是根据你的JStorm版本来确定，比如当0.9.6.1时，是ln -s jstorm-0.9.6.1 ROOT
#另外不是 ln -s jstorm-ui-0.9.6.3.war ROOT 这个要小心

cd ../bin
./startup.sh
```

查看http://192.168.11.73:8080/

5. 启动JStorm

```
nohup jstorm nimbus &
#查看$JSTORM_HOME/logs/nimbus.log检查有无错误
nohup jstorm supervisor &
#查看$JSTORM_HOME/logs/supervisor.log检查有无错误
```

2016/7/14 

