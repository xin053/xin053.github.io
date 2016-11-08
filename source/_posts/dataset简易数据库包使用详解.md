---
title: dataset简易数据库包使用详解
date: 2016-11-08 13:33:15
categories: 
- Python模块学习
tags:
- Python
- dataset
---

## dataset简介

dataset号称是为懒人所写的数据库,并说明了很多程序员存储数据都会使用不易查询和更新的CSV和JSON格式，而不是数据库，主要原因是数据库的相关代码比较复杂，而dataset正式解决这个问题，为程序员提供更方便的数据库操作

![](https://dataset.readthedocs.io/en/latest/_static/dataset-logo.png)

```python
import dataset

db = dataset.connect('sqlite:///:memory:')

table = db['sometable']
table.insert(dict(name='John Doe', age=37))
table.insert(dict(name='Jane Doe', age=34, gender='female'))

john = table.find_one(name='John Doe')
```

<!-- more -->

**Features:**

- **Automatic schema**: If a table or column is written that does not exist in the database, it will be created automatically.
- **Upserts**: Records are either created or updated, depending on whether an existing version can be found.
- **Query helpers** for simple queries such as [`all`](https://dataset.readthedocs.io/en/latest/api.html#dataset.Table.all) rows in a table or all [`distinct`](https://dataset.readthedocs.io/en/latest/api.html#dataset.Table.distinct) values across a set of columns.
- **Compatibility**: Being built on top of [SQLAlchemy](http://www.sqlalchemy.org/), `dataset` works with all major databases, such as SQLite, PostgreSQL and MySQL.
- **Scripted exports**: Data can be exported based on a scripted configuration, making the process easy and replicable.

## dataset使用

### 连接数据库

```python
import dataset

# connecting to a SQLite database
db = dataset.connect('sqlite:///mydatabase.db')
# connecting to a MySQL database with user and password
db = dataset.connect('mysql://user:password@localhost/mydatabase')
# connecting to a PostgreSQL database
db = dataset.connect('postgresql://scott:tiger@localhost:5432/mydatabase')
```

### 插入数据

dataset会根据输入自动创建表和字段名

```python
# get a reference to the table 'user'
table = db['user']
# table = db.get_table('user')

# Insert a new record.
table.insert(dict(name='John Doe', age=46, country='China'))
# dataset will create "missing" columns any time you insert a dict with an unknown key
table.insert(dict(name='Jane Doe', age=37, country='France', gender='female'))
```

将产生(主键`id`自动生成):

| id   | country | name     | age  | gender |
| ---- | ------- | -------- | ---- | ------ |
| 1    | China   | John Doe | 46   |        |
| 2    | France  | Jane Doe | 37   | female |

### 更新记录

```python
table.update(dict(name='John Doe', age=47), ['name'])
```

第二个参数相当于sql update语句中的`where`，用来过滤出需要更新的记录

### 事务操作

事务操作可以简单的使用上下文管理器来实现,出现异常，将会回滚

```python
with dataset.connect() as tx:
    tx['user'].insert(dict(name='John Doe', age=46, country='China'))
```

等同于:

```python
db = dataset.connect()
db.begin()
try:
    db['user'].insert(dict(name='John Doe', age=46, country='China'))
    db.commit()
except:
    db.rollback()
```

也可以嵌套使用:

```python
db = dataset.connect()
with db as tx1:
    tx1['user'].insert(dict(name='John Doe', age=46, country='China'))
    with db as tx2:
        tx2['user'].insert(dict(name='Jane Doe', age=37, country='France', gender='female'))
```

### 其他操作

```python
>>> print(db)
<Database(sqlite:///mydatabase.db)>
>>> print(db.tables)
['user']
>>> print(db['user'].columns)
['id', 'country', 'name', 'age', 'gender']
>>> print(len(db['user']))
2

>>> table = db['user']
>>> table
<Table(user)>
>>> table.table
Table('user', MetaData(bind=Engine(sqlite:///mydatabase.db)), Column('id', INTEGER(), table=<user>, primary_key=True, nullable=False), Column('country', TEXT(), table=<user>), Column('name', TEXT(), table=<user>), Column('age', INTEGER(), table=<user>), Column('gender', TEXT(), table=<user>), schema=None)
```

### 从表获取数据

```python
>>> users = db['user'].all()
>>> users
<dataset.persistence.util.ResultIter at 0x157c27ef978>

>>> for user in db['user']:
        print(user['age'])
OrderedDict([('id', 1), ('country', 'China'), ('name', 'John Doe'), ('age', 47), ('gender', None)])
OrderedDict([('id', 2), ('country', 'France'), ('name', 'Jane Doe'), ('age', 37), ('gender', 'female')])

>>> chinese_users = table.find(country='China')
>>> chinese_users
<dataset.persistence.util.ResultIter at 0x157c2816978>

>>> john = table.find_one(name='John Doe')
>>> john
OrderedDict([('id', 1),
             ('country', 'China'),
             ('name', 'John Doe'),
             ('age', 47),
             ('gender', None)])

>>> elderly_users = table.find(table.table.columns.age >= 70)
```

获取非重复数据

```python
# Get one user per country
db['user'].distinct('country')
```

### 删除记录

```python
table.delete(place='Berlin')
```

### 执行SQL语句

```python
result = db.query('SELECT country, COUNT(*) c FROM user GROUP BY country')
for row in result:
   print(row['country'], row['c'])
```

### 导出数据

```python
# export all users into a single JSON
result = db['users'].all()
dataset.freeze(result, format='json', filename='users.json')
```

## 参考文档

- [dataset官方文档](https://dataset.readthedocs.io/en/latest/)
