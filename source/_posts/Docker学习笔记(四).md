---
title: Docker学习笔记(四)
date: 2016-10-09 09:27:01
categories: 
- Docker
tags:
- Docker
---

## 使用docker nginx

首先参考官方docker nginx

[nginx](https://hub.docker.com/r/_/nginx/)

可以看到有很多不同标签的镜像，有基于debian的，也有基于alpine的，这里我们选择alpine，我们首先pull选择的镜像

```bash
docker pull nginx:1.11.4-alpine
```

最好还是看下该镜像的[Dockerfile](https://github.com/nginxinc/docker-nginx/blob/0dd9ef6a337474293b5e36c95a85da99b11e1a0a/mainline/alpine/Dockerfile)文件，对镜像的内容有些了解还是很有用的

下载完镜像之后我们继续查看文档，第一个例子就是利用nginx托管简单的静态文件

```bash
docker run --name nginx-test -v /some/content:/usr/share/nginx/html:ro -d nginx:1.11.4-alpine
```

`--name`设置容器的名称，`-v`挂载宿主主机目录到nginx容器指定的web目录，`ro`表示只读，`rw`表示可读可写，`-d`表示在宿主主机的后台运行，之后就是利用的镜像，下面是一片文章来回答docker容器在后台运行和前台运行的区别：

[Docker 容器后台运行和前台运行的区别](http://zhidao.baidu.com/link?url=os7Z3_MlRjE4TLqKutNp_CoRv2C2-fAEllErdGlIm1txwIa_nnBdbig9t9KKJwQRNLN1brBM9gACaHfwOrww_eAVN8cDZKViYF3By_R0sLW)

那么到目前为止一个简单的web应用就搭建好了，现在来说明一下windows环境下的各种问题

## windows环境说明

- 物理机win10 x64企业版
- 本人安装的是docker toolbox(在virtual box中创建一个名字为`default`的虚拟机来作为docker容器的宿主机)

而每个容器其实也就相当于是一个linux，所以这样其实在我电脑上有三层:

> win10物理机 -> default(linux)虚拟机(宿主机) -> 容器

这所以强调这一点是因为如果是正常的生产环境的话，应该是直接在一台linux服务器中安装docker，然后运行容器，也就是只有两层:

> linux(宿主机) -> 容器

那么三层会带来什么影响呢?

### 挂载卷

在选择挂载目录时，Kitematic提示只能挂载物理机上本用户的目录到`/usr/share/nginx/html`，也就是只能挂载`C:\Users\zzx`(zzx为我windows当前用户)目录下的文件夹到`/usr/share/nginx/html`，又看到`default`虚拟机设置中与windows物理机的共享文件夹是`C:\Users`，以为是只设置了c盘的原因，于是添加了一个f盘，发现还是只能挂载`C:\Users\zzx`下的目录，无奈，创建了`C:\Users\zzx\.docker\docker\static_web`,并将`spring`帮助文档中的`index.html`放在了该目录下，可以通过Kitematic修改挂载目录，挺方便的:

![](http://i.imgur.com/Fw45l7l.png)

那我我们如何访问到该`index.html`文件呢?由此引发第二个问题

### 如何访问

首先查看该容器`Ports`信息:

![](http://i.imgur.com/OdTEMPg.png)

可以看到容器暴露了80和443端口，而容器的80和443端口分别映射到宿主机的32769和32768端口，而192.168.99.100则是宿主机的ip，也可以通过以下命令查看端口映射:

```bash
docker port docker-name/id 需要查询的端口
```

![](http://i.imgur.com/0cpGsii.png)

那么在物理机windows中，就需要用`http://192.168.99.100:32769/`来访问该`index.html`文件了。因为windows只能和宿主机通信，只能看到32769和32768这两个端口，而宿主机才能和容器通信，宿主机才能看到容器暴露的80和443端口

![](http://i.imgur.com/QmYHUFM.png)

### 容器的ip

每个容器的ip都是docker设置的，并且使用的是`172.17.x.x`这个网段，所以容器间能够实现共享，因为都在同一个子网下，而宿主机和容器是通过网桥连接的，网桥的一端是宿主机中的适配器，另一端则是容器中代表`172.17.x.x`这个ip的适配器

在我此次实验中，容器`nginx-test`的ip是`172.17.0.2`

所以在宿主机中能够通过`http://localhost/`访问到`index.html`文件

## 连接容器

```bash
docker run -p 4567 --name webapp --link redis:db -t -i -v $PWD/webapp:/opt/webapp alpine sh
```

以上命令基于alpine镜像创建了一个名为`webapp`的容器，并暴露了该容器的4567端口，同时挂载了一个目录，最主要的是`--linke`选项

`--link`标志创建了两个容器间的父子连接，该标识需要两个参数，一个是要连接的容器的名称，第二个是连接后容器的别名。以上操作，我们把新容器连接到redis容器，并使用db作为别名。

通过把容器连接在一起，可以让父容器直接访问任何子容器的公开端口。更好的是，只有使用`--link`标志连接到这个容器的容器才能连接到这个端口，容器的端口不需要对本地宿主机公开。通过这个安全模型，可以限制容器化应用程序的被攻击面，减少应用暴露的网络。