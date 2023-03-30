---
title: Redis集群搭建
subtitle: 基于docker搭建redis集群
date: 2023-03-17T10:12:29+08:00
draft: false
tags: [DoItYourself]
categories: [Skills]
url: /posts/skills/redis/
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
|-v ~/Docker/redis/share/redis-node-1:/data|挂载宿主机目录到docker内部|
|redis:latest|redis镜像和版本号|
|--cluster-enabled yes|开启redis集群|
|--appendonly yes|开启持久化|
|--port 6381|redis端口号|
### 3. 构建集群关系
* 进入其中一个容器
```bash
docker exec -it redis-node-1 bash
```
* 构建关系
```bash
redis-cli --cluster create 127.0.0.1:6381 127.0.0.1:6382 127.0.0。1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 --cluster-replicas 1
```
> --cluster-replicas 1 表示集群主节点需要多少个从节点，这里用了6台，即3台服务器构成集群，每台服务器设置1台从服务器

* 查看集群状态

到这里redis集群已经搭建完成了，我们可以对集群做一系列的操作了。
```bash
redis-cli -h 127.0.0.1 -p 6381 # 进入端口为6381的redis容器
cluster info
cluster nodes # 查看主从节点和槽点范围
```
## 三、集群操作
### 1. 主从容错切换
停止一个master，根据`cluster nodes`观察结果。结论：slave节点自动成为master节点，启动之前停止的节点，会自动加入集群。
### 2. 主从扩容
新建两个节点6387、6388。
```bash
redis-cli --cluster add-node 127.0.0.1:6387 127.0.0.1:6381 # 将6387节点作为master加入原集群。
redis-cli --cluster check 127.0.0.1:6381 # 检查是否加入该集群
redis-cli --cluster reshard 127.0.0.1:6381 # 重新分配槽号
redis-cli --cluster check 127.0.0.1:6381 # 查看分配情况
redis-cli --cluster add-node 127.0.0.1:6388 127.0.0.1:6387 --cluster-slave --cluster-master-id {id}
redis-cli --cluster check 127.0.0.1:6381 # 查看分配情况
```

### 3. 主从缩容
将6387、6388两个节点下线
```bash
redis-cli --cluster check 127.0.0.1:6388 # 检查集群情况，获得节点id
redis-cli --cluster del-node 127.0.0.1:6388 {id} # 删除节点6388
redis-cli --cluster reshard 127.0.0.1:6381 # 将6387的槽号清空，重新分配给master（会弹出命令选择分配和删除的id）
redis-cli --cluster check 127.0.0.1:6381 # 检查集群，确认已经清空
redis-cli --cluster del-node 127.0.0.1:6387 {id} # 删除节点6387
```