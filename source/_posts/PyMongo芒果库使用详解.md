---
title: PyMongo芒果库使用详解
date: 2016-11-09 22:09:36
categories:
- Python模块学习
tags:
- Python
- PyMongo
---

## PyMongo简介

MongoDB官方出的针对python平台的库，相当于数据库的客户端，所以需要安装MongoDB的服务器端，按照[Install MongoDB Community Edition on Windows](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)说明可以在windows平台上安装MongoDB

并在管理员权限的cmd窗口运行:

```powershell
"D:\MongoDB\Server\3.2\bin\mongod.exe" --config "D:\MongoDB\Server\3.2\mongod.cfg" --install --serviceName "MongoDB"
```

将会产生系统服务，`mongod.cfg`文件内容:

```
systemLog:
    destination: file
    path: F:\cookies\MongoDB\log\mongod.log
storage:
    dbPath: F:\cookies\MongoDB\database
```

<!-- more -->

## PyMongo使用

针对PyMongo3.3.1

### 创建客户端

```python
>>> from pymongo import MongoClient
>>> client = MongoClient()
```

不指定服务器地址和端口就是默认localhost下的27017端口,也就是:

```python
>>> client = MongoClient('localhost', 27017)
```

### 创建数据库

```python
>>> db = client.test_database
```

底层检查是否有`test_database`这个属性，如果有，获取的就是`test_database`数据库，如果没有，则创建`test_database`数据库,也可以使用如下方式创建数据库:

```python
>>> db = client['test-database']
```

### 创建集合

```python
>>> collection = db.my_collection
>>> collection = db['my-collection']
```

与创建数据库基本一样

### 获取集合

```python
>>> db.collection_names()
['my_collection']
```

### 插入文档

之前的各种操作都不会产生数据文件，只有在插入文档的时候，才连接服务器，产生相应的数据文件

```python
>>> db.my_collection.insert_one({"x": 10})
<pymongo.results.InsertOneResult at 0x2248b7f3af8>
```

也可以在插入文档的同时返回插入文档的`id`:

```python
>>> my_document_id = db.my_collection.insert_one({"x": 10}).inserted_id
>>> my_document_id
ObjectId('582404cff67a2f29f4cb8565')
```

`insert_many()`可以插入多个文档

### 查找文档

`find_one()`查找的是符合条件的第一个文档

```python
>>> db.my_collection.find_one({"x":10})
{'_id': ObjectId('582404adf67a2f29f4cb8564'), 'x': 10}
```

根据`id`查找文档:

```python
>>> db.my_collection.find_one({"_id":my_document_id})
{'_id': ObjectId('582404cff67a2f29f4cb8565'), 'x': 10}
```

也可以如下:

```python
>>> from bson.objectid import ObjectId
>>> db.my_collection.find_one({"_id":ObjectId('582404cff67a2f29f4cb8565')})
```

输出所有符合条件的文档:

```python
for collection in db.my_collection.find({"x":10}).sort("_id"):
    print(collection)
    
{'x': 10, '_id': ObjectId('582404adf67a2f29f4cb8564')}
{'x': 10, '_id': ObjectId('582404cff67a2f29f4cb8565')}
```

`sort("_id")`表示按`id`列排序

### 统计

获取集合中文档数:

```python
>>> db.my_collection.count()
2
```

### 创建索引

我们先将`ObjectId('582404adf67a2f29f4cb8564')`中的`x`值改为11

然后在`x`上创建索引

```python
>>> db.my_collection.create_index([('x', pymongo.ASCENDING)],unique=True)
'x_1'
```

列出所有的索引:

```python
>>> list(db.my_collection.index_information())
['_id_', 'x_1']
```

索引`'_id_'`是根据`_id`自动创建的

其他基础操作比如更新，删除的语法与命令行Mongo类似，在此不赘述

以上便是PyMongo的基本操作，高级操作可参考:

http://api.mongodb.com/python/3.3.1/examples/aggregation.html

API参考:

http://api.mongodb.com/python/3.3.1/api/index.html

## 参考文档

- [PyMongo github README](https://github.com/mongodb/mongo-python-driver)
- [PyMongo 3.3.1 doc](http://api.mongodb.com/python/3.3.1/tutorial.html)
- [Introduction to MongoDB](https://docs.mongodb.com/getting-started/python/introduction/)
