---
title: venv创建Python虚拟环境
date: 2016-06-30 22:31:13
categories: 
- Python模块学习
tags:
- Python
- venv
- virtual environments
---

## Python虚拟环境简介
众说周知，Python有很多版本，其中具有画时代意义的版本就是2.7和3，由于Python各版本之间不兼容，所以不像一般软件那样，直接安装便可以覆盖老版本，Python的不同大版本可以同时存在，例如电脑上可以同时存在Python 2.7，Python 3.4和Python 3.5，而Python的小版本(版本下的小更新)会覆盖之前的安装，比如目前电脑上是Python 3.5，那么安装Python 3.5.1便会覆盖3.5，之后再安装Python 3.5.2，便会覆盖Python 3.5.1，哦，对了，我指的是window环境下python安装包，非便携版。

Python极强的能力主要由其庞大的库来支撑，Python 2.7下有很多很多库，目前也有很多库已经兼容了Python 3，但是还是有大量的库只兼容Python 2.7.有人说Python 3会毁灭Python，但我并不认为是这样，Python 3带来了很多特性已经性能等方面的提升，Python 3必定是一种趋势，要想华丽的蜕变，就必须选择舍弃，所以Python官网发布了Python 3.

<!-- more -->

现在说下为什么需要Python虚拟运行环境，首先Python安装的大部分第三方包都是装在Python安装目录下的Lib下的site-packages目录下的，而如果你想同时在2.7和3下进行开发，就需要同时在各自的site-packages目录下各安装包，而如果以后出来了Python 3.6，又需要在其site-packages下安装常用的第三方包，简直复杂，梦魇，然后有些项目需要2.7，而有的项目需要3，如果运行这两个项目，就需要切换PATH，而如果某个大项目同时用到2.7和3，就没办法了，而通过创建Python虚拟环境之后，相当于有一个目录，这个目录中有site-packages目录，其他目录和一些脚本，如激活虚拟环境的脚本，同时该目录(虚拟环境)下有特定版本的Python解释器，所以可以创建一个虚拟环境，然后把所有的包都装在这个下面，以后Python更新就不用再去下第三方包了，只用update虚拟环境中的Python解释器就行了。而对于使用不同Python版本的项目，可以创建一个2.7的虚拟环境和一个3的虚拟环境，然后项目分别和这两个虚拟环境进行绑定，运行就没有问题了，总的来说，虚拟环境是一个与系统Python环境相隔离的Python环境。

## venv模块简介
Python3.3以上的版本通过venv模块原生支持虚拟环境，可以代替Python之前的virtualenv。

该venv模块提供了创建轻量级“虚拟环境”，提供与系统Python的隔离支持。每一个虚拟环境都有其自己的Python二进制（允许有不同的Python版本创作环境），并且可以拥有自己独立的一套Python包。

需要注意的是，在Python3.3中使用”venv”命令创建的环境不包含”pip”，你需要进行手动安装。在Python3.4中改进了这一个缺陷。

官方文档：[venv — Creation of virtual environments](https://docs.python.org/3/library/venv.html)

## venv创建虚拟环境
首先选择一个目录，这个目录就是虚拟环境，也就是虚拟环境中的Python解释器，第三方包都会在这个目录。在我的电脑上我选择的是：

	F:\pythonVE

打开cmd后cd切换到该目录，然后执行：

```bash
F:\pythonVE>python -m venv .
```

表示在当前目录下创建虚拟环境，对了我PATH里Python为3.5.2

执行完毕后，看下该目录下的内容：

![](http://i.imgur.com/nEoobC9.png)

看下`pyvenv.cfg`文件内容：

```bash
home = D:\Python 3.5
include-system-site-packages = false
version = 3.5.2
```

很明了，就不解释了。

Lib下的 site-packages 用来存放第三方包，Scripts存在一些可用的脚本。

下面是”venv”的详细使用参数:

```bash
usage: venv [-h] [--system-site-packages] [--symlinks | --copies] [--clear]
            [--upgrade] [--without-pip]
            ENV_DIR [ENV_DIR ...]

Creates virtual Python environments in one or more target directories.

positional arguments:
  ENV_DIR             A directory to create the environment in.

optional arguments:
  -h, --help             show this help message and exit
  --system-site-packages Give the virtual environment access to the system
                         site-packages dir.
  --symlinks             Try to use symlinks rather than copies, when symlinks
                         are not the default for the platform.
  --copies               Try to use copies rather than symlinks, even when
                         symlinks are the default for the platform.
  --clear                Delete the contents of the environment directory if it
                         already exists, before environment creation.
  --upgrade              Upgrade the environment directory to use this version
                         of Python, assuming Python has been upgraded in-place.
  --without-pip          Skips installing or upgrading pip in the virtual
                         environment (pip is bootstrapped by default)
```

创建虚拟环境后，如果想使用该虚拟环境，需要先激活该虚拟环境。

到虚拟目录下的Scripts目录执行activate.bat

```bash
F:\pythonVE\Scripts>activate.bat
```

会发现cmd变成这个样子了:

![](http://i.imgur.com/f9oziA1.png)

可以发现目前已经在这个虚拟环境下了

现在来测试下，首先我的pip版本不是最新，更到最新再说：

```bash
python -m pip install --upgrade pip
```

然后可以看到`F:\pythonVE\Lib\site-packages`目录下的pip确实更新到最新版了。

然后先安装著名的`requests`模块看看:

```bash
pip install requests
```

同样在`F:\pythonVE\Lib\site-packages`目录下确实看到了requests目录。

值得一提的是pycharm新建项目的时候可以选择python环境，其中就可以选择虚拟环境，其他工具应该也可以。

继续测试：

```bash
(pythonVE) F:\pythonVE\Scripts>python
Python 3.5.2 (v3.5.2:4def2a2901a5, Jun 25 2016, 22:18:55) [MSC v.1900 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import requests
>>> print(requests)
<module 'requests' from 'F:\\pythonVE\\lib\\site-packages\\requests\\__init__.py'>
>>>
```

发现使用的`requests`模块确实来自虚拟环境

不过项目开发时，要确保自己用的什么环境哦

## 参考文章

- http://blog.csdn.net/geekun/article/details/51325383

