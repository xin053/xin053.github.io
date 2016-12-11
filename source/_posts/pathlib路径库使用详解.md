---
title: pathlib路径库使用详解
date: 2016-07-03 17:41:44
categories:
- Python模块学习
tags:
- Python
- pathlib
---

## pathlib简介
pathlib库在python 3.4以后已经成为标准库，基本上可以代替`os.path`来处理路径。它采用完全面对对象的编程方式。

总共有6个类用来处理路径，大体可以分为两类：

1. pure paths 单纯的路径计算操作而没有IO功能
2. concrete paths 路经计算操作和IO功能

这6个类的继承关系如下：

![](http://pathlib.readthedocs.io/en/pep428/_images/pathlib-inheritance.png)

可以看到`PurePath`是所有类的基类,**我们重点要掌握`PurePath`和`Path`这两个类,在Windows平台下路径对象会有Windows前缀,Unix平台上路径对象会有Posix前缀**。

<!-- more -->

## 基本使用
### 列出所有子目录

```python
>>> import pathlib
>>> p = pathlib.Path('.')
>>> [x for x in p.iterdir() if x.is_dir()]
[WindowsPath('.git'), WindowsPath('.idea'), WindowsPath('.vscode'), 
WindowsPath('1_函数参数'), WindowsPath('2_生成器'), WindowsPath('3_常用函数'), 
WindowsPath('4_装饰器), WindowsPath('5_常用模块')]
# 在linux环境下，上述的WindowsPath都会变为PosixPath
```

### 列出指定类型的文件

```python
list(p.glob('**/*.py'))
```

### 路径拼接
可以使用`/`符号来拼接路径

```python
>>> p = pathlib.Path(r'F:\cookies\python')
>>> q = p / 'learnPython'
>>> print(q)
F:\cookies\python\learnPython
```

### 查询属性

```python
>>> q.exists()
True
>>> q.is_dir()
True
```

### 打开文件

```python
>>> q = q / "hello_world.py"
>>> with q.open() as f:
>>> print(f.readline())
#!/usr/bin/env python
```

## Pure paths
### 产生Pure paths的三种方式

> class pathlib.PurePath(*pathsegments)

```python
>>> PurePath('setup.py')
PurePosixPath('setup.py')     # Running on a Unix machine
PureWindowsPath('setup.py')   # Running on a Windows machine
```

```python
>>> PurePath('foo', 'some/path', 'bar')
PureWindowsPath('foo/some/path/bar')
>>> PurePath(Path('foo'), Path('bar'))
PureWindowsPath('foo/bar')
```

如果参数为空，则默认指定当前文件夹

```python
>>> PurePath()
PureWindowsPath('.')
```

当同时指定多个绝对路径,则使用最后一个

```python
>>> PureWindowsPath('c:/Windows', 'd:bar')
PureWindowsPath('d:bar')
```

在Windows平台上，参数路径上如果有`\`或者`/`,则使用之前设置的盘符

```python
>>> PureWindowsPath('F:\cookies\python\learnPython','\game')
PureWindowsPath('F:/game')
```

> class pathlib.PurePosixPath(*pathsegments)

```python
>>> PurePosixPath('/etc')
PurePosixPath('/etc')
```

> class pathlib.PureWindowsPath(*pathsegments)

```python
>>> PureWindowsPath('c:/Program Files/')
PureWindowsPath('c:/Program Files')
```

### Path计算

```python
>>> PurePosixPath('foo') == PurePosixPath('FOO')
False
>>> PureWindowsPath('foo') == PureWindowsPath('FOO')
True
>>> PureWindowsPath('FOO') in { PureWindowsPath('foo') }
True
>>> PureWindowsPath('C:') < PureWindowsPath('d:')
True
>>> PureWindowsPath('foo') == PurePosixPath('foo')
False
>>> PureWindowsPath('foo') < PurePosixPath('foo')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unorderable types: PureWindowsPath() < PurePosixPath()
```

### str() 和 bytes()

```python
>>> p = PurePath('/etc')
>>> str(p)
'/etc'
>>> p = PureWindowsPath('c:/Program Files')
>>> str(p)
'c:\\Program Files'
>>> bytes(p)
b'/etc'
```

### 常用属性和方法

```python
>>> PureWindowsPath('c:/Program Files/').drive
'c:'
>>> PurePosixPath('/etc').root
'/'

>>> p = PureWindowsPath('c:/foo/bar/setup.py')
>>> p.parents[0]
PureWindowsPath('c:/foo/bar')
>>> p.parents[1]
PureWindowsPath('c:/foo')
>>> p.parents[2]
PureWindowsPath('c:/')

>>> PureWindowsPath('//some/share/setup.py').name
'setup.py'
>>> PureWindowsPath('//some/share').name
''

>>> PurePosixPath('my/library/setup.py').suffix
'.py'
>>> PurePosixPath('my/library.tar.gz').suffix
'.gz'
>>> PurePosixPath('my/library').suffix
''

>>> PurePosixPath('my/library.tar.gar').suffixes
['.tar', '.gar']
>>> PurePosixPath('my/library.tar.gz').suffixes
['.tar', '.gz']
>>> PurePosixPath('my/library').suffixes
[]

>>> PurePosixPath('my/library.tar.gz').stem
'library.tar'
>>> PurePosixPath('my/library.tar').stem
'library'
>>> PurePosixPath('my/library').stem
'library'

>>> p = PureWindowsPath('c:\\windows')
>>> str(p)
'c:\\windows'
>>> p.as_posix()
'c:/windows'

>>> p = PurePosixPath('/etc/passwd')
>>> p.as_uri()
'file:///etc/passwd'
>>> p = PureWindowsPath('c:/Windows')
>>> p.as_uri()
'file:///c:/Windows'

>>> PurePath('a/b.py').match('*.py')
True
>>> PurePath('/a/b/c.py').match('b/*.py')
True
>>> PurePath('/a/b/c.py').match('a/*.py')
False

>>> p = PurePosixPath('/etc/passwd')
>>> p.relative_to('/')
PurePosixPath('etc/passwd')
>>> p.relative_to('/etc')
PurePosixPath('passwd')
>>> p.relative_to('/usr')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "pathlib.py", line 694, in relative_to
    .format(str(self), str(formatted)))
ValueError: '/etc/passwd' does not start with '/usr'

>>> p = PureWindowsPath('c:/Downloads/pathlib.tar.gz')
>>> p.with_name('setup.py')
PureWindowsPath('c:/Downloads/setup.py')
>>> p = PureWindowsPath('c:/')
>>> p.with_name('setup.py')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/antoine/cpython/default/Lib/pathlib.py", line 751, in with_name
    raise ValueError("%r has an empty name" % (self,))
ValueError: PureWindowsPath('c:/') has an empty name

>>> p = PureWindowsPath('c:/Downloads/pathlib.tar.gz')
>>> p.with_suffix('.bz2')
PureWindowsPath('c:/Downloads/pathlib.tar.bz2')
>>> p = PureWindowsPath('README')
>>> p.with_suffix('.txt')
PureWindowsPath('README.txt')
```

## Concrete paths
### 产生Concrete paths的三种方式

> class pathlib.Path(*pathsegments)

```python
>>> Path('setup.py')
PosixPath('setup.py')       # Running on a Unix machine
WindowsPath('setup.py')     # Running on a Windows machine
```

> class pathlib.PosixPath(*pathsegments)

```python
>>> PosixPath('/etc')
PosixPath('/etc')
```

> class pathlib.WindowsPath(*pathsegments)

```python
>>> WindowsPath('c:/Program Files/')
WindowsPath('c:/Program Files')
```

### 常用方法

`cwd()`设置path对象为当前路径

```python
>>> Path.cwd()
WindowsPath('D:/Python 3.5')
```

`stat()`获取文件或目录属性

```python
>>> p = Path('setup.py')
>>> p.stat().st_size
956
```

`chmod()`Unix系统修改文件或目录权限

`exists()`判断文件或目录是否存在

```python
>>> from pathlib import *
>>> Path('.').exists()
True
>>> Path('setup.py').exists()
True
>>> Path('/etc').exists()
True
>>> Path('nonexistentfile').exists()
False
```

`glob()`列举文件

```python
>>> sorted(Path('.').glob('*.py'))
[PosixPath('pathlib.py'), PosixPath('setup.py'), PosixPath('test_pathlib.py')]
>>> sorted(Path('.').glob('*/*.py'))
[PosixPath('docs/conf.py')]
>>> sorted(Path('.').glob('**/*.py'))
[PosixPath('build/lib/pathlib.py'),
 PosixPath('docs/conf.py'),
 PosixPath('pathlib.py'),
 PosixPath('setup.py'),
 PosixPath('test_pathlib.py')]
# The "**" pattern means "this directory and all subdirectories, recursively"
```

`is_dir()`判断是否是目录

`is_file()`判断是否是文件

`is_symlink()`判断是否是链接文件

`iterdir()`如果path指向一个目录，则返回该目录下所有内容的生成器

`mkdir(mode=0o777, parents=False)`创建目录

`open(mode='r', buffering=-1, encoding=None, errors=None, newline=None)`打开文件

`owner()`获取文件所有者

`rename(target)`修改名称

```python
>>> p = Path('foo')
>>> p.open('w').write('some text')
9
>>> target = Path('bar')
>>> p.rename(target)
>>> target.open().read()
'some text'
```

`resolve()`Make the path absolute, resolving any symlinks. A new path object is returned

```python
>>> p = Path()
>>> p
PosixPath('.')
>>> p.resolve()
PosixPath('/home/antoine/pathlib')
```

`rmdir()`删除目录，目录必须为空

`touch(mode=0o777, exist_ok=True)`创建空文件

## 其他相关文章

- [pathlib官方文档](http://pathlib.readthedocs.io/en/pep428/#concrete-paths)