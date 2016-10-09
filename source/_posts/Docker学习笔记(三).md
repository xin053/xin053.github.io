---
title: Docker学习笔记(三)
date: 2016-10-07 18:44:42
categories: 
- Docker
tags:
- Docker
---

## 深入docker镜像

docker镜像是由文件系统叠加而成，最底端是一个引导文件系统，即bootfs，这很像典型的linux的引导文件系统。docker用户几乎永远不会和引导系统有什么交互。实际上，当一个容器启动后，它会被移到内存中，而引导文件系统则会被卸载，以留出更多的内存共initrd磁盘镜像使用。

到目前为止，docker看起来还很像一个典型的linux虚拟化栈。实际上，docker镜像的第二层是root文件系统rootfs，它位于引导文件系统之上。

在传统的linux引导过程中，root文件系统会最先以只读的方式加载，当引导结束并完成了完整性检查之后，它才会被切换为读写模式，但是在docker里，root文件系统永远只能是只读状态，而且docker利用联合加载技术又会在root文件系统层上加载更多的只读文件系统。联合加载指的是一次同时加载多个文件系统，但是在外面看起来只能看到一个文件系统，联合加载会将各文件系统叠加在一起，这样最终的文件系统会包含所有底层的文件和目录。

<!-- more -->

docker将这样的文件系统成为镜像，一个镜像可以放在另一个镜像的顶部。位于下面的镜像成为父镜像。最底层的镜像称为基础镜像。最后，当从一个镜像启动容器时，docker会在该镜像的最顶层加载一个读写文件系统。我们想在docker中运行的程序就是在这个读写层中执行的

![](http://i.imgur.com/0NK2H01.png)

当docker第一次启动一个容器时，初始的读写层是空的，当文件系统发生变化时，这些变化都会应用到这一层上。比如，如果想修改一个文件，这个文件首先会从该读写层下面的只读层复制到该读写层。该文件的只读版本依然存在，但是已经被读写层中的该文件副本所隐藏

通常这种机制被称为写时复制。每个只读镜像层都是只读的，并且以后永远不会变化。当创建一个新容器时，docker会构建一个镜像栈，并在栈的最顶端添加一个读写层。这个读写层再加上其下面的镜像层以及一些配置数据，就构成了一个容器。容器是可以修改的，并且是可以启动和停止的。容器的这种特点加上镜像分层框架，使我们可以快速构建镜像并运行包含我们自己的应用程序和服务的容器。

## 列出镜像

```bash
docker images
```

## 仓库

docker hub中有两种仓库

- 用户仓库    镜像都是有docker用户创建的
- 顶层仓库    镜像是由docker内部人员管理的

用户仓库的命名由用户名和仓库名两部分组成，如jamtur01/puppet

顶层仓库只包含库名部分，如ubuntu仓库

## 构建镜像

一般来说，我们不是真正创建新镜像，而是基于一个已有的基础镜像，构建新镜像而已。

创建static_web文件夹并在其中创建Dockerfile文件:

```dockerfile
# version:0.0.1
FROM ubuntu:14.04
MAINTAINER James Turnbull "james@example.com"
RUN apt-get update
RUN apt-get install -y nginx
RUN echo 'Hi,I am in your container' > /usr/share/nginx/html/index.index
EXPOSE 80
```

每条指令都会创建一个新的镜像层并对镜像进行提交。docker大体上按照如下流程执行dockerfile中的指令

- docker从基础镜像运行一个容器
- 执行一条指令，对容器做出修改
- 执行类似docker commit的操作，提交一个新的镜像层
- docker再基于刚提交的镜像运行一个新容器
- 执行dockefile中的下一条指令，直到所有指令都执行完毕

接着输入以下命令来创建我们的镜像：

```bash
cd static_web
docker build -t="jamtur01/static_web"
```

`-t`选项指定仓库名和镜像名称

也可以通过下面命令来为镜像设置一个标签

```bash
docker build -t="jamtur01/static_web:v1"
```

如果没有指定任何标签，docker会自动为镜像设置一个`latest`标签

以上的例子告诉docker在当前目录寻找Dockerfile文件，也可以指定一个git仓库的源地址来指定Dockerfile的位置，如：

```bash
docker build -t="jamtur01/static_web:v1" git@github.com:jamtur01/docker-static_web
```

该git仓库的根目录下存在Dockerfile文件

而根目录下的`.dockerignore`文件的功能与`.gitignore`文件作用一样

## 构建缓存

默认构建镜像时，会使用缓存技术，所以如果Dockerfile文件内容没有变化，再次构建镜像时，产生的镜像与之前一样，如果有`apt-get update`等命令，这样docker将不会再次更新apt包，所以如果想要docker构建镜像时再一次的执行Dockerfile里面的命令时，需要明确指定不使用缓存功能

```bash
docker build --no-cache -t="jamtur01/static_web"
```

## 构建历史

```bash
docker history docker-name/id
```

可以查看镜像是如何构建出来的

## Dockerfile指令

### CMD

CMD指令用于指定一个容器启动时要运行的命令。这有点类似与RUN命令，只是RUN指令是指定镜像被构建时要运行的指令，而CMD是指定容器被启动时要运行的命令。这和使用`docker run`命令启动容器时执行要运行的命令非常类似

```bash
docker run -i -t jamtur01/static_web /bin/true
```

与在Dockerfile中使用如下代码是等效的

```dockerfile
CMD ["/bin/true"]
```

也可以指定运行使用的参数

```dockerfile
CMD ["/bin/bash", "-l"]
```

这里我们将`-l`参数传递给了`/bin/bash`

最后，需要注意的是，`docker run`命令可以覆盖`CMD`指令

### ENTRYPOINT

`ENTRYPOINT`指令不容易在启动容器时被覆盖，`docker run`命令行中指定的任何参数都会被当作参数再次传递给`ENTYRPOINT`指定的命令

```dockerfile
ENTRYPOINT ["/usr/sbin/nginx"]
CMD ["-h"]
```

此时当我们启动一个容器，任何在命令行中指定的参数都会被传递给Nginx守护进程。如果在启动容器时不指定任何参数，则在CMD指令中指定的`-h`参数会被传递给Nginx守护进程。

这使得我们可以构建一个镜像，该镜像既可以运行一个默认的命令，同时它也支持通过`docker run`命令行为该命令指定可覆盖的选项或者标志。

### WORKDIR

用于切换工作目录

```dockerfile
WORKDIR /opt/webapp/db
RUN bundle install
WORKDIR /opt/webapp
ENTRYPOINT ["rackup"]
```



### ENV

在镜像构建过程中设置环境变量

```dockerfile
ENV RVM_PATH /home/rvm
```

这个新的环境变量可以在后续的任何`RUN`等指令中使用

```dockerfile
ENV TARGET_DIR /opt/app
WORKDIR $TARGET_DIR
```

### USER

用来指定该镜像会以什么样的用户去运行

```dockerfile
USER nginx
```

指定该镜像启动的容器会以nginx用户的身份来运行

如果没有使用`USER`指令指定用户，默认用户为root

### VOLUME

用来向基于镜像创建的容器添加卷，一盒卷是可以存在于一个或多个容器内的特定目录，这个目录可以绕过联合文件系统，并提供如下共享数据或者对数据进行持久化的功能

- 卷可以在容器间共享和重用
- 一个容器可以不是必须和其他容器共享卷
- 对卷的修改是立刻生效的
- 对卷的修改不会对更新镜像产生影响
- 卷会一直存在直到没有任何容器再使用它

卷的功能让我们可以将数据(如源代码),数据库或者其他内容添加到镜像中而不是将这写内容提交到镜像中，并且允许我们在多个容器间共享这些内容，我们可以利用此功能来测试容器和内部的应用程序代码，管理日志，或者处理容器内部的数据库。

```dockerfile
VOLUME ["/opt/project"]
```

该指令会为基于此镜像创建的任何容器创建一个名为`/opt/project`的挂载点

### ADD

用来将构建环境下的文件和目录复制到镜像中

```dockerfile
ADD software.lic /opt/applition/software.lic
```

该指令将会将构建目录下的`software.lic`文件复制到镜像中的`/opt/applition/software.lic`,如果目标地址或目的地址以`/`结尾，那么docker将会认为是一个目录，否则是文件。

```dockerfile
ADD latest.tar.gz /var/www/wordpress
```

该命令会将归档文件`latest.tar.gz`解开到`/var/www/wordpress`目录下

### COPY

非常类似与`ADD`，它们的根本不同是`COPY`只关心在构建上下文中复制本地文件，而不会去做文件提取和解压

如果目的地址不存在，docker会自动创建所有需要的目录结构

### ONBUILD

为镜像添加触发器，当一个镜像被用作其他镜像的基础镜像时，该镜像中的触发器将会被执行，并且只会在子镜像中执行，不会在孙子镜像中执行

## 将镜像推送到Docker Hub

```bash
docker push username/imagename
```

将会将镜像推送到到docker hub

## 参考文章

- [Docker 中 latest 标签引发的困惑](http://www.linuxidc.com/Linux/2015-01/112548.htm)