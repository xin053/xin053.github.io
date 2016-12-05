---
title: re正则库使用详解
date: 2016-12-01 16:00:45
categories: 
- Python模块学习
tags:
- Python
- re
---

## re简介

正则表达式会被python解释器编译成字节码，这样查找的效率比单纯用python代码实现查找要快，但是匹配统一内容可以有多种不同的正则表达式，并且他们的效率各不相同


## 特殊符号

```python
. ^ $ * + ? { } [ ] \ | ( )
```

匹配这些特殊符号需要使用`\`进行转义

<!-- more -->

### `.`

匹配除换行符以外的任意字符，如果指定了`DOTALL`标志，则匹配所有字符，但注意`.`表示仅仅匹配一个字符

```python
import re
re.findall(r'.', '\r\nabc')
# ['\r', 'a', 'b', 'c']
re.findall(r'.', '\r\nabc', flags=re.DOTALL)
# ['\r', '\n', 'a', 'b', 'c']
```

### `^`

匹配字符串的开始，当指定`MULTILINE`标志，则匹配每一行的开头

```python
re.findall(r'ab.', 'abcdefabhy')
# ['abc', 'abh']
re.findall(r'^ab.', 'abcdefabhy')
# ['abc']
re.findall(r'^ab.',
           '''abcd
           abcd
           acd
           abcd''')
# ['abc']
re.findall(r'^ab.',
           '''abcd
           abcd
           acd
           abcd''', flags=re.MULTILINE)
# ['abc', 'abc', 'abc']
```

### `### ### 

匹配字符串的结尾，当指定`MULTILINE`标志，则匹配每一行的结尾(匹配换行符之前的)

```python
re.findall(r'.ab$', 'aabcbab')
# ['bab']
re.findall(r'ab.$', 'aabcbab')
# []
re.findall(r'ab.$', 'aabcbab1\n') # 注意换行符不是结尾，换行符之前的才是结尾
# ['ab1']
```

### `*`

`*`表示0个或多个前一字符或正则

```python
re.findall(r'ab*c', 'ac.abc.abbbbc')
# ['ac', 'abc', 'abbbbc']
```

### `+`

`+`表示1个或多个前一字符或正则

```python
re.findall(r'ab+c', 'ac.abc.abbbbc')
# ['abc', 'abbbbc']
```

### `?`

`?`表示0个或1个前一字符或正则

```python
re.findall(r'ab?c', 'ac.abc.abbbbc')
# ['ac', 'abc']
```

### `*?` ` +?` ` ??` 

`*` `+` `?` 都是贪婪的，会匹配最长的

```python
re.findall(r'<.*>', '<a> b <c>')
# ['<a> b <c>']
```

在这些操作符后面添加`?`能够使之变为不贪婪的，也就是匹配最短的

```python
re.findall(r'<.*?>', '<a> b <c>')
# ['<a>', '<c>']
```

### `{m}`

`{m}`表示m个前一字符或正则

```python
re.findall(r'a{3}b', 'aabaaabaaaab')
# ['aaab', 'aaab']
```

### `{m,n}`

`{m,n}`表示m到n个前一字符或正则  注意:`,`后面没有空格

```python
re.findall(r'a{2,3}b', 'aabaaabaaaab')
# ['aab', 'aaab', 'aaab']
```

省略m表示没有下限，省略n表示没有上限

```python
re.findall(r'a{,3}b', 'babaabaaabaaaab')
# ['b', 'ab', 'aab', 'aaab', 'aaab']
re.findall(r'a{2,}b', 'babaabaaabaaaab')
# ['aab', 'aaab', 'aaaab']
```

### `{m,n}?`

`{m,n}`会匹配最长的，在后面加`?`，则匹配最短的

```python
re.findall(r'a{2,4}', 'aaaa')
# ['aaaa']
re.findall(r'a{2,4}?', 'aaaa')
# ['aa', 'aa']
```

### `[]`

`[]`指定一组字符

```python
re.findall(r'[a-z]', 'adfzADFZ059')
# ['a', 'd', 'f', 'z']
re.findall(r'[a-zA-Z0-9]', 'adfzADFZ059')
# ['a', 'd', 'f', 'z', 'A', 'D', 'F', 'Z', '0', '5', '9']
```

很多特殊符号在`[]`环境内无效,其他特殊符号需要转义:

```python
re.findall(r'[.$*+?{}|()]', '.^$*+?{}[]\|()')
# ['.', '$', '*', '+', '?', '{', '}', '|', '(', ')']
```

`[]`内的`^`表示非，`^^`表示除`^`以外的全部字符:

```python
re.findall(r'[^5]', '1359')
# ['1', '3', '9']
re.findall(r'[^^]', '1359^')
# ['1', '3', '5', '9']
```

### `|`

`|`也就是或，注意也是短路操作

```python
re.findall(r'a|bc', 'acbcabc')
# ['a', 'bc', 'a', 'bc']
re.findall(r'[a|b]c', 'acbcabc')
# ['ac', 'bc', 'bc']
```

### `(...)`

匹配圆括号里的RE匹配的内容，并指定组的开始和结束位置。组里面的内容可以被提取,要匹配`(`和`)`，则需要使用转义符号或者是`[(]`,`[)]`

### `(?aiLmsux)`

`i`,`L`,`m`,`s`,`u`,`x`里的一个或多个字母。表达式不匹配任何字符，但是指定相应的标志：`re.I`(忽略大小写)、`re.L`(依赖locale)、`re.M`(多行模式)、`re.S`(.匹配所有字符)、`re.U`(依赖Unicode)、`re.X`(详细模式)

```python
re.findall(r'(?i)ab', 'abABAbaB')
# ['ab', 'AB', 'Ab', 'aB']
```

### `(?P<name>...)`

和普通的圆括号类似，但是子串匹配到的内容将可以用命名的`name`参数来提取。组的`name`必须是有效的python标识符，而且在本表达式内不重名。命名了的组和普通组一样，也用数字来提取，也就是说名字只是个额外的属性。

```python
m = re.match('(?P<name>\w+)', 'zzx:22')
m.group('name')
# 'zzx'
m.group(1)
# 'zzx'
```

## special sequences 

### `\number`

表示之前的分组

```python
re.match(r'(.+) \1 (abc) \2', '55 55 abc abc')
# <_sre.SRE_Match object; span=(0, 13), match='55 55 abc abc'>
```

### `\A`

仅匹配字符串的开头

```python
re.findall(r'\Aabc', 'abcabc')
# ['abc']
```

### `\b`

表示单词开始和结尾处的空白字符以及非字母非数字的字符

```python
re.findall(r'\babc\b', 'abc.')
# ['abc']
re.findall(r'\babc\b', 'abc!')
# ['abc']
re.findall(r'\babc\b', 'abca')
# []
```

### `\B`

`\b`的反面

```python
re.findall(r'py\B', 'python')
# ['py']
re.findall(r'py\B', 'py.')
# []
```

### `\s`

匹配空白字符,包括`[ \t\n\r\f\v] `

```python
re.findall(r'aa\s+bb', 'aa \n\t  bb')
# ['aa \n\t  bb']
```

### `\S`

`\s`的反面

```python
re.findall(r'aa\S+bb', 'aahg.!bb')
# ['aahg.!bb']
re.findall(r'aa\S+bb', 'aa bb')
# []
```

### `\w`

匹配数字和字母

```python
re.findall(r'\w+', 'aa3bb 45AS')
# ['aa3bb', '45AS']
```

### `\W`

`\w`的反面

```python
re.findall(r'\W+', 'aa3bb .! 45AS')
# [' .! ']
```

### `\Z`

匹配字符串结尾

```python
re.findall(r'ab\Z', 'abab')
# ['ab']
```

## `re`模块方法

### `re.compile(pattern, flags=0) `

编译一个正则表达式为一个正则表达式对象，之后就可以使用该对象对字符串进行匹配了

### `re.search(pattern, string, flags=0) `

从字符串的开头开始搜索匹配，返回匹配到的第一个

### `re.match(pattern, string, flags=0) `

返回字符串中匹配的第一个

### `re.fullmatch(pattern, string, flags=0) `

对整个字符串进行匹配

### `re.split(pattern, string, maxsplit=0, flags=0) `

凭正则表达式分割字符串

### `re.findall(pattern, string, flags=0) `

如果匹配模式中包含分组，则返回分组，如果有多个分组，则返回分组组成的元组

### `re.finditer(pattern, string, flags=0) `

返回迭代器

### `re.sub(pattern, repl, string, count=0, flags=0) `

替换

## Match Objects

像`match()` `search()`等方法返回的就是一个`Match`对象，该对象包括的属性和方法请看[官方文档](https://docs.python.org/3/library/re.html#match-objects)

注意，关于分组，第0组就是匹配到的字符串

```python
a = re.match(r'\babc\b', 'abc!')
a.group()
# 'abc'
```

## 参考文档

- [re官方文档](https://docs.python.org/3/library/re.html)
- [Regular Expression HOWTO](https://docs.python.org/3/howto/regex.html)