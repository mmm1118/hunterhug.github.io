---
layout: post
title: "What and How Consul?"
date: 2017-11-23
author: silly
desc: "Just introduce!"
categories: ["工具"]
tags: ["consul"]
permalink: "/tool/consul.html"
---

I move this page to my blog: [https://www.consul.io/intro/index.html](https://www.consul.io/intro/index.html)

What is `consul`:  A consul is an official who is sent by his or her government to live in a foreign city in order to look after all the people there that belong to his or her own country.

# Introduction to Consul

Welcome to the intro guide to Consul! This guide is the best place to start with Consul. We cover what Consul is, what problems it can solve, how it compares to existing software, and how you can get started using it. If you are familiar with the basics of Consul, the documentation provides a more detailed reference of available features.

## What is Consul?

Consul has multiple components, but as a whole, it is a tool for discovering and configuring services in your infrastructure. It provides several key features:

Service Discovery: Clients of Consul can provide a service, such as api or mysql, and other clients can use Consul to discover providers of a given service. Using either DNS or HTTP, applications can easily find the services they depend upon.

Health Checking: Consul clients can provide any number of health checks, either associated with a given service ("is the webserver returning 200 OK"), or with the local node ("is memory utilization below 90%"). This information can be used by an operator to monitor cluster health, and it is used by the service discovery components to route traffic away from unhealthy hosts.

KV Store: Applications can make use of Consul's hierarchical key/value store for any number of purposes, including dynamic configuration, feature flagging, coordination, leader election, and more. The simple HTTP API makes it easy to use.

Multi Datacenter: Consul supports multiple datacenters out of the box. This means users of Consul do not have to worry about building additional layers of abstraction to grow to multiple regions.

Consul is designed to be friendly to both the DevOps community and application developers, making it perfect for modern, elastic infrastructures.

## Basic Architecture of Consul

Consul is a distributed, highly available system. This section will cover the basics, purposely omitting some unnecessary detail, so you can get a quick understanding of how Consul works. For more detail, please refer to the in-depth architecture overview.

Every node that provides services to Consul runs a Consul agent. Running an agent is not required for discovering other services or getting/setting key/value data. The agent is responsible for health checking the services on the node as well as the node itself.

The agents talk to one or more Consul servers. The Consul servers are where data is stored and replicated. The servers themselves elect a leader. While Consul can function with one server, 3 to 5 is recommended to avoid failure scenarios leading to data loss. A cluster of Consul servers is recommended for each datacenter.

Components of your infrastructure that need to discover other services or nodes can query any of the Consul servers or any of the Consul agents. The agents forward queries to the servers automatically.

Each datacenter runs a cluster of Consul servers. When a cross-datacenter service discovery or configuration request is made, the local Consul servers forward the request to the remote datacenter and return the result.

# Consul vs. Other Software

See how Consul compares to other software to assess how it fits into your existing infrastructure.

The problems Consul solves are varied, but each individual feature has been solved by many different systems. Although there is no single system that provides all the features of Consul, there are other options available to solve some of these problems.

In this section, we compare Consul to some other options. In most cases, Consul is not mutually exclusive with any other system.

See: [https://www.consul.io/intro/vs/zookeeper.html](https://www.consul.io/intro/vs/zookeeper.html)

# Getting Started

Continue onwards with the getting started guide to get Consul up and running.

## Install

Go to [https://www.consul.io/downloads.html](https://www.consul.io/downloads.html) download the binary package. Then:

```
# In Linux Ubuntu

tar -zxvf consul_1.0.1_linux_amd64.zip
sudo mv consul /usr/local/bin/

$ consul
Usage: consul [--version] [--help] <command> [<args>]

```

## Run the Consul Agent

After Consul is installed, the agent must be run. The agent can run either in server or client mode. Each datacenter must have at least one server, though a cluster of 3 or 5 servers is recommended. A single server deployment is highly discouraged as data loss is inevitable in a failure scenario.

All other agents run in client mode. A client is a very lightweight process that registers services, runs health checks, and forwards queries to servers. The agent must be running on every node that is part of the cluster.

For simplicity, we'll start the Consul agent in development mode for now. This mode is useful for bringing up a single-node Consul environment quickly and easily. It is not intended to be used in production as it does not persist any state.

```
-$ consul agent -dev
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
           Version: 'v0.7.0'
         Node name: 'Armons-MacBook-Air'
        Datacenter: 'dc1'
            Server: true (bootstrap: false)
       Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2016/09/15 10:21:10 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:127.0.0.1:8300 Address:127.0.0.1:8300}]
    2016/09/15 10:21:10 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
    2016/09/15 10:21:10 [INFO] serf: EventMemberJoin: Armons-MacBook-Air 127.0.0.1
    2016/09/15 10:21:10 [INFO] serf: EventMemberJoin: Armons-MacBook-Air.dc1 127.0.0.1
    2016/09/15 10:21:10 [INFO] consul: Adding LAN server Armons-MacBook-Air (Addr: tcp/127.0.0.1:8300) (DC: dc1)
    2016/09/15 10:21:10 [INFO] consul: Adding WAN server Armons-MacBook-Air.dc1 (Addr: tcp/127.0.0.1:8300) (DC: dc1)
    2016/09/15 10:21:13 [DEBUG] http: Request GET /v1/agent/services (180.708µs) from=127.0.0.1:52369
    2016/09/15 10:21:13 [DEBUG] http: Request GET /v1/agent/services (15.548µs) from=127.0.0.1:52369
    2016/09/15 10:21:17 [WARN] raft: Heartbeat timeout from "" reached, starting election
    2016/09/15 10:21:17 [INFO] raft: Node at 127.0.0.1:8300 [Candidate] entering Candidate state in term 2
    2016/09/15 10:21:17 [DEBUG] raft: Votes needed: 1
    2016/09/15 10:21:17 [DEBUG] raft: Vote granted from 127.0.0.1:8300 in term 2. Tally: 1
    2016/09/15 10:21:17 [INFO] raft: Election won. Tally: 1
    2016/09/15 10:21:17 [INFO] raft: Node at 127.0.0.1:8300 [Leader] entering Leader state
    2016/09/15 10:21:17 [INFO] consul: cluster leadership acquired
    2016/09/15 10:21:17 [DEBUG] consul: reset tombstone GC to index 3
    2016/09/15 10:21:17 [INFO] consul: New leader elected: Armons-MacBook-Air
    2016/09/15 10:21:17 [INFO] consul: member 'Armons-MacBook-Air' joined, marking health alive
    2016/09/15 10:21:17 [INFO] agent: Synced service 'consul'
```

As you can see, the Consul agent has started and has output some log data. From the log data, you can see that our agent is running in server mode and has claimed leadership of the cluster. Additionally, the local member has been marked as a healthy member of the cluster.

## Cluster Members

If you run consul members in another terminal, you can see the members of the Consul cluster. We'll cover joining clusters in the next section, but for now, you should only see one member (yourself):

```
$ consul members
Node                Address            Status  Type    Build     Protocol  DC
Armons-MacBook-Air  172.20.20.11:8301  alive   server  0.6.1dev  2         dc1
```

In My PC, It look like:

```
jinhan@jinhan-chen-110:~$ consul members
Node             Address         Status  Type    Build  Protocol  DC   Segment
jinhan-chen-110  127.0.0.1:8301  alive   server  1.0.1  2         dc1  <all>
```

The output shows our own node, the address it is running on, its health state, its role in the cluster, and some version information. Additional metadata can be viewed by providing the -detailed flag.

The output of the members command is based on the gossip protocol and is eventually consistent. That is, at any point in time, the view of the world as seen by your local agent may not exactly match the state on the servers. For a strongly consistent view of the world, use the HTTP API as it forwards the request to the Consul servers:

you can `curl localhost:8500/v1/agent/members` see more.


In addition to the HTTP API, the DNS interface can be used to query the node. Note that you have to make sure to point your DNS lookups to the Consul agent's DNS server which runs on port 8600 by default. The format of the DNS entries (such as "Armons-MacBook-Air.node.consul") will be covered in more detail later.

HTTP API
```
curl localhost:8500/v1/catalog/nodes
```

DNS
```
dig @127.0.0.1 -p 8600 Armons-MacBook-Air.node.consul
...

;; QUESTION SECTION:
;Armons-MacBook-Air.node.consul.    IN  A

;; ANSWER SECTION:
Armons-MacBook-Air.node.consul. 0 IN    A   127.0.0.1
```
You can use Ctrl-C (the interrupt signal) to gracefully halt the agent. After interrupting the agent, you should see it leave the cluster and shut down.

By gracefully leaving, Consul notifies other cluster members that the node left. If you had forcibly killed the agent process, other members of the cluster would have detected that the node failed. When a member leaves, its services and checks are removed from the catalog. When a member fails, its health is simply marked as critical, but it is not removed from the catalog. Consul will automatically try to reconnect to failed nodes, allowing it to recover from certain network conditions, while left nodes are no longer contacted.

Additionally, if an agent is operating as a server, a graceful leave is important to avoid causing a potential availability outage affecting the consensus protocol. See the [guides](https://www.consul.io/docs/guides/index.html) section for details on how to safely add and remove servers.

More wait...