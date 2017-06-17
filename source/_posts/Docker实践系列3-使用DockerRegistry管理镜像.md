---
title: Docker实践系列3-使用DockerRegistry管理镜像
date: 2017-06-04 23:51:56
tags: DOCKER
categories: DOCKER
---
- Docker Hub相当于maven中央仓库
- Docker Registry相当于局域网的mavan仓库（nexus）

### 1.使用DockerHub
1. 注册登录docker hub
2. 在docker hub上创建仓库
3. 推送本地镜像到docker hub

```
$ docker push cai/java
```
输出没有授权，需要登录

```
$ docker login
```

### 2.搭建Docker Registry
#### 2.1.启动Docker Registry

```
$ docker run -d -p 50000:50000 -v ~/docker-registry:/tmp/registry registry
```
- -d:表示将在后台启动该容器。
- -p:表示端口映射。左边的为宿主机的端口，右边的为容器内部需要暴露的端口
- -v:推送的镜像文件会放在容器的/tmp/registry,这样设置能可通过宿主机访问已推送的镜像，对镜像文件备份。

还可以通过访问127.0.0.1:50000/v1/search，搜索docker registry中的镜像仓库。
#### 2.2.重命名镜像标签
由于cai/java镜像默认的注册中心为Docker Hub，我们使用docker push命令推送的目标地址实际上都是docker hub，因此cai/java镜像的完整名称应为docker.io/cai/java。如果我们打算将cai/java镜像推送至本地的docker registry，则需将镜像名称修改为127.0.0.1:5000/cai/java。

```
$ docker tag fwfdfw21213e2 127.0.0.1:5000/cai/java
```

#### 2.3.推送镜像
```
$ docker push 127.0.0.1:5000/cai/java
```
一般情况下，我们可将Docker Registry与Nginx集成，将它们部署在一台稳定的服务器上，通过Nginx反向代理的方式来调用Docker Registry,并将IP绑定到一个内部域名上，比如docker-registry.xxx.com。


