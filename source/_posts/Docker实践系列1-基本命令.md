---
title: Docker实践系列1-基本命令
date: 2017-05-30 17:00:24
tags: DOCKER
categories: DOCKER
---

## 1.如何使用Docker
### 1.1.Docker镜像常用操作
#### 1.1.1.列出本地可用镜像
```bash
$ docker images
```
#### 1.1.2.拉取镜像
```bash
$ docker pull centos
```
执行以上命令后，docker客户端将连接docker hub，并自动从centos镜像仓库中下载最新版本的centos镜像。下载完后可再次通过docker images命令列出镜像：

| REPOSITORY | TAG     | IMAGE ID     | CREATED      | SIZE     |
| ---------- | ------- | ------------ | ------------ | -------- |
| centos     | lastest | d0e7f81ca65c | 3 months ago | 196.6 MB |
- IMAGE ID:表示镜像的标识符，具备唯一性。这里看到的是一串12位的字符串，实际上它是64位完整镜像ID的缩略表达形式。

#### 1.1.3.搜索镜像
```bash
$ docker search centos
```
#### 1.1.4.删除镜像
```bash
$ docker rmi centos
$ docker rmi -f centos #强制删除有容器在跑的镜像
$ docker rmi -f `docker images -a -q` #一次性删除所有镜像
```
使用以上命令在Docker hub中搜索centos镜像。
#### 1.1.5.导出与导入镜像
使用以下命令导出centos镜像为一个tar文件（镜像包）：
```bash
$ docker save centos > centos.tar
```
若不指定导出的tar文件路径，则导出的centos.tar文件在当前目录上。

导出的centos.tar镜像包文件可随时在另一台Docker机器上导入：
```bash
$ docker load < centos.tar
```
当我们获取了centos镜像后，就能基于该镜像启动相应的容器。
### 1.2.Docker容器常用操作
#### 1.2.1.创建并启动容器
使用以下命令在运行centos镜像，从而创建并启动centos容器：
```bash
$ docker run -i -t centos /bin/bash
```
需要注意的是该命令实际上首先从本地获取centos镜像，若本地没有，则从docker hub拉取centos镜像并放入本地，随后才根据镜像创建并启动centos容器。
- -i选项：表示启动容器后，打开标准输入设备（STDIN）,可使用键盘进行输入。
- -t选项：表示启动容器后，分配一个伪终端（pseudo-TTY）,将于服务器建立一个会话。
- centos参数：表示需要运行的镜像名称，标准格式为centos:latest,若为latest版本，则可省略。
- /bin/bash参数：表示运行容器中的bash应用程序，因为我们此时并不需要运行其他程序，只想进入到容器中。
#### 1.2.2.列出容器
```bash
$ docker ps
```
| CONTAINER ID | IMAGE  | COMMAND     | CREATED     | STATUS       | PROTS | NAMES       |
| ------------ | ------ | ----------- | ----------- | ------------ | ----- | ----------- |
| 21e21e1j212j | centos | "/bin/bash" | 8 hours age | Up 5 seconds |       | ekdsaf_dfds |
- COMMAND:表示启动容器时运行的命令。
- NAMES:表示容器的名称，由docker引擎自动生成，也可以在docker run命令中通过--name选项来指定。

#### 1.2.3.进入容器
使用以下命令进入运行中的容器(需指定容器ID或容器名称)：
```bash
$ docker attach 21e21e1j212j 
```
只能进入运行中的容器，不能进入已停止的容器。此外，当打开多个终端并进入容器时，在一个终端中执行了某命令，该命令自动同步到其他终端中。例如在一个终端中执行了exit命令，将从终端中退出容器，此时其他终端也会自动退出容器。
#### 1.2.4.执行命令
使用以下命令向运行中的容器执行具体的命令(需指定容器ID或容器名称)：
```bash
$ docker exec -i -t 21e21e1j212j ls -l
```
#### 1.2.5.停止容器
使用以下命令停止运行中的容器(需指定容器ID或容器名称)：
```bash
$ docker stop 21e21e1j212j 
```
使用该命令可对容器发送SIGTERM信号，将等待一段很短的时间，再对容器发送SIGKILL信号，立刻终止容器。
#### 1.2.6.终止容器
使用以下命令终止运行中的容器(需指定容器ID或容器名称)：
```bash
$ docker kill 21e21e1j212j 
```
对容器发送SIGKILL信号，立刻终止容器
#### 1.2.7.启动容器
使用以下命令启动已停止的容器(需指定容器ID或容器名称)：
```bash
$ docker start 21e21e1j212j 
```
#### 1.2.8.重启容器
使用以下命令重启运行中的容器(需指定容器ID或容器名称)：
```bash
$ docker restart 21e21e1j212j 
```
该命令实际上首先执行了docker stop，再执行docker start.
#### 1.2.9.删除容器
使用以下命令删除已停止的容器(需指定容器ID或容器名称)：
```bash
$ docker rm 21e21e1j212j
$ docker rm -f 21e21e1j212j #强制删除运行中的容器
或 $ docker rm -f `docker ps -a -q` #一次性删除所有容器
```
#### 1.2.10.导出与导入容器
使用以下命令导出容器为一个tar文件（容器包）：
```bash
$ docker export 21e21e1j212j > centos.tar
```
导出的centos.tar容器包可随时在另一个docker机器上导入为镜像，命令为：
```bash
$ docker import centos.tar cai/centos:latest
```
需要注意的是，我们之前用docker load 命令（从镜像包中导入镜像）与现在用docker import(从容器包中导入镜像)都可以导入镜像，区别是容器包不包含任何历史记录，相当于容器的当前快照，镜像包则包含所有的历史记录，因此镜像包的体积较大。
### 1.3.Docker命令汇总
```bash
$ docker help
```

