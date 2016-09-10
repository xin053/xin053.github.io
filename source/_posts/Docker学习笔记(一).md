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

<!-- more -->

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
Docker可以帮你构建和部署容器，你只需把自己的应用程序或者服务打包放进容器即可。容器是基于镜像启动的，容器中可以运行一个或多个进程。镜像是Docker生命周期中的构建或打包阶段，而容器则是启动或执行阶段。

# Docker安装
Docker是基于linux中某些特性产生的，所以只支持linux系统，同时需要内核3.10以上，其他需要请看[官网](https://docs.docker.com)，目前支持的linux系统也比较少，所以对于mac和win而言，只能通过安装虚拟机来安装docker。由于我本机是win10，所以以下说明win系统下docker的安装，其他请看[官网](https://docs.docker.com)

官网提供`docker for windows`这款软件，但是很多要求，比如：

> Docker for Windows requires 64bit Windows 10 Pro, Enterprise and Education (1511 November update, Build 10586 or later) and Microsoft Hyper-V.

且不说对系统版本的问题，现在单说Hyper-V，这个是微软自带的虚拟机，启用之后，整个物理机处于虚拟化状态，在这之上是不能再开启vmware和virtual box等虚拟机软件的，这并不是关不关闭某些服务的问题，而是根本就不行。也就是不能共存。虽然可以通过添加启动项让系统启动的时候选择是否是Hyper-V环境，但是还是很麻烦。

而楼主需要vmware中的kali，所以想着既然微软自带hyper-V，便打算在这上面安装kali系统，结果折腾了半天，放弃了，原因如下：

- 虽然直接跟硬件打交道，但是对于图形化界面而言太卡了，即使分配2g内存，也卡卡卡卡，试试就知道，卡到怀疑这个软件，这个是我不使用这个的最主要原因
- 没有vmtools这种软件，屏幕分辨率只有一种，对于笔记本而言，出现上下滑动的滚动条极其不方便，虽然可以通过修改grub来指定特定的分辨率，但是不能自适应还是不方便
- 不能从物理机到虚拟机的自由拷贝，虽然这点也可以解决，不过因为第一点原因，太卡，最终还是放弃了hyper-v，我觉得我再也不会使用这个虚拟机了，不是说它不行，而是不适合我。

所以这里还是推荐使用`toolbox`，也就是docker早期提供的解决方案，这个软件中除docker必须组件外，还包括virtual box安装包，git安装包，安装过这两个的，可以在安装过程中取消勾选。

安装之后会生成两个快捷方式：`Docker Quickstart Terminal`和`Kitematic (Alpha)`，后者是管理docker的图形化软件，是辅助软件，不过很方便。

我们首先打开前者，首次打开会自动创建虚拟机，并运行虚拟机，配置好各种设置，最后显示如下说明安装完成：

**注意：即使关闭命令行窗口，后台载有docker的虚拟机还是在运行，需要打开virtual box手动关闭。关闭虚拟机后，以后打开`Docker Quickstart Terminal`会自动开启该虚拟机**

```bash


                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/

docker is configured to use the default machine with IP 192.168.99.100
For help getting started, check out the docs at https://docs.docker.com

Start interactive shell

zzx@zhouzixin MINGW64 ~
$
```

然后，可以检测下docker一些命令是否能够正常工作：

```bash
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world

c04b14da8d14: Pull complete
Digest: sha256:0256e8a36e2070f7bf2d0b0763dbabdd67798512411de4cdcf9431a1feb60fd9
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

```bash
$ docker version
Client:
 Version:      1.12.1
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   23cf638
 Built:        Thu Aug 18 17:52:38 2016
 OS/Arch:      windows/amd64

Server:
 Version:      1.12.1
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   23cf638
 Built:        Thu Aug 18 17:52:38 2016
 OS/Arch:      linux/amd64
```

```bash
$ docker info
Containers: 1
 Running: 0
 Paused: 0
 Stopped: 1
Images: 1
Server Version: 1.12.1
Storage Driver: aufs
 Root Dir: /mnt/sda1/var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 3
 Dirperm1 Supported: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: host null bridge overlay
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Security Options: seccomp
Kernel Version: 4.4.17-boot2docker
Operating System: Boot2Docker 1.12.1 (TCL 7.2); HEAD : ef7d0b4 - Thu Aug 18 21:18:06 UTC 2016
OSType: linux
Architecture: x86_64
CPUs: 1
Total Memory: 995.9 MiB
Name: default
ID: DIO7:T6MX:DIRC:2GND:ZQZE:XQW6:3GQF:KUKW:6NYB:NGXQ:Y67A:TBCH
Docker Root Dir: /mnt/sda1/var/lib/docker
Debug Mode (client): false
Debug Mode (server): true
 File Descriptors: 13
 Goroutines: 24
 System Time: 2016-09-10T06:02:28.737254965Z
 EventsListeners: 0
Registry: https://index.docker.io/v1/
Labels:
 provider=virtualbox
Insecure Registries:
 127.0.0.0/8
```

至此，docker的安装完成，至于linux和mac下的安装，请参见[官网](https://docs.docker.com)