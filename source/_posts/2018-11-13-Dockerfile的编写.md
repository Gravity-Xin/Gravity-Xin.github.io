---
title: Dockerfile的编写
date: 2018-11-13 15:51:44
categories: 
- Docker/K8S
tags: 
- Dockerfile
comments: true
---

## 构建Docker镜像的方式

* `docker container commit`: 通过容器构建
* `docker image build`: 通过Dockerfile文件构建

## Dockefile的组成

Dockerfile是一个文本文件，其中包含了一条条的指令(Instruction)，每条指令构建一层

* `FROM image:tag`: 指定基础镜像，必须是第一条指令 **scratch空白镜像**
* `MAINTAINER name`: 指定镜像的作者信息
* `RUN`: 指定镜像构建过程中运行的命令 **每一个RUN命令都会在当前镜像的上层创建一个新的镜像用于执行该命令，因此为了避免镜像的臃肿，应该将RUN中的命令使用&&进行连结**
  * `RUN command param1 param2`: shell模式 -> /bin/bash command param1 param2
  * `RUN {"command", "param1", "param2"}`: exec模式 -> 可以指定其他shell来运行命令
* `EXPOSE port`: 指定运行该镜像的容器使用的端口，可指定多个 **在运行该镜像时，仍然需要再次指定容器的端口**
* `CMD`: 指定运行该镜像时，容器中默认运行的命令 **如果启动容器时，同时指定了命令，则CMD会被覆盖**
  * `CMD command param1 param2`: shell模式
  * `CMD {"command", "param1", "param2"}`: exec模式
  * `CMD {"param1", "param2"}`: 作为ENTRYPOINT指定的默认参数
* `ENTRYPOINT`: 与CMD指定类似 **如果运行容器时，同时指定了命令，则ENTRYPOINT不会被覆盖，除非使用了docker image run的--entrypoint参数**
  * `ENTRYPOINT command param1 param2`: shell模式
  * `ENTRYPOINT {"command", "param1", "param2"}`: exec模式
* `ADD src dest`: 将文件或目录复制到镜像中，src为构建目录的相对路径，dest为镜像中的绝对路径
* `COPY src dest`: 将文件或目录复制到镜像中，src为构建目录的相对路径，dest为镜像中的绝对路径 **如果时单纯的复制文件，推荐使用COPY指令**
* `VOLUME ["/data]`: 向镜像运行的容器中添加数据卷
* `WORKDIR path`: 设置容器中运行的命令的工作目录，与CMD和ENTRYPOIN指令配合使用
* `ENV key=value`: 设置容器中的环境变量
* `USE username`: 设置容器中运行命令的用户，默认使用root用户来运行命令
* `ONBUILD [INSTRUCTION]`: 添加镜像触发器，当该镜像被其他镜像作为基础镜像时，执行触发器
* `HEATHCHECK [OPTIONS] command`: 容器健康检查

## 构建过程

* 从基础镜像中运行一个容器
* 执行一条指令，对容器进行修改
* 执行类似docker commit的操作，提交一个新的镜像层
* 基于刚提交的新的镜像层，运行一个新的容器
* 再次执行Dockerfile中的下一条指令
* 反复执行上述过程，直到所有指令执行完毕

`docker image build`会自动删除构建过程中运行的中间容器，但是不会删除构建过程中产生的中间层镜像，以方便使用缓存的镜像加快构建速度

## 构建Golang应用

```dockefile
FROM scratch
ADD main /
CMD ["/main"]
```