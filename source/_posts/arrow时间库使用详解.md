---
title: arrow时间库使用详解
date: 2016-07-04 19:43:58
categories:
- Python模块学习
tags:
- Python
- arrow
---

## arrow简介
arrow是一个提供了更易懂和友好的方法来创建、操作、格式化和转化日期、时间和时间戳的python库。可以完全替代datetime，支持python2和3

## 基本使用

```python
>>> import arrow
>>> utc = arrow.utcnow()
>>> utc
<Arrow [2013-05-11T21:23:58.970460+00:00]>

>>> utc = utc.replace(hours=-1)
>>> utc
<Arrow [2013-05-11T20:23:58.970460+00:00]>

>>> local = utc.to('US/Pacific')
>>> local
<Arrow [2013-05-11T13:23:58.970460-07:00]>

>>> arrow.get('2013-05-11T21:23:58.970460+00:00')
<Arrow [2013-05-11T21:23:58.970460+00:00]>

>>> local.timestamp
1368303838

>>> local.format()
'2013-05-11 13:23:58 -07:00'

>>> local.format('YYYY-MM-DD HH:mm:ss ZZ')
'2013-05-11 13:23:58 -07:00'

>>> local.humanize()
'an hour ago'

>>> local.humanize(locale='ko_kr')
'1시간 전'
```

<!-- more -->

## arrow对象的创建
### 以当前时间获取arrow对象

```python
>>> arrow.now()
<Arrow [2016-07-04T20:34:26.918272+08:00]>
>>> arrow.utcnow()
<Arrow [2016-07-04T12:34:34.296419+00:00]>
>>> arrow.now('US/Pacific')
<Arrow [2016-07-04T05:35:19.395502-07:00]>
```

默认的`now()`便会以系统时区作为arrow对象的时区，同时是本地时间格式。而`utcnow()`获取的是世界标准时间格式，所以一般情况下我们使用最多的就是`arrow.now()`

### 以指定时间戳获取arrow对象

```python
>>> arrow.get(1367900664)
<Arrow [2013-05-07T04:24:24+00:00]>

>>> arrow.get('1367900664')
<Arrow [2013-05-07T04:24:24+00:00]>

>>> arrow.get(1367900664.152325)
<Arrow [2013-05-07T04:24:24.152325+00:00]>

>>> arrow.get('1367900664.152325')
<Arrow [2013-05-07T04:24:24.152325+00:00]>
```

### 其他获取arrow对象的方式

```python
>>> arrow.get('2013-05-05 12:30:45', 'YYYY-MM-DD HH:mm:ss')
<Arrow [2013-05-05T12:30:45+00:00]>

>>> arrow.get('2013-09-30T15:34:00.000-07:00')
<Arrow [2013-09-30T15:34:00-07:00]>

>>> arrow.get(2016,7,4)
<Arrow [2016-07-04T00:00:00+00:00]>
>>> arrow.get(2016,7,4,20)
<Arrow [2016-07-04T20:00:00+00:00]>
>>> arrow.get(2016,7,4,20,14)
<Arrow [2016-07-04T20:14:00+00:00]>
>>> arrow.get(2016,7,4,20,14,15)
<Arrow [2016-07-04T20:14:15+00:00]>

>>> arrow.Arrow(2013, 5, 5)
<Arrow [2013-05-05T00:00:00+00:00]>
```

## arrow对象的属性

```python
>>> a = arrow.now()
>>> a
<Arrow [2016-07-04T20:17:09.633154+08:00]>

>>> a.timestamp
1467634629

>>> a.year
2016
>>> a.month
7
>>> a.day
4
>>> a.hour
20
>>> a.minute
17
>>> a.second
9
>>> a.microsecond
633154
>>> a.week
27
```

## 时间的计算
### 基本计算

```python
>>> a = arrow.now()
>>> a
<Arrow [2016-07-04T20:17:09.633154+08:00]>

>>> a.replace(hour=4)
<Arrow [2016-07-04T04:17:09.633154+08:00]>

>>> a.replace(hour=4)
<Arrow [2016-07-04T04:17:09.633154+08:00]>

>>> a.replace(hours=4)
<Arrow [2016-07-05T00:17:09.633154+08:00]>

>>> b = a.replace(hour=4)
>>> b
<Arrow [2016-07-04T04:17:09.633154+08:00]>
>>> b.replace(hours=4)
<Arrow [2016-07-04T08:17:09.633154+08:00]>

>>> a
<Arrow [2016-07-04T20:17:09.633154+08:00]>
```

需要注意的是`replace()`方法是产生一个新的arrow对象，所以原来的`a`没变，另外注意`hour`与`hours`的区别，前者是设置时间，取值为0-23，而后者是在原来时间的基础上加减，取值可正可负。

### 转换为指定时间格式

```python
>>> arrow.now().format('YYYY-MM-DD HH:mm:ss ZZ')
'2016-07-04 20:32:01 +08:00'
```

### 本地时间与标准时间的转换

```python
>>> a = arrow.now()
>>> a
<Arrow [2016-07-04T20:17:09.633154+08:00]>
>>> a.to('local')
<Arrow [2016-07-04T20:17:09.633154+08:00]>
>>> a.to('utc')
<Arrow [2016-07-04T12:17:09.633154+00:00]>
>>> a
<Arrow [2016-07-04T20:17:09.633154+08:00]>
```

同样`to()`也是产生新的arrow对象，对原对象没有影响

## Ranges & spans

Get the time span of any unit:

```python
>>> arrow.utcnow().span('hour')
(<Arrow [2013-05-07T05:00:00+00:00]>, <Arrow [2013-05-07T05:59:59.999999+00:00]>)
```

Or just get the floor and ceiling:

```python
>>> arrow.utcnow().floor('hour')
<Arrow [2013-05-07T05:00:00+00:00]>

>>> arrow.utcnow().ceil('hour')
<Arrow [2013-05-07T05:59:59.999999+00:00]>
```

You can also get a range of time spans:

```python
>>> start = datetime(2013, 5, 5, 12, 30)
>>> end = datetime(2013, 5, 5, 17, 15)
>>> for r in arrow.Arrow.span_range('hour', start, end):
...     print r
...
(<Arrow [2013-05-05T12:00:00+00:00]>, <Arrow [2013-05-05T12:59:59.999999+00:00]>)
(<Arrow [2013-05-05T13:00:00+00:00]>, <Arrow [2013-05-05T13:59:59.999999+00:00]>)
(<Arrow [2013-05-05T14:00:00+00:00]>, <Arrow [2013-05-05T14:59:59.999999+00:00]>)
(<Arrow [2013-05-05T15:00:00+00:00]>, <Arrow [2013-05-05T15:59:59.999999+00:00]>)
(<Arrow [2013-05-05T16:00:00+00:00]>, <Arrow [2013-05-05T16:59:59.999999+00:00]>)
```

Or just iterate over a range of time:

```python
>>> start = datetime(2013, 5, 5, 12, 30)
>>> end = datetime(2013, 5, 5, 17, 15)
>>> for r in arrow.Arrow.range('hour', start, end):
...     print repr(r)
...
<Arrow [2013-05-05T12:30:00+00:00]>
<Arrow [2013-05-05T13:30:00+00:00]>
<Arrow [2013-05-05T14:30:00+00:00]>
<Arrow [2013-05-05T15:30:00+00:00]>
<Arrow [2013-05-05T16:30:00+00:00]>
```

## Tokens

|&nbsp; 	|Token	|Output|
|:-----:|:-----:|:----:|
|Year	|YYYY	|2000, 2001, 2002 ... 2012, 2013|
| 	&nbsp;    |YY	    |00, 01, 02 ... 12, 13|
|Month	|MMMM	|January, February, March ...|
|&nbsp; 	|MMM	|Jan, Feb, Mar ...|
|&nbsp; 	|MM|	01, 02, 03 ... 11, 12|
|&nbsp; 	|M|	1, 2, 3 ... 11, 12|
|Day of Year|	DDDD|	001, 002, 003 ... 364, 365|
|&nbsp; 	|DDD|	1, 2, 3 ... 4, 5|
|Day of Month|	DD|	01, 02, 03 ... 30, 31|
|&nbsp;| 	D|	1, 2, 3 ... 30, 31|
|Day of Week|	dddd|	Monday, Tuesday, Wednesday ...|
|&nbsp; 	|ddd	|Mon, Tue, Wed ...|
|&nbsp; 	|d	|1, 2, 3 ... 6, 7|
|Hour|	HH|	00, 01, 02 ... 23, 24|
|&nbsp; 	|H|	0, 1, 2 ... 23, 24|
|&nbsp;| 	hh|	01, 02, 03 ... 11, 12|
|&nbsp; 	|h|	1, 2, 3 ... 11, 12|
|AM / PM|	A|	AM, PM|
|&nbsp; 	|a|	am, pm|
|Minute|	mm|	00, 01, 02 ... 58, 59|
|&nbsp; |	m|	0, 1, 2 ... 58, 59|
|Second|	ss|	00, 01, 02 ... 58, 59|
|&nbsp; 	|s|	0, 1, 2 ... 58, 59|
|Sub-second|	SSS|	000, 001, 002 ... 998, 999|
|&nbsp; 	|SS|	00, 01, 02 ... 98, 99|
|&nbsp;| 	S|	0, 1, 2 ... 8, 9|
|Timezone|	ZZ|	-07:00, -06:00 ... +06:00, +07:00|
|&nbsp;| 	Z|	-0700, -0600 ... +0600, +0700|
|Timestamp|	X|	1381685817|

## API Guide

- [API Guide](https://arrow.readthedocs.io/en/latest/#api-guide)

## 其他相关文章

- [arrow官方文档](https://arrow.readthedocs.io/en/latest/#api-guide)