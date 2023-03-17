---
title: Redis集群搭建
subtitle: 基于docker搭建redis集群
date: 2023-03-17T10:12:29+08:00
draft: false
tags: [DoItYourself]
categories: [Skills]
url: /posts/skills/Redis/
---

## 一、Redis集群
### 集群的三种模式
* 主从模式：
将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)，数据的复制是单向的，只能由主节点到从节点。默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

* 哨兵模式（Redis-Sentinel）：
基于主从模式，实现了监控 Redis 系统的运行状态，通过投票机制，从 slave 中选举出新的 master 以保证集群正常运行。

* Redis Cluster：
redis的哨兵模式基本已经可以实现高可用、读写分离，但是在这种模式每台redis服务器都存储相同的数据，很浪费内存资源,所以在redis3.0上加入了Cluster集群模式，实现了redis的分布式存储，也就是说每台redis节点存储着不同的内容根据官方推荐，集群部署至少要3台以上的master节点，最好使用3主3从六个节点的模式。
Cluster 集群由多个redis服务器组成的分布式网络服务集群，集群之中有多个master主节点，每一个主节点都可读可写，节点之间会相互通信，两两相连，redis集群无中心节点。

## 二、集群的搭建
> 本文基于Cluster模式使用docker容器搭建集群，所有容器均在一台服务器上，仅做练手使用，无法达到高可用的目的。
### 1. 获取镜像
```bash
docker pull redis 
```
### 2. 构建节点容器
这里构建6个节点，达到搭建集群最低3个master的基本要求。
```bash
docker run -d --name redis-node-1 --net host --privileged=true -v ~/Docker/redis/share/redis-node-1:/data redis:latest --cluster-enabled yes --appendonly yes --port 6381
docker run -d --name redis-node-2 --net host --privileged=true -v ~/Docker/redis/share/redis-node-2:/data redis:latest --cluster-enabled yes --appendonly yes --port 6382
docker run -d --name redis-node-3 --net host --privileged=true -v ~/Docker/redis/share/redis-node-3:/data redis:latest --cluster-enabled yes --appendonly yes --port 6383
docker run -d --name redis-node-4 --net host --privileged=true -v ~/Docker/redis/share/redis-node-4:/data redis:latest --cluster-enabled yes --appendonly yes --port 6384
docker run -d --name redis-node-5 --net host --privileged=true -v ~/Docker/redis/share/redis-node-5:/data redis:latest --cluster-enabled yes --appendonly yes --port 6385
docker run -d --name redis-node-6 --net host --privileged=true -v ~/Docker/redis/share/redis-node-6:/data redis:latest --cluster-enabled yes --appendonly yes --port 6386 
```
命令分步解释：
|命令|说明|
|:--:|:--:|
|docker run|创建并运行docker容器实例|
|--name redis-node-1|容器命名|
|--net host|容器使用宿主机的ip和端口|
|--privileged=true|获取宿主机root权限|

