---
title: Docker学习笔记(一)
date: 2016-07-13 12:16:39
categories: 
- Docker
tags:
- Docker
---

# 简介
Docker是容器，容器只能运行于底层宿主机相同或相似的操作系统。例如，可以在Ubuntu服务器上运行RedHat Enterprise Linux，但却无法在Ubuntu服务器上运行Windows

容器本身就比较复杂，不易安装，管理和自动化也很困难，而Docker就是为改变这一切而生。

## Docker简介
Docker是一个能够把开发的应用程序自动部署到容器的开源引擎。

使用Docker时，开发人员只需要关心容器中运行的应用程序，而维护人员只需要关心如何管理容器。

Docker的目标之一就是缩短代码从开发，测试到部署，上线运行的周期，让你的应用程序具备可移植性，易于构建，并易于协作。

## Docker组件
Docker核心组件：

- [Docker客户端和服务器端](#Docker客户端和服务器端)
- [Docker镜像](#Docker镜像)
- [Registry](#Registry)
- [Docker容器](#Docker容器)

### Docker客户端和服务器端
Docker是一个C/S架构的程序

![](http://i.imgur.com/Q9SPlJo.png)

### Docker镜像
用户基于镜像来运行自己的容器。可以把镜像当作容器的“源代码”。镜像体积很小，易于分享，存储和更新。

### Registry
Docker用Registry来保存用户构建的镜像。Registry分为公共和私有两种。Docker公司运营的公共Registry叫做Docker Hub。

### Docker容器