---
title: tablib库使用详解
date: 2016-07-10 23:58:34
categories: 
- Python模块学习
tags:
- Python
- tablib
---

## tablib简介
tablib为requests作者`kennethreitz`维护，支持python2到python3.简单的说就是一个通用的数据集，操作类似数据库，但又不是一个数据库的代替，因为缺少查操作，但是可以通过tablib将数据集轻松转为xls、csv、yaml等格式。简单的来说就是用来处理`tabular dataset`，为这些不同格式的数据集提供一个统一的格式。

目前支持下面这些输出格式：

- Excel (Sets + Books)
- JSON (Sets + Books)
- YAML (Sets + Books)
- HTML (Sets)
- TSV (Sets)
- OSD (Sets)
- CSV (Sets)
- DBF (Sets)

<!-- more -->

## 基本使用
### 创建Dataset对象

```python
import tablib
headers = ('first_name', 'last_name')

data = [
    ('John', 'Adams'),
    ('George', 'Washington')
]

data = tablib.Dataset(*data, headers=headers)
```

这样相当于构造了一张表：

| first_name | last_name  |
| :--------: | :--------: |
|    John    |   Adams    |
|   George   | Washington |

其中最重要的就是`Dataset`对象，当然该对象的创建也可以不输入参数，直接`data = tablib.Dataset()`创建出一个`Dataset`对象，然后通过`data.headers = ['first_name', 'last_name']`设置表头，当然也可以使用`data.headers = ('first_name', 'last_name')`,因为不管是用列表还是元组，tablib都会自动帮我们处理好，我们可以通过`data.append(['Henry', 'Ford'])`或者`data.append(('Henry', 'Ford'))`来向表中添加一条记录。

我们可以通过`data.dict`来查看目前表中的所有数据：

```python
>>> data.dict
[OrderedDict([('First Name', 'John'), ('Last Name', 'Adams')]), OrderedDict([('First Name', 'George'), ('Last Name', 'Washington')]), OrderedDict([('First Name', 'Henry'), ('Last Name', 'Ford')])]
```

也可以通过`print(data)`显示更人性化的输出：

```python
>>> print(data)
First Name|Last Name 
----------|----------
John      |Adams     
George    |Washington
Henry     |Ford      
```

### Dataset属性

```python
>>> print(data)
First Name|Last Name|age
----------|---------|---
John      |Adams    |90
Henry     |Ford     |83
>>> data.height
2
>>> data.width
3
```

`data.height`输出当前记录(行)总数
`data.width`输出当前属性(列)总数

### 常用方法
`lpop()`,`lpush(row, tags=[])`,`lpush_col(col, header=None)`是对列的相关操作

`pop()`,`rpop()`,`rpush(row, tags=[])`,`rpush_col(col, header=None)`是对行的相关操作

`remove_duplicates()`去除重复的记录

`sort(col, reverse=False)`根据列进行排序

`subset(rows=None, cols=None)`返回子Dataset

`wipe()`清空Dataset，包括表头和内容

### 新增列

```python
>>> data.append_col((90, 67, 83), header='age')
```

这样表就变成了：

| first_name | last_name  | age  |
| :--------: | :--------: | :--: |
|    John    |   Adams    |  90  |
|   George   | Washington |  67  |
|   Henry    |    Ford    |  83  |

```python
>>> print(data)
First Name|Last Name |age
----------|----------|---
John      |Adams     |90
George    |Washington|67
Henry     |Ford      |83
```

### 对记录操作

```python
>>> print(data[:2])
[('John', 'Adams', 90), ('George', 'Washington', 67)]
>>> print(data[2:])
[('Henry', 'Ford', 83)]
```

### 对属性操作

```python
>>> print(data['first_name'])
['John', 'George', 'Henry']

>>> print(data)
First Name|Last Name |age
----------|----------|---
John      |Adams     |90
George    |Washington|67
Henry     |Ford      |83
>>> data.get_col(1)
['Adams', 'Washington', 'Ford']
```

### 删除记录

```python
>>> del data[1]
>>> print(data)
First Name|Last Name|age
----------|---------|---
John      |Adams    |90
Henry     |Ford     |83
```
可见记录也是从0开始索引的

删除记录操作也支持切片

### 删除属性

```python
del data['Col Name']
```

### 导入数据

```python
imported_data = Dataset().load(open('data.csv').read())
```

### 导出数据
#### csv

```python
>>> data.csv
'First Name,Last Name,age\r\nJohn,Adams,90\r\nHenry,Ford,83\r\n'
>>> print(data.csv)
First Name,Last Name,age
John,Adams,90
Henry,Ford,83
```

#### json

```python
>>> data.json
'[{"First Name": "John", "Last Name": "Adams", "age": 90}, {"First Name": "Henry", "Last Name": "Ford", "age": 83}]'
>>> print(data.json)
[{"First Name": "John", "Last Name": "Adams", "age": 90}, {"First Name": "Henry", "Last Name": "Ford", "age": 83}]
```

#### yaml

```python
>>> data.yaml
'- {First Name: John, Last Name: Adams, age: 90}\n- {First Name: Henry, Last Name: Ford, age: 83}\n'
>>> print(data.yaml)
- {First Name: John, Last Name: Adams, age: 90}
- {First Name: Henry, Last Name: Ford, age: 83}
```

#### excel

```python
>>> with open('people.xls', 'wb') as f:
...     f.write(data.xls)
```

注意要以二进制形式打开文件

#### dbf

```python
>>> with open('people.dbf', 'wb') as f:
...     f.write(data.dbf)
```

## 高级使用
### 动态列
可以将一个函数指定给Dataset对象

```python
import random

def random_grade(row):
	"""Returns a random integer for entry."""
	return (random.randint(60,100)/100.0)

data.append_col(random_grade, header='Grade')

>>> data.yaml
- {Age: 22, First Name: Kenneth, Grade: 0.6, Last Name: Reitz}
- {Age: 20, First Name: Bessie, Grade: 0.75, Last Name: Monke}
```

函数的参数`row`传入的是每一行记录，所以可以根据传入的记录进行更一步的计算：

```python
def guess_gender(row):
	"""Calculates gender of given student data row."""
	m_names = ('Kenneth', 'Mike', 'Yuri')
	f_names = ('Bessie', 'Samantha', 'Heather')

	name = row[0]

	if name in m_names:
		return 'Male'
	elif name in f_names:
		return 'Female'
	else:
		return 'Unknown'

>>> data.yaml
- {Age: 22, First Name: Kenneth, Gender: Male, Last Name: Reitz}
- {Age: 20, First Name: Bessie, Gender: Female, Last Name: Monke}
```

### tag
可以给记录添加tag，之后通过tag来过滤记录：

```python
students = tablib.Dataset()

students.headers = ['first', 'last']

students.rpush(['Kenneth', 'Reitz'], tags=['male', 'technical'])
students.rpush(['Bessie', 'Monke'], tags=['female', 'creative'])

>>> students.filter(['male']).yaml
- {first: Kenneth, Last: Reitz}
```

### Excel Workbook With Multiple Sheets
it’s quite common to group multiple spreadsheets into a single Excel file, known as a Workbook.

```python
book = tablib.Databook((data1, data2, data3))

with open('students.xls', 'wb') as f:
	f.write(book.xls)
```
## 参考文档

- [tablib官方文档](http://docs.python-tablib.org/en/latest/)