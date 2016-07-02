---
title: pip包管理器使用详解
date: 2016-07-02 16:10:15
categories: 
- Python模块学习
tags:
- Python
- pip
---

## pip简介
pip为包管理器，跟linux上众多的包管理器的功能大致相同，就是对包进行管理，使得包的安装，更新和卸载更容易。

pip包管理器下载的python库来自：[PyPI](https://pypi.python.org/pypi/)

pip在python2.7.9以上和3.4以上自带，通过venv,virtualenv和pyvenv创建的虚拟环境默认也会安装，不过最好通过以下命令更新到最新版
Windows:

```bash
python -m pip install -U pip
```

Linux或Mac

```bash
pip install -U pip
```

<!-- more -->

## pip使用
### 包的安装

```bash
pip install PackageName                # latest version
pip install PackageName==1.0.4         # specific version
```

例如，如果我们安装requests包，那么输入命令：

```bash
pip install requests
```

### 显示包文件

```bash
pip show --files PackageName
```

该命令能够显示包的简介，包安装的位置，以及整个包包含的文件

### 显示过期的包

```bash
pip list --outdated
```

该命令列出所有可以更新的包，而

```bash
pip list
```

可以列出当前python环境下安装的所有包

### 包的更新

```bash
pip install -U PackageName
```

该命令将指定包更新到仓库最新版

### 包的卸载

```bash
pip uninstall PackageName
```

## pip help

```bash
Usage:
  pip <command> [options]

Commands:
  install                     Install packages.
  download                    Download packages.
  uninstall                   Uninstall packages.
  freeze                      Output installed packages in requirements format.
  list                        List installed packages.
  show                        Show information about installed packages.
  search                      Search PyPI for packages.
  wheel                       Build wheels from your requirements.
  hash                        Compute hashes of package archives.
  completion                  A helper command used for command completion
  help                        Show help for commands.

General Options:
  -h, --help                  Show help.
  --isolated                  Run pip in an isolated mode, ignoring
                              environment variables and user configuration.
  -v, --verbose               Give more output. Option is additive, and can be
                              used up to 3 times.
  -V, --version               Show version and exit.
  -q, --quiet                 Give less output.
  --log <path>                Path to a verbose appending log.
  --proxy <proxy>             Specify a proxy in the form
                              [user:passwd@]proxy.server:port.
  --retries <retries>         Maximum number of retries each connection should
                              attempt (default 5 times).
  --timeout <sec>             Set the socket timeout (default 15 seconds).
  --exists-action <action>    Default action when a path already exists:
                              (s)witch, (i)gnore, (w)ipe, (b)ackup.
  --trusted-host <hostname>   Mark this host as trusted, even though it does
                              not have valid or any HTTPS.
  --cert <path>               Path to alternate CA bundle.
  --client-cert <path>        Path to SSL client certificate, a single file
                              containing the private key and the certificate
                              in PEM format.
  --cache-dir <dir>           Store the cache data in <dir>.
  --no-cache-dir              Disable the cache.
  --disable-pip-version-check
                              Don't periodically check PyPI to determine
                              whether a new version of pip is available for
                              download. Implied with --no-index.
```