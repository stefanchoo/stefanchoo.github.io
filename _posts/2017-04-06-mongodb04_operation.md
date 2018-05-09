---
layout: post
title: "MongoDB 操作"
description: "NoSQL 文档数据库"
category : [SQL]
tags : [MongoDB, NoSQL]
---
{% include JB/setup %}

## MongoDB - 连接

#### **语法**

```sql
mongodb://[username:password@]host1[:port1],host2[:port2],...[,hostN[:portN]][/[database][?options]]
```

- **mongodb://** 这是固定的格式，必须要指定。
- **username:password@** 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登陆这个数据库
- **host1** 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。<u>如果要连接复制集，请指定多个主机地址</u>。
- **portX** 可选的指定端口，如果不填，默认为`27017`
- **/database **如果指定username:password@，连接并验证登陆指定数据库。若不指定，默认打开 test 数据库。
- **?options** 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开

标准的连接格式包含了多个选项(options)，如下所示：

| 选项                  | 描述                                       |
| ------------------- | ---------------------------------------- |
| replicaSet=name     | 验证replica set的名称。 Impliesconnect=replicaSet. |
| slaveOk=true\|false | true:在connect=direct模式下，驱动会连接第一台机器，即使这台服务器不是主。在connect=replicaSet模式下，驱动会发送所有的写请求到主并且把读取操作分布在其他从服务器。false: 在 connect=direct模式下，驱动会自动找寻主服务器. 在connect=replicaSet 模式下，驱动仅仅连接主服务器，并且所有的读写命令都连接到主服务器。 |
| safe=true\|false    | true: 在执行更新操作之后，驱动都会发送getLastError命令来确保更新成功。(还要参考 wtimeoutMS).false: 在每次更新之后，驱动不会发送getLastError来确保更新成功。 |
| w=n                 | 驱动添加 { w : n } 到getLastError命令. 应用于safe=true。 |
| wtimeoutMS=ms       | 驱动添加 { wtimeout : ms } 到 getlasterror 命令. 应用于 safe=true. |
| fsync=true\|false   | true: 驱动添加 { fsync : true } 到 getlasterror 命令.应用于 safe=true.false: 驱动不会添加到getLastError命令中。 |
| journal=true\|false | 如果设置为 true, 同步到 journal (在提交到数据库前写入到实体中). 应用于 safe=true |
| connectTimeoutMS=ms | 可以打开连接的时间。                               |
| socketTimeoutMS=ms  | 发送和接受sockets的时间。                         |



### 更多连接实例

连接本地数据库服务器，端口是默认的。

```ruby
mongodb://localhost
```

使用用户名fred，密码foobar登录localhost的admin数据库。

```ruby
mongodb://fred:foobar@localhost
```

使用用户名fred，密码foobar登录localhost的baz数据库。

```ruby
mongodb://fred:foobar@localhost/baz
```

连接 replica pair, 服务器1为example1.com服务器2为example2。

```ruby
mongodb://example1.com:27017,example2.com:27017
```

连接 replica set 三台服务器 (端口 27017, 27018, 和27019):

```ruby
mongodb://localhost,localhost:27018,localhost:27019
```

连接 replica set 三台服务器, 写入操作应用在主服务器 并且分布查询到从服务器。

```ruby
mongodb://host1,host2,host3/?slaveOk=true
```

直接连接第一个服务器，无论是replica set一部分或者主服务器或者从服务器。

```ruby
mongodb://host1,host2,host3/?connect=direct;slaveOk=true
```

当你的连接服务器有优先级，还需要列出所有服务器，你可以使用上述连接方式。

安全模式连接到localhost:

```ruby
mongodb://localhost/?safe=true
```

以安全模式连接到replica set，并且等待至少两个复制服务器成功写入，超时时间设置为2秒。

```ruby
mongodb://host1,host2,host3/?safe=true;w=2;wtimeoutMS=2000
```



## MongoDB创建数据库

#### 语法

```Ruby
use DATABASE_NAME
```

1. 如果数据库不存在，则创建数据库，否则切换到制定的数据库
2. 显示所有数据库 `show dbs`
3. 当数据库中没有集合时，是不会显示的

## MongoDB删除数据库

#### 语法

```ruby
db.dropDatabase()
```

先要切换到想要删除的数据库：`use DATABASE_NAME`

## MongoDB创建集合

#### 语法

```ruby
db.createCollection(name, options)
```

**参数说明**

- name: 要创建的集合名称
- options: 可选参数, 指定有关内存大小及索引的选项

options 可以是如下参数：

| 字段          | 类型   | 描述                                       |
| ----------- | ---- | ---------------------------------------- |
| capped      | 布尔   | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。**当该值为 true 时，必须指定 size 参数。** |
| autoIndexId | 布尔   | （可选）如为 true，自动在 _id 字段创建索引。默认为 false。    |
| size        | 数值   | （可选）为固定集合指定一个最大值（以字节计）。**如果 capped 为 true，也需要指定该字段。** |
| max         | 数值   | （可选）指定固定集合中包含文档的最大数量。                    |

在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。

#### **实例**

在 test 数据库中创建 runoob 集合：

```ruby
> use test
switched to db test
> db.createCollection("runoob")
{ "ok" : 1 }
>
```

如果要查看已有集合，可以使用 show collections 命令：

```ruby
> show collections
runoob
system.indexes
```

下面是带有几个关键参数的 createCollection() 的用法：

创建固定集合 mycol，整个集合空间大小 6142800 KB, 文档最大个数为 10000 个。

```ruby
> db.createCollection("mycol", { capped : true, autoIndexId : true, size : 
   6142800, max : 10000 } )
{ "ok" : 1 }
>
```

在 MongoDB 中，你不需要创建集合。当你插入一些文档时，MongoDB 会自动创建集合。

```Ruby
> db.mycol2.insert({"name" : "菜鸟教程"})
> show collections
mycol2
...
```

# MongoDB 删除集合

#### **语法**

```ruby
db.collection.drop()
```

**参数说明**

- 无

**返回值**

如果成功删除选定集合，则 drop() 方法返回 true，否则返回 false。

#### **实例**

在数据库 test 中，我们可以先通过 show collections 命令查看已存在的集合：

```Ruby
> use test
switched to db mydb
> show collections
mycol
mycol2
>
```

接着删除集合 mycol2 : 

```Ruby
> db.mycol2.drop()
true
>
```

通过 show collections 再次查看数据库 test 中的集合：

```ruby
> show collections
mycol
>
```

# MongoDB  插入文档

BSON是一种类json的一种二进制形式的存储格式，简称为Binary JSON

#### **插入文档**

MongoDB 使用 insert() 或 save() 方法向集合中插入文档，语法如下：

```ruby
db.COLLECTION_NAME.insert(document)
```

- db.collection.insertOne(): 向指定集合中插入一条文档数据
- db.collection.insertMany(): 向指定集合中插入多条文档数据

#### **实例**

以下文档可以存储在 MongoDB 的 runoob 数据库 的 col 集合中：

```ruby
> db.col.insert({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})
```

```Ruby
> var document = db.mycol.insertOne({"a": 3})
> document
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5aed6c888ed7436bd454555c")
}
```

```Ruby
> var res = db.mycol.insertMany([{"b": 4}, {"c":5}])
> res
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5aed6cd58ed7436bd454555d"),
		ObjectId("5aed6cd58ed7436bd454555e")
	]
}

```

# MongoDB 更新文档

MongoDB 使用 **update()** 和 **save()** 方法来更新集合中的文档。接下来让我们详细来看下两个函数的应用及其区别。

------

## update() 方法

update() 方法用于更新已存在的文档。语法格式如下：

```ruby
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

- db.collection.updateOne() 向指定集合更新单个文档
- db.collection.updateMany() 向指定集合更新多个文档

**参数说明：**

- **query **: update的查询条件，类似sql update查询内where后面的。
- **update **: update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- **upsert **: 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- **multi **: 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- **writeConcern **:可选，抛出异常的级别。

#### **实例**

```ruby
> db.col.find()
{ "_id" : ObjectId("5aed6ba48ed7436bd454555b"), "title" : "MongoDB 教程", "description" : "MongoDB 是一个 Nosql 数据库", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "mongodb", "database", "NoSQL" ], "likes" : 100 }
> db.col.update({'title':'MongoDB 教程'}, {$set:{'title':'MongoDB'}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.col.find().pretty()
{
	"_id" : ObjectId("5aed6ba48ed7436bd454555b"),
	"title" : "MongoDB",
	"description" : "MongoDB 是一个 Nosql 数据库",
	"by" : "菜鸟教程",
	"url" : "http://www.runoob.com",
	"tags" : [
		"mongodb",
		"database",
		"NoSQL"
	],
	"likes" : 100
}
```

```Ruby
> db.test.insert(
... [
... {"name":"abc","age":"25","status":"zxc"},
... {"name":"dec","age":"19","status":"qwe"},
... {"name":"asd","age":"30","status":"nmn"},
... ]
... )
BulkWriteResult({
	"writeErrors" : [ ],
	"writeConcernErrors" : [ ],
	"nInserted" : 3,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})
> db.test.updateOne({'age':{$gt:'19'}}, {$set:{'name':'abcd'}})
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
> db.test.find().pretty()
{
	"_id" : ObjectId("5aed71338ed7436bd454555f"),
	"name" : "abcd",
	"age" : "25",
	"status" : "zxc"
}
{
	"_id" : ObjectId("5aed71338ed7436bd4545560"),
	"name" : "dec",
	"age" : "19",
	"status" : "qwe"
}
{
	"_id" : ObjectId("5aed71338ed7436bd4545561"),
	"name" : "asd",
	"age" : "30",
	"status" : "nmn"
}
> db.test.updateMany({'age':{$gte:'19'}}, {$set:{'name':'abcd'}})
{ "acknowledged" : true, "matchedCount" : 3, "modifiedCount" : 1 }
> db.test.find().pretty()
{
	"_id" : ObjectId("5aed71338ed7436bd454555f"),
	"name" : "abcd",
	"age" : "25",
	"status" : "zxc"
}
{
	"_id" : ObjectId("5aed71338ed7436bd4545560"),
	"name" : "abcd",
	"age" : "19",
	"status" : "qwe"
}
{
	"_id" : ObjectId("5aed71338ed7436bd4545561"),
	"name" : "abcd",
	"age" : "30",
	"status" : "nmn"
}
```

## save() 方法 

如id存在，则替换，否则相当于insert

```Ruby
> db.mycol.find()
{ "_id" : ObjectId("5aed69ec8ed7436bd454555a"), "name" : "菜鸟教程" }
{ "_id" : ObjectId("5aed6c888ed7436bd454555c"), "a" : 3 }
{ "_id" : ObjectId("5aed6cd58ed7436bd454555d"), "b" : 4 }
{ "_id" : ObjectId("5aed6cd58ed7436bd454555e"), "c" : 5 }
> db.mycol.save({ "_id" : ObjectId("5aed6c888ed7436bd454555c"), "a" : 4 })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.mycol.find()
{ "_id" : ObjectId("5aed69ec8ed7436bd454555a"), "name" : "菜鸟教程" }
{ "_id" : ObjectId("5aed6c888ed7436bd454555c"), "a" : 4 }
{ "_id" : ObjectId("5aed6cd58ed7436bd454555d"), "b" : 4 }
{ "_id" : ObjectId("5aed6cd58ed7436bd454555e"), "c" : 5 }
```

# MongoDB 删除文档

#### **语法**

```ruby
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

- db.collection.deleteOne() 删除单个文档
- db.collection.deleteMany() 删除多个文档

#### **实例**

```Ruby
> db.test.find().pretty()
{
	"_id" : ObjectId("5aed71338ed7436bd454555f"),
	"name" : "abcd",
	"age" : "25",
	"status" : "zxc"
}
{
	"_id" : ObjectId("5aed71338ed7436bd4545560"),
	"name" : "abcd",
	"age" : "19",
	"status" : "qwe"
}
{
	"_id" : ObjectId("5aed71338ed7436bd4545561"),
	"name" : "abcd",
	"age" : "30",
	"status" : "nmn"
}
> db.test.deleteOne({'name': 'abcd'})
{ "acknowledged" : true, "deletedCount" : 1 }
> db.test.find().pretty()
{
	"_id" : ObjectId("5aed71338ed7436bd4545560"),
	"name" : "abcd",
	"age" : "19",
	"status" : "qwe"
}
{
	"_id" : ObjectId("5aed71338ed7436bd4545561"),
	"name" : "abcd",
	"age" : "30",
	"status" : "nmn"
}
> db.test.deleteMany({'name': 'abcd'})
{ "acknowledged" : true, "deletedCount" : 2 }
> db.test.find().pretty()
```



# MongoDB 查询文档

#### **语法**

```ruby
db.collection.find(query, projection)
```

- **query** ：可选，使用查询操作符指定查询条件
- **projection** ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

若不指定 projection，则默认返回所有键，指定 projection 格式如下，有两种模式

```Ruby
db.collection.find(query, {title: 1, by: 1}) // inclusion模式 指定返回的键，不返回其他键
db.collection.find(query, {title: 0, by: 0}) // exclusion模式 指定不返回的键,返回其他键
db.collection.find(query, {_id:0, title: 1, by: 1}) 
```

_id 键默认返回，需要主动指定 _id:0 才会隐藏

两种模式不可混用（因为这样的话无法推断其他键是否应返回）

如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：

```ruby
> db.col.find().pretty()
```

pretty() 方法以格式化的方式来显示所有文档。

除了 find() 方法之外，还有一个 findOne() 方法，它只返回一个文档。

## MongoDB 与 RDBMS Where 语句比较

如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

| 操作    | 格式                       | 范例                                       | RDBMS中的类似语句         |
| ----- | ------------------------ | ---------------------------------------- | ------------------- |
| 等于    | `{<key>:<value>`}        | `db.col.find({"by":"菜鸟教程"}).pretty()`    | `where by = '菜鸟教程'` |
| 小于    | `{<key>:{$lt:<value>}}`  | `db.col.find({"likes":{$lt:50}}).pretty()` | `where likes < 50`  |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `db.col.find({"likes":{$lte:50}}).pretty()` | `where likes <= 50` |
| 大于    | `{<key>:{$gt:<value>}}`  | `db.col.find({"likes":{$gt:50}}).pretty()` | `where likes > 50`  |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `db.col.find({"likes":{$gte:50}}).pretty()` | `where likes >= 50` |
| 不等于   | `{<key>:{$ne:<value>}}`  | `db.col.find({"likes":{$ne:50}}).pretty()` | `where likes != 50` |

## MongoDB AND 条件

MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，即常规 SQL 的 AND 条件。

#### 语法

```ruby
> db.col.find({key1:value1, key2:value2}).pretty()
```

#### **实例**

```Ruby
> db.test.find({"name":"abc","age":"25"}).pretty()
{
	"_id" : ObjectId("5aed75de8ed7436bd4545562"),
	"name" : "abc",
	"age" : "25",
	"status" : "zxc"
}
```

## MongoDB OR 条件

MongoDB OR 条件语句使用了关键字 **$or**,语法格式如下：

```Ruby
> db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

### 实例

```Ruby
> db.test.find({$or:[{"name":"abc"},{"age":{$gt:"19"}}]}).pretty()
{
	"_id" : ObjectId("5aed75de8ed7436bd4545562"),
	"name" : "abc",
	"age" : "25",
	"status" : "zxc"
}
{
	"_id" : ObjectId("5aed75de8ed7436bd4545564"),
	"name" : "asd",
	"age" : "30",
	"status" : "nmn"
}
```

## AND 和 OR 联合使用

```ruby
> db.test.find({"status":"zxc", $or:[{"name":"abc"},{"age":{$gt:"19"}}]}).pretty()
{
	"_id" : ObjectId("5aed75de8ed7436bd4545562"),
	"name" : "abc",
	"age" : "25",
	"status" : "zxc"
}
```

# MongoDB $type 操作符

**描述**

在本章节中，我们将继续讨论MongoDB中条件操作符 $type。

$type操作符是基于BSON类型来检索集合中匹配的数据类型，并返回结果。

MongoDB 中可以使用的类型如下表所示：

| **类型**                  | **数字** | **备注**           |
| ----------------------- | ------ | ---------------- |
| Double                  | 1      |                  |
| String                  | 2      |                  |
| Object                  | 3      |                  |
| Array                   | 4      |                  |
| Binary data             | 5      |                  |
| Undefined               | 6      | 已废弃。             |
| Object id               | 7      |                  |
| Boolean                 | 8      |                  |
| Date                    | 9      |                  |
| Null                    | 10     |                  |
| Regular Expression      | 11     |                  |
| JavaScript              | 13     |                  |
| Symbol                  | 14     |                  |
| JavaScript (with scope) | 15     |                  |
| 32-bit integer          | 16     |                  |
| Timestamp               | 17     |                  |
| 64-bit integer          | 18     |                  |
| Min key                 | 255    | Query with `-1`. |
| Max key                 | 127    |                  |

## MongoDB 操作符 - $type 实例

如果想获取 "col" 集合中 title 为 String 的数据，你可以使用以下命令：

```Ruby
db.col.find({"title" : {$type : 2}})
```



# MongoDB Limit与Skip方法

------

## MongoDB Limit() 方法

如果你需要在MongoDB中读取指定数量的数据记录，可以使用MongoDB的Limit方法，limit()方法接受一个数字参数，该参数指定从MongoDB中读取的记录条数。

语法：

```Ruby
> db.COLLECTION_NAME.find().limit(NUMBER)
```

### 实例

```ruby
> db.test.find({"age":{$gt:"18"}}, {_id:0, status:0}).pretty()
{ "name" : "abc", "age" : "25" }
{ "name" : "dec", "age" : "19" }
{ "name" : "asd", "age" : "30" }
> db.test.find({"age":{$gt:"18"}}, {_id:0, status:0}).limit(2).pretty()
{ "name" : "abc", "age" : "25" }
{ "name" : "dec", "age" : "19" }
```

## MongoDB Skip() 方法

我们除了可以使用limit()方法来读取指定数量的数据外，还可以使用skip()方法来跳过指定数量的数据，skip方法同样接受一个数字参数作为跳过的记录条数。

#### **语法**

```ruby
> db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
```

#### **实例**

```ruby
> db.test.find({"age":{$gt:"18"}}, {_id:0, status:0}).limit(2).skip(1).pretty()
{ "name" : "dec", "age" : "19" }
{ "name" : "asd", "age" : "30" }
```
# MongoDB 排序

------

## MongoDB sort()方法

在MongoDB中使用使用sort()方法对数据进行排序，sort()方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而-1是用于降序排列。

#### **语法**

sort()方法基本语法如下所示：

```ruby
> db.COLLECTION_NAME.find().sort({KEY:1})
```

#### **实例**

```ruby
> db.col.find({},{"title":1,"likes":1, _id:0}).sort({"likes":-1})
{ "title" : "PHP 教程", "likes" : 200 }
{ "title" : "Java 教程", "likes" : 150 }
{ "title" : "MongoDB", "likes" : 110 }
> db.col.find({},{"title":1,"likes":1, _id:0}).sort({"likes":1})
{ "title" : "MongoDB", "likes" : 110 }
{ "title" : "Java 教程", "likes" : 150 }
{ "title" : "PHP 教程", "likes" : 200 }
```

##### 注意

skip()，limit()，sort() 三个放在一起执行的时候，不管其位置如何，执行的顺序总是 sort() > skip() > limit() 

如要按照位置顺序来执行，则需要借助管道流aggregate

可参考：https://blog.csdn.net/thewindkee/article/details/54670750



## MongoDB索引

索引能够极大地提高效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。

##### 索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中的一列或多列的值进行排序的一种结构。

#### ensureIndex() 方法

#### 语法

```ruby
> db.COLLECTION_NAME.ensureIndex({KEY:1})
```

语法中的 Key 值为你要创建的索引字段，1为指定按升序创建索引，如果你想按照降序来创建索引指定为-1即可。

### 实例

```ruby
> db.col.ensureIndex({"title":1})
```

ensureIndex() 方法中你也可以设置使用多个字段创建索引（关系型数据库中称作复合索引）。

```ruby
> db.col.ensureIndex({"title":1,"description":-1})
```

ensureIndex() 接收可选参数，可选参数列表如下：

| Parameter          | Type          | Description                              |
| ------------------ | ------------- | ---------------------------------------- |
| background         | Boolean       | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为**false**。 |
| unique             | Boolean       | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**.   |
| name               | string        | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| dropDups           | Boolean       | 在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 **false**. |
| sparse             | Boolean       | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. |
| expireAfterSeconds | integer       | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。         |
| v                  | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。       |
| weights            | document      | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language   | string        | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语      |
| language_override  | string        | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language. |

### 实例

在后台创建索引：

```ruby
> db.col.ensureIndex({"title":1},{"background":1})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
> db.col.ensureIndex({"likes":1})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 2,
	"numIndexesAfter" : 3,
	"ok" : 1
}
> db.col.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "test.col"
	},
	{
		"v" : 2,
		"key" : {
			"title" : 1
		},
		"name" : "title_1",
		"ns" : "test.col",
		"background" : 1
	},
	{
		"v" : 2,
		"key" : {
			"likes" : 1
		},
		"name" : "likes_1",
		"ns" : "test.col"
	}
]
```

通过在创建索引时加background:true 的选项，让创建工作在后台执行

查看索引：`db.collection.getIndexes()`，默认_id作为索引，由数据库本身来维持，且不可删除。