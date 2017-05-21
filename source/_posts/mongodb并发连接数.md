---
title: mongodb并发连接数
date: 2017-05-20 00:01:49
tags: mongodb
categories: 工作笔记
---

高并发时生产应用出现mongodb连接超时的异常：
com.mongodb.DBPortPool$ConnectionWaitTimeOut: Connection wait timeout after 120000 ms

查看mongodb服务器当前连接数
```bash
#./mongo admin -u admin -p admin
MongoDB shell version: 2.2.0
connecting to: admin
>  db.serverStatus().connections;
{ "current" : 174, "available" : 645 }
```
不是mongodb自身的原因，查看应用的mongodb的连接池最大连接数connectionsPerHost原来只设了50，调大到200解决。(PS:网上有资料说Mongodb某个版本的java驱动有这个bug)

附资料：
关于MONGODB最大连接数的查看与修改

在Linux平台下，无论是64位或者32位的MongoDB默认最大连接数都是819，WIN平台不知道，估计也没有人在 WIN平台下使用MongoDB做生产环境
```bash
[root@DELL113 mongodb-linux-i686-2.4.1]# mongo admin -u root -p password
MongoDB shell version: 2.4.1
connecting to: 192.168.6.42/admin
> db.serverStatus().connections
{ "current" : 1, "available" : 818, "totalCreated" : NumberLong(1) }
```
途中available显示818少了一个，表示空闲的。current表示已经占用了的连接数，两数一加就等于819，如果我现在在连接一个，那么available就是817，current就是2
```bash
[root@DELL113 mongodb-linux-i686-2.4.1]# ./bin/mongo 192.168.6.42
MongoDB shell version: 2.4.1
connecting to: 192.168.6.42/test
> db.serverStatus().connections
{ "current" : 1, "available" : 818, "totalCreated" : NumberLong(1) }
> db.serverStatus().connections
{ "current" : 2, "available" : 817, "totalCreated" : NumberLong(2) }
```
819个连接数对于一般的站点我认为已经够用，并且都是现连现取现断。但这个连接数也可以修改，只要在启动的时候加入--maxConns即可
服务器启动
```bash
[root@lee mongodb-linux-x86_64-2.4.1]# ./bin/mongod --dbpath=/root/db --maxConns=2000
Wed Apr 3 11:06:21.905 [initandlisten] MongoDB starting : pid=2812 port=27017 dbpath=/root/db 64-bit host=lee
Wed Apr 3 11:06:21.957 [initandlisten] db version v2.4.1
Wed Apr 3 11:06:21.957 [initandlisten] git version: 1560959e9ce11a693be8b4d0d160d633eee75110
Wed Apr 3 11:06:21.957 [initandlisten] build info: Linux ip-10-2-29-40 2.6.21.7-2.ec2.v1.2.fc8xen #1 SMP Fri Nov 20 17:48:28 EST 2009 x86_64 BOOST_LIB_VERSION=1_49
Wed Apr 3 11:06:21.957 [initandlisten] allocator: tcmalloc
Wed Apr 3 11:06:21.957 [initandlisten] options: { dbpath: "/root/db", maxConns: 2000 }
Wed Apr 3 11:06:21.982 [initandlisten] journal dir=/root/db/journal
Wed Apr 3 11:06:21.982 [initandlisten] recover : no journal files present, no recovery needed
Wed Apr 3 11:06:22.297 [initandlisten] preallocateIsFaster=true 2.62
Wed Apr 3 11:06:22.717 [initandlisten] --maxConns too high, can only handle 819
Wed Apr 3 11:06:22.724 [initandlisten] waiting for connections on port 27017
Wed Apr 3 11:06:22.725 [websvr] admin web console waiting for connections on port 28017
Wed Apr 3 11:06:25.126 [initandlisten] connection accepted from 192.168.4.86:53917 #1 (1 connection now open)
```
查询最大连接数
```bash
[root@DELL113 mongodb-linux-i686-2.4.1]# ./bin/mongo 192.168.6.42
MongoDB shell version: 2.4.1
connecting to: 192.168.6.42/test
> db.serverStatus().connections
{ "current" : 1, "available" : 818, "totalCreated" : NumberLong(1) }
> 
```
发现还是819？其实是Linux默认进程能打开最大文件数有关，可以通过ulimit 解决
```bash
[root@lee mongodb-linux-x86_64-2.4.1]# ulimit -n 2500
[root@lee mongodb-linux-x86_64-2.4.1]# ./bin/mongod --dbpath=/root/db --maxConns=2000
Wed Apr 3 11:11:07.013 [initandlisten] MongoDB starting : pid=2930 port=27017 dbpath=/root/db 64-bit host=lee
Wed Apr 3 11:11:07.013 [initandlisten] db version v2.4.1
Wed Apr 3 11:11:07.013 [initandlisten] git version: 1560959e9ce11a693be8b4d0d160d633eee75110
Wed Apr 3 11:11:07.013 [initandlisten] build info: Linux ip-10-2-29-40 2.6.21.7-2.ec2.v1.2.fc8xen #1 SMP Fri Nov 20 17:48:28 EST 2009 x86_64 BOOST_LIB_VERSION=1_49
Wed Apr 3 11:11:07.013 [initandlisten] allocator: tcmalloc
Wed Apr 3 11:11:07.013 [initandlisten] options: { dbpath: "/root/db", maxConns: 2000 }
Wed Apr 3 11:11:07.031 [initandlisten] journal dir=/root/db/journal
Wed Apr 3 11:11:07.031 [initandlisten] recover : no journal files present, no recovery needed
Wed Apr 3 11:11:07.170 [initandlisten] waiting for connections on port 27017
Wed Apr 3 11:11:07.171 [websvr] admin web console waiting for connections on port 28017
Wed Apr 3 11:11:10.076 [initandlisten] connection accepted from 192.168.4.86:53161 #1 (1 connection now open)
```
再查看最大连接数，搞定
```bash
[root@DELL113 mongodb-linux-i686-2.4.1]# ./bin/mongo 192.168.6.42
MongoDB shell version: 2.4.1
connecting to: 192.168.6.42/test
> db.serverStatus().connections
{ "current" : 1, "available" : 1999, "totalCreated" : NumberLong(1) }
> 
```
关于ulimit的更多知识大家可以去网上检索检索

客户端程序通常是通过DRIVER来链接，由于每次建立链接的成本都挺高，因此都用链接池来实现，SPRING DATA MONGODB中是如下配置
```bash
mongo.dbname=cms
#线程池的大小
mongo.connectionsPerHost=100
#这个*mongo.connectionsPerHost则是如果链接数大于100的等待xttk数
mongo.threadsAllowedToBlockForConnectionMultiplier=4
#等待线程的等待时间
mongo.maxWaitTime=1500
mongo.socketTimeout=1500
mongo.connectTimeout=1000
mongo.autoConnectRetry=true
mongo.socketKeepAlive=true
mongo.slaveOk=true
```



