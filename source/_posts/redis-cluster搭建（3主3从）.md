---
title: redis-cluster搭建（3主3从）
date: 2017-04-25 14:14:45
tags: redis
categories: 中间件
---
###  一、搭建环境

- os:centos6.4

- redis:3.0.5

<!-- more -->

###  二、搭建步骤

#### 1.安装必要的依赖
- 安装gcc：yum install gcc
- 安装rubygems：yum install rubygems
- 安装ruby的redis驱动：gem install redis

#### 2..获取安装包
使用下载好的包redis-3.0.5.tar.gz或直接命令下载：
wget [http://download.redis.io/releases/redis-3.0.5.tar.gz](http://download.redis.io/releases/redis-3.0.5.tar.gz)

将安装包放到/soft/目录下:mv redis-3.0.5.tar.gz /soft/

#### 3.安装编译
- cd /soft
- tar zxvf redis-3.0.5.tar.gz 
- cd redis-3.0.5
- make MALLOC=libc
- make install PREFIX=/usr/local/redis


#### 4.创建文件夹

- mkdir /data/redis-cluster -p
- cd /data/redis-cluster/
- mkdir 7000 7001 7002 7003 7004 7005

#### 5.拷贝及修改配置文件

- cp /soft/redis-3.0.5/redis.conf  /data/redis-cluster/7000/

- cp /soft/redis-3.0.5/redis.conf  /data/redis-cluster/7001/

- cp /soft/redis-3.0.5/redis.conf  /data/redis-cluster/7002/

- cp /soft/redis-3.0.5/redis.conf  /data/redis-cluster/7003/

- cp /soft/redis-3.0.5/redis.conf  /data/redis-cluster/7004/

- cp /soft/redis-3.0.5/redis.conf  /data/redis-cluster/7005/

- vim /data/redis-cluster/7000/redis.conf
  分别修改以上路径的配置文件内容项：7000-7005

```
   daemonize yes
   pidfile /var/run/redis_7000.pid
   port 7000
   cluster-enabled yes
   cluster-config-file nodes_7000.conf
   cluster-node-timeout 15000
   appendonly yes
```

#### 6.启动6个redis实例

-  /usr/local/redis/bin/redis-server /data/redis-cluster/7000/redis.conf 
-  /usr/local/redis/bin/redis-server /data/redis-cluster/7001/redis.conf 
-  /usr/local/redis/bin/redis-server /data/redis-cluster/7002/redis.conf 
-  /usr/local/redis/bin/redis-server /data/redis-cluster/7003/redis.conf 
-  /usr/local/redis/bin/redis-server /data/redis-cluster/7004/redis.conf 
-  /usr/local/redis/bin/redis-server /data/redis-cluster/7005/redis.conf 

#### 7.将集群管理程序复制到redis安装目录下
cp /soft/redis-3.0.5/src/redis-trib.rb /usr/local/redis/bin/

#### 8.创建集群
/usr/local/redis/bin/redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 

       使用create命令 --replicas 1 参数表示为每个主节点创建一个从节点，其他参数是实例的地址集合

创建成功：

![](http://ooxz0ztfx.bkt.clouddn.com/rediscluster/6.png)



#### 9.检验测试

9.1测试存取值

​	客户端连接集群redis-cli需要带上 -c ，redis-cli -c -p 端口号

![](http://ooxz0ztfx.bkt.clouddn.com/rediscluster/1.png)

​	发现在几个节点中跳来跳去。

9.2测试集群节点选举

查看集群目前状况：/usr/local/redis/bin/redis-cli -c  -p 7000 cluster nodes
![](http://ooxz0ztfx.bkt.clouddn.com/rediscluster/2.png)

可以看出7000,7001,7002为master节点，7003,7004,7005为对应从节点

现在杀掉7001节点的进程，再看看集群状况：
![](http://ooxz0ztfx.bkt.clouddn.com/rediscluster/3.png)

可以看到，节点7001为fail状态，它的从节点7004被选举为master节点

现在我们将7001节点重新启动，看是否会自动加入集群中以及充当master节点还是slave节点。
![](http://ooxz0ztfx.bkt.clouddn.com/rediscluster/4.png)

可以看到7001节点恢复了，成为了7004节点的从节点。
### 三、附录
##### redis配置参数说明
　　daemonize：如需要在后台运行，把该项的值改为yes

　　pdifile：把pid文件放在/var/run/redis.pid，可以配置到其他地址

　　bind：指定redis只接收来自该IP的请求，如果不设置，那么将处理所有请求，在生产环节中最好设置该项

　　port：监听端口，默认为6379

　　timeout：设置客户端连接时的超时时间，单位为秒

　　loglevel：等级分为4级，debug，revbose，notice和warning。生产环境下一般开启notice

　　logfile：配置log文件地址，默认使用标准输出，即打印在命令行终端的端口上

　　database：设置数据库的个数，默认使用的数据库是0

　　save：设置redis进行数据库镜像的频率

　　rdbcompression：在进行镜像备份时，是否进行压缩

　　dbfilename：镜像备份文件的文件名

　　dir：数据库镜像备份的文件放置的路径

　　slaveof：设置该数据库为其他数据库的从数据库

　　masterauth：当主数据库连接需要密码验证时，在这里设定

　　requirepass：设置客户端连接后进行任何其他指定前需要使用的密码

　　maxclients：限制同时连接的客户端数量

　　maxmemory：设置redis能够使用的最大内存

　　appendonly：开启appendonly模式后，redis会把每一次所接收到的写操作都追加到appendonly.aof文件中，当redis重新启动时，会从该文件恢复出之前的状态

　　appendfsync：设置appendonly.aof文件进行同步的频率

　　vm_enabled：是否开启虚拟内存支持

　　vm_swap_file：设置虚拟内存的交换文件的路径

　　vm_max_momery：设置开启虚拟内存后，redis将使用的最大物理内存的大小，默认为0

　　vm_page_size：设置虚拟内存页的大小

　　vm_pages：设置交换文件的总的page数量

　　vm_max_thrrads：设置vm IO同时使用的线程数量