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

1. 生成器是特殊的迭代器,关键字`yield`就表示是一个生成器,其`__next__()`方法就是执行到下一次`yield`,其`__iter__()`方法就是返回自身

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
