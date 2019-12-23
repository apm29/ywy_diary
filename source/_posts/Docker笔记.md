---
title: Docker笔记
tags:
  - Docker
categories:
  - 笔记
toc: false
date: 2019-12-20 03:08:17
---

### 1. Docker Install

参考Docker网站[Ubuntu安装Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/)的教程安装

### 2.切换Aws-Ubuntu到Root

 
```
sudo -s
```

### 3.Docker 启动

* 显示基本信息
```
 $ docker version
    #或者
 $ docker info
```
* Docker 需要用户具有 sudo 权限，为了避免每次命令都输入sudo，可以把用户加入 Docker 用户组[（官方文档）](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)。
```
$ sudo usermod -aG docker $USER
```
* 手动启动
安装之后, 你需要启动Docker Daemon. 大多数Linux发行版使用 `systemctl` 来启动服务. 如果不行可以用`service`方式.

* systemctl:

```
$ sudo systemctl start docker
```
service:

```
$ sudo service docker start
```

* 配置为自动启动
大多数Linux发行版 (RHEL, CentOS, Fedora, Ubuntu 16.04 and higher) 使用 `systemd` 来配置启动时运行哪些服务. Ubuntu 14.10 以下使用 `upstart`.

* systemd

```
$ sudo systemctl enable docker
```
禁用
```
$ sudo systemctl disable docker
```

### Image
> Docker 把应用程序及其依赖，打包在 image 文件里面。只有通过这个文件，才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。

> image 是二进制文件。实际开发中，一个 image 文件往往通过继承另一个 image 文件，加上一些个性化设置而生成。举例来说，你可以在 Ubuntu 的 image 基础上，往里面加入 Apache 服务器，形成你的 image。可以从[DockerHub](https://hub.docker.com/)下载想要的Image

```
# 列出本机的所有 image 文件。
$ docker image ls

# 删除 image 文件
$ docker image rm [imageName]
```