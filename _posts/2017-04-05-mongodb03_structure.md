---
layout: post
title: "MongoDB 结构"
description: "NoSQL 文档数据库"
category : [SQL]
tags : [MongoDB, NoSQL]
---
{% include JB/setup %}

## MongoDB概念解析

| SQL术语/概念    | MongoDB术语/概念 | 解释/说明                   |
| ----------- | ------------ | ----------------------- |
| database    | database     | 数据库                     |
| table       | collection   | 数据库表/集合                 |
| row         | document     | 数据记录/文档                 |
| column      | field        | 数据字段/域                  |
| index       | index        | 索引                      |
| table joins | aggregate    | 聚合实现（look up）           |
| primary key | primary key  | 主键，MongoDB自动将_id字段设置为主键 |

## 数据库(DataBase)

启动时出现警告，请参考官网文档：https://www.mongodb.com/blog/post/mongodb-security-part-ii-10-mistakes-that-can 或 https://blog.csdn.net/q1056843325/article/details/70941697

```Shell
$ mongo
MongoDB shell version v3.6.4
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.6.4
...     
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
runoob  0.000GB
> use runoob
switched to db runoob
```

- **admin**: 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器
- **local:** 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
- **config**: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息



## 文档(Document）

文档是一组键值(key-value)对（即BSON，Binary JSON）。MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型，这与关系型数据库有很大的区别，也是 MongoDB 非常突出的特点。

例子如下：

```json
{"site":"www.runoob.com", "name":"菜鸟教程"}
```

注意：

1. 文档中的键/值对是有序的。
2. 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型（甚至可以是整个嵌入的文档)。
3. MongoDB区分类型和大小写。
4. MongoDB的文档不能有重复的键。
5. 文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。



## 集合(Collection)

集合存在于数据库中，集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。

```Json
{"site":"www.baidu.com"}
{"site":"www.google.com","name":"Google"}
{"site":"www.runoob.com","name":"菜鸟教程","num":5}
```

当第一个文档插入时，集合就会被创建。

### capped collections

Capped collections 是固定大小的 collection，是高性能自动维护对象插入的顺序。它非常适合类似记录日志的功能和标准的collection不同，你必须要显式的创建一个capped collection， 指定一个collection的大小，单位是字节。collection的数据存储空间值提前分配的。当集合大小达到最大值时，最早插入的数据会被覆盖掉

```sql
db.createCollection("capcoll", {capped:true, size: 100000})
```

- 在capped collection中，你能添加新的对象。
- 能进行更新，然而，对象不会增加存储空间。如果增加，更新就会失败 。
- ~~数据库不允许进行删除~~。可使用drop()方法删除collection所有的行。
- 注意: 删除之后，你必须显式的重新创建这个collection。
- 在32bit机器中，capped collection最大存储为1e9( 1X109)个字节。



## 元数据

数据库的信息是存储在集合中，它们使用了系统的命名空间：

```sql
dbname.system.*
```

在MongoDB数据库中名字空间 <dbname>.system.* 是包含多种系统信息的特殊集合(Collection)，如下:

| 集合命名空间                   | 描述                      |
| ------------------------ | ----------------------- |
| dbname.system.namespaces | 列出所有名字空间。               |
| dbname.system.indexes    | 列出所有索引。                 |
| dbname.system.profile    | 包含数据库概要(profile)信息。     |
| dbname.system.users      | 列出所有可访问数据库的用户。          |
| dbname.local.sources     | 包含复制对端（slave）的服务器信息和状态。 |

对于修改系统集合中的对象有如下限制。

在{{system.indexes}}插入数据，可以创建索引。但除此之外该表信息是不可变的(特殊的drop index命令将自动更新相关信息)。 

{{system.users}}是可修改的。 {{system.profile}}是可删除的

## 数据类型

| 数据类型               | 描述                                       |
| ------------------ | ---------------------------------------- |
| String             | 字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。 |
| Integer            | 整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。   |
| Boolean            | 布尔值。用于存储布尔值（真/假）。                        |
| Double             | 双精度浮点值。用于存储浮点值。                          |
| Min/Max keys       | 将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。      |
| Array              | 用于将数组或列表或多个值存储为一个键。                      |
| Timestamp          | 时间戳。记录文档修改或添加的具体时间。                      |
| Object             | 用于内嵌文档。                                  |
| Null               | 用于创建空值。                                  |
| Symbol             | 符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。 |
| Date               | 日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。 |
| Object ID          | 对象 ID。用于创建文档的 ID。                        |
| Binary Data        | 二进制数据。用于存储二进制数据。                         |
| Code               | 代码类型。用于在文档中存储 JavaScript 代码。             |
| Regular expression | 正则表达式类型。用于存储正则表达式。                       |

### ObjectId

ObjectId 类似唯一主键，可以很快的去生成和排序，包含 12 bytes，含义是：

- 前 4 个字节表示创建 **unix**时间戳,格林尼治时间 **UTC** 时间，比北京时间晚了 8 个小时
- 接下来的 3 个字节是机器标识码
- 紧接的两个字节由进程 id 组成 PID
- 最后三个字节是随机数

MongoDB 中存储的文档必须有一个 _id 键。这个键的值可以是任何类型的，默认是个 ObjectId 对象

由于 ObjectId 中保存了创建的时间戳，所以你不需要为你的文档保存时间戳字段，你可以通过 getTimestamp 函数来获取文档的创建时间:

```SQL
> var newObject = ObjectId()
> newObject.getTimestamp()
ISODate("2017-11-25T07:21:10Z")
```

ObjectId 转为字符串

```SQL
> newObject.str
5a1919e63df83ce79df8b38f
```

### 字符串

**BSON 字符串都是 UTF-8 编码。**

### 时间戳

BSON 有一个特殊的时间戳类型用于 MongoDB 内部使用，与普通的 日期 类型不相关。 时间戳值是一个 64 位的值。其中：

- 前32位是一个 time_t 值（与Unix新纪元相差的秒数）
- 后32位是在某秒中操作的一个递增的`序数`

在单个 mongod 实例中，时间戳值通常是唯一的。

在复制集中， oplog 有一个 ts 字段。这个字段中的值使用BSON时间戳表示了操作时间。

> BSON 时间戳类型主要用于 MongoDB 内部使用。在大多数情况下的应用开发中，你可以使用 BSON 日期类型。

### 日期

表示当前距离 Unix新纪元（1970年1月1日）的毫秒数。日期类型是有符号的, 负数表示 1970 年之前的日期。

```sql
> var mydate1 = new Date()     //格林尼治时间
> mydate1
ISODate("2018-03-04T14:58:51.233Z")
> typeof mydate1
object

> var mydate2 = ISODate() //格林尼治时间
> mydate2
ISODate("2018-03-04T15:00:45.479Z")
> typeof mydate2
object
```
### 配置（v3.6）

MAC版使用brew 安装：`brew install mongodb`

启动：`brew services start mongodb`

停止：`brew services stop mongodb`

重启：`brew services restart mongodb`

配置文件：`/usr/local/etc/mongod.conf`

配置文件格式采用yaml的方式，参考：https://docs.mongodb.com/manual/reference/configuration-options/index.html#net-options

```yaml
systemLog:
  destination: file
  path: /usr/local/var/log/mongodb/mongo.log
  logAppend: true
storage:
  dbPath: /usr/local/var/mongodb
net:
 # bindIp: 127.0.0.1   # 只能通过localhost(127.0.0.1)来访问，配置成 0.0.0.0，所有可用的IP4可访问
  bindIpAll: true      # 所有IP4 IP6 均可访问
```