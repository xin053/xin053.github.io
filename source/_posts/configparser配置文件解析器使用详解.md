---
title: configparser配置文件解析器使用详解
date: 2016-07-18 14:30:59
categories: 
- Python模块学习
tags:
- Python
- configparser
---

## configparser简介
python2下该模块名为ConfigParser，到3才改为configparser,可以看官方ConfigParser模块的说明

https://docs.python.org/2/library/configparser.html

本文介绍python3中configparser模块的使用，configparser模块是用来解析ini配置文件的解析器，关于ini配置文件的结构可以看python官方文档中的介绍：

[ini文件结构](https://docs.python.org/3.5/library/configparser.html#supported-ini-file-structure)

ini文件结构需要注意一下几点：

- 键值对可用`=`或者`:`进行分隔
- `section`的名字是区分大小写的,而`key`的名字是不区分大小写的
- 键值对中头部和尾部的空白符会被去掉
- 值可以为多行
- 配置文件可以包含注释，注释以`#`或者`;`为前缀

***注意：configparser有default_section的概念,默认为`[DEFAULT]`节,也就是之后的所有的section都有该默认section中的键值对,详情参见[configparser源码的`__init__()`方法](https://hg.python.org/cpython/file/3.5/Lib/configparser.py)***

<!-- more -->

## 基本使用

为了创建如下ini文件：
configparser模块主要使用ConfigParser类来解析ini文件

```ini
[DEFAULT]
ServerAliveInterval = 45
Compression = yes
CompressionLevel = 9
ForwardX11 = yes

[bitbucket.org]
User = hg

[topsecret.server.com]
Port = 50022
ForwardX11 = no
```

我们可以使用如下代码：

```python
>>> import configparser
>>> config = configparser.ConfigParser()
>>> config['DEFAULT'] = {'ServerAliveInterval': '45',
...                      'Compression': 'yes',
...                      'CompressionLevel': '9'}
>>> config['bitbucket.org'] = {}
>>> config['bitbucket.org']['User'] = 'hg'
>>> config['topsecret.server.com'] = {}
>>> topsecret = config['topsecret.server.com']
>>> topsecret['Port'] = '50022'     # mutates the parser
>>> topsecret['ForwardX11'] = 'no'  # same here
>>> config['DEFAULT']['ForwardX11'] = 'yes'
>>> with open('example.ini', 'w') as configfile:
...   config.write(configfile)
```

然后我们再读取该ini文件：

```python
>>> import configparser
>>> config = configparser.ConfigParser()
>>> config.sections()
[]
>>> config.read('example.ini')
['example.ini']
>>> config.sections()
['bitbucket.org', 'topsecret.server.com']
>>> 'bitbucket.org' in config
True
>>> 'bytebong.com' in config
False
>>> config['bitbucket.org']['User']
'hg'
>>> config['DEFAULT']['Compression']
'yes'
>>> topsecret = config['topsecret.server.com']
>>> topsecret['ForwardX11']
'no'
>>> topsecret['Port']
'50022'
>>> for key in config['bitbucket.org']: print(key)
...
user
compressionlevel
serveraliveinterval
compression
forwardx11
>>> config['bitbucket.org']['ForwardX11']
'yes'
```

**The only bit of magic involves the DEFAULT section which provides default values for all other sections. Note also that keys in sections are case-insensitive and stored in lowercase**

除了可以使用列表的方式获取值，也可以通过`section`级别的`get()`方法获取，同时该函数可以指定默认值

```python
>>> topsecret.get('Port')
'50022'
>>> topsecret.get('CompressionLevel')
'9'
>>> topsecret.get('Cipher', '3des-cbc')
'3des-cbc'
```

而解析器级别的`get()`函数的默认值是通过`fallback`参数指定的：

```python
>>> config.get('bitbucket.org', 'monster',
...            fallback='No such things as monsters')
'No such things as monsters'
```

需要注意的是，无论是通过列表方式获取值，还是通过`get()`方法获取值，获取到的数据都字符串类型，如果想要获取指定类型的数据，可以使用如下的几个方法:

- getint()
- getfloat()
- getboolean()

同时需要注意`getboolean()`方法能判断True/False的值有： 'yes'/'no', 'on'/'off', 'true'/'false' 和 '1'/'0' 

## Interpolation
创建ConfigParser()类的时候可以指定interpolation参数，如果将interpolation设置为`BasicInterpolation()`，则配置文件中的`%(key)s`结构会被解析，如，比如`example.ini`文件内容如下：

```ini
[Paths]
home_dir: /Users
my_dir: %(home_dir)s/lumberjack
my_pictures: %(my_dir)s/Pictures
```

```python
>>> import configparser
>>> config = configparser.ConfigParser(interpolation=configparser.BasicInterpolation())
>>> config.read(r'F:\coding\python\example.ini')
['F:\\coding\\python\\example.ini']
>>> config['Paths']['my_dir']
'/Users/lumberjack'
```

可以看到`%(home_dir)s`被解析成了`/Users`，说白了，相当于配置文件中的变量

创建ConfigParser()类的时候如果没有指定interpolation参数，则不会解析`%(key)s`，只会返回字符串而已

```python
>>> config['Paths']['my_dir']
'%(home_dir)s/lumberjack'
```

当然Interpolation还有更高级的使用方法，创建ConfigParser()类的时候指定interpolation参数为`ExtendedInterpolation()`，那么解析器会解析`${section:key}`结构，那么上面的ini文件应该写成如下的格式:

```ini
[Paths]
home_dir: /Users
my_dir: ${home_dir}/lumberjack
my_pictures: ${my_dir}/Pictures
```

并且`ExtendedInterpolation()`也能解析更复杂的，像下面这样的ini文件:

```ini
[Common]
home_dir: /Users
library_dir: /Library
system_dir: /System
macports_dir: /opt/local

[Frameworks]
Python: 3.2
path: ${Common:system_dir}/Library/Frameworks/

[Arthur]
nickname: Two Sheds
last_name: Jackson
my_dir: ${Common:home_dir}/twosheds
my_pictures: ${my_dir}/Pictures
python_dir: ${Frameworks:path}/Python/Versions/${Frameworks:Python}
```

## ConfigParser
ConfigParser对象的其他方法，如：

- add_section(section)
- has_section(section)
- options(section)
- has_option(section, option)
- remove_option(section, option)
- remove_section(section)

都很常用，具体就不介绍了，看名字就知道是干什么的了

## 相关文档

- [configparser官方文档](https://docs.python.org/3.5/library/configparser.html)
- [configparser源代码](https://hg.python.org/cpython/file/3.5/Lib/configparser.py)