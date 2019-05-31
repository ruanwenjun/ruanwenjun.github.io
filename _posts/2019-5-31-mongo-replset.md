---
layout: post
title: 关于MongoDB副本集
tags: [MongoDB]
---
目录：
- [副本集](#副本集)
- [单机搭建Mongo副本](#单机搭建mongo副本)

Mongo副本集是为了实现高可用、数据冗余。一个MongoDB副本集由一组拥有相同数据的MongoDB进程组成。试想，你的服务需要连一台Mongo，如果这个Mongo突然挂了，那么你的服务就挂了。但是如果你的Mongo设置了副本，那么如果这台Mongo挂了，你的服务会自动连到这台Mongo的副本。所以你的服务对外还是好的。

## 副本集
副本集中有两个角色Primary和Second。所有的写请求都发送到Primary，Second负责从Primary中同步数据。一个Replica set只能有一个Primary，当Primary挂了之后，Second中会选举出一个来当Primary。

PSS架构：包含一个Primary和若干Second。该架构模式下，Replica set存活节点数必须为奇数，以保证选举投票时可以出现大多数。

PSA架构：由偶数个节点加一个Arbiter构成

## 单机搭建Mongo副本
新建三个配置文件:config1,config2,config3
```java
dbpath=/data/mongo_set/db1 
logpath=/data/mongo_set/logs/db1.log 
port=20001
replSet=rs
```
```java
dbpath=/data/mongo_set/db2
logpath=/data/mongo_set/logs/db2.log 
port=20002
replSet=rs
```
```java
dbpath=/data/mongo_set/db3
logpath=/data/mongo_set/logs/db3.log 
port=20003
replSet=rs
```
然后启动这三个Mongo
```sh
./mongod —-config config1
./mongod —config config2
./mongod —config config3
```
连到任意一台mongo,执行
```
config={_id:"rs",members:[{_id:0,host:”localhost:20001"},{_id:1,host:”localhost:20002"},{_id:2,host:”localhost:20003"}]}
rs.initiate(config)
```
至此，mongo副本已经搭建完毕。可以使用rs.status()命令查看副本状态。同时可以模拟当primary挂了之后，secondary是否能够自动提供服务

关于更多副本集可以参考[https://docs.mongodb.com/manual/replication/](https://docs.mongodb.com/manual/replication/)


