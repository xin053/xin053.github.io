---
title: furl链接解析库使用详解
date: 2016-11-13 21:53:46
categories:
- Python模块学习
tags:
- Python
- furl
---

## furl简介

```http
scheme://username:password@host:port/path?query#fragment
```

- **scheme** is the scheme string (all lowercase) or None. None means no scheme. An empty string means a protocol relative URL, like `//www.google.com`.
- **username** is the username string for authentication.
- **password** is the password string for authentication with **username**.
- **host** is the domain name, IPv4, or IPv6 address as a string. Domain names are all lowercase.
- **port** is an integer or None. A value of None means no port specified and the default port for the given **scheme** should be inferred, if possible.
- **path** is a Path object comprised of path segments.
- **query** is a Query object comprised of query arguments.
- **fragment** is a Fragment object comprised of a Path and Query object separated by an optional `?` separator.

```python
>>> from furl import furl
>>> f = furl('http://user:pass@www.google.com:99/')
>>> f.scheme, f.username, f.password, f.host, f.port
('http', 'user', 'pass', 'www.google.com', 99)
```

<!-- more -->

## furl使用

### 端口

会根据协议自动识别默认端口,目前仅支持ftp，ssh，http，https

```python
>>> f = furl('https://secure.google.com/')
>>> f.port
443

>>> f = furl('unknown://www.google.com/')
>>> print f.port
None
```

### netloc

```python
>>> furl('http://www.google.com/').netloc
'www.google.com'

>>> furl('http://www.google.com:99/').netloc
'www.google.com:99'

>>> furl('http://user:pass@www.google.com:99/').netloc
'user:pass@www.google.com:99'

>>> furl('http://www.baidu.com?username=zzx').netloc
'www.baidu.com'
```

### origin

```python
>>> furl('http://www.google.com/').origin
'http://www.google.com'

>>> furl('http://www.google.com:99/').origin
'http://www.google.com:99'

>>> furl('http://www.baidu.com?username=zzx').origin
'http://www.baidu.com'
```

### Path

```python
>>> f = furl('http://www.google.com/a/large%20ish/path')
>>> f.path
Path('/a/large ish/path')
>>> f.path.segments
['a', 'large ish', 'path']
>>> str(f.path)
'/a/large%20ish/path'

>>> f.path.segments = ['a', 'new', 'path', '']
>>> str(f.path)
'/a/new/path/'

>>> f.path = 'o/hi/there/with%20some%20encoding/'
>>> f.path.segments
['o', 'hi', 'there', 'with some encoding', '']
>>> str(f.path)
'/o/hi/there/with%20some%20encoding/'

>>> f.url
'http://www.google.com/o/hi/there/with%20some%20encoding/'

>>> f.path.segments = ['segments', 'are', 'maintained', 'decoded', '^`<>[]"#/?']
>>> str(f.path)
'/segments/are/maintained/decoded/%5E%60%3C%3E%5B%5D%22%23%2F%3F'
```

可以注意到链接末尾的`/`被解析为`''`,因为它被当作是一个目录:

```python
>>> f = furl('http://www.google.com/a/directory/')
>>> f.path.isdir
True
>>> f.path.isfile
False

>>> f = furl('http://www.google.com/a/file')
>>> f.path.isdir
False
>>> f.path.isfile
True
```

对path进行规范化:

```python
>>> f = furl('http://www.google.com////a/./b/lolsup/../c/')
>>> f.path.normalize()
>>> f.url
'http://www.google.com/a/b/c/'
```

### 参数处理

```python
>>> f = furl('http://www.google.com/?one=1&two=2')
>>> f.query
Query('one=1&two=2')
>>> f.query.params
omdict1D([('one', '1'), ('two', '2')])
>>> str(f.query)
'one=1&two=2'

>>> f = furl('http://www.google.com/?one=1&two=2')
>>> f.query.params
omdict1D([('one', '1'), ('two', '2')])
>>> f.args
omdict1D([('one', '1'), ('two', '2')])
>>> f.args is f.query.params
True
```

有关query属性的例子:

```python
>>> f = furl('http://www.google.com/?space=jams&space=slams')
>>> f.args['space']
'jams'
>>> f.args.getlist('space')
['jams', 'slams']
>>> f.args.addlist('repeated', ['1', '2', '3'])
>>> str(f.query)
'space=jams&space=slams&repeated=1&repeated=2&repeated=3'
>>> f.args.popvalue('space')
'slams'
>>> f.args.popvalue('repeated', '2')
'2'
>>> str(f.query)
'space=jams&repeated=1&repeated=3'
```

`''`与`None`参数:

```python
>>> f = furl('http://sprop.su')
>>> f.args['param'] = ''
>>> f.url
'http://sprop.su/?param='

>>> f = furl('http://sprop.su')
>>> f.args['param'] = None
>>> f.url
'http://sprop.su/?param'
```

### Fragment

```python
>>> f = furl('http://www.google.com/#/fragment/path?with=params')
>>> f.fragment
Fragment('/fragment/path?with=params')
>>> f.fragment.path
Path('/fragment/path')
>>> f.fragment.query
Query('with=params')
>>> f.fragment.separator
True

>>> f = furl('http://www.google.com/#/fragment/path?with=params')
>>> str(f.fragment)
'/fragment/path?with=params'
>>> f.fragment.path.segments.append('file.ext')
>>> str(f.fragment)
'/fragment/path/file.ext?with=params'

>>> f = furl('http://www.google.com/#/fragment/path?with=params')
>>> str(f.fragment)
'/fragment/path?with=params'
>>> f.fragment.args['new'] = 'yep'
>>> str(f.fragment)
'/fragment/path?new=yep&with=params'
```

fragment的分隔符是`?`

### Encoding

```python
>>> f = furl('http://www.google.com/')
>>> f.path = 'some encoding here'
>>> f.args['and some encoding'] = 'here, too'
>>> f.url
'http://www.google.com/some%20encoding%20here?and+some+encoding=here,+too'
>>> f.set(host=u'ドメイン.テスト', path=u'джк', query=u'☃=☺')
>>> f.url
'http://xn--eckwd4c7c.xn--zckzah/%D0%B4%D0%B6%D0%BA?%E2%98%83=%E2%98%BA'
```

### Inline manipulation

```python
>>> from furl import furl
>>> f = furl('http://www.google.com/?one=1&two=2')
>>> f.args['three'] = '3'
>>> del f.args['one']
>>> f.url
'http://www.google.com/?two=2&three=3'

>>> furl('http://www.google.com/?one=1').add({'two':'2'}).url
'http://www.google.com/?one=1&two=2'

>>> furl('http://www.google.com/?one=1&two=2').set({'three':'3'}).url
'http://www.google.com/?three=3'

>>> furl('http://www.google.com/?one=1&two=2').remove(['one']).url
'http://www.google.com/?two=2'

>>> url = 'http://www.google.com/#fragment' 
>>> furl(url).add(args={'example':'arg'}).set(port=99).remove(fragment=True).url
'http://www.google.com:99/?example=arg'

>>> f = furl().set(
...   scheme='https', host='secure.google.com', port=99, path='index.html',
...   args={'some':'args'}, fragment='great job')
>>> f.url
'https://secure.google.com:99/index.html?some=args#great%20job'

>>> url = 'https://secure.google.com:99/a/path/?some=args#great job'
>>> furl(url).remove(args=['some'], path='path/', fragment=True, port=True).url
'https://secure.google.com/a/'
```

### copy

**copy()** creates and returns a new furl object with an identical URL.

```python
>>> f = furl('http://www.google.com')
>>> f.copy().set(path='/new/path').url
'http://www.google.com/new/path'
>>> f.url
'http://www.google.com'
```

### join

```python
>>> f = furl('http://www.google.com')
>>> f.join('new/path').url
'http://www.google.com/new/path'
>>> f.join('replaced').url
'http://www.google.com/new/replaced'
>>> f.join('../parent').url
'http://www.google.com/parent'
>>> f.join('path?query=yes#fragment').url
'http://www.google.com/path?query=yes#fragment'
>>> f.join('unknown://www.yahoo.com/new/url/').url
'unknown://www.yahoo.com/new/url/'
```

## 参考文档

- [官方API文档](https://github.com/gruns/furl/blob/master/API.md)
- [github](https://github.com/gruns/furl)
