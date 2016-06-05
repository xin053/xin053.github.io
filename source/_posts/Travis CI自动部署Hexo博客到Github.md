layout: travis
title: Travis CI自动部署Hexo博客到Github
date: 2016-06-05 20:37:10
categories: 
- Hexo
tags:
- Hexo
- Travis
---

## 使用Travis CI自动部署你的Hexo博客到Github上
### 问题由来
一般通过`Hexo`命令行三部曲：

	- hexo clean
	- hexo g(enerate)
	- hexo d(eploy)
就可以清除项目缓存，生成静态网页，然后`push`静态网页到`Github`，然后访问`https://GithubID.github.io`就可以访问更新后的`blog`了。

通过另外三部曲：

	- hexo clean
	- hexo g(enerate)
	- hexo s(erver)
即可开启本地服务器，通过浏览器访问`http://localhost:4000`便可以看到更新后的`blog`。

<!-- more -->

**但是：** 由于上传到`github`上的只是静态网页文件，所以只能在具有整个网站项目源代码的电脑上进行写作，与文章同步与更新，而且每次都要执行`hexo`三部曲，不符合使用`git`的习惯。

### 解决方法
通过`Travis CI`自动部署，自动发布到`github`上，配置一下，便可实现在每次`git push`之后自动执行一个脚本，执行生成并发布等命令。相当于不在手动执行`hexo`三部曲，只用修改后，按照以往的`git`操作：

	add->commit->push

便可自动将源代码`push`到`github`,然后由`travis`自动`clone`项目，并由`travis`的虚拟机自动执行`hexo`三部曲,从而`deploy`更新后的网站到`github`

### 技巧
在本地新建`dev`分支，并在远程新建`dev`分支，每次本地修改提交到远程`dev`分支(本地新建`dev`分支只是为了好操作，不新建`dev`分支也可以,也就是本地`master`提交到远程`dev`)，`travis`配置文件`.vravis.yml`中监视远程dev分支的改变，`build`成功后添加动作：`hexo deploy`，其中`_config.yml`中`deploy`的分支为远程`master`，这样实现的效果是：

**当`git push dev`时，远程仓库的`dev`是整个项目源代码，而`travis`检测到`dev`分支改变，便会`clone dev`分支，并`build`，然后`deploy`到远程`master`分支，所以远程仓库`master`是整个`blog`的静态网页集合，于是实现了仓库中既有`blog`项目源代码，又有构建成功的整个项目，并且处在不同分支上。**

### 具体配置
参考以下文章：

- [使用Travis自动部署Hexo(1)](http://www.jianshu.com/p/7f05b452fd3a "使用Travis自动部署Hexo(1)")
- [使用Travis自动部署Hexo(2)](http://www.jianshu.com/p/fff7b3384f46 "使用Travis自动部署Hexo(2)")
- [Hexo 自动部署到 Github](http://www.tuicool.com/articles/AZf2Yzb "Hexo 自动部署到 Github")
- [手把手教你使用Travis CI自动部署你的Hexo博客到Github上](http://www.2cto.com/kf/201605/505702.html "手把手教你使用Travis CI自动部署你的Hexo博客到Github上")

### 配置过程中遇到的问题
按照上面的第一篇配置，由于我是`windows`环境，所以按照上面的配置出现了错误，和博主下面说明的情况一致，没有解决。

于是尝试第二篇中的做法，结合第三篇文章中的配置说明，其中

    # 这里的 REPO_TOKEN 是变量名,在后面的配置文件中会用到
    # TOKEN 是上面github生成的Token
    travis encrypt 'REPO_TOKEN=<TOKEN>' --add

在`cmd`中运行上面的命令报错：

	系统找不到指定文件

没有解决，最终查到`travis`使用方法，用下面指令代替：

    travis encrypt -r pwmckenna/node-travis-encrypt GH_TOKEN=0efdabf1c44122b90db5****** --add

其实也就是将`GH_TOKEN=TOKEN`这个环境变量加密,以便在`.travis.yml`能够使用`GH_TOKEN`访问到`TOKEN`

附上`.travis.yml`配置：

    language: node_js

    node_js: stable

    branches:
      only:
      - dev      # 监视github仓库中的dev分支，分支出现变化就执行build

    before_install:
    - export TZ='Asia/Shanghai'
    - npm install -g hexo
    - npm install -g hexo-cli

    install:
    - npm install

    script:
    - git submodule init      # 用于更新主题
    - git submodule update
    - hexo clean && hexo g

    env:
      global:
        secure: ***HcNd3E02rGSZ3mzYHmNwsTKBIXa8cdOHIqmnuV8P9xAFRaDRUEdE8gqFmNCeKd5hriM64sO5BGU/szI7Q2uJNhUgDg0Rw/UZMbZCei5Pf112qzDpbb/ok1PUU9Q282sI1YVf8poBUvMHmoHLOMayR25IjIysb5aE+8kpipDbReU=

    after_success:
    - git config --global user.name "xin053"
    - git config --global user.email "13207130066.cool@163.com"
    - sed -i'' "s~git@github.com:xin053/xin053.github.io.git~https://${GH_TOKEN}:x-oauth-basic@github.com/xin053/xin053.github.io.git~" _config.yml
    - hexo deploy

`_config.yml`的`deploy`配置如下：

    deploy:
      type: git
      repository: git@github.com:xin053/xin053.github.io.git
      branch: master      # 将build后的静态网页发布到github仓库master分支

其中还要注意`TOKEN`的权限，`github`默认生成的`TOKEN`权限很低，如果使用默认的，`build`过程中虽然`travis`显示构建成功，但是`log`中显示`hexo deploy`失败，原因是权限问题，所以建议设置权限如下，即全部勾选。

![](http://i.imgur.com/HaFgkwf.png)

`travis`的`log`中最后显示如下图所示时，说明已经部署成功了：

![](http://i.imgur.com/47CPLk0.png)

而最后一篇文章则是通过`travis`网站来设置环境变量，更方便，也更快捷

![](http://i.imgur.com/MoGZ0UC.png)

在网站中设置了环境变量之后，就可以去掉`.travis.yml`中的`env`项了。

设置好后，只用在网站项目源码根目录下执行：

    git init

初始化git,仅第一次需要，然后

    git add .
    git commit -am "update"

第一次提交到远程仓库要先添加远程仓库：

    git remote add master git@github.com:xin053/xin053.github.io.git

将本地`dev`与远程默认分支进行关联

    git push -f --set-upstream dev dev

如果原理`github`下是原来的静态网站，者这里用`-f`强推，覆盖掉原来的项目就行。

之后每次就只用

    git push dev

也就是每次写完博客之后，在网站项目源代码根目录执行三部曲：

    git add .
    git commit -am "update"
    git push dev

就可以提交最新源代码到`github`的`dev`分支,同时`travis`会自动`build`，生成最新的静态网页`deploy`到`github`的`master`分支。

`build`记录可以参见`https://travis-ci.org/`

![](http://i.imgur.com/yEmpXuC.png)