---
title: BeautifulSoup html与xml解析库使用详解
date: 2016-11-14 15:38:10
categories:
- Python模块学习
tags:
- Python
- BeautifulSoup
---

## BeautifulSoup简介

BeautifulSoup 3只支持python 2，并且已经停止开发，BeautifulSoup支持python2和3，以下使用方法参考4.4版说明文档

![](http://www.crummy.com/software/BeautifulSoup/bs4/doc/_images/6.1.jpg)

<!-- more -->

## BeautifulSoup使用

### 解析器比较

| 解析器           | 使用方法                                     | 优势                            | 劣势                                  |
| ------------- | ---------------------------------------- | ----------------------------- | ----------------------------------- |
| Python标准库     | `BeautifulSoup(markup,"html.parser")`    | Python的内置标准库执行速度适中文档容错能力强     | Python 2.7.3 or 3.2.2)前 的版本中文档容错能力差 |
| lxml HTML 解析器 | `BeautifulSoup(markup,"lxml")`           | 速度快文档容错能力强                    | 需要安装C语言库                            |
| lxml XML 解析器  | `BeautifulSoup(markup,["lxml-xml"])``BeautifulSoup(markup,"xml")` | 速度快唯一支持XML的解析器                | 需要安装C语言库                            |
| html5lib      | `BeautifulSoup(markup,"html5lib")`       | 最好的容错性以浏览器的方式解析文档生成HTML5格式的文档 | 速度慢不依赖外部扩展                          |

如果不指定解析器，BeautifulSoup会自动选择最合适的解析器来解析文档

### 对象种类

Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种: `Tag` , `NavigableString` , `BeautifulSoup` , `Comment` .

#### Tag

`Tag` 对象与XML或HTML原生文档中的tag相同:

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)
# <class 'bs4.element.Tag'>
str(tag)
# '<b class="boldest">Extremely bold</b>'
```

每个tag都有name和attribute:

```python
tag.name
# 'b'
tag.attrs
# {'class': ['boldest']}
tag['class']
# ['boldest']
```

可以通过直接赋值来增加或修改tag的名字和属性:

```python
tag.name = "blockquote"
tag
# <blockquote class="boldest">Extremely bold</blockquote>

tag['class'] = 'verybold'
tag['id'] = 1
tag
# <blockquote class="verybold" id="1">Extremely bold</blockquote>
```

通过`del`删除属性:

```python
del tag['class']
del tag['id']
tag
# <blockquote>Extremely bold</blockquote>
print(tag.get('class'))
# None
```

对于多值属性,会返回一个列表，使用的时候注意是返回列表还是字符串:

```python
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.p['class']
# ["body", "strikeout"]

css_soup = BeautifulSoup('<p class="body"></p>')
css_soup.p['class']
# ["body"]
```

如果转换的文档是XML格式,那么tag中不包含多值属性

```python
xml_soup = BeautifulSoup('<p class="body strikeout"></p>', 'xml')
xml_soup.p['class']
# 'body strikeout'
```

#### NavigableString

字符串常被包含在tag内.Beautiful Soup用 `NavigableString` 类来包装tag中的字符串:

```python
tag.string
# 'Extremely bold'
type(tag.string)
# <class 'bs4.element.NavigableString'>
```

tag中包含的字符串不能编辑,但是可以被替换成其它的字符串,用 [replace_with()](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#replace-with) 方法:

```python
tag.string.replace_with("No longer bold")
tag
# <blockquote class="verybold" id="1">No longer bold</blockquote>
```

如果想在Beautiful Soup之外使用 `NavigableString` 对象,需要调用 `unicode()` 方法,将该对象转换成普通的Unicode字符串,否则就算Beautiful Soup已方法已经执行结束,该对象的输出也会带有对象的引用地址.这样会浪费内存.

#### BeautifulSoup

`BeautifulSoup` 对象表示的是一个文档的全部内容.大部分时候,可以把它当作 `Tag` 对象,它支持 [遍历文档树](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id18) 和 [搜索文档树](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id27) 中描述的大部分的方法.

因为 `BeautifulSoup` 对象并不是真正的HTML或XML的tag,所以它没有name和attribute属性.但有时查看它的 `.name` 属性是很方便的,所以 `BeautifulSoup` 对象包含了一个值为 “[document]” 的特殊属性 `.name`

```python
soup.name
# '[document]'
```

#### Comment

```python
markup = "<b><!--Hey, buddy. Want to buy a used parser?--></b>"
soup = BeautifulSoup(markup)
comment = soup.b.string
type(comment)
# <class 'bs4.element.Comment'>
```

`Comment` 对象是一个特殊类型的 `NavigableString` 对象:

```python
comment
# 'Hey, buddy. Want to buy a used parser'
```

但是当它出现在HTML文档中时, `Comment` 对象会使用特殊的格式输出:

```python
print(soup.b.prettify())
# <b>
#  <!--Hey, buddy. Want to buy a used parser?-->
# </b>
```

### 遍历文档树

我们测试的文档内容:

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
    <body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')
```

通过点取属性的方式只能获得当前名字的第一个tag:

```python
soup.body.b
# <b>The Dormouse's story</b>
soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```

使用`find_all()`获取所有的tag:

```python
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

tag的 `.contents` 属性可以将tag的子节点以列表的方式输出:

```python
head_tag = soup.head
head_tag
# <head><title>The Dormouse's story</title></head>

head_tag.contents
[<title>The Dormouse's story</title>]

title_tag = head_tag.contents[0]
title_tag
# <title>The Dormouse's story</title>
title_tag.contents
# ['The Dormouse's story']
```

字符串没有 `.contents` 属性,因为字符串没有子节点:

```python
text = title_tag.contents[0]
text.contents
# AttributeError: 'NavigableString' object has no attribute 'contents'
```

通过tag的 `.children` 生成器,可以对tag的子节点进行循环:

```python
for child in title_tag.children:
    print(child)
    # The Dormouse's story
```

`.descendants` 属性可以对所有tag的子孙节点进行递归循环

`BeautifulSoup` 有一个直接子节点(<html>节点),却有很多子孙节点:

```python
len(list(soup.children))
# 1
len(list(soup.descendants))
# 25
```

输出所有`string`:

```python
for string in soup.stripped_strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u"The Dormouse's story"
    # u'Once upon a time there were three little sisters; and their names were'
    # u'Elsie'
    # u','
    # u'Lacie'
    # u'and'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'...'
```

通过 `.parent` 属性来获取某个元素的父节点.

通过元素的 `.parents` 属性可以递归得到元素的所有父辈节点

在文档树中,使用 `.next_sibling` 和 `.previous_sibling` 属性来查询兄弟节点

通过 `.next_siblings` 和 `.previous_siblings` 属性可以对当前节点的兄弟节点迭代输出

`.next_element` 属性指向解析过程中下一个被解析的对象(字符串或tag),结果可能与 `.next_sibling`相同,但通常是不一样的.

通过 `.next_elements` 和 `.previous_elements` 的迭代器就可以向前或向后访问文档的解析内容,就好像文档正在被解析一样

### 搜索文档树

除了`find_all()`之外，搜索也支持正则表达式:

```python
import re
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
# body
# b
```

下面代码找到文档中所有`<a>`标签和`<b>`标签:

```python
soup.find_all(["a", "b"])
# [<b>The Dormouse's story</b>,
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

`True` 可以匹配任何值,下面代码查找到所有的tag,但是不会返回字符串节点

```python
for tag in soup.find_all(True):
    print(tag.name)
# html
# head
# title
# body
# p
# b
# p
# a
# a
# a
# p
```

如果包含一个名字为 `id` 的参数,Beautiful Soup会搜索每个tag的”id”属性.

```python
soup.find_all(id='link2')
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

如果传入 `href` 参数,Beautiful Soup会搜索每个tag的”href”属性:

```python
soup.find_all(href=re.compile("elsie"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```

下面的例子在文档树中查找所有包含 `id` 属性的tag,无论 `id` 的值是什么:

```python
soup.find_all(id=True)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

使用多个指定名字的参数可以同时过滤tag的多个属性:

```python
soup.find_all(href=re.compile("elsie"), id='link1')
# [<a class="sister" href="http://example.com/elsie" id="link1">three</a>]
```

通过 `string` 参数可以搜搜文档中的字符串内容.与 `name` 参数的可选值一样, `string` 参数接受 [字符串](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id30) , [正则表达式](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id31) , [列表](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id32), [True](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#true) . 看例子:

```python
soup.find_all(string="Elsie")
# [u'Elsie']

soup.find_all(string=["Tillie", "Elsie", "Lacie"])
# [u'Elsie', u'Lacie', u'Tillie']

soup.find_all(string=re.compile("Dormouse"))
[u"The Dormouse's story", u"The Dormouse's story"]

def is_the_only_string_within_a_tag(s):
    ""Return True if this string is the only child of its parent tag.""
    return (s == s.parent.string)

soup.find_all(string=is_the_only_string_within_a_tag)
# [u"The Dormouse's story", u"The Dormouse's story", u'Elsie', u'Lacie', u'Tillie', u'...']
```

虽然 `string` 参数用于搜索字符串,还可以与其它参数混合使用来过滤tag.Beautiful Soup会找到`.string` 方法与 `string` 参数值相符的tag.下面代码用来搜索内容里面包含“Elsie”的`<a>`标签:

```python
soup.find_all("a", string="Elsie")
# [<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>]
```

限制返回结果的个数:

```python
soup.find_all("a", limit=2)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

下面两行代码是等价的:

```python
soup.find_all("a")
soup("a")
```

这两行代码也是等价的:

```python
soup.title.find_all(string=True)
soup.title(string=True)
```

`find_all()` 和 `find()` 只搜索当前节点的所有子节点,孙子节点等. `find_parents()` 和`find_parent()` 用来搜索当前节点的父辈节点

`find_next_siblings()` 方法返回所有符合条件的后面的兄弟节点, `find_next_sibling()` 只返回符合条件的后面的第一个tag节点.

`find_previous_siblings()` 方法返回所有符合条件的前面的兄弟节点, `find_previous_sibling()` 方法返回第一个符合条件的前面的兄弟节点

`find_all_next()`方法返回所有符合条件的节点, `find_next()` 方法返回第一个符合条件的节点

`find_all_previous()` 方法返回所有符合条件的节点, `find_previous()` 方法返回第一个符合条件的节点.

CSS选择器:对于熟悉css选择器的开发人员来说，使用这种方法来查找比较简单:

```python
soup.select("title")
# [<title>The Dormouse's story</title>]

soup.select("p nth-of-type(3)")
# [<p class="story">...</p>]
```

通过tag标签逐层查找:

```python
soup.select("body a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("html head title")
# [<title>The Dormouse's story</title>]
```

找到某个tag标签下的直接子标签 [[6\]](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id93) :

```python
soup.select("head > title")
# [<title>The Dormouse's story</title>]

soup.select("p > a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("p > a:nth-of-type(2)")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

soup.select("p > #link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("body > a")
# []
```

找到兄弟节点标签:

```python
soup.select("#link1 ~ .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie"  id="link3">Tillie</a>]

soup.select("#link1 + .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

通过CSS的类名查找:

```python
soup.select(".sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("[class~=sister]")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

通过tag的id查找:

```python
soup.select("#link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("a#link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

同时用多种CSS选择器查询元素:

```python
soup.select("#link1,#link2")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

通过是否存在某个属性来查找:

```python
soup.select('a[href]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

通过属性的值来查找:

```python
soup.select('a[href="http://example.com/elsie"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select('a[href^="http://example.com/"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href$="tillie"]')
# [<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href*=".com/el"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```

返回查找到的元素的第一个

```python
soup.select_one(".sister")
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```

### 修改文档树

`Tag.insert()` 方法与 `Tag.append()` 方法类似,区别是不会把新元素添加到父节点 `.contents` 属性的最后,而是把元素插入到指定的位置.与Python列表总的 `.insert()` 方法的用法下同:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
tag = soup.a

tag.insert(1, "but did not endorse ")
tag
# <a href="http://example.com/">I linked to but did not endorse <i>example.com</i></a>
tag.contents
# [u'I linked to ', u'but did not endorse', <i>example.com</i>]
```

`Tag.clear()` 方法移除当前tag的内容:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
tag = soup.a

tag.clear()
tag
# <a href="http://example.com/"></a>
```

`PageElement.extract()` 方法将当前tag移除文档树,并作为方法结果返回:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

i_tag = soup.i.extract()

a_tag
# <a href="http://example.com/">I linked to</a>

i_tag
# <i>example.com</i>

print(i_tag.parent)
None
```

这个方法实际上产生了2个文档树: 一个是用来解析原始文档的 `BeautifulSoup` 对象,另一个是被移除并且返回的tag.被移除并返回的tag可以继续调用 `extract` 方法:

```python
my_string = i_tag.string.extract()
my_string
# u'example.com'

print(my_string.parent)
# None
i_tag
# <i></i>
```

`Tag.decompose()` 方法将当前节点移除文档树并完全销毁:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

soup.i.decompose()

a_tag
# <a href="http://example.com/">I linked to</a>
```

`PageElement.replace_with()` 方法移除文档树中的某段内容,并用新tag或文本节点替代它:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

new_tag = soup.new_tag("b")
new_tag.string = "example.net"
a_tag.i.replace_with(new_tag)

a_tag
# <a href="http://example.com/">I linked to <b>example.net</b></a>
```

`replace_with()` 方法返回被替代的tag或文本节点,可以用来浏览或添加到文档树其它地方

`PageElement.wrap()` 方法可以对指定的tag元素进行包装,并返回包装后的结果:

```python
soup = BeautifulSoup("<p>I wish I was bold.</p>")
soup.p.string.wrap(soup.new_tag("b"))
# <b>I wish I was bold.</b>

soup.p.wrap(soup.new_tag("div"))
# <div><p><b>I wish I was bold.</b></p></div>
```

该方法在 Beautiful Soup 4.0.5 中添加

`Tag.unwrap()` 方法与 `wrap()` 方法相反.将移除tag内的所有tag标签,该方法常被用来进行标记的解包:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

a_tag.i.unwrap()
a_tag
# <a href="http://example.com/">I linked to example.com</a>
```

与 `replace_with()` 方法相同, `unwrap()` 方法返回被移除的tag

## 参考文档

- [官网文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
- [中文4.4文档](https://beautifulsoup.readthedocs.io/zh_CN/latest/)