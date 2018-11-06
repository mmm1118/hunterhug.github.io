---
layout: post
title: "Hadoop安装配置"
date: 2016-08-22
author: silly
categories: ["工具"]
desc: "Hadoop安装部署,直接从官网中翻译，亲测成功，版本2.7.2"
tags: ["大数据","hadoop"]
permalink: "/tool/hadoop_install.html"
---

安装的hadoop是2.7.2

# 一．单机模式
默认情况下，hadoop是非分布式模式，作为一个单一的Java进程，容易调试．

1.下载[安装包](http://www.apache.org/dyn/closer.cgi/hadoop/common/)
2.安装必要环境

```
    # java必须安装
    $ sudo apt-get install ssh
    $ sudo apt-get install rsync
```

3.导入环境变量,编辑etc/hadoop/hadoop-env.sh或者`/etc/profile.d/*.sh`

```
    # set to the root of your Java installation
    export JAVA_HOME=/usr/java/jdk1.8.0_92
```

4.查看命令

```
  $ bin/hadoop
```

5.运行mapreduce,将一些xml作为输入数据，运行jar包找出符合正则的语句，jar包源码待解释．

```
  $ mkdir input
  $ cp etc/hadoop/*.xml input
  $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
  $ cat output/*
```

# 二．伪分布模式
在一台机器可以伪装分布式，所有的hadoop守护进程,每一个都运行在单一的Java进程．

守护进程：Daemon

1.配置etc/hadoop/core-site.xml:注意：localhost可以改成主机名，且host文件要改

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

2.配置etc/hadoop/hdfs-site.xml:

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

3.授权公钥

```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 

```

4.格式化文件系统

```
  $ bin/hdfs namenode -format
```

5.启动　Start NameNode daemon and DataNode daemon:

```
    [root@storm2 hadoop]# sbin/start-dfs.sh 
    Starting namenodes on [localhost]
    localhost: starting namenode, logging to /usr/local/hadoop-2.7.2/logs/hadoop-root-namenode-storm2.novalocal.out
```

The hadoop daemon log output is written to the $HADOOP_LOG_DIR directory (defaults to $HADOOP_HOME/logs).

6.浏览器输入http://ip:50070

<img src="/picture/hadoop.jpg"/>

7.在文件系统上建立文件夹Make the HDFS directories required to execute MapReduce jobs:

```
  $ bin/hdfs dfs -mkdir /user
  $ bin/hdfs dfs -mkdir /user/<username>
```

```
[root@storm2 hadoop]# bin/hdfs 
Usage: hdfs [--config confdir] [--loglevel loglevel] COMMAND
       where COMMAND is one of:
  dfs                  run a filesystem command on the file systems supported in Hadoop.
  classpath            prints the classpath
  namenode -format     format the DFS filesystem
  secondarynamenode    run the DFS secondary namenode
  namenode             run the DFS namenode
  journalnode          run the DFS journalnode
  zkfc                 run the ZK Failover Controller daemon
  datanode             run a DFS datanode
  dfsadmin             run a DFS admin client
  haadmin              run a DFS HA admin client
  fsck                 run a DFS filesystem checking utility
  balancer             run a cluster balancing utility
  jmxget               get JMX exported values from NameNode or DataNode.
  mover                run a utility to move block replicas across
                       storage types
  oiv                  apply the offline fsimage viewer to an fsimage
  oiv_legacy           apply the offline fsimage viewer to an legacy fsimage
  oev                  apply the offline edits viewer to an edits file
  fetchdt              fetch a delegation token from the NameNode
  getconf              get config values from configuration
  groups               get the groups which users belong to
  snapshotDiff         diff two snapshots of a directory or diff the
                       current directory contents with a snapshot
  lsSnapshottableDir   list all snapshottable dirs owned by the current user
						Use -help to see options
  portmap              run a portmap service
  nfs3                 run an NFS version 3 gateway
  cacheadmin           configure the HDFS cache
  crypto               configure HDFS encryption zones
  storagepolicies      list/get/set block storage policies
  version              print the version
```

8.复制本机文件到文件系统，Copy the input files into the distributed filesystem:

```
bin/hdfs dfs -put etc/hadoop /user/<username>
```

9.跑Mapreduce，Run some of the examples provided:

```
# bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep /user/silly/hadoop /user/silly/output 'dfs[a-z.]+'

```

10.检查执行后的文内容，Examine the output files: Copy the output files from the distributed filesystem to the local filesystem and examine them:

```
[root@storm2 hadoop]# bin/hdfs dfs -get /user/silly/output output
[root@storm2 hadoop]# ls
bin  include  lib      LICENSE.txt  NOTICE.txt  README.txt  share
etc  input    libexec  logs         output      sbin
[root@storm2 hadoop]# ll output/
总用量 4
-rw-r--r--. 1 root root 197 8月  23 03:48 part-r-00000
-rw-r--r--. 1 root root   0 8月  23 03:48 _SUCCESS
[root@storm2 hadoop]# cat output/part-r-00000 
6	dfs.audit.logger
4	dfs.class
3	dfs.server.namenode.
2	dfs.period
2	dfs.audit.log.maxfilesize
2	dfs.audit.log.maxbackupindex
1	dfsmetrics.log
1	dfsadmin
1	dfs.servers
1	dfs.replication
1	dfs.file
```

或者

```
[root@storm2 hadoop]# bin/hdfs dfs -cat /user/silly/output/*
```

11.停止

```
[root@storm2 hadoop]# sbin/stop-dfs.sh 
Stopping namenodes on [localhost]
localhost: stopping namenode
localhost: stopping datanode
Stopping secondary namenodes [0.0.0.0]
0.0.0.0: stopping secondarynamenode

```

# 三．伪分布使用YARN(Yet Another Resource Negotiator，另一种资源协调者)
You can run a MapReduce job on YARN in a pseudo-distributed mode by setting a few parameters and running ResourceManager daemon and NodeManager daemon in addition.
再加一点参数和开两个守护进程，资源管理器和节点管理器，就可以用YARN来跑任务了．

1.接着上面伪分布式还需要配置etc/hadoop/mapred-site.xml:

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

2.编辑etc/hadoop/yarn-site.xml:

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

3.启动两个守护进程

```
  $ sbin/start-yarn.sh
```

4.打开资源管理器ResourceManager - http://localhost:8088/

<img src="/picture/hadoop1.jpg"/>

5.可以跑任务了

6.关闭

```
$ sbin/stop-yarn.sh
```
