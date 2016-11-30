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

1. Python模块搜索机制
   1. 程序当前目录
   2. PYTHONPATH(环境变量)目录
   3. 标准库目录


2. CPython,PyPy,Jython
   1. CPython即底层用c/c++语言实现的Python,也是我们通常所用的python解释器
   2. PyPy是使用Python实现的python解释器，速度比CPython快
   3. Jython可以让python code跑在JVM上,并可以调用java code的解释器


3. 一个项目标准文件层次结构

   ![](http://i.imgur.com/jwwhOiY.png)

4. import导入模块顺序
   1. 标准库
   2. 第三方库
   3. 本地库

5. 命名
   1. 类命名采用驼峰命名法
   2. 异常定义使用Error前缀
   3. 函数命名使用小写,并用下划线连接每个word
   4. 模块命名使用小写，word直接连接(较多)或者用下划线

6. 库API
   1. 公共API：库暴露给用户使用的API
   2. 私有API：为实现功能，库内部使用的API，函数名前面有下划线

7. 库更新时应该注意的问题
   1. 废弃的接口不要立即删除
   2. 可以在新版本中使用`warnings`对使用废弃API的程序员显示警告信息

8. pip安装包
   1. 如果在linux系统，可以使用`pip install --user 包名`来将包安装在`home`目录以避免在系统层面安装而造成操作系统目录的污染

9. for循环
   1. 当使用for循环的时候，for语句就会自动的通过`__iter__()`方法来获得迭代器对象，并且通过`__next__()`方法来获取下一个元素

10. 可迭代与迭代器
   1. 如果一个对象具有`__iter__()`方法，那么它便是一个可迭代对象(iterable)
   2. 如果一个可迭代对象具有`next()`(python 2)或者`__next__()`(python 3)方法,并且函数返回是`return`而不是`yield`,那么它就是一个迭代器(iterator)

11. 生成器
   1. 生成器是特殊的迭代器,关键字`yield`就表示是一个生成器,其`__next__()`方法就是执行到下一次`yield`,其`__iter__()`方法就是返回自身

12. 列表解析与生成器表达式
   1. `[]`返回列表
   2. `()`返回生成器

13. 迭代器与生成器
   1. 两者都比list节省内存，也就是运行的时候才生成数据，而不是像list一次性存储全部数据
   2. 生成器的编写比自定义一个迭代器简单很多,使用`yield`关键字或者`()`表达式就可以返回一个生成器,而不用自己写`__iter__()`与`__next__()`方法，并且生成器可以做更多的事情
   3. 生成器在进行迭代的时候只是暂停,并不是结束，而`finally`语句中的内容是在介绍的时候才执行的
   4. `list()` `tuple()` `max()` `min()`等函数都支持迭代器或者生成器作为参数,但是要注意如果迭代器或者生成器是无限的，那么很明显会一直循环，所以使用前要搞清楚迭代器或生成器是否是有限的

14. 列表与元组
   1. 列表与远足最大的区别就是前者可变，后者不可变，所以元组较安全

15. 绑定与非绑定方法

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

16. 待定
