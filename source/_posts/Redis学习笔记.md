---
title: Redis学习笔记
date: 2016-11-12 11:20:36
categories:
- Redis
tags:
- Redis
---

## Redis简介

Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。

特点:

- Redis是完全在内存中保存数据的数据库，使用磁盘只是为了持久性目的
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。

优点:

- **异常快速: **Redis是非常快的，每秒可以执行大约110000设置操作，81000个/每秒的读取操作。
- **支持丰富的数据类型: **Redis支持最大多数开发人员已经知道如列表，集合，可排序集合，哈希等数据类型。
  这使得在应用中很容易解决的各种问题，因为我们知道哪些问题处理使用哪种数据类型更好解决。
- **操作都是原子的 : **所有 Redis 的操作都是原子，从而确保当两个客户同时访问 Redis 服务器得到的是更新后的值（最新值）。
- **MultiUtility工具:** Redis是一个多功能实用工具，可以在很多如：缓存，消息传递队列中使用（Redis原生支持发布/订阅），在应用程序中，如：Web应用程序会话，网站页面点击数等任何短暂的数据

<!-- more -->

因为redis原生支持linux，所以出现了https://github.com/MSOpenTech/redis，支持windows平台，下载安装包安装即可，并且可以设置最高使用的内存大小，更多配置参考安装目录下的配置文件

## Redis使用

### 连接redis服务器

```powershell
C:\WINDOWS\system32>redis-cli -h 127.0.0.1 -p 6379 -a "123" -n 0
127.0.0.1:6379>
```

`-a`后面是密码,`-n`表示连接第几个数据库，默认连接编号为0的数据库

如果默认是本机6397端口,没有密码，可以直接使用以下连接:

```powershell
C:\Users\zzx>redis-cli
127.0.0.1:6379>
```

输入`quit`退出

### 数据类型

#### String

string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象

string类型是Redis最基本的数据类型，一个键最大能存储512MB。

```powershell
127.0.0.1:6379> SET name 'zzx'
OK
127.0.0.1:6379> GET name
"zzx"
```

`SET`与`GET`都可以使用小写，但是一般都是用大写，好区分是不是redix命令

其他命令:http://www.runoob.com/redis/redis-strings.html

更多命令见参考文档

#### Hash

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

```powershell
127.0.0.1:6379> HMSET user:1 username zzx password 123 age 22
OK
127.0.0.1:6379> HGETALL user:1
1) "username"
2) "zzx"
3) "password"
4) "123"
5) "age"
6) "22"
```

`user:1`是key

每个 hash 可以存储2^32-1键值对（40多亿）

其他命令:http://www.runoob.com/redis/redis-hashes.html

更多命令见参考文档

#### List

```powershell
127.0.0.1:6379> LPUSH test_list this is
(integer) 2
127.0.0.1:6379> LPUSH test_list a
(integer) 3
127.0.0.1:6379> LPUSH test_list test for list
(integer) 6
127.0.0.1:6379> LRANGE test_list 0 6
1) "list"
2) "for"
3) "test"
4) "a"
5) "is"
6) "this"
```

列表最多可存储2^32-1 元素

其他命令:http://www.runoob.com/redis/redis-lists.html

更多命令见参考文档

#### Set

通过hash实现的，不能保证顺序，元素唯一性

```powershell
127.0.0.1:6379> SADD test_set this is a
(integer) 3
127.0.0.1:6379> SADD test_set test for set
(integer) 3
127.0.0.1:6379> SADD test_set this
(integer) 0
127.0.0.1:6379> SMEMBERS test_set
1) "test"
2) "this"
3) "set"
4) "for"
5) "is"
6) "a"
```

对于已经存在与set中的元素会返回0

其他命令:http://www.runoob.com/redis/redis-sets.html

更多命令见参考文档

#### zset(有序集合)

元素不重复并且保持插入元素的顺序,与Set不同的是，zset中的每个元素有都个`score`属性，可以理解为权重，内部是按照权重的大小进行排序的

```powershell
127.0.0.1:6379> ZADD test_zset 1 this 2 is 3 a 4 test 0 for 7 zset
(integer) 6
127.0.0.1:6379> ZRANGE test_zset 0 7
1) "for"
2) "this"
3) "is"
4) "a"
5) "test"
6) "zset"
```

其他命令:http://www.runoob.com/redis/redis-sorted-sets.html

更多命令见参考文档

### redis key

以上的`name`,`test_list`,`test_set`,`test_zset`和`uesr:1`都是key

可以通过`DEL`命令来删除key

| 序号   | 命令及描述                                    |
| ---- | ---------------------------------------- |
| 1    | [DEL key](http://www.runoob.com/redis/keys-del.html)该命令用于在 key 存在时删除 key。 |
| 2    | [DUMP key](http://www.runoob.com/redis/keys-dump.html) 序列化给定 key ，并返回被序列化的值。 |
| 3    | [EXISTS key](http://www.runoob.com/redis/keys-exists.html) 检查给定 key 是否存在。 |
| 4    | [EXPIRE key](http://www.runoob.com/redis/keys-expire.html) seconds为给定 key 设置过期时间。 |
| 5    | [EXPIREAT key timestamp](http://www.runoob.com/redis/keys-expireat.html) EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 |
| 6    | [PEXPIRE key milliseconds](http://www.runoob.com/redis/keys-pexpire.html) 设置 key 的过期时间以毫秒计。 |
| 7    | [PEXPIREAT key milliseconds-timestamp](http://www.runoob.com/redis/keys-pexpireat.html) 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计 |
| 8    | [KEYS pattern](http://www.runoob.com/redis/keys-keys.html) 查找所有符合给定模式( pattern)的 key 。 |
| 9    | [MOVE key db](http://www.runoob.com/redis/keys-move.html) 将当前数据库的 key 移动到给定的数据库 db 当中。 |
| 10   | [PERSIST key](http://www.runoob.com/redis/keys-persist.html) 移除 key 的过期时间，key 将持久保持。 |
| 11   | [PTTL key](http://www.runoob.com/redis/keys-pttl.html) 以毫秒为单位返回 key 的剩余的过期时间。 |
| 12   | [TTL key](http://www.runoob.com/redis/keys-ttl.html) 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 |
| 13   | [RANDOMKEY](http://www.runoob.com/redis/keys-randomkey.html) 从当前数据库中随机返回一个 key 。 |
| 14   | [RENAME key newkey](http://www.runoob.com/redis/keys-rename.html) 修改 key 的名称 |
| 15   | [RENAMENX key newkey](http://www.runoob.com/redis/keys-renamenx.html) 仅当 newkey 不存在时，将 key 改名为 newkey 。 |
| 16   | [TYPE key](http://www.runoob.com/redis/keys-type.html) 返回 key 所储存的值的类型。 |

### 事务

| 序号   | 命令及描述                                    |
| ---- | ---------------------------------------- |
| 1    | [DISCARD](http://www.runoob.com/redis/transactions-discard.html) 取消事务，放弃执行事务块内的所有命令。 |
| 2    | [EXEC](http://www.runoob.com/redis/transactions-exec.html) 执行所有事务块内的命令。 |
| 3    | [MULTI](http://www.runoob.com/redis/transactions-multi.html) 标记一个事务块的开始。 |
| 4    | [UNWATCH](http://www.runoob.com/redis/transactions-unwatch.html) 取消 WATCH 命令对所有 key 的监视。 |
| 5    | [WATCH key [key ...\]](http://www.runoob.com/redis/transactions-watch.html) 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。 |

### Redis服务器

输入`INFO`可以获取 Redis 服务器的各种信息和统计数值

### Redis数据备份与恢复

```powershell
127.0.0.1:6379> save
OK
```

该命令将在 redis 安装目录中创建`dump.rdb`文件。

如果需要恢复数据，只需将备份文件 `dump.rdb` 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 `CONFIG` 命令

```powershell
127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "D:\\Redis"
```

创建 redis 备份文件也可以使用命令 `BGSAVE`，该命令在后台执行。

### 删除数据库

`FLUSHDB` 清除一个数据库，`FLUSHALL`清除整个redis数据

### Redis管道

Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。这意味着通常情况下一个请求会遵循以下步骤：

- 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
- 服务端处理命令，并将结果返回给客户端。

Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。

## redis-py

`redis-py`是python实现的redis客户端，关于`redis-py`的使用参考:

- https://github.com/andymccurdy/redis-py#scan-iterators
- https://redis-py.readthedocs.io/en/latest/

## 参考文档

- https://github.com/MSOpenTech/redisredis-py[Memory Configuration For Redis 3.0](https://github.com/MSOpenTech/redis/wiki/Memory-Configuration-For-Redis-3.0)
- [Windows下Redis的安装使用教程](http://www.jb51.net/article/84071.htm)
- [Redis 教程](http://www.runoob.com/redis/redis-tutorial.html)
- [Redis快速入门](http://www.yiibai.com/redis/redis_quick_guide.html)
- [Redis 命令参考](http://redisdoc.com/)
- [Command reference -Redis](http://redis.io/commands)
