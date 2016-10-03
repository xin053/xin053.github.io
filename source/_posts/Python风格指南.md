---
title: Python风格指南
date: 2016-10-03 19:34:51
categories: 
- Python
tags:
- Python
---

## 导入

> 仅对包和模块使用导入

虽然也可以导入某个函数，但这样做明显使得结构不清晰，所以应该仅仅对包和模块使用导入

## 包

> 使用模块的全路径名来导入每个模块

避免模块名冲突. 查找包更容易.

## 异常

> 允许使用异常, 但必须小心

模块或包应该定义自己的特定域的异常基类, 这个基类应该从内建的`Exception`类继承. 模块的异常基类应该叫做`Error`.

```python
class Error(Exception):
    pass
```

永远不要使用 `except:` 语句来捕获所有异常, 也不要捕获`Exception` 或者 `StandardError`,因为这样容易隐藏真正的bug

当捕获异常时, 使用 `as` 而不要用逗号. 例如:

```python
try:
    raise Error
except Error as error:
    pass
```

<!-- more -->

## 全局变量

> 避免全局变量

避免使用全局变量, 用类变量来代替. 但也有一些例外:

- 脚本的默认选项.
- 模块级常量. 例如:　PI = 3.14159. 常量应该全大写, 用下划线连接.
- 有时候用全局变量来缓存值或者作为函数返回值很有用.
- 如果需要, 全局变量应该仅在模块内部可用, 并通过模块级的公共函数来访问.

## 嵌套/局部/内部类或函数

> 鼓励使用嵌套/本地/内部类或函数

## 列表推导

> 可以在简单情况下使用

简单的列表推导可以比其它的列表创建方法更加清晰简单. 生成器表达式可以十分高效, 因为它们避免了创建整个列表.

适用于简单情况. 每个部分应该单独置于一行: 映射表达式, for语句, 过滤器表达式. 禁止多重for语句或过滤器表达式. 复杂情况下还是使用循环.

## 生成器

> 按需使用生成器.

## Lambda函数

> 适用于单行函数

适用于单行函数. 如果代码超过60-80个字符, 最好还是定义成常规(嵌套)函数.

## 条件表达式

> 适用于单行函数

条件表达式是对于if语句的一种更为简短的句法规则. 例如:` x = 1 if cond else 2 `.

适用于单行函数. 在其他情况下，推荐使用完整的if语句.

## True/False的求值

> 尽可能使用隐式false

Python在布尔上下文中会将某些值求值为false. 按简单的直觉来讲, 就是所有的”空”值都被认为是false. 因此`0`， `None`, `[]`, `{}`, `“”` 都被认为是`false`.

尽可能使用隐式的false, 例如: 使用 `if foo:` 而不是` if foo != []:` 不过还是有一些注意事项需要你铭记在心:

- 永远不要用`==`或者`!=`来比较单件, 比如`None`. 使用`is`或者`is not`.
- 对于序列(字符串, 列表, 元组), 要注意空序列是false. 

## 线程

> 不要依赖内建类型的原子性.

虽然Python的内建类型例如字典看上去拥有原子操作, 但是在某些情形下它们仍然不是原子的

优先使用`Queue`模块的 `Queue` 数据类型作为线程间的数据通信方式. 另外, 使用`threading`模块及其锁原语(locking primitives). 了解条件变量的合适使用方式, 这样你就可以使用 `threading.Condition` 来取代低级别的锁了.

## 分号

> 不要在行尾加分号, 也不要用分号将两条命令放在同一行.

## 行长度

> 每行不超过80个字符

例外:

- 长的导入模块语句
- 注释里的URL

不要使用反斜杠连接行.

Python会将 圆括号, 中括号和花括号中的行隐式的连接起来 , 你可以利用这个特点. 如果需要, 你可以在表达式外围增加一对额外的圆括号.

```python
Yes: foo_bar(self, width, height, color='black', design=None, x='foo',
             emphasis=None, highlight=0)

     if (width == 0 and height == 0 and
         color == 'red' and emphasis == 'strong'):
```

## 空行

> 顶级定义之间空两行, 方法定义之间空一行

## 空格

> 按照标准的排版规范来使用标点两边的空格

括号内不要有空格.

```python
Yes: spam(ham[1], {eggs: 2}, [])
```

```python
No:  spam( ham[ 1 ], { eggs: 2 }, [ ] )
```

## Shebang

> 大部分.py文件不必以#!作为文件的开始. 根据 [PEP-394](http://www.python.org/dev/peps/pep-0394/) , 程序的main文件应该以 `#!/usr/bin/python2`或者 `#!/usr/bin/python3`开始.

#!先用于帮助内核找到Python解释器, 但是在导入模块时, 将会被忽略. 因此只有被直接执行的文件中才有必要加入#!.

## 注释

> 确保对模块, 函数, 方法和行内注释使用正确的风格

```python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table.  Silly things may happen if
    other_silly_variable is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
         'Zim': ('Irk', 'Invader'),
         'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
    pass
```

## 类

> 类应该在其定义下有一个用于描述该类的文档字符串. 如果你的类有公共属性(Attributes), 那么文档中应该有一个属性(Attributes)段. 并且应该遵守和函数参数相同的格式.

```python
class SampleClass(object):
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```

## 类

> 如果一个类不继承自其它类, 就显式的从object继承. 嵌套类也一样.

```python
Yes: class SampleClass(object):
         pass


     class OuterClass(object):

         class InnerClass(object):
             pass


     class ChildClass(ParentClass):
         """Explicitly inherits from another class already."""
```

```python
No: class SampleClass:
        pass


    class OuterClass:

        class InnerClass:
            pass
```

继承自 `object` 是为了使属性(properties)正常工作, 并且这样可以保护你的代码, 使其不受 [PEP-3000](http://www.python.org/dev/peps/pep-3000/) 的一个特殊的潜在不兼容性影响. 这样做也定义了一些特殊的方法, 这些方法实现了对象的默认语义, 包括:

`__new__, __init__, __delattr__, __getattribute__, __setattr__, __hash__, __repr__, and __str__`

## 字符串

> 即使参数都是字符串, 使用%操作符或者格式化方法格式化字符串. 不过也不能一概而论, 你需要在+和%之间好好判定.

避免在循环中用`+`和`+=`操作符来累加字符串.作为替代方案, 你可以将每个子串加入列表, 然后在循环结束后用 `.join` 连接列表. 
## 文件和sockets

> 在文件和sockets结束时, 显式的关闭它.

推荐使用 `with`语句 以管理文件:

```python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print line
```

对于不支持使用`with`语句的类似文件的对象,使用 `contextlib.closing()`:

```python
import contextlib

with contextlib.closing(urllib.urlopen("http://www.python.org/")) as front_page:
    for line in front_page:
        print line
```

## TODO注释

> 为临时代码使用TODO注释, 它是一种短期解决方案. 不算完美, 但够好了.

当你写了一个TODO, 请注上你的名字.

```python
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```

## 导入格式

> 每个导入应该独占一行

导入应该按照从最通用到最不通用的顺序分组:

- 标准库导入
- 第三方库导入
- 应用程序指定导入

## 命名

> module_name, package_name, ClassName, method_name, ExceptionName, function_name, GLOBAL_VAR_NAME, instance_var_name, function_parameter_name, local_var_name.

命名约定:

- 所谓”内部(Internal)”表示仅模块内可用, 或者, 在类内是保护或私有的.
- 用单下划线(_)开头表示模块变量或函数是protected的(使用import * from时不会包含).
- 用双下划线(__)开头的实例变量或方法表示类内私有.
- 将相关的类和顶级函数放在同一个模块里. 不像Java, 没必要限制一个类一个模块.
- 对类名使用大写字母开头的单词(如CapWords, 即Pascal风格), 但是模块名应该用小写加下划线的方式(如lower_with_under.py). 尽管已经有很多现存的模块使用类似于CapWords.py这样的命名, 但现在已经不鼓励这样做, 因为如果模块名碰巧和类名一致, 这会让人困扰.

Python之父Guido推荐的规范:

|Type                           |Public|                  Internal|
|:---:|:---:|:---:|
|Modules                        |lower_with_under|        _lower_with_under|
|Packages                       |lower_with_under|             |
|Classes                        |CapWords         |       _CapWords|
|Exceptions                     |CapWords         |                |
|Functions                      |lower_with_under()|      _lower_with_under()|
|Global/Class Constants         |CAPS_WITH_UNDER    |     _CAPS_WITH_UNDER|
|Global/Class Variables         |lower_with_under   |     _lower_with_under|
|Instance Variables             |lower_with_under   |     _lower_with_under (protected) or __lower_with_under (private)|
|Method Names                   |lower_with_under() |     _lower_with_under() (protected) or __lower_with_under() (private)|
|Function/Method Parameters     |lower_with_under   |               |   
|Local Variables                |lower_with_under   |               |         

## Main

> 即使是一个打算被用作脚本的文件, 也应该是可导入的. 并且简单的导入不应该导致这个脚本的主功能(main functionality)被执行, 这是一种副作用. 主功能应该放在一个main()函数中.

在Python中, pydoc以及单元测试要求模块必须是可导入的. 你的代码应该在执行主程序前总是检查 `if __name__ == '__main__'` , 这样当模块被导入时主程序就不会被执行.

```python
def main():
      ...

if __name__ == '__main__':
    main()
```

**所有的顶级代码在模块导入时都会被执行. 要小心不要去调用函数, 创建对象, 或者执行那些不应该在使用pydoc时执行的操作.**

## 参考文档

- [Python风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/contents/)
