---
title: Docker学习笔记(二)
date: 2016-09-20 16:04:51
categories: 
- Docker
tags:
- Docker
---

## 解决问题
日常升级完VirtualBox后，发现打开docker虚拟机出差，报错：

```
Failed to open/create the internal network 'HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter'
```

解决方法：

- 打开网络和共享中心
- 更多适配器设置
- 选择 对应的网络适配器 adapter （如： VirtualBox Host-Only Ethernet Adapter）
- 右键“属性”
- 然后勾选 VirtualBox NDIS6 Bridged Networking Driver 选项，确定

<!-- more -->

## 使用docker

### 创建容器

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

注意区分容器和镜像，我们可以基于同一个镜像创建多个容器，但是每个容器的id和名称都是不同的，如果我们创建容器时不指定容器的名字，docker会帮我们自动随机生成一个名称，也可以通过`--name`来指定名称。如果创建容器使用的镜像在本地不存在，docker会自动去docker hub下载相应的镜像，这里强烈推荐alpine镜像

### 查看容器

首先通过Kitematic 下载alpine镜像,然后启动容易，输入

```bash
docker ps -a
```

查看所有的容器，包括正在运行和已经停止的

![](http://i.imgur.com/cBXdSTK.png)

输入：

```bash
docker ps
```

则是查看正在运行的容器

### 重启和停止容器

```bash
docker start/stop/restart docker-name/id
```

![](http://i.imgur.com/2PQcNnG.png)

docker容器在重新启动的时候，会沿用`docker run`命令指定的参数来运行。

容器没有stop时，这时候可以使用：

```bahs
docker attach docker-name/id
```

附着到正在运行的容器上

如果退出容器的shell，容器也会随之停止运行

## 创建守护式容器

以上创建的都是交互式运行的容器，我们也可以创建长期运行的守护式容器，非常适合运行应用程序和服务

```bash
docker run --name alpine-test -d alpine sh -c "while true;do echo hello world;sleep 1;done"
```

该命令基于alpine镜像创建一个一个名称为alpine-test的容器，`-d`表示是守护式容器，执行该命令只会返回容器的id，该容器运行时每过1秒会输出一个`hello world`，通过`docker logs -f docker-name/id`可以查看容器运行日志

## 其他常用docker命令

### 查看容器内的进程

```bash
docker top docker-name/id
```

### 在容器内部运行进程

例子：

```bash
docker exec -d docker-name/id touch /etc/config_file
```

创建了一个后台运行的进程

```bash
docker exec -t -i docker-name/id /bin/bash
```

创建了一个前台运行的进程,也就是创建了一个bash会话

### 自动重启容器

```bash
docker run --restart=always --name alpine-test -d alpine sh
```

创建的容器会在出现错误停止运行后自动重启容器

### 查看容器信息

```bash
docker inspect docker-name/id
```

可以查看创建的容器的相关信息

### 删除容器

```bash
docker rm docker-name/id
```

删除容器之后要先停止容器

## 参考文章

- [Docker微容器Alpine Linux](http://blog.csdn.net/zdy0_2004/article/details/50785506)

- [alpine](https://hub.docker.com/r/_/alpine/)

- [1science](https://hub.docker.com/u/1science/)/[alpine](https://hub.docker.com/r/1science/alpine/)

  ​

  ​