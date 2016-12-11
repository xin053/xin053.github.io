---
title: shortuuid库使用详解
date: 2016-07-07 01:01:31
categories:
- Python模块学习
tags:
- Python
- shortuuid
---

## shortuuid简介
UUID是指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。在UUID的算法中，可能会用到诸如网卡MAC地址，IP，主机名，进程ID等信息以保证其独立性。

UUID的唯一缺陷在于生成的结果串会比较长。关于UUID这个标准使用最普遍的是微软的GUID(Globals Unique Identifiers)。在ColdFusion中可以用CreateUUID()函数很简单地生成UUID，其格式为：xxxxxxxx-xxxx- xxxx-xxxxxxxxxxxxxxxx(8-4-4-16)，其中每个 x 是 0-9 或 a-f 范围内的一个十六进制的数字。而标准的UUID格式为：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx (8-4-4-4-12)

shortuuid使用python内置的uuid模块产生标准32位uuid，然后进行base57编码，使用的字符集为数字，字母大小写，然后再除去结果中相近的字符：` l, 1, I, O 和 0`

<!-- more -->

## 基本使用
可以使用shortuuid模块的`uuid()`方法快速产生uuid:

```python
>>> import shortuuid
>>> shortuuid.uuid()
'vytxeTZskVKR7C7WgdSP3d'
>>> shortuuid.uuid()
'c6Xi3vjP6DpyLF5WipPSD4'
```

也可以以一个name来产生uuid:

```python
>>> shortuuid.uuid(name="example.com")
'wpsWLdLt9nscn2jbTD3uxe'
>>> shortuuid.uuid(name="example.com")
'wpsWLdLt9nscn2jbTD3uxe'
>>> shortuuid.uuid(name="http://example.com")
'c8sh5y9hdSMS6zVnrvf53T'
```

也可以产生定长的uuid:

```python
>>> shortuuid.uuid(pad_length=32)
'jy2ksEmHC46d2uvrvZxMEG2222222222'
>>> shortuuid.uuid(pad_length=32)
'2HxCkUWH9QgeVPMptqhv2T2222222222'
>>> shortuuid.uuid(pad_length=40)
'pmmFaPm7EyMHCqWgYat8M8222222222222222222'
```

指定`pad_length`将会使用字符集中的第一个字符填充为指定长度

获取使用的字符集:

```python
>>> shortuuid.get_alphabet()
'23456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz'
```

设置字符集:

```python
>>> shortuuid.set_alphabet("aaaaabcdefgh1230123")
>>> shortuuid.uuid()
'0agee20aa1hehebcagddhedddc0d2chhab3b'

>>> shortuuid.get_alphabet()
'0123abcdefgh'
```

shortuuid会自动去除字符集中重复的字符

`encode()`与`decode()`

```python
>>> import uuid ; u = uuid.uuid4() ; u
UUID('6ca4f0f8-2508-4bac-b8f1-5d1e3da2247a')
>>> s = shortuuid.encode(u) ; s
'cu8Eo9RyrUsV4MXEiDZpLM'
>>> shortuuid.decode(s) == u
True

>>> shortuuid.decode(shortuuid.uuid())
UUID('7f4e0771-d675-42b7-a1ff-dff3386d3de8')
>>> shortuuid.decode(shortuuid.uuid())
UUID('8a2dbc65-d688-474c-a3fa-4e12db6ce4a7')
>>> shortuuid.decode(shortuuid.uuid())
UUID('37e2d25e-aeb0-48be-84ea-cb41793a1228')
>>> shortuuid.decode(shortuuid.uuid(name='zzx'))
UUID('041f18b2-18fc-5d7b-a8a7-b21c3843e9e7')
>>> shortuuid.decode(shortuuid.uuid(name='zzx'))
UUID('041f18b2-18fc-5d7b-a8a7-b21c3843e9e7')
```

## ShortUUID类
当你需要在每个线程中有不用的字符集来产生uuid时，可以使用`ShortUUID`类

```python
>>> su = shortuuid.ShortUUID(alphabet="01345678")
>>> su.uuid()
'034636353306816784480643806546503818874456'
>>> su.get_alphabet()
'01345678'
>>> su.set_alphabet("21345687654123456")
>>> su.get_alphabet()
'12345678'
```

当然`ShortUUID`类也有这些方法：

- encode()
- decode()
- uuid()
- random()
- get_alphabet()
- set_alphabet()
- encoded_length()