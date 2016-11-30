---
title: Python描述符descriptor
date: 2016-11-29 18:40:14
categories: 
- Python
tags:
- Python
- descriptor
---

## 简介

### Python描述符(descriptor)解密

原文链接： [Chris Beaumont](http://nbviewer.ipython.org/urls/gist.github.com/ChrisBeaumont/5758381/raw/descriptor_writeup.ipynb) 翻译： [极客范 ](http://www.geekfan.net/)- [慕容老匹夫](http://www.geekfan.net/author/murong/)

转载链接： [http://www.geekfan.net/7862/](http://www.geekfan.net/7862/)

Python中包含了许多内建的语言特性，它们使得代码简洁且易于理解。这些特性包括列表/集合/字典推导式，属性（property）、以及装饰器（decorator）。对于大部分特性来说，这些“中级”的语言特性有着完善的文档，并且易于学习。

但是这里有个例外，那就是描述符。至少对于我来说，描述符是Python语言核心中困扰我时间最长的一个特性。这里有几点原因如下：

1. 有关描述符的官方文档相当难懂，而且没有包含优秀的示例告诉你为什么需要编写描述符（我得为Raymond Hettinger辩护一下，他写的其他主题的Python文章和视频对我的帮助还是非常大的）
2. 编写描述符的语法显得有些怪异
3. 自定义描述符可能是Python中用的最少的特性，因此你很难在开源项目中找到优秀的示例

但是一旦你理解了之后，描述符的确还是有它的应用价值的。这篇文章告诉你描述符可以用来做什么，以及为什么应该引起你的注意。

<!-- more -->

## 一句话概括：描述符就是可重用的属性

在这里我要告诉你：从根本上讲，描述符就是可以重复使用的属性。也就是说，描述符可以让你编写这样的代码：

```python
f = Foo()
b = f.bar
f.bar = c
del f.bar
```

而在解释器执行上述代码时，当发现你试图访问属性`b = f.bar`、对属性赋值`f.bar = c`或者删除一个实例变量的属性`del f.bar`时，就会去调用自定义的方法。

让我们先来解释一下为什么把对函数的调用伪装成对属性的访问是大有好处的。

## property——把函数调用伪装成对属性的访问

想象一下你正在编写管理电影信息的代码。你最后写好的Movie类可能看上去是这样的：

```python
class Movie(object):
    def __init__(self, title, rating, runtime, budget, gross):
        self.title = title
        self.rating = rating
        self.runtime = runtime
        self.budget = budget
        self.gross = gross
 
    def profit(self):
        return self.gross - self.budget
```

你开始在项目的其他地方使用这个类，但是之后你意识到：如果不小心给电影打了负分怎么办？你觉得这是错误的行为，希望`Movie`类可以阻止这个错误。 你首先想到的办法是将`Movie`类修改为这样：

```python
class Movie(object):
    def __init__(self, title, rating, runtime, budget, gross):
        self.title = title
        self.rating = rating
        self.runtime = runtime
        self.gross = gross
        if budget < 0:
            raise ValueError("Negative value not allowed: %s" % budget)
        self.budget = budget
 
    def profit(self):
        return self.gross - self.budget
```

但这行不通。因为其他部分的代码都是直接通过`Movie.budget`来赋值的,这个新修改的类只会在`__init__`方法中捕获错误的数据，但对于已经存在的类实例就无能为力了。如果有人试着运行`m.budget = -100`，那么谁也没法阻止。作为一个Python程序员同时也是电影迷，你该怎么办？

幸运的是，Python的`property`解决了这个问题。如果你从未见过`property`的用法，下面是一个示例：

```python
class Movie(object):
    def __init__(self, title, rating, runtime, budget, gross):
        self._budget = None
 
        self.title = title
        self.rating = rating
        self.runtime = runtime
        self.gross = gross
        self.budget = budget
 
    @property
    def budget(self):
        return self._budget
 
    @budget.setter
    def budget(self, value):
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self._budget = value
 
    def profit(self):
        return self.gross - self.budget
 
m = Movie('Casablanca', 97, 102, 964000, 1300000)
print m.budget       # calls m.budget(), returns result
try:
    m.budget = -100  # calls budget.setter(-100), and raises ValueError
except ValueError:
    print "Woops. Not allowed"
 
964000
Woops. Not allowed
```

我们用`@property`装饰器指定了一个`getter`方法，用`@budget.setter`装饰器指定了一个`setter`方法。当我们这么做时，每当有人试着访问`budget`属性，Python就会自动调用相应的`getter/setter`方法。比方说，当遇到`m.budget = value`这样的代码时就会自动调用`budget.setter`

花点时间来欣赏一下Python这么做是多么的优雅：如果没有`property`，我们将不得不把所有的实例属性隐藏起来，提供大量显式的类似`get_budget`和`set_budget`方法。像这样编写类的话，使用起来就会不断的去调用这些`getter/setter`方法，这看起来就像臃肿的Java代码一样。更糟的是，如果我们不采用这种编码风格，直接对实例属性进行访问。那么稍后就没法以清晰的方式增加对非负数的条件检查——我们不得不重新创建`set_budget`方法，然后搜索整个工程中的源代码，将`m.budget = value`这样的代码替换为`m.set_budget(value)`。太蛋疼了！！

因此，`property`让我们将自定义的代码同变量的访问/设定联系在了一起，同时为你的类保持一个简单的访问属性的接口。干得漂亮！

## property的不足

对`property`来说，最大的缺点就是它们不能重复使用。举个例子，假设你想为`rating`，`runtime`和`gross`这些字段也添加非负检查。下面是修改过的新类：

```python
class Movie(object):
    def __init__(self, title, rating, runtime, budget, gross):
        self._rating = None
        self._runtime = None
        self._budget = None
        self._gross = None
 
        self.title = title
        self.rating = rating
        self.runtime = runtime
        self.gross = gross
        self.budget = budget
 
    #nice
    @property
    def budget(self):
        return self._budget
 
    @budget.setter
    def budget(self, value):
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self._budget = value
 
    #ok    
    @property
    def rating(self):
        return self._rating
 
    @rating.setter
    def rating(self, value):
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self._rating = value
 
    #uhh...
    @property
    def runtime(self):
        return self._runtime
 
    @runtime.setter
    def runtime(self, value):
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self._runtime = value        
 
    #is this forever?
    @property
    def gross(self):
        return self._gross
 
    @gross.setter
    def gross(self, value):
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self._gross = value        
 
    def profit(self):
        return self.gross - self.budget
```

可以看到代码增加了不少，但重复的逻辑也出现了不少。虽然`property`可以让类从外部看起来接口整洁漂亮，**但是却做不到内部同样整洁漂亮。**

## 描述符登场（最终的大杀器）

这就是描述符所解决的问题。描述符是`property`的升级版，允许你为重复的`property`逻辑编写单独的类来处理。下面的示例展示了描述符是如何工作的（现在还不必担心`NonNegative`类的实现）：

```python
from weakref import WeakKeyDictionary
 
class NonNegative(object):
    """A descriptor that forbids negative values"""
    def __init__(self, default):
        self.default = default
        self.data = WeakKeyDictionary()
 
    def __get__(self, instance, owner):
        # we get here when someone calls x.d, and d is a NonNegative instance
        # instance = x
        # owner = type(x)
        return self.data.get(instance, self.default)
 
    def __set__(self, instance, value):
        # we get here when someone calls x.d = val, and d is a NonNegative instance
        # instance = x
        # value = val
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self.data[instance] = value
 
class Movie(object):
 
    #always put descriptors at the class-level
    rating = NonNegative(0)
    runtime = NonNegative(0)
    budget = NonNegative(0)
    gross = NonNegative(0)
 
    def __init__(self, title, rating, runtime, budget, gross):
        self.title = title
        self.rating = rating
        self.runtime = runtime
        self.budget = budget
        self.gross = gross
 
    def profit(self):
        return self.gross - self.budget
 
m = Movie('Casablanca', 97, 102, 964000, 1300000)
print m.budget  # calls Movie.budget.__get__(m, Movie)
m.rating = 100  # calls Movie.budget.__set__(m, 100)
try:
    m.rating = -1   # calls Movie.budget.__set__(m, -100)
except ValueError:
    print "Woops, negative value"
 
964000
Woops, negative value
```

这里引入了一些新的语法，我们一条条的来看：

`NonNegative`是一个描述符对象，因为它定义了`__get__`，`__set__`或`__delete__`方法。

`Movie`类现在看起来非常清晰。我们在类的层面上创建了4个描述符，把它们当做普通的实例属性。显然，描述符在这里为我们做非负检查。

### 访问描述符

当解释器遇到`print m.buget`时，它就会把`budget`当作一个带有`__get__ `方法的描述符，调用`Movie.budget.__get__`方法并将方法的返回值打印出来，而不是直接传递`m.budget`来打印。这和你访问一个`property`相似，Python自动调用一个方法，同时返回结果。

`__get__`接收2个参数：一个是点号左边的实例对象（在这里，就是m.budget中的m），另一个是这个实例的类型`Movie`。在一些Python[文档](http://docs.python.org/2/reference/datamodel.html#implementing-descriptors)中，`Movie`被称作描述符的所有者（owner）。如果我们需要访问`Movie.budget`，Python将会调用`Movie.budget.__get__(None, Movie)`。可以看到，第一个参数要么是所有者的实例，要么是`None`。这些输入参数可能看起来很怪，但是这里它们告诉了你描述符属于哪个对象的一部分。当我们看到`NonNegative`类的实现时这一切就合情合理了。

### 对描述符赋值

当解释器看到`m.rating = 100`时，Python识别出`rating`是一个带有`__set__`方法的描述符，于是就调用`Movie.rating.__set__(m, 100)`。和`__get__`一样，`__set__`的第一个参数是点号左边的类实例`m.rating = 100`中的`m`。第二个参数是所赋的值（100）。

### 删除描述符

为了说明的完整，这里提一下删除。如果你调用`del m.budget`，Python就会调用`Movie.budget.__delete__(m)`。

## NonNegative类是如何工作的？

带着前面的困惑，我们终于要揭示`NonNegative`类是如何工作的了。每个`NonNegative`的实例都维护着一个字典，其中保存着所有者实例和对应数据的映射关系。当我们调用`m.budget`时，`__get__`方法会查找与`m`相关联的数据，并返回这个结果（如果这个值不存在，则会返回一个默认值）。`__set__`采用的方式相同，但是这里会包含额外的非负检查。我们使用`WeakKeyDictionary`来取代普通的字典以防止内存泄露——我们可不想仅仅因为它在描述符的字典中就让一个无用的实例一直存活着。

使用描述符会有一点别扭。因为它们作用于类的层次上，每一个类实例都共享同一个描述符。这就意味着对不同的实例对象而言，描述符不得不手动地管理不同的状态，同时需要显式的将类实例作为第一个参数准确传递给`__get__`、`__set__`以及`__delete__`方法。

我希望这个例子解释清楚了描述符可以用来做什么——它们提供了一种方法将`property`的逻辑隔离到单独的类中来处理。如果你发现自己正在不同的`property`之间重复着相同的逻辑，那么本文也许会成为一个线索供你思考为何用描述符重构代码是值得一试的。

## 秘诀和陷阱

### 把描述符放在类的层次上（class level）

为了让描述符能够正常工作，它们必须定义在类的层次上。如果你不这么做，那么Python无法自动为你调用`__get__`和`__set__`方法。

```python
class Broken(object):
    y = NonNegative(5)
    def __init__(self):
        self.x = NonNegative(0)  # NOT a good descriptor
 
b = Broken()
print "X is %s, Y is %s" % (b.x, b.y)
 
X is <__main__.NonNegative object at 0x10432c250>, Y is 5
```

可以看到，访问类层次上的描述符`y`可以自动调用`__get__`。但是访问实例层次上的描述符x只会返回描述符本身，真是魔法一般的存在啊。

### 确保实例的数据只属于实例本身

你可能会像这样编写`NonNegative`描述符：

```python
class BrokenNonNegative(object):
    def __init__(self, default):
        self.value = default
 
    def __get__(self, instance, owner):
        return self.value
 
    def __set__(self, instance, value):
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self.value = value
 
class Foo(object):
    bar = BrokenNonNegative(5) 
 
f = Foo()
try:
    f.bar = -1
except ValueError:
    print "Caught the invalid assignment"
 
Caught the invalid assignment
```

这么做看起来似乎能正常工作。但这里的问题就在于所有`Foo`的实例都共享相同的`bar`，这会产生一些令人痛苦的结果：

```python
class Foo(object):
    bar = BrokenNonNegative(5) 
 
f = Foo()
g = Foo()
 
print "f.bar is %s\ng.bar is %s" % (f.bar, g.bar)
print "Setting f.bar to 10"
f.bar = 10
print "f.bar is %s\ng.bar is %s" % (f.bar, g.bar)  #ouch
f.bar is 5
g.bar is 5
Setting f.bar to 10
f.bar is 10
g.bar is 10
```

这就是为什么我们要在`NonNegative`中使用数据字典的原因。`__get__`和`__set__`的第一个参数告诉我们需要关心哪一个实例。`NonNegative`使用这个参数作为字典的`key`，为每一个`Foo`实例单独保存一份数据。

```python
class Foo(object):
    bar = NonNegative(5)
 
f = Foo()
g = Foo()
print "f.bar is %s\ng.bar is %s" % (f.bar, g.bar)
print "Setting f.bar to 10"
f.bar = 10
print "f.bar is %s\ng.bar is %s" % (f.bar, g.bar)  #better
f.bar is 5
g.bar is 5
Setting f.bar to 10
f.bar is 10
g.bar is 5
```

这就是描述符最令人感到别扭的地方（坦白的说，我不理解为什么Python不让你在实例的层次上定义描述符，并且总是需要将实际的处理分发给`__get__`和`__set__`。这么做行不通一定是有原因的）

### 注意不可哈希的描述符所有者

`NonNegative`类使用了一个字典来单独保存专属于实例的数据。这个一般来说是没问题的，除非你用到了不可哈希（unhashable）的对象：

```python
class MoProblems(list):  #you can't use lists as dictionary keys
    x = NonNegative(5)
 
m = MoProblems()
print m.x  # womp womp
 
TypeError
Traceback (most recent call last)
<ipython-input-8-dd73b177bd8d> in <module>()
      3 
      4 m = MoProblems()
----> 5 print m.x  # womp womp
 
<ipython-input-3-6671804ce5d5> in __get__(self, instance, owner)
      9         # instance = x
     10         # owner = type(x)
---> 11         return self.data.get(instance, self.default)
     12 
     13     def __set__(self, instance, value):
 
TypeError: unhashable type: 'MoProblems'
```

因为`MoProblems`的实例（`list`的子类）是不可哈希的，因此它们不能为`MoProblems`.`x`用做数据字典的key。有一些方法可以规避这个问题，但是都不完美。最好的方法可能就是给你的描述符加标签了。

```python
class Descriptor(object):
 
    def __init__(self, label):
        self.label = label
 
    def __get__(self, instance, owner):
        print '__get__', instance, owner
        return instance.__dict__.get(self.label)
 
    def __set__(self, instance, value):
        print '__set__'
        instance.__dict__[self.label] = value
 
class Foo(list):
    x = Descriptor('x')
    y = Descriptor('y')
 
f = Foo()
f.x = 5
print f.x
 
__set__
__get__ [] <class '__main__.Foo'>
5
```

这种方法依赖于Python的方法解析顺序（即，MRO）。我们给Foo中的每个描述符加上一个标签名，名称和我们赋值给描述符的变量名相同，比如`x = Descriptor(‘x’)`。之后，描述符将特定于实例的数据保存在`f.__dict__['x']`中。这个字典条目通常是当我们请求`f.x`时Python给出的返回值。然而，由于`Foo.x `是一个描述符，Python不能正常的使用`f.__dict__[‘x’]`，但是描述符可以安全的在这里存储数据。只是要记住，不要在别的地方也给这个描述符添加标签。

```python
class Foo(object):
    x = Descriptor('y')
 
f = Foo()
f.x = 5
print f.x
 
f.y = 4    #oh no!
print f.x
__set__
__get__ <__main__.Foo object at 0x10432c810> <class '__main__.Foo'>
5
__get__ <__main__.Foo object at 0x10432c810> <class '__main__.Foo'>
4
```

我不喜欢这种方式，因为这样的代码很脆弱也有很多微妙之处。但这个方法的确很普遍，可以用在不可哈希的所有者类上。David Beazley在他的[书](http://www.amazon.com/Python-Essential-Reference-4th-Edition/dp/0672329786/)中用到了这个方法。

### 在元类中使用带标签的描述符

由于描述符的标签名和赋给它的变量名相同，所以有人使用元类来自动处理这个簿记（bookkeeping）任务。

```python
class Descriptor(object):
    def __init__(self):
        #notice we aren't setting the label here
        self.label = None
 
    def __get__(self, instance, owner):
        print '__get__. Label = %s' % self.label
        return instance.__dict__.get(self.label, None)
 
    def __set__(self, instance, value):
        print '__set__'
        instance.__dict__[self.label] = value
 
class DescriptorOwner(type):
    def __new__(cls, name, bases, attrs):
        # find all descriptors, auto-set their labels
        for n, v in attrs.items():
            if isinstance(v, Descriptor):
                v.label = n
        return super(DescriptorOwner, cls).__new__(cls, name, bases, attrs)
 
class Foo(object):
    __metaclass__ = DescriptorOwner
    x = Descriptor()
 
f = Foo()
f.x = 10
print f.x
 
__set__
__get__. Label = x
10
```

我不会去解释有关元类的细节——参考文献中David Beazley已经在他的文章中解释的很清楚了。 需要指出的是元类自动的为描述符添加标签，并且和赋给描述符的变量名字相匹配。

尽管这样解决了描述符的标签和变量名不一致的问题，但是却引入了复杂的元类。虽然我很怀疑，但是你可以自行判断这么做是否值得。

### 访问描述符的方法

描述符仅仅是类，也许你想要为它们增加一些方法。举个例子，描述符是一个用来回调`property`的很好的手段。比如我们想要一个类的某个部分的状态发生变化时就立刻通知我们。下面的大部分代码是用来做这个的：

```python
class CallbackProperty(object):
    """A property that will alert observers when upon updates"""
    def __init__(self, default=None):
        self.data = WeakKeyDictionary()
        self.default = default
        self.callbacks = WeakKeyDictionary()
 
    def __get__(self, instance, owner):
        return self.data.get(instance, self.default)
 
    def __set__(self, instance, value):        
        for callback in self.callbacks.get(instance, []):
            # alert callback function of new value
            callback(value)
        self.data[instance] = value
 
    def add_callback(self, instance, callback):
        """Add a new function to call everytime the descriptor updates"""
        #but how do we get here?!?!
        if instance not in self.callbacks:
            self.callbacks[instance] = []
        self.callbacks[instance].append(callback)
 
class BankAccount(object):
    balance = CallbackProperty(0)
 
def low_balance_warning(value):
    if value < 100:
        print "You are poor"
 
ba = BankAccount()
 
# will not work -- try it
#ba.balance.add_callback(ba, low_balance_warning)
```

这是一个很有吸引力的模式——我们可以自定义回调函数用来响应一个类中的状态变化，而且完全无需修改这个类的代码。这样做可真是替人分忧解难呀。现在，我们所要做的就是调用`ba.balance.add_callback(ba, low_balance_warning)`，以使得每次`balance`变化时`low_balance_warning`都会被调用。

但是我们是如何做到的呢？当我们试图访问它们时，描述符总是会调用`__get__`。就好像`add_callback`方法是无法触及的一样！其实关键在于利用了一种特殊的情况，即，当从类的层次访问时，`__get__`方法的第一个参数是`None`。

```python
class CallbackProperty(object):
    """A property that will alert observers when upon updates"""
    def __init__(self, default=None):
        self.data = WeakKeyDictionary()
        self.default = default
        self.callbacks = WeakKeyDictionary()
 
    def __get__(self, instance, owner):
        if instance is None:
            return self        
        return self.data.get(instance, self.default)
 
    def __set__(self, instance, value):
        for callback in self.callbacks.get(instance, []):
            # alert callback function of new value
            callback(value)
        self.data[instance] = value
 
    def add_callback(self, instance, callback):
        """Add a new function to call everytime the descriptor within instance updates"""
        if instance not in self.callbacks:
            self.callbacks[instance] = []
        self.callbacks[instance].append(callback)
 
class BankAccount(object):
    balance = CallbackProperty(0)
 
def low_balance_warning(value):
    if value < 100:
        print "You are now poor"
 
ba = BankAccount()
BankAccount.balance.add_callback(ba, low_balance_warning)
 
ba.balance = 5000
print "Balance is %s" % ba.balance
ba.balance = 99
print "Balance is %s" % ba.balance
Balance is 5000
You are now poor
Balance is 99
```

## 个人总结

- 描述符伪装成类的属型，而当类的实例通过点操作符访问时，实际是就是调用描述符中三个方法之一
- 属性查找的顺序是:"类 -> 基类 -> 实例",并不是首先就在表示实例的那片内存中查找属性，而是首先在类中查找，因为python需要首先判断该'属性'是否是描述符(伪装的属性)，如果是描述符，那么则不是调用`__setattr__()`或者`__getattr__()`方法对`__dict__`字典进行处理，而是调用描述符的`__get__()`,`__set__()`和`__delete__()`方法
- 由于描述符只能作为类的属性，所以该类的多个实例都是公用的这个描述符，所以一般在描述符中的`__init__()`函数中创建一个字典，以类实例的地址(例子中的`instance`)参数作为key，以要这个实例的数据作为value
- 类中的普通方法第一个参数是`self`,因为实例化类时，会自动将分配给实例的内存地址传递该self，也就是所谓的绑定，该函数也就成为绑定函数了，而给实例动态添加的方法以及类之外定义的方法就不需要`self`参数了
- 以底层的思维了看待类和对象，都是内存中分配的地址空间而已，虽然有书上说类也是对象，但是不好理解，从底层就容易理解一些，先划分区域，并写入相应数据，然后这就是类，然后以这个类实例化时，就是再划分一块内存，写于相应数据(为了节省空间，不会完全复制类中的属性和方法，只会简单的赋值一些属性表示该对象是那个类的实例)，然后这就是类。类属性就是属性的值只在代表类的那块内存中，而不在代表对象的那块内存中

## 参考文档

- [如何理解 Python 的 Descriptor？](https://www.zhihu.com/question/25391709)
- [Python 的 descriptor（上）](https://segmentfault.com/a/1190000004478718)
- [Python描述符（descriptor）解密](http://www.geekfan.net/7862/)
- [Descriptor HowTo Guide](https://docs.python.org/3/howto/descriptor.html)