---
title: Python3.6更新内容
date: 2016-12-23 19:15:12
categories:
- Python
tags:
- Python
---

## Python3.6
北京时间2016年12月23日晚上6点半左右，[python官网](https://www.python.org/)放出了python3.6.0正式版，安装后，可以看到windows版具体编译时间是2016年12月23日早上8点6分。可以说python3.6从测试到正式发布已经有很长一段时间了，并且官方表示，2017年初开始对3.6版本进行各种bug修复等改进，也就是3.6.x的版本，关于python3.6相较于3.5有哪些变化，请看[What’s New In Python 3.6](https://docs.python.org/3.6/whatsnew/3.6.html)
本文主要讲解如何将工作环境从python3.5转到python3.6，以及python3.6新功能的介绍。

![](https://www.python.org/static/img/python-logo.png)

<!-- more -->

## 工作环境

由于python的每个版本，例如3.5和3.6安装时安装目录是分开的(windows环境)，而如果我们将python第三方库安装在python安装目录下的话，那么现在我如果使用3.6，又得重新将3.6的安装目录添加到环境变量`PATH`，并且将大量第三方库安装到3.6安装目录，但是这样就引发了一个问题，那就是多份第三方库都存在于电脑中，当然也可以删除3.5相关的所有文件，但是实际上重新安装常用的那些库又很麻烦，所以我将python虚拟环境当作我的工作环境，也就是在`F:\pythonVE`目录创建一个python虚拟环境，将第三方库都安装在这个虚拟环境中，所以现在刚刚安装好python3.6，只用在cmd执行:

```powershell
python -m venv --upgrade F:\pythonVE
```

注意这里的`python`是3.6中的`python.exe`,`--upgrade`参数的意思就是将虚拟环境中的python版本升级为此python版本(3.6版本)

所以`PAHT`中只用添加虚拟环境的路径就可以了，然后就是慢慢更新第三方包了，毕竟第三方包适配3.6也需要时间，但是毫无疑问，会很快。**jupyter的`ipython-qtconsole.exe`现在就用不了，因为pyqt还没支持3.6(毕竟3.6今天才出23333)，不过相信过几天就可以用了，python3已经是趋势，不要告诉我你的主要工作环境是python2(话说12月17号更新了python2.7.13)**

**注意有些包还是要手动更新的，例如windows上无法编译lxml，所以一般都是下载编译好的进行安装，之前下载的是支持python3.5的lxml，现在需要卸载当前库，并手动下载编译好的支持3.6的lxml进行安装,有些包使用pip安装的时候会提示编码问题，简单的方法就是从[Unofficial Windows Binaries for Python Extension Packages](http://www.lfd.uci.edu/~gohlke/pythonlibs/)下载，然后直接安装**

***以上只是本人环境，因为我目前只把python当作工具，所以不会像开发库一样考虑版本兼容等情况，不过一般还是建议将常用包放在python安装目录下，对于特定的项目构建虚拟环境，在虚拟环境中安装与python版本相适应的包进行开发。***

## What’s New In Python 3.6

主要改变:

- PEP 468 - Preserving the order of **kwargs in a function
- PEP 487 - Simpler customization of class creation
- PEP 495 - Local Time Disambiguation
- PEP 498 - Literal String Formatting
- PEP 506 - Adding A Secrets Module To The Standard Library
- PEP 509 - Add a private version to dict
- PEP 515 - Underscores in Numeric Literals
- PEP 519 - Adding a file system path protocol
- PEP 520 - Preserving Class Attribute Definition Order
- PEP 523 - Adding a frame evaluation API to CPython
- PEP 524 - Make os.urandom() blocking on Linux (during system startup)
- PEP 525 - Asynchronous Generators (provisional)
- PEP 526 - Syntax for Variable Annotations (provisional)
- PEP 528 - Change Windows console encoding to UTF-8
- PEP 529 - Change Windows filesystem encoding to UTF-8
- PEP 530 - Asynchronous Comprehensions

### PEP 498: Formatted string literals

```python
>>> name = "Fred"
>>> f"He said his name is {name}."
'He said his name is Fred.'
>>> width = 10
>>> precision = 4
>>> value = decimal.Decimal("12.34567")
>>> f"result: {value:{width}.{precision}}"  # nested fields
'result:      12.35'
```

在字符串前面加`f`，表示该字符串将被格式化，类似于对字符串进行`str.format()`操作，不得不说，确实很方便

### PEP 526: Syntax for variable annotations

提供变量声明语法,，包括类中的变量，实例中的变量和函数参数

```python
primes: List[int] = []

captain: str  # Note: no initial value!

class Starship:
    stats: Dict[str, int] = {}
```

```python
>>> class Starship:
...     stats: str
...
>>> Starship.__annotations__
{'stats': <class 'str'>}
```

当然，python始终是一门动态语言，所以这些类型声明实际上只是将这些类型信息存储在类或者模块的`__annotations__`属性中，并不会在运行时检擦这些属性，只是起到提示的作用，当然，这个特性确实也很有用处，具体类型声明语法请看[PEP 484](https://www.python.org/dev/peps/pep-0484/)

### PEP 515: Underscores in Numeric Literals

能够在数字间添加下划线以提高阅读性

```python
>>> 1_000_000_000_000_000
1000000000000000
>>> type(1_000_000_000_000_000)
<class 'int'>
>>> 0x_FF_FF_FF_FF
4294967295
```

同时字符串格式化也支持这种下划线的格式化方式:

```python
>>> '{:_}'.format(1000000)
'1_000_000'
>>> '{:_x}'.format(0xFFFFFFFF)
'ffff_ffff'
>>> '{:_X}'.format(0xFFfFFFFF)
'FFFF_FFFF'
```

当然也可以使用二进制`b`，八进制`o`

### PEP 525: Asynchronous Generators

异步生成器，python3.6中可以在同一函数体中使用`await`和`yield`

```python
class Ticker:
    """Yield numbers from 0 to `to` every `delay` seconds."""

    def __init__(self, delay, to):
        self.delay = delay
        self.i = 0
        self.to = to

    def __aiter__(self):
        return self

    async def __anext__(self):
        i = self.i
        if i >= self.to:
            raise StopAsyncIteration
        self.i += 1
        if i:
            await asyncio.sleep(self.delay)
        return i
```

以上代码现在可以简写为:

```python
async def ticker(delay, to):
    """Yield numbers from 0 to `to` every `delay` seconds."""
    for i in range(to):
        yield i
        await asyncio.sleep(delay)
```

### PEP 530: Asynchronous Comprehensions

可以在列表，元组，字典，生成器表达式中使用`async for`和`await`

```python
result = []
async for i in aiter():
    if i % 2:
        result.append(i)
```

可以简写为:

```python
result = [i async for i in aiter() if i % 2]
```

有关`await`的例子:

```python
result = [await fun() for fun in funcs if await condition()]
```

### PEP 487: Simpler customization of class creation

现在可以不用使用元类来自定义子类的创建

当子类被创建时，基类中的`__init_subclass__()`类方法将被调用

```python
class PluginBase:
    subclasses = []

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        cls.subclasses.append(cls)

class Plugin1(PluginBase):
    pass

class Plugin2(PluginBase):
    pass
```

### PEP 487: Descriptor Protocol Enhancements

描述符中新增了` __set_name__()`方法，当描述符被实例化时，便会调用` __set_name__()`方法

```python
class IntField:
    def __get__(self, instance, owner):
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        if not isinstance(value, int):
            raise ValueError(f'expecting integer in {self.name}')
        instance.__dict__[self.name] = value

    # this is the new initializer:
    def __set_name__(self, owner, name):
        self.name = name

class Model:
    int_field = IntField() # 将会调用__set_name__()方法，将属性名int_field保存起来
```

### PEP 519: Adding a file system path protocol

在大多数眼中，路径就是字符串或者是字节对象,以至于python标准库`pathlib`较少被使用。现在提供了一个`os.PathLike`接口，只要实现了`__fspath__()`方法，那么这个对象就表示是一个路径，并且可以使用`os.fspath()`,` os.fsdecode()`, 或者 `os.fsencode()`方法或者这个路径对象的字符串或字节表示

```python
>>> import pathlib
>>> with open(pathlib.Path("README")) as f:
...     contents = f.read()
...
>>> import os.path
>>> os.path.splitext(pathlib.Path("some_file.txt"))
('some_file', '.txt')
>>> os.path.join("/a/b", pathlib.Path("c"))
'/a/b/c'
>>> import os
>>> os.fspath(pathlib.Path("some_file.txt"))
'some_file.txt'
```

 ### PEP 529: Change Windows filesystem encoding to UTF-8

现在的python3.6版本使得我们可以在windows平台是正确使用字节对象表示的路径，而不会造成数据丢失，事实上，该字节对象就是通过` sys.getfilesystemencoding()`编码的，也就是`UTF-8`

### PEP 528: Change Windows console encoding to UTF-8

The default console on Windows will now accept all Unicode characters and provide correctly read str objects to Python code. `sys.stdin`, `sys.stdout` and`sys.stderr` now default to utf-8 encoding.

只想说，简直是福音，再也不用担心控制台输出乱码了。。。

### PEP 520: Preserving Class Attribute Definition Order

类中定义的属性的顺序在`__dict__`中将被保留

### PEP 468: Preserving Keyword Argument Order

`**kwargs` in a function signature is now guaranteed to be an insertion-order-preserving mapping.

#### New dict implementation

新的dict实现，比原来的实现快20% 到25%不说，还保留了顺序，也就是说dict现在是有序的。。。所以要OrderedDict何用？不过，官方也说了，现在只是暂时这样，有可能之后的版本又变成无序的了

```python
>>> b = {'one': 1, 'two': 2, 'three': 3}
>>> b
{'one': 1, 'two': 2, 'three': 3}
```

### 其他改动

添加了[`secrets`](https://docs.python.org/3.6/library/secrets.html#module-secrets)模块

改进了`re`模块，在正则表达式中添加了修饰符跨度的支持，Examples: `'(i:p)ython'` matches `'python'` and `'Python'`, but not `'PYTHON'`; `'(?i)g(?-i:v)r'`matches `'GvR'` and `'gvr'`, but not `'GVR'`

更多细节改动参考[官网What’s New In Python 3.6](https://docs.python.org/3.6/whatsnew/3.6.html)

## 参考文档

- [What’s New In Python 3.6](https://docs.python.org/3.6/whatsnew/3.6.html)