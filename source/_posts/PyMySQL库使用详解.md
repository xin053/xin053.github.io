---
title: PyMySQL库使用详解
date: 2016-11-06 20:19:12
categories:
- Python模块学习
tags:
- Python
- PyMySQL
---

## PyMySQL简介

一个比较方便的连接mysql使用的python库，官网给的例子很简单，但是看下源码发现内容还是很多的，很多函数都没有介绍，所以只有在使用的时候查看源代码了。从github上该项目所获得的星数来看，该库还是很出名的。

<!-- more -->

## PyMySQL使用

```python
import pymysql.cursors

# Connect to the database
connection = pymysql.connect(host='localhost',
                             user='user',
                             password='passwd',
                             db='db',
                             charset='utf8mb4',
                             cursorclass=pymysql.cursors.DictCursor)

try:
    with connection.cursor() as cursor:
        # Create a new record
        sql = "INSERT INTO `users` (`email`, `password`) VALUES (%s, %s)"
        cursor.execute(sql, ('webmaster@python.org', 'very-secret'))

    # connection is not autocommit by default. So you must commit to save
    # your changes.
    connection.commit()

    with connection.cursor() as cursor:
        # Read a single record
        sql = "SELECT `id`, `password` FROM `users` WHERE `email`=%s"
        cursor.execute(sql, ('webmaster@python.org',))
        result = cursor.fetchone()
        print(result)
finally:
    connection.close()
```

结果:

```python
{'password': 'very-secret', 'id': 1}
```

## 参考文档

- [PyMySQL官方文档](http://pymysql.readthedocs.io/)