---
title: MongoDB学习笔记
date: 2016-11-09 12:08:29
categories:
- MongoDB
tags:
- MongoDB
---

## MongoDB简介

![](https://avatars3.githubusercontent.com/u/45120?v=3&s=200)

MongoDB是对象型数据库，mysql等关系型数据库的表格式固定，如果想增添带有更多信息的属性就需要重新建一张表，然后用外键进行关联，这样查询也会造成表之间的`join`，效率低，而且结构越复杂，表越多，表之间的关系就越紧密，会影响表之间的清晰度。而对象型数据库将每条记录看作是一个文档，以json格式存放在一个文件中，并且每个文档结构可以不同，一个文档中就包含了这条记录的所有相关信息，以面对对象的思维来看就是一个对象，文档的集合也就是关系型数据库记录的集合，也就是表

<!-- more -->

## MongoDB使用

### 创建数据库

```
use DATABASE_NAME
```

### 删除数据库

```
db.dropDatabase()
```

`db`这个变量的值就是我们当前使用的数据库

### 创建集合

```
db.createCollection(name, options)
```

在该命令中，`name` 是所要创建的集合名称。`options` 是一个用来指定集合配置的文档。

### 删除集合

```
db.COLLECTION_NAME.drop()
```

### 数据类型

- **String**：字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。
- **Integer**：整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。
- **Boolean**：布尔值。用于存储布尔值（真/假）。
- **Double**：双精度浮点值。用于存储浮点值。
- **Min/Max keys**：将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。
- **Arrays**：用于将数组或列表或多个值存储为一个键。
- **Timestamp**：时间戳。记录文档修改或添加的具体时间。
- **Object**：用于内嵌文档。
- **Null**：用于创建空值。
- **Symbol**：符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。
- **Date**：日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。
- **Object ID**：对象 ID。用于创建文档的 ID。
- **Binary Data**：二进制数据。用于存储二进制数据。
- **Code**：代码类型。用于在文档中存储 JavaScript 代码。
- **Regular expression**：正则表达式类型。用于存储正则表达式。

### 插入文档

```
db.COLLECTION_NAME.insert(document)
```

```
db.mycol.insert({
   _id: ObjectId(7df78ad8902c),
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
})
```

在插入的文档中，如果我们没有指定 `_id` 参数，那么 MongoDB 会自动为文档指定一个唯一的 ID

### 查询文档

```
db.COLLECTION_NAME.find()
```

```
db.mycol.find().pretty()
{
   "_id": ObjectId(7df78ad8902c),
   "title": "MongoDB Overview", 
   "description": "MongoDB is no sql database",
   "by": "tutorials point",
   "url": "http://www.tutorialspoint.com",
   "tags": ["mongodb", "database", "NoSQL"],
   "likes": "100"
}
```

用格式化方式显示结果，使用的是 `pretty()` 方法。除了 `find()` 方法之外，还有一个 `findOne()` 方法，它只返回一个文档。

| 操作    | 格式                       | 范例                                       | RDBMS中的类似语句                    |
| ----- | ------------------------ | ---------------------------------------- | ------------------------------ |
| 等于    | `{<key>:<value>}`        | `db.mycol.find({"by":"tutorials point"}).pretty()` | `where by = 'tutorials point'` |
| 小于    | `{<key>:{$lt:<value>}}`  | `db.mycol.find({"likes":{$lt:50}}).pretty()` | `where likes < 50`             |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `db.mycol.find({"likes":{$lte:50}}).pretty()` | `where likes <= 50`            |
| 大于    | `{<key>:{$gt:<value>}}`  | `db.mycol.find({"likes":{$gt:50}}).pretty()` | `where likes > 50`             |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `db.mycol.find({"likes":{$gte:50}}).pretty()` | `where likes >= 50`            |
| 不等于   | `{<key>:{$ne:<value>}}`  | `db.mycol.find({"likes":{$ne:50}}).pretty()` | `where likes != 50`            |

`and`语法就用逗号表示:

```
db.mycol.find({key1:value1, key2:value2}).pretty()
```

`or`语法:

```
db.mycol.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

### 更新文档

```
db.COLLECTION_NAME.update(SELECTIOIN_CRITERIA, UPDATED_DATA)
```

MongoDB 中的 `update()` 与 `save()` 方法都能用于更新集合中的文档。`update()` 方法更新已有文档中的值，而`save()` 方法则是用传入该方法的文档来替换已有文档。

MongoDB 默认只更新单个文档，要想更新多个文档，需要把参数 `multi` 设为 `true`。

```
db.mycol.update({'title':'MongoDB Overview'},{$set:{'title':'New MongoDB Tutorial'}},{multi:true})
```

```
db.mycol.update({title:'Seven'}, {$inc:{likes:2}})
```

`$inc`表示将`likes`值加2

### 删除文档

```
db.COLLECTION_NAME.remove(DELLETION_CRITTERIA)
```

如果有多个记录，而你只想删除第一条记录，那么就设置 `remove()` 方法中的 `justOne` 参数：

```
db.COLLECTION_NAME.remove(DELETION_CRITERIA,1)
```

如果没有指定删除标准，则 MongoDB 会将集合中所有文档都予以删除。

```
db.COLLECTION_NAME.remove()
```

### 映射

MongoDB 的查询文档曾介绍过 `find()` 方法，它可以利用 AND 或 OR 条件来获取想要的字段列表。在 MongoDB 中执行 `find()` 方法时，显示的是一个文档的所有字段。要想限制，可以利用 0 或 1 来设置字段列表。1 用于显示字段，0 用于隐藏字段。

```
db.COLLECTION_NAME.find({},{KEY:1})
```

假如 mycol 集合拥有下列数据：

```
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point Overview"}
```

```
db.mycol.find({},{"title":1,_id:0})
{"title":"MongoDB Overview"}
{"title":"NoSQL Overview"}
{"title":"Tutorials Point Overview"}
```

注意：在执行 `find()` 方法时，`_id` 字段是一直显示的。如果不想显示该字段，则可以将其设为 0。

### 限制记录

```
db.COLLECTION_NAME.find().limit(NUMBER)
```

### 记录排序

MongoDB 中的文档排序是通过 `sort()` 方法来实现的。`sort()` 方法可以通过一些参数来指定要进行排序的字段，并使用 1 和 -1 来指定排序方式，其中 1 表示升序，而 -1 表示降序。

```
db.COLLECTION_NAME.find().sort({KEY:1})
```

### 索引

```
db.COLLECTION_NAME.ensureIndex({KEY:1})
```

这里的 key 是想创建索引的字段名称，1 代表按升序排列字段值。-1 代表按降序排列。

获取索引信息:

```
db.mycol.getIndexes()
```

将返回所有索引，包括其名字。

删除索引:

```
db.mycol.dropIndex('index_name')
```

### 聚合

相当于关系型数据库中的`group by`

```
db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

比如有集合:

```
{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by_user: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
},
{
   _id: ObjectId(7df78ad8902d)
   title: 'NoSQL Overview', 
   description: 'No sql database is very fast',
   by_user: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 10
},
{
   _id: ObjectId(7df78ad8902e)
   title: 'Neo4j Overview', 
   description: 'Neo4j is no sql database',
   by_user: 'Neo4j',
   url: 'http://www.neo4j.com',
   tags: ['neo4j', 'database', 'NoSQL'],
   likes: 750
}
```

假如想从上述集合中，归纳出一个列表，以显示每个用户写的教程数量，需要像下面这样使用 `aggregate()` 方法：

```
db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "tutorials point",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
```

| 表达式         | 描述                                       | 范例                                       |
| ----------- | ---------------------------------------- | ---------------------------------------- |
| `$sum`      | 对集合中所有文档的定义值进行加和操作                       | `db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}])` |
| `$avg`      | 对集合中所有文档的定义值进行平均值                        | `db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}])` |
| `$min`      | 计算集合中所有文档的对应值中的最小值                       | `db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}])` |
| `$max`      | 计算集合中所有文档的对应值中的最大值                       | `db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}])` |
| `$push`     | 将值插入到一个结果文档的数组中                          | `db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}])` |
| `$addToSet` | 将值插入到一个结果文档的数组中，但不进行复制                   | `db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}])` |
| `$first`    | 根据成组方式，从源文档中获取第一个文档。但只有对之前应用过 `$sort`管道操作符的结果才有意义。 | `db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}])` |
| `$last`     | 根据成组方式，从源文档中获取最后一个文档。但只有对之前进行过 `$sort`管道操作符的结果才有意义。 | `db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}])` |

### 事务

```
db.mycol.findAndModify(
            {
            query:{'title':'Forrest Gump'},
            update:{$inc:{likes:10}}
            }
              )
```

`query`是查找出匹配的文档，和`find`是一样的，而`update`则是更新`likes`这个项目。注意由于MongoDB只支持单个文档的atomic operation，因此如果`query`出多于一个文档，则只会对第一个文档进行操作。

### 正则表达式

```
db.mycol.find({title:/.*b$/}).pretty()
```

注意以上匹配都是区分大小写的，如果你要让其不区分大小写，则可以：

```
db.mycol.find({title:{$regex:'fight.*b',$options:'$i'}}).pretty()
```

`$i`是insensitive的意思。这样的话，即使是小写的fight，也能搜到了。

## 参考文档

- [MongoDB 极简实践入门](https://github.com/StevenSLXie/Tutorials-for-Web-Developers/blob/master/MongoDB%20%E6%9E%81%E7%AE%80%E5%AE%9E%E8%B7%B5%E5%85%A5%E9%97%A8.md)
- [极客学院 Mongodb 教程](http://wiki.jikexueyuan.com/project/mongodb/)
- https://university.mongodb.com/
- [MongoDB 3.2 中文文档](http://docs.mongoing.com/manual-zh/)
- [MongoDB Tutorials](https://docs.mongodb.com/manual/tutorial/)
- [Install MongoDB Community Edition on Windows](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)
