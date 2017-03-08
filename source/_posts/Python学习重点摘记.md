---
title: Python学习重点摘记
date: 2016-10-30 20:01:37
sticky: 8
categories:
- Python
tags:
- Python
---

## 说明
该文档为学习python过程中觉得非常重要的内容(利于理解某些实现原理)，也就是作为一名pythoner，必须知道的内容，但是好记性不如烂笔头，还是记下来，定时复习还是比较好的。

**持续更新**

<!-- more -->

## 重点

### Python模块搜索机制

1. 程序当前目录
2. PYTHONPATH(环境变量)目录
3. 标准库目录

### CPython,PyPy,Jython

1. CPython即底层用c/c++语言实现的Python,也是我们通常所用的python解释器
2. PyPy是使用Python实现的python解释器，速度比CPython快
3. Jython可以让python code跑在JVM上,并可以调用java code的解释器

### 一个项目标准文件层次结构

![](http://i.imgur.com/jwwhOiY.png)

### import导入模块顺序

1. 标准库
2. 第三方库
3. 本地库

### 命名

1. 类命名采用驼峰命名法
2. 异常定义使用Error前缀
3. 函数命名使用小写,并用下划线连接每个word
4. 模块命名使用小写，word直接连接(较多)或者用下划线

### 库API

1. 公共API：库暴露给用户使用的API
2. 私有API：为实现功能，库内部使用的API，函数名前面有下划线

### 库更新时应该注意的问题

1. 废弃的接口不要立即删除
2. 可以在新版本中使用`warnings`对使用废弃API的程序员显示警告信息

### pip安装包

1. 如果在linux系统，可以使用`pip install --user 包名`来将包安装在`home`目录以避免在系统层面安装而造成操作系统目录的污染

### for循环

1. 当使用for循环的时候，for语句就会自动的通过`__iter__()`方法来获得迭代器对象，并且通过`__next__()`方法来获取下一个元素

### 可迭代与迭代器

1. 如果一个对象具有`__iter__()`方法，那么它便是一个可迭代对象(iterable)
2. 如果一个可迭代对象具有`next()`(python 2)或者`__next__()`(python 3)方法,并且函数返回是`return`而不是`yield`,那么它就是一个迭代器(iterator)

### 生成器

1. 生成器是特殊的迭代器,关键字`yield`就表示该方法返回一个生成器对象,其`__next__()`方法就是执行到下一次`yield`,其`__iter__()`方法就是返回自身，可以用for循环执行生成器中的代码。当循环结束，或不满足`if/else`条件时，导致函数运行但不命中`yield`关键字，此时生成器触发`StopIteration`异常。
2. 调用生成器时`__next__()`方法时，实际上是将函数状态挂起，也就是保存了栈帧的状态

```python
def test():
    index = 0
    for i in range(3):
        index = i
        yield i
    print('hello')    
    if index > 2:
        yield '1:'+str(index)
    print('world')
    
a = test()
a.__next__()
# 0
a.__next__()
# 1
a.__next__()
# 2
a.__next__()
# hello
# world
# ---------------------------------------------------------------------------
# StopIteration                             Traceback (most recent call last)
# <ipython-input-41-73aa2c76d676> in <module>()
# ----> 1 a.__next__()

# StopIteration:
# 最后一次__next__()到代码末尾，所以触发异常
```

```python
def test():
    index = 0
    for i in range(3):
        index = i
        yield i
    print('hello')    
    if index > 1:
        yield '1:'+str(index)
    print('world')
    
a = test()
a.__next__()
# 0
a.__next__()
# 1
a.__next__()
# 2
a.__next__()
# hello
# '1:2'
a.__next__()
# world
# ---------------------------------------------------------------------------
# StopIteration                             Traceback (most recent call last)
# <ipython-input-48-73aa2c76d676> in <module>()
# ----> 1 a.__next__()

# StopIteration: 

list(test())
# hello
# world
# [0, 1, 2, '1:2']
# list内部循环执行__next__(),然后append到列表中返回，所以后输出列表
```

### `yield`表达式

```python
def counter(maximum):
    i = 0
    while i < maximum:
        val = (yield i)
        # If value provided, change counter
        if val is not None:
            i = val
        else:
            i += 1
```

```python
>>> it = counter(10)  
>>> next(it)  
0
>>> next(it)  
1
>>> it.send(8)  
8
>>> next(it)  
9
>>> next(it)  
Traceback (most recent call last):
  File "t.py", line 15, in ?
    it.next()
StopIteration

```
可以通过`send()`方法向生成器发送数据，`(yield i)`表达式的值就是`send(value)`中的`value`,而默认情况，`(yield i)`表达式的值为`None`

### 列表解析与生成器表达式

1. `[]`返回列表
2. `()`返回生成器

### 迭代器与生成器

1. 两者都比list节省内存，也就是运行的时候才生成数据，而不是像list一次性存储全部数据
2. 生成器的编写比自定义一个迭代器简单很多,使用`yield`关键字或者`()`表达式就可以返回一个生成器,而不用自己写`__iter__()`与`__next__()`方法，并且生成器可以做更多的事情
3. 生成器在进行迭代的时候只是暂停,并不是结束，而`finally`语句中的内容是在介绍的时候才执行的
4. `list()` `tuple()` `max()` `min()`等函数以及`in`,`not in`操作符都支持迭代器或者生成器作为参数,但是要注意如果迭代器或者生成器是无限的，那么很明显会一直循环，所以使用前要搞清楚迭代器或生成器是否是有限的
5. 对字典进行`for`操作，默认是获取的`key`的迭代器，并且注意普通字典是无序的，要想获取`value`的迭代器或者是`key/value`的迭代器，就调用字典的`values()`和`items()`方法

### 迭代器与内置函数

1. `map(f, iterA, iterB, ...)` returns an iterator over the sequence 
   `f(iterA[0], iterB[0]), f(iterA[1], iterB[1]), f(iterA[2], iterB[2]), ....`
2. `filter(predicate, iter)` returns an iterator over all the sequence elements that meet a certain condition
3. `enumerate(iter)` counts off the elements in the iterable, returning 2-tuples containing the count and each element
4. `sorted(iterable, key=None, reverse=False)` collects all the elements of the iterable into a list, sorts the list, and returns the sorted result
5. `any(iter)` and `all(iter)` built-ins look at the **truth values of an iterable’s contents**. `any()` returns `True` if any element in the iterable is a true value, and `all()` returns `True` if all of the elements are true values
6. `zip(iterA, iterB, ...)` takes one element from each iterable and returns them in a tuple.It doesn’t construct an in-memory list and exhaust all the input iterators before returning; instead tuples are constructed and returned only if they’re requested. (The technical term for this behaviour is lazy evaluation.)
7. `functools.reduce(func, iter, [initial_value])` cumulatively performs an operation on all the iterable’s elements and, therefore, can’t be applied to infinite iterables. `func` must be a function that takes two elements and returns a single value. `functools.reduce()` takes the first two elements `A` and `B` returned by the iterator and calculates `func(A, B)`. It then requests the third element, `C`, calculates `func(func(A, B), C)`, combines this result with the fourth element returned, and continues until the iterable is exhausted. If the iterable returns no values at all, a `TypeError` exception is raised. If the initial value is supplied, it’s used as a starting point and `func(initial_value, A)` is the first calculation.参考 [python的reduce()函数](https://www.cnblogs.com/XXCXY/p/5180245.html) `reduce`常与`operator`模块中的函数搭配使用

### 列表与元组

1. 列表与远足最大的区别就是前者可变，后者不可变，所以元组较安全

### 绑定与非绑定方法

```python
         >>> class D(object):
         ...     def f(self, x):
         ...         return x
         ...
         >>> d = D()
         >>> D.__dict__['f']  # Stored internally as a function
         <function f at 0x00C45070>
         >>> D.f              # Get from a class becomes an unbound method
         <unbound method D.f>
         >>> d.f              # Get from an instance becomes a bound method
         <bound method D.f of <__main__.D object at 0x00B18C90>>
```

### log

1. 级别从低到高为`DEBUG` `INFO` `WARNING` `ERROR` `CRITICAL`
2. 默认级别为`WARNING`，只有`WARNING`和之上的级别的log控制台才会被输出
3. [python logging模块使用教程](http://www.jianshu.com/p/feb86c06c4f4) [Python logging模块详解](http://blog.csdn.net/zyz511919766/article/details/25136485)

### sorted

1. 可处理简单列表排序和针对对象中某一属性进行排序，并且与`sort()`不同，前者不改变原列表，后者直接修改原列表
2. 使用方法参考[Sorting HOW TO](https://docs.python.org/3/howto/sorting.html)

### ipaddress

1. 该模块能提供一些方法对ipv4和ipv6进行识别和处理

### 类

当一个**类定义**被执行，发生了这些事:

1. 一个合适的元类被确定
2. 类命名空间被准备好
3. 类主体被执行
4. 类对象被创建

**类是元类的实例**，在python中，一切都是对象，类也是对象

### 特殊方法

1. `__new__()`是特殊的不用声明的静态方法
2. `__mro__()`返回类所有的父类

### 文件操作

1. `os.listdir()`获取的只是指定路径下所有文件名组成的字符串列表，`os.scandir()`这可以返回一个生成器，每个元素是`DirEntry`对象，保留了文件相关的信息
2. `os.remove()`只能删除文件，`os.rmdir()`只能删除空目录，`shutil.rmtree()`可以递归删除目录

### subprocess替代

1. [`delegator` — `requests`作者最新作品](https://github.com/kennethreitz/delegator.py)
2. [`envoy` — 对标准库subprocess的封装，比较好用](https://github.com/kennethreitz/envoy)
3. [`sh` — linux平台下subprocess的替代](https://github.com/amoffat/sh)

### keyring

1. 该模块能够将密码保存在系统keyring服务中

### random 与urandom

1. random是标准库中的一个模块，产生的随机数不安全
2. urandom()是os模块中的方法，使用基于系统的随机数生成器，是安全的

### python2与3的编码问题

1. Python2有两种表示字符序列的类型，分别叫做`str`和`Unicode`，`str`实例包含原始的8位值；而`Unicode`的实例，则包含`Unicode`字符。

   1. `str`格式本质含义是“某种编码格式”，绝大多数情况下，被引号框起来的字符串，就是`str`，这时的字符串编码类型，其实就是你Python文件的编码类型，比如在Windows里，默认用的是GBK编码。
   2. `Unicode`格式的含义就是“用unicode编码的字符串”。Python在进入2.0版后正式定义了了Unicode字符串这个奇怪的特性，目的就是为了处理太多种语言编码的文本。从那时开始，Python语言中的字符串类型就分为两种：一种是传统的Python字符串（各种花样编码），另一种则是新出现的`Unicode`

   ![](https://ooo.0o0.ooo/2016/11/16/582c111e3fa73.png)

2. Python3也有两种表示字符序列的类型：`bytes`和`str`。前者的实例包含原始的8位值，后者的实例包含`Unicode`字符,可以说python3的`str`，就是python2的`Unicode`

   1. `str`格式的定义变更为”Unicode类型的字符串“，也就是说在默认情况下，被引号框起来的字符串，是使用`Unicode`编码的。
   2. 而“不是Unicode的某种编码格式”，比如UTF-8、GBK，这些编码方式被定义为了`bytes`，这里的`bytes`和py2中的`str`有很多相似的地方

3. 我们需要编写两个辅助（helper）函数，以便在这两种情况之间转换，使得转换后的输入数据能够符合开发者的预期

   ```python
   #在Python3中，我们需要编写接受str或bytes，并总是返回str的方法：
   def to_str(bytes_or_str):
     if isinstance(bytes_or_str, bytes):
       value = bytes_or_str.decode('utf-8')
     else:
       value = bytes_or_str
     return value # Instance of str
     
   #另外，还需要编写接受str或bytes，并总是返回bytes的方法：
   def to_bytes(bytes_or_str):
     if isinstance(bytes_or_str, str):
       value = bytes_or_str.encode('utf-8)
     else:
       value = bytes_or_str
     return value # Instance of bytes
     
   #在Python2中，需要编写接受str或unicode，并总是返回unicode的方法：
   #python2
   def to_unicode(unicode_or_str):
     if isinstance(unicode_or_str, str):
       value = unicode_or_str.decode('utf-8')
     else:
       value = unicode_or_str
     return value # Instance of unicode
     
   #另外，还需要编写接受str或unicode，并总是返回str的方法：
   #Python2
   def to_str(unicode_or_str):
     if isinstance(unicode_or_str, unicode):
       value = unicode_or_str.encode('utf-8')
     else:
       value = unicode_or_str
     reutrn vlaue # Instance of str
   ```


### contextlib

1. 使用`contextlib`标准库可以快速构造上下文管理器
2. 使用`contextlib`库中的`contextmanager`装饰器装饰一个返回生成器的函数，就构造了一个上下文管理器，其中`yield`前面的内容是`with`表达式执行之前的操作，`yield`之后的内容是`with`表达式执行之后的操作，`yield`如果有返回的内容，则会返回给`with`表达式中`as`之后的变量

### 打包python包到pypi

1. [`flit` — 很方便](https://github.com/takluyver/flit)

### 装饰器

```python
def identity(f):
    return f

@identity
def foo():
    return 'bar'
```

上述就是一个最简单的装饰器的实现，实际上，装饰器就是实现了下面的操作:

```python
def foo():
    return 'bar'

foo = identity(foo)
```

注册装饰器的实现:

```python
_functions = {}
def register(f):
    global _functions
    _functions[f.__name__] = f
    return f
    
@register
def foo():
    return 'bar'
```

上述装饰器没有使用被装饰函数的参数，返回的还是原来的函数，只是在返回之前做了一些事情

而大多数装饰器都会用到被装饰函数的参数，这时候就不能返回函数本身了，因为最后返回函数本身，就没有渠道获取被装饰函数的参数，所以这时候就需要返回一个对被装饰函数进行包装的函数，例如:

```python
def check_is_admin(f):
    def wrapper(*args, **kwargs):
        if kwargs.get('username') != 'admin':
            raise Exception("This user is not allowd to get food")
        return f(*args,**kwargs)
    return wrapper

class Store(object):
    @check_is_admin
    def get_food(self, username, food):
        return self.storage.get(food)
    
    @check_is_admin
    def put_food(self, username, food):
        self.storage.put(food)
```

`get_food`被装饰后，实际上我们在调用`get_food`时，调用的是`wrapper`函数，也就是说`get_food`被包装了，所以`wrapper`可以获取到输入的参数，并对参数进行一些操作,并且注意上述代码要传入一个名为`username`的关键字参数，例如:

```python
get_food(username='admin', food='apple')
```

为此，我们需要一个更加智能的装饰器，它能够查看被装饰函数的参数，并从中提取需要的参数，`inspect`模块能实现这样的需求

并且由于实际上我们调用的是包装函数`wrapper`,那么像`__name__`等返回的就是`wrapper`函数的信息，而不是`get_food`函数的信息,为此，`functools`提供`wraps`装饰器用来将被装饰函数一些特有属性赋值给包装函数，这样我们使用`wrapper`函数时，就像真的在使用原`get_food`函数一样

```python
import functools
import inspect

def check_is_admin(f):
    @functools.wraps(f)
    def wrapper(*args, **kwargs):
        func_args = inspect.getcallargs(f, *args, **kwargs)
        if func_args.get('username') != 'admin':
            raise Exception("This user is not allowd to get food")
        return f(*args,**kwargs)
    return wrapper
```

