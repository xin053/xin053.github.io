---
title: os库常用方法使用介绍
date: 2016-11-29 09:27:27
categories: 
- Python模块学习
tags:
- Python
- os
---

## os简介
与系统相依赖的一些操作，有些操作只支持unix系统

## os常用方法

### environ与getenv

获取环境变量

```python
import os
os.environ["PYTHON_HOME"]
# 'F:\\pythonVE'
os.getenv('PYTHON_HOME')
# 'F:\\pythonVE'
```

<!-- more -->

### 用户与用户组

获取当前进程或者指定pid进程的用户和用户组，仅支持unix，详情见[`os`](https://docs.python.org/3/library/os.html#os.getegid)

其中windows平台也可以使用的:

获取当前登陆用户:

```python
os.getlogin() 
# 'zzx'
```

### chdir与getcwd

改变与获取当前工作路径

```python
os.getcwd()
# 'F:\\pythonVE\\Scripts'
os.chdir('..')
os.getcwd()
# 'F:\\pythonVE'
```

### listdir与scandir

枚举指定目录,不指定`path`参数则默认当前路径

```python
os.listdir()
# ['Include', 'Lib', 'pip-selfcheck.json', 'pyvenv.cfg', 'Scripts', 'share']
os.listdir('.')
# ['Include', 'Lib', 'pip-selfcheck.json', 'pyvenv.cfg', 'Scripts', 'share']
```

`scandir()`与`listdir()`作用相同，但是返回的是迭代器

```python
a = os.scandir()
a
# <nt.ScandirIterator at 0x187d4cc5440>
a.__next__()
# <DirEntry 'Include'>
a.__next__()
# <DirEntry 'Lib'>
```

而`DirEntry`对象包含了与文件相关的属性，详情见:[`os.DirEntry`](https://docs.python.org/3/library/os.html#os.DirEntry)

### 文件系统相关

- `mkdir()` 创建目录
- `remove()` 删除文件
- `rmdir()` 删除目录
- `rename()` 重命名

### stat

文件相关信息

```python
>>> import os
>>> statinfo = os.stat('somefile.txt')
>>> statinfo
os.stat_result(st_mode=33188, st_ino=7876932, st_dev=234881026,
st_nlink=1, st_uid=501, st_gid=501, st_size=264, st_atime=1297230295,
st_mtime=1297230027, st_ctime=1297230027)
>>> statinfo.st_size
264
```

### startfile

使用电脑上默认应用打开指定文件

### 分隔符 换行符相关

```python
>>> os.curdir
'.'
>>> os.pardir
'..'
>>> os.sep
'\\'
>>> os.altsep
'/'
>>> os.extsep
'.'
>>> os.pathsep
';'
>>> os.defpath
'.;C:\\bin'
>>> os.linesep
'\r\n'
```

### 参考文档

- [`os`官方文档](https://docs.python.org/3/library/os.html)

