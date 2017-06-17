---
title: Docker实践系列2-制作镜像
date: 2017-05-31 23:39:55
tags: DOCKER
categories: DOCKER
---

### 1.手工制作Java镜像
1. 从docker hub中获取一份centos镜像到本地，准备oracle jdk的压缩包放在宿主机的/software目录下。
2. 启动centos容器

```bash
$ docker run -i -t -v /software:/mnt/software centos /bin/bash
```
-v选项，即数据卷（DataVolume），用于将宿主机上的磁盘挂载到容器中。格式为“宿主机路径:容器路径”，宿主机路径可以为相对路径，容器路径必须为绝对路径，此外，多次使用-v选项，可以同时挂载多个宿主机路径到容器中。
3. 进入容器后，执行命令,安装JDK
```bash
#验证是否挂载成功
ll /mnt/software
#解压jdk压缩包
tar -zxf /mnt/software/jdk-8u91-linux-x64.tar.gz -C /opt
#查看解压文件夹
ll /opt
#为了便于访问与升级，建立一个软连接，用于快速访问JDK跟目录
ln -s /opt/jdk1.8.0_91 /opt/jdk
ll /opt/jdk
#验证JDK是否安装成功
/opt/jdk/bin/java -version
```
4. 提交镜像

再打开一个终端，查看当前运行中的容器
```bash
$ docker ps
#提交容器为一个新的镜像
docker commit be3dsddwewq2 cai/java
#验证查看本地是否保护cai/java镜像
$ docker images
```
5. 验证镜像

用刚才创建的镜像启动一个容器，若容器能成功输出java版本号，则说明镜像是可用的。
```bash
$ docker run --rm cai/java /opt/jdk/bin/java -version
```
-rm选项，表示当容器退出时可自动删除容器，也就是运行完java -version命令后，容器自动退出。当我们不想保留该容器，自动执行docker rm操作，将自己删除。

### 2.使用Dockerfile构建镜像
#### 2.1.Dockerfile基本结构
Dockerfile实际上是一个编写Docker镜像的脚本，该脚本有一个固定的格式。我们首先创建一个空白的文本文件，文件名为Dockerfile。其实叫其他名字也可以，只是Dockerfile是默认的文件名而已。
1. 设置基础镜像

当前构建的镜像必须继承于某个“基础镜像”,例如基于centos最新版镜像：

```
FROM centos:latest
```
FROM指令有固定的格式，即“仓库名:标签名”,若使用最新版本，则latest可以省略。
2. 设置维护者信息

```
MAINTAINER "CAI"<403411876@qq.com>
```

3. 设置需要添加到容器的文件

例如我们从Oracle官网下载的JDK压缩包，可使用ADD指令将此文件添加到容器的指定目录中：

```
ADD jdk-8u91-linux-x64.tar.gz /opt
```
- ADD指令的第一个参数为宿主机的来源路径（可用相对路径），第二个参数是容器的目标路径（必须为绝对路径）。ADD指令将自动解压来源路径中的压缩包，将解压后的文件复制到目标路径中。此外我们一般将Dockerfile文件与需要添加到容器的文件放在同一目录下，这样有助于编写来源路径。
- 还有一个与ADD指令功能类似的COPY指令，只是后者只能完成简单的复制，而没有自动解压的功能。
4. 设置镜像制作过程中需要执行的命令

```
RUN ln -s /opt/jdk1.8.0_91 /opt/jdk
```
如果需要执行多条命令，我们可以使用多行RUN命令，但如果将多条指令通过\命令换行符合并成一条，这样将减少所构建的镜像的体积。原因是，在镜像中每执行一条命令，都会形成新的“镜像层”，我们需要尽可能减少镜像层，从而减少镜像体积。
5. 设置容器启动时需要执行的命令

我们在使用docker run命令时，可在命令的最后一段添加一个容器启动时需要执行的命令，在Dockerfile中也有相对应的CMD指令

```
CMD /opt/jdk/bin/java -version
```
- 如果使用docker run命令时指定了需要执行的命令，那么该命令将覆盖Dockerfile中通过CMD设置的命令。
- 还有一个类似的ENTRYPOINT指令，他所执行的指令不能被docker run命令所覆盖。
- CMD指令要么没有，要么只有一条，CMD指令还能与ENTRYPOINT指令联合使用。

#### 2.2.使用Dockerfile构建镜像
Dockerfile内容如下：
```
FROM centos:latest
MAINTAINER "CAI"<403411876@qq.com>
ADD jdk-8u91-linux-x64.tar.gz /opt
RUN ln -s /opt/jdk1.8.0_91 /opt/jdk
CMD /opt/jdk/bin/java -version

```
使用docker build 命令读取Dockerfile文件，并构建一个镜像

```bash
$ docker bulid -t cai/java .
```
-t选项指定镜像的名称，并读取当前的目录(即.目录)中的Dockerfile文件。

通过docker images发现这次构建的镜像与之前手工构建的镜像所包括的仓库名与标签名完全相同，之前的都被更新为<none>了。用以下命令修改执行的仓库名和标签名

```
$ docker tag eirj313213 cai/java:1.0
```

Dockerfile也提供设置环境变量的指令，ENV
```
FROM centos:latest
MAINTAINER "CAI"<403411876@qq.com>
ADD jdk-8u91-linux-x64.tar.gz /opt
RUN ln -s /opt/jdk1.8.0_91 /opt/jdk
ENV JAVA_HOME /opt/jdk
ENV PATH $JAVA_HOME/bin:$PATH
CMD java -version

```
#### 2.3.Dockerfile的其他指令