---
title: Docker实践系列4-Springboot与Docker整合
date: 2017-06-06 22:11:47
tags: DOCKER
categories: DOCKER
---

#### 1.添加Dockerfile
这个文件放在src/main/resources资源目录下
```shell
FROM java
MAINTAINER "cai"<403411876@qq.com>
ADD @project.build.finalName@.jar app.jar
EXPOSE 8080
CMD java -jar app.jar
```
- 使用Docker官方提供的Java镜像，它是基于Openjdk制作的，也可以使用我们自己基于oracle jdk制作的镜像。
- 通过ADD命令将jar包添加到镜像中 。Springboot对maven的“资源过滤”特性进行了一些改进，我们需要使用@project.build.finalName@来获取jar包文件。
- EXPOSE暴露8080端口
- CMD执行启动命令

#### 2.使用Maven构建Docker镜像。
Spotify公司开源了一款docker-maven-plugin的Maven插件，我们可使用Maven命令来构建Docker镜像

将以下配置到pom.xml文件

```xml
<plugin>
  <groupId>com.spotify</groupId>
  <artifactId>docker-maven-plugin</artifactId>
  <version>0.4.10</version>
  <configuration>
    <imageName>${project.groupId}/${project.artifactId}:${project.version}</imageName>
    <dockerDirectory>${project.build.outputDirectory}/</dockerDirectory>
    <resources>
      <resource>
        <directory>${project.build.directory}</directory>
        <include>${project.build.finalName}.jar</include>
      </resource>
    </resources>
  </configuration>
</plugin>
```
- imageName:用于指定Docker镜像的完整名称
- dockerDirectory:用于指定Dockerfile文件所在的目录,指定为${project.build.outputDirectory}是为了读取经maven资源过滤后的Dockerfile文件，该文件中的@project.build.finalName@占位符此时已经被替换为实际内容
- resources/resource/directory:用于指定需要复制的跟目录，其中${project.build.directory}表示target目录。
- resources/resource/include:用于指定需要复制的文件。即为Maven打包后生产的jar文件。

执行mvn docker:build后，在target目录下生成了一个docker目录，该目录中不仅包含classes目录中的所有文件，还包含了一个打包生成的jar文件。随后，docker-maven-plugin插件就在docker目录下执行docker build命令来构建镜像。

如果打算将构建的镜像推送到Docker registry中，那么pom.xml可以定义其地址
```xml
<properties>
  <docker.registry>127.0.0.1:5000</docker.registry>
</properties>
```
将此属性作物imagName的前缀，再将imageName改为
```xml
<imageName>${docker.registry}/${project.groupId}/${project.artifactId}:${project.version}</imageName>
```
最后执行 mvn docker:build docker:push命令先构建后推送

查看命令帮助：mvn docker:help -Ddetail=true

#### 3.启动Springboot的docker容器

```shell
$ docker run -d -p 58080:8080 127.0.0.1:50000/com.cai/hello:1.0.0
```

#### 4.调整docker容器内存限制
查看容器运行情况
```shell
$ docker stats
```

docker run的-m参数调整容器内存

```bash
$ doceker run \
-d \
-p 58080:8080 \
-m 512m \
127.0.0.1:50000/com.cai/hello:1.0.0
```
