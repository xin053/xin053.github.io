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

**但是：** 由于上传到`github`上的只是静态网页文件，所以只能在具有整个网站项目源代码的电脑上进行写作，与文章同步与更新，别人就算`fork`也只是些静态网页文件，基本上不能在其上进行修改变为自己的`blog`，相当于并没有开源。

### 解决方法
通过`Travis CI`自动部署，发布到`github`上的是整个网站项目的源代码，配置一下，便可实现在每次`git push`之后自动执行一个脚本，执行生成并发布等命令。

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
      - master

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
    - sed -i'' "/^ *repo/s~github\.com~${GH_TOKEN}@github.com~" _config.yml
    - hexo deploy

而最后一篇文章则是通过`travis`网站来设置环境变量，更方便，也更快捷

![](http://i.imgur.com/MoGZ0UC.png)

在网站中设置了环境变量之后，就可以去掉`.travis.yml`中的`env`项了。

设置好后，只用在网站项目源码根目录下执行：

    git init

初始化git,仅第一次需要，然后

    git add .
    git commit -am "update"

第一次提交到远程仓库要先建立连接：

    git remote add master git@github.com:xin053/xin053.github.io.git

将本地`master`与远程默认分支进行关联

    git push -f --set-upstream master master

如果原理`github`下是原来的静态网站，者这里用`-f`强推，覆盖掉原来的项目就行。

之后没吃就只用

    git push

也就是每次写完博客之后，在网站项目源代码根目录执行三部曲：

    git add .
    git commit -am "update"
    git push

就可以提交最新源代码到`github`,同时`travis`会自动`build`

`build`记录可以参见`https://travis-ci.org/`

![](http://i.imgur.com/yEmpXuC.png)