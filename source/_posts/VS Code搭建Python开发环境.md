---
title: VS Code搭建Python开发环境
date: 2016-06-11 10:18:39
categories:
- Python
tags:
- Python
- VS Code
---

## 编辑器感想

贴吧总是看到

> 高手都是用记事本写代码的

我只想表示：那你慢慢用记事本吧，我用`VS Code`。

作为码农，基本每天都要敲代码，这就是命吧。一般较大的项目都会用`IDE`进行开发：

- Win32汇编：RadASM 2.0，RadASM 3.0，WinAsm
- C/C++：VC++6.0，Dev-c++，CodeBlocks，VS2015
- Java：Eclipse，MyEclipse，IntelliJ IDEA
- Android：Eclipse ADT，Android Studio
- Python：PyCharm

以上基本是我用过的全部`IDE`，对于开发周期较长的项目，用`IDE`开发确实比较方便，尤其配合`github`使用，例如我最喜欢的`PyCharm`，`Android Studio`，`IntelliJ IDEA`这一系列具有方便`github`功能的`IDE`，还有号称世界上最强大的IDE：`VS`

但是我只想说，电脑配置不够，占用内存太高，用`AS`编译一个`android`项目都要几分钟。所以一般对于小`demo`，或者小的项目，找一个比较实用的编辑器能提供很多帮助。

<!-- more -->

下面说下常用的几个编辑器：

- Vim：linux环境下的神器，搭个完整功能的IDE出来妥妥的。
- Emacs：我是操作系统，刷微博什么的都不是事。
- Atom：拥有众多插件，非常多，提供很多便捷，markdown的代码块的预览都能显示代码高亮，但是占用内存太大是硬伤。用这个进行开发基本可以不使用大型IDE了。
- Sublime Text：现在最新版本是3系列，不过我看到很多人还用2系列进行开发，也有很多插件，不过要钱，得自己找破解版。这个也是我用过时间最长的编辑器。
- VS Code：在我接触VS Code之后，基本就没再使用Sublime Text了，原因就不解释。体验过才知道VS Code有多强大。毕竟出自微软。

知乎上有个`Atom`、`Sublime Text`、`VSCode` 三者的比较：

[2016年看，Atom、Sublime Text、VSCode 三者比较，各有哪些优势和弱势？](https://www.zhihu.com/question/41857899)

`VS Code`刚开源的时候还看过一篇文章，讲的是`VS Code`如何血虐`Sublime Text`，找不到链接了。

## VS Code简单介绍

微软开源项目，一个极为强大的第三方编辑器，目前最新版本为1.2

别的不多少，晒下界面：

![](http://i.imgur.com/mgejfhd.png)

`VS Code`的分屏，`git`，调试，常用插件以及基本配置和使用就不说了，附上官网链接：

[https://code.visualstudio.com](https://code.visualstudio.com)

## 配置Python开发环境

简介的`Python`搭配简洁的`VS Code`，真是绝配，比`Python`搭配`Sublime Text`方便多了。

`VS Code`是有工作空间的概念的，用`VS Code`打开一个文件夹，该文件夹就可以算是一个工作空间，`VS Code`会在该文件下新建`.vscode`文件夹，里面存放的是该工作空间的配置文件，所以以后在该工作空间下的任何代码都会使用该配置文件。

首先我们需要下载`Python`，并将`Python`加到环境变量`Path`中。

然后打开`VS Code`，安装`python`插件，具体怎么安装，看下面链接：

[VS Code Python插件](https://marketplace.visualstudio.com/items?itemName=donjayamanne.python)

该插件提供`python`调试，`Lint`，以及代码补全等功能，具体看上面的链接

话说插件的更新不会删除老版本，希望`VS Code`后期能够改进吧：

![](http://i.imgur.com/4lpO5Wj.png)

### 配置Python运行环境

`VS Code`有`task`的概念，具体看官网，在`.vscode`文件夹下有三个文件：

![](http://i.imgur.com/qfBXugz.png)

其中`tasks.json`就是配置运行环境的，`settings.json`下的配置能够覆盖`VS Code`的默认配置，`launch.json`配置调试环境。

下面是`tasks.json`的配置

``` json
{
    "version": "0.1.0",
    "command": "python",
    "isShellCommand": true,
    "args": ["${file}"],
    "showOutput": "always"
}
```

参数都很好理解，其中`command`是`python`可执行文件的路径，如果配置到了`path`中，就可以直接写`python`，`args`就是运行的参数，也就是当前打开的文件名。

配置好运行环境后，按下`ctrl + shift + B`就可以执行了

![](http://i.imgur.com/L4ZKdUa.png)

### 配置Python调试环境

`launch.json`的配置如下：

``` json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python",
            "type": "python",
            "request": "launch",
            "stopOnEntry": true,
            "program": "${file}",
            "debugOptions": [
                "WaitOnAbnormalExit",
                "WaitOnNormalExit",
                "RedirectOutput"
            ]
        },
        {
            "name": "Python Console App",
            "type": "python",
            "request": "launch",
            "stopOnEntry": true,
            "program": "${file}",
            "externalConsole": true,
            "debugOptions": [
                "WaitOnAbnormalExit",
                "WaitOnNormalExit"
            ]
        },
        {
            "name": "Django",
            "type": "python",
            "request": "launch",
            "stopOnEntry": true,
            "program": "${workspaceRoot}/manage.py",
            "args": [
                "runserver",
                "--noreload"
            ],
            "debugOptions": [
                "WaitOnAbnormalExit",
                "WaitOnNormalExit",
                "RedirectOutput",
                "DjangoDebugging"
            ]
        },
        {
            "name": "Watson",
            "type": "python",
            "request": "launch",
            "stopOnEntry": true,
            "program": "${workspaceRoot}/console.py",
            "args": [
                "dev",
                "runserver",
                "--noreload=True"
            ],
            "debugOptions": [
                "WaitOnAbnormalExit",
                "WaitOnNormalExit",
                "RedirectOutput"
            ]
        }
    ]
}
```

如果`python`可执行文件不在`path`中，就需要添加`pythonPath`属性配置，如：

	"pythonPath": "D:\\Python 3.5\\python.exe"

点左边蜘蛛(调试)按钮，设置好断点，就可以开开心心调试了，附上截图：

![](http://i.imgur.com/EMelrIc.png)

有没有感受到 `VS Code`加`Python`插件组合的强大呢？
