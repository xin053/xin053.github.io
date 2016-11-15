---
title: Python官方文档摘记
date: 2016-10-06 17:40:00
sticky: 10
categories: 
- Python
tags:
- Python
---

## 说明
python官方文档无疑是每个学习python的pythoner必看的，除了[The Python Tutorial](https://docs.python.org/3/tutorial/index.html)新手必看之外，最重要莫过于[The Python Language Reference](https://docs.python.org/3/reference/index.html)和[The Python Standard Library](https://docs.python.org/3/library/index.html)
前者简单介绍了有关python的实现，后者则是python的标准库的详解，无疑是很好的学习资源，在此记下对于我日常而言使用比较多的库，当作索引，以便将来查阅。

**python官方文档版本：3.5.2**
**持续更新**

<!-- more -->

The Python Language Reference：

- [Lexical analysis 词法分析 简单介绍了python一些功能的实现原理](https://docs.python.org/3/reference/lexical_analysis.html)
- [Data model 介绍了很多双下划线的私有属性 值得反复看](https://docs.python.org/3/reference/datamodel.html)
- [The import system 导入系统介绍](https://docs.python.org/3/reference/import.html)

The Python Standard Library：

- [Built-in Functions 必须熟练掌握](https://docs.python.org/3/library/functions.html)
- [Built-in Types 基础内容 要熟悉](https://docs.python.org/3/library/stdtypes.html)
- [Text Processing Services 文本处理服务](https://docs.python.org/3.5/library/text.html)
  - [`format` — 字符串格式化](https://docs.python.org/3.5/library/string.html)
  - [`re` — 正则表达式操作](https://docs.python.org/3.5/library/re.html)
- [Data Types](https://docs.python.org/3/library/datatypes.html)

  - [`datetime` — 基础时间和日期模块](https://docs.python.org/3/library/datetime.html)
  - [`calendar` — General calendar-related functions](https://docs.python.org/3/library/calendar.html)
  - [`collections` — 提供代替内置列表元组字典的多种数据结构](https://docs.python.org/3/library/collections.html#module-collections)
    - [`ChainMap` — 通过链表将多个map链在一起 查找时优先查找前面的](https://docs.python.org/3/library/collections.html#collections.ChainMap)
    - [`Counter` — Dict subclass for counting hashable items](https://docs.python.org/3/library/collections.html#collections.Counter)
    - [`deque` — 双端队列](https://docs.python.org/3/library/collections.html#collections.deque)
    - [`defaultdict` — 给字典的value添加默认数据类型](https://docs.python.org/3/library/collections.html#collections.defaultdict)
    - [`namedtuple` — 命名元组](https://docs.python.org/3/library/collections.html#collections.namedtuple)
    - [`OrderedDict` — 有序字典 Dictionary that remembers insertion order](https://docs.python.org/3/library/collections.html#collections.OrderedDict)
    - [`UserDict` — dict 的一个封装类 主要用来拷贝一个字典的数据](https://docs.python.org/3/library/collections.html#collections.UserDict)
    - [`UserList`](https://docs.python.org/3/library/collections.html#collections.UserList)
    - [`UserString`](https://docs.python.org/3/library/collections.html#collections.UserString)
  - [`heapq` — 利用堆排序算法排序模块](https://docs.python.org/3/library/heapq.html)
  - [`bisect` — 利用二分法将元素插入到已排序列表后依然保持已排序](https://docs.python.org/3/library/bisect.html)
  - [`weakref` — 弱引用 用来避免循环引用导致对象没有被GC回收](https://docs.python.org/3/library/weakref.html)
  - [`types` — Define names for built-in types that aren't directly accessible as a builtin](https://docs.python.org/3/library/types.html)
  - [`copy` — 分浅拷贝(不拷贝子对象)与深拷贝(拷贝子对象)](https://docs.python.org/3/library/copy.html)
  - [`pprint` — Data pretty printer](https://docs.python.org/3/library/pprint.html)
  - [`reprlib` — 可以限制字符串长度的repr](https://docs.python.org/3/library/reprlib.html)
  - [`enum` — 3.4出现的](https://docs.python.org/3/library/enum.html)
- [Numeric and Mathematical Modules](https://docs.python.org/3/library/numeric.html)
  - [`math` — 常用数学操作 例如绝对值 阶乘 sin(x) cos(x)等](https://docs.python.org/3/library/math.html)
  - [`decimal` — 更精确的数据类型 注意对于float数据要先转成字符串](https://docs.python.org/3/library/decimal.html)
  - [`fractions` — 分数  如果需要准确精度也需要导入decimal包](https://docs.python.org/3/library/fractions.html)
  - [`random` — 产生随机数 可指定产生哪个范围的随机数也能指定返回整数 但是不能用在安全方面](https://docs.python.org/3/library/random.html)
  - [`statistics` — 统计学函数 例如求均值 中值 众数 标准差和方差](https://docs.python.org/3/library/statistics.html)
- [Functional Programming Modules](https://docs.python.org/3/library/functional.html)
  - [`itertools` — 提供多种常用便利操作 组合等等 返回值为生成器或者迭代器](https://docs.python.org/3/library/itertools.html)
  - [`functools` — 提供一些方法对函数进行封装](https://docs.python.org/3/library/functools.html)
  - [`operator` — 各种常用的操作符都封装到了该模块中 例如加减乘除 大小比较等等](https://docs.python.org/3/library/operator.html)
- [File and Directory Access](https://docs.python.org/3/library/filesys.html)
  - [`pathlib` — 路径库](https://docs.python.org/3/library/pathlib.html)  [pathlib路径库使用详解](https://xin053.github.io/2016/07/03/pathlib%E8%B7%AF%E5%BE%84%E5%BA%93%E4%BD%BF%E7%94%A8%E8%AF%A6%E8%A7%A3/)
  - [`os.path` — 同样是路径库 与pathlib不同的是后者上升到了对象级别](https://docs.python.org/3/library/os.path.html)
  - [`stat` — 显示文件mode等信息 也就是linux中 `ls -al` 显示的那些内容](https://docs.python.org/3/library/stat.html)
  - [`tempfile` — 创建临时文件或临时目录 使用特定的函数可以使得创建的临时文件在关闭的时候自动删除](https://docs.python.org/3/library/tempfile.html)
  - [`glob` — 匹配文件后缀名  可以包含路径](https://docs.python.org/3/library/glob.html)
  - [`fnmatch` — 匹配文件名  不能包含路径](https://docs.python.org/3/library/fnmatch.html)
  - [`linecache` — 获取文件指定行内容并缓存](https://docs.python.org/3/library/linecache.html)
  - [`shutil` — 提供高层次的文件操作 包括文件复制删除修改权限等 也有linux下`which`命令的功能 也提供简单的文件打包和解包](https://docs.python.org/3/library/shutil.html)


