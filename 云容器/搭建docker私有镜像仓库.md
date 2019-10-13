---
title: 搭建docker私有镜像仓库
date: 2019/2/6 20:00:00
updated: 2019/2/6 20:00:00
categories: 
- 云容器
tags: 
- 云容器
- docker
- 开发环境
---

本文记录如何使用registry镜像创建docker私有仓库，并试验将一个本地镜像推送到私有仓库。

# 搭建私有仓库
docker官方提供了一个registry镜像供我们搭建本地私有的仓库环境。

执行命令，这个命令会自动下载registry容器，并创建私有仓库服务。仓库默认被创建在`/tmp/registry`下：
```shell
$ docker run -d -p 5000:5000 registry
```
用这种方式创建，一旦容器被删除，则镜像也会丢失。我们也可以指定一个其他目录来创建仓库：
```shell
$ docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry
# 看一下端口
$ netstat -nlp
```
现在本地已经在5000端口自动启动了一个私有仓库服务。
<!-- more -->

# 测试服务
接下来我们push一个镜像到刚刚搭建的私有仓库，试试灵不灵。
```shell
$ docker pull hello-world

# 打个标签
$ docker tag hello-world:latest 127.0.0.1:5000/myhello-world:latest
$ docekr images
# 这两个hello-world镜像实际上指向的是同一个镜像文件

# 推送到私有仓库
$ docker push 127.0.0.1:5000/myhello-world:latest

# 验证一下是否成功
# api文档：https://docs.docker.com/registry/spec/api/#listing-repositories
$ curl -XGET http://127.0.0.1:5000/v2/_catalog
$ docker pull 127.0.0.1:5000/myhello-world:latest
```
这样就搞定了。