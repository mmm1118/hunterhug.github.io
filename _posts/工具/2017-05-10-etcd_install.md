---
layout: post
title: "Etcd安装和使用"
date: 2017-05-10
author: silly
desc: "meven安装"
categories: ["工具"]
tags: ["Etcd"]
permalink: "/tool/etcd-install.html"
--- 

# 一.安装

# 1.1 二进制安装

从这里下载: [etcd-v3.2.11-linux-amd64.tar.gz](https://github.com/coreos/etcd/releases/)

下载包后解压即可运行：

```
# 解压
tar zxvf etcd-v3.2.11-linux-amd64.tar.gz 
cd etcd-v3.2.11-linux-amd64

# ETCD版本
etcd --version

# 客户端接口版本
etcdctl --version

# API3的要这样
ETCDCTL_API=3 etcdctl version

# 启动也很简单
./etcd

# 试试
ETCDCTL_API=3 etcdctl --endpoints=localhost:2379 put foo bar
ETCDCTL_API=3 etcdctl --endpoints=localhost:2379 get foo
```

# 1.2 源码安装

安装好Golang环境: 见[Golang环境配置](/tool/golang-config.html)

```
go get -u -v https://github.com/coreos/etcd
./build
```

启动:

```
./etcd
```

# 1.3 docker安装

拉镜像:

```
docker pull quay.io/coreos/etcd
```

启动:

```
docker run -it --rm -p 2379:2379 -p 2380:2380 --name etcd quay.io/coreos/etcd
```

查询:

```
docker exec -it etcd etcdctl member list
```

# 二.启动详细说明

## 2.1单机启动

```
./etcd --name my-etcd-1  --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://0.0.0.0:2379 --listen-peer-urls http://0.0.0.0:2380 --initial-advertise-peer-urls http://0.0.0.0:2380  --initial-cluster my-etcd-1=http://0.0.0.0:2380
```

## 2.2集群启动

我们用三个端口2380,2381,2382来模拟集群(这三个是成员之间通信),2379,2389,2399是给客户端连接的.公网IP是: `172.16.13.90`, 如果在本机模拟集群, 可以将`172.16.13.90`改为`0.0.0.0`

带advertise参数是广播参数: 如`--listen-client-urls`和`--advertise-client-urls`, 前者是Etcd端监听客户端的url,后者是Etcd客户端请求的url, 两者端口是相同的, 只不过后者一般为公网IP, 暴露给外部使用.

```
./etcd --name my-etcd-1  --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://172.16.13.90:2379 --listen-peer-urls http://0.0.0.0:2380 --initial-advertise-peer-urls http://172.16.13.90:2380  --initial-cluster-token etcd-cluster-test --initial-cluster-state new --initial-cluster my-etcd-1=http://172.16.13.90:2380,my-etcd-2=http://172.16.13.90:2381,my-etcd-3=http://172.16.13.90:2382

./etcd --name my-etcd-2  --listen-client-urls http://0.0.0.0:2389 --advertise-client-urls http://172.16.13.90:2389 --listen-peer-urls http://0.0.0.0:2381 --initial-advertise-peer-urls http://172.16.13.90:2381  --initial-cluster-token etcd-cluster-test --initial-cluster-state new --initial-cluster my-etcd-1=http://172.16.13.90:2380,my-etcd-2=http://172.16.13.90:2381,my-etcd-3=http://172.16.13.90:2382

./etcd --name my-etcd-3  --listen-client-urls http://0.0.0.0:2399 --advertise-client-urls http://172.16.13.90:2399 --listen-peer-urls http://0.0.0.0:2382 --initial-advertise-peer-urls http://172.16.13.90:2382  --initial-cluster-token etcd-cluster-test --initial-cluster-state new --initial-cluster my-etcd-1=http://172.16.13.90:2380,my-etcd-2=http://172.16.13.90:2381,my-etcd-3=http://172.16.13.90:2382
```

查看成员:

```
etcdctl member list
```

使用时需要指定endpoints(默认本地端口2379), 集群时数据会迅速同步:

```
ETCDCTL_API=3 etcdctl --endpoints=127.0.0.1:2389 put foo xx
ETCDCTL_API=3 etcdctl --endpoints=127.0.0.1:2379 get foo
```

# 2.3参数说明

|参数|使用说明|
|----|----|
|--name etcd0|本member的名字|	 
|--initial-advertise-peer-urls http://192.168.2.55:2380|其他member使用，其他member通过该地址与本member交互信息。一定要保证从其他member能可访问该地址。静态配置方式下，该参数的value一定要同时在--initial-cluster参数中存在。memberID的生成受--initial-cluster-token和--initial-advertise-peer-urls影响。|
|--listen-peer-urls  http://0.0.0.0:2380|本member侧使用，用于监听其他member发送信息的地址。ip为全0代表监听本member侧所有接口|
|--listen-client-urls http://0.0.0.0:2379|本member侧使用，用于监听etcd客户发送信息的地址。ip为全0代表监听本member侧所有接口|	 
|--advertise-client-urls http://192.168.2.55:2379|etcd客户使用，客户通过该地址与本member交互信息。一定要保证从客户侧能可访问该地址	|
|--initial-cluster-token etcd-cluster-2	|用于区分不同集群。本地如有多个集群要设为不同。| 
|--initial-cluster etcd0=http://192.168.2.55:2380,etcd1=http://192.168.2.54:2380,etcd2=http://192.168.2.56:2380|本member侧使用。描述集群中所有节点的信息，本member根据此信息去联系其他member。memberID的生成受--initial-cluster-token和--initial-advertise-peer-urls影响。|
|--initial-cluster-state new|用于指示本次是否为新建集群。有两个取值new和existing。如果填为existing，则该member启动时会尝试与其他member交互。集群初次建立时，要填为new，经尝试最后一个节点填existing也正常，其他节点不能填为existing。集群运行过程中，一个member故障后恢复时填为existing，经尝试填为new也正常。|
|-data-dir|指定节点的数据存储目录，这些数据包括节点ID，集群ID，集群初始化配置，Snapshot文件，若未指定-wal-dir，还会存储WAL文件；如果不指定会用缺省目录。|
|-discovery http://192.168.1.163:20003/v2/keys/discovery/78b12ad7-2c1d-40db-9416-3727baf686cb|用于自发现模式下，指定第三方etcd上key地址，要建立的集群各member都会向其注册自己的地址。|

# 三.使用详细说明

ETCD API有两种, 一种是3, 一种是2, 默认为2, 我们主要用3:

API3:

```
jinhan@jinhan-chen-110:~/code/src/github.com/coreos/etcd/bin$ ETCDCTL_API=3 etcdctl put mykey "this is awesome"
OK
jinhan@jinhan-chen-110:~/code/src/github.com/coreos/etcd/bin$ ETCDCTL_API=3 etcdctl get mykey
mykey
this is awesome
```

API2是这样的(可以不加前缀)

```
jinhan@jinhan-chen-110:~/code/src/github.com/coreos/etcd/bin$ ETCDCTL_API=2 etcdctl set /local/dd d
d
jinhan@jinhan-chen-110:~/code/src/github.com/coreos/etcd/bin$ ETCDCTL_API=2 etcdctl get /local/dd
d
jinhan@jinhan-chen-110:~/code/src/github.com/coreos/etcd/bin$ ETCDCTL_API=2 etcdctl set /local/dd d
d
jinhan@jinhan-chen-110:~/code/src/github.com/coreos/etcd/bin$ ETCDCTL_API=2 etcdctl get /local/dd
d
```

命令详解:

API2:

```
jinhan@jinhan-chen-110:~/code/src/github.com/coreos/etcd/bin$ ETCDCTL_API=2 etcdctl 
NAME:
   etcdctl - A simple command line client for etcd.

USAGE:
   etcdctl [global options] command [command options] [arguments...]
   
VERSION:
   3.1.0-rc.1+git
   
COMMANDS:
     backup          backup an etcd directory
     cluster-health  check the health of the etcd cluster
     mk              make a new key with a given value
     mkdir           make a new directory
     rm              remove a key or a directory
     rmdir           removes the key if it is an empty directory or a key-value pair
     get             retrieve the value of a key
     ls              retrieve a directory
     set             set the value of a key
     setdir          create a new directory or update an existing directory TTL
     update          update an existing key with a given value
     updatedir       update an existing directory
     watch           watch a key for changes
     exec-watch      watch a key for changes and exec an executable
     member          member add, remove and list subcommands
     user            user add, grant and revoke subcommands
     role            role add, grant and revoke subcommands
     auth            overall auth controls

GLOBAL OPTIONS:
   --debug                          output cURL commands which can be used to reproduce the request
   --no-sync                        don't synchronize cluster information before sending request
   --output simple, -o simple       output response in the given format (simple, `extended` or `json`) (default: "simple")
   --discovery-srv value, -D value  domain name to query for SRV records describing cluster endpoints
   --insecure-discovery             accept insecure SRV records describing cluster endpoints
   --peers value, -C value          DEPRECATED - "--endpoints" should be used instead
   --endpoint value                 DEPRECATED - "--endpoints" should be used instead
   --endpoints value                a comma-delimited list of machine addresses in the cluster (default: "http://127.0.0.1:2379,http://127.0.0.1:4001")
   --cert-file value                identify HTTPS client using this SSL certificate file
   --key-file value                 identify HTTPS client using this SSL key file
   --ca-file value                  verify certificates of HTTPS-enabled servers using this CA bundle
   --username value, -u value       provide username[:password] and prompt if password is not supplied.
   --timeout value                  connection timeout per request (default: 2s)
   --total-timeout value            timeout for the command execution (except watch) (default: 5s)
   --help, -h                       show help
   --version, -v                    print the version
```

API3:

```  
jinhan@jinhan-chen-110:~/code/src/github.com/coreos/etcd/bin$ ETCDCTL_API=3 etcdctl 
NAME:
	etcdctl - A simple command line client for etcd3.

USAGE:
	etcdctl

VERSION:
	3.1.0-rc.1+git

API VERSION:
	3.1


COMMANDS:
	get			Gets the key or a range of keys
	put			Puts the given key into the store
	del			Removes the specified key or range of keys [key, range_end)
	txn			Txn processes all the requests in one transaction
	compaction		Compacts the event history in etcd
	alarm disarm		Disarms all alarms
	alarm list		Lists all alarms
	defrag			Defragments the storage of the etcd members with given endpoints
	endpoint health		Checks the healthiness of endpoints specified in `--endpoints` flag
	endpoint status		Prints out the status of endpoints specified in `--endpoints` flag
	watch			Watches events stream on keys or prefixes
	version			Prints the version of etcdctl
	lease grant		Creates leases
	lease revoke		Revokes leases
	lease timetolive	Get lease information
	lease keep-alive	Keeps leases alive (renew)
	member add		Adds a member into the cluster
	member remove		Removes a member from the cluster
	member update		Updates a member in the cluster
	member list		Lists all members in the cluster
	snapshot save		Stores an etcd node backend snapshot to a given file
	snapshot restore	Restores an etcd member snapshot to an etcd directory
	snapshot status		Gets backend snapshot status of a given file
	make-mirror		Makes a mirror at the destination etcd cluster
	migrate			Migrates keys in a v2 store to a mvcc store
	lock			Acquires a named lock
	elect			Observes and participates in leader election
	auth enable		Enables authentication
	auth disable		Disables authentication
	user add		Adds a new user
	user delete		Deletes a user
	user get		Gets detailed information of a user
	user list		Lists all users
	user passwd		Changes password of user
	user grant-role		Grants a role to a user
	user revoke-role	Revokes a role from a user
	role add		Adds a new role
	role delete		Deletes a role
	role get		Gets detailed information of a role
	role list		Lists all roles
	role grant-permission	Grants a key to a role
	role revoke-permission	Revokes a key from a role
	help			Help about any command

OPTIONS:
      --cacert=""				verify certificates of TLS-enabled secure servers using this CA bundle
      --cert=""					identify secure client using this TLS certificate file
      --command-timeout=5s			timeout for short running command (excluding dial timeout)
      --dial-timeout=2s				dial timeout for client connections
      --endpoints=[127.0.0.1:2379]		gRPC endpoints
  -h, --help[=false]				help for etcdctl
      --hex[=false]				print byte strings as hex encoded strings
      --insecure-skip-tls-verify[=false]	skip server certificate verification
      --insecure-transport[=true]		disable transport security for client connections
      --key=""					identify secure client using this TLS key file
      --user=""					username[:password] for authentication (prompt if password is not supplied)
  -w, --write-out="simple"			set the output format (json, proto, simple, table)

```
