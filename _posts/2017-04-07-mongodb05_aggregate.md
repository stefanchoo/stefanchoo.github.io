---
layout: post
title: "MongoDB 聚合"
description: "NoSQL 文档数据库"
category : [SQL]
tags : [MongoDB, NoSQL]
---
{% include JB/setup %}

## MongoDB聚合

MongoDB中聚合（aggregate）主要用于处理数据（诸如统计平均值，求和等），并返回计算后的数据结果。

#### aggregate()方法

#### 语法

```ruby
> db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

#### 实例

集合中的数据如下：

```json
{
    "_id" : ObjectId("5aefc46e58072d466fd8277e"),
    "title" : "MongoDB Overview",
    "description" : "MongoDB is no sql database",
    "by_user" : "runoob.com",
    "url" : "http://www.runoob.com",
    "tags" : [ 
        "mongodb", 
        "database", 
        "NoSQL"
    ],
    "likes" : 100.0
},
{
    "_id" : ObjectId("5aefc46e58072d466fd8277f"),
    "title" : "NoSQL Overview",
    "description" : "No sql database is very fast",
    "by_user" : "runoob.com",
    "url" : "http://www.runoob.com",
    "tags" : [ 
        "mongodb", 
        "database", 
        "NoSQL"
    ],
    "likes" : 10.0
},
{
    "_id" : ObjectId("5aefc46e58072d466fd82780"),
    "title" : "Neo4j Overview",
    "description" : "Neo4j is no sql database",
    "by_user" : "Neo4j",
    "url" : "http://www.neo4j.com",
    "tags" : [ 
        "neo4j", 
        "database", 
        "NoSQL"
    ],
    "likes" : 750.0
}
```

统计以上集合中每个作者所写的文章数，使用aggregate()计算结果如下:

```Ruby
> db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "runoob.com",
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

类似SQL： select by_user, count(*) from mycol group by by_user

下表展示了一些聚合的表达式:

| 表达式       | 描述                      | 实例                                       |
| --------- | ----------------------- | ---------------------------------------- |
| $sum      | 计算总和。                   | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}]) |
| $avg      | 计算平均值                   | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}]) |
| $min      | 获取集合中所有文档对应值得最小值。       | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}]) |
| $max      | 获取集合中所有文档对应值得最大值。       | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}]) |
| $push     | 在结果文档中插入值到一个数组中。        | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}]) |
| $addToSet | 在结果文档中插入值到一个数组中，但不创建副本。 | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}]) |
| $first    | 根据资源文档的排序获取第一个文档数据。     | db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}]) |
| $last     | 根据资源文档的排序获取最后一个文档数据     | db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}]) |

#### 管道的概念

管道在Unix和Linux中一般用于将当前命令的输出结果作为下一个命令的参数。

MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

表达式：处理输入文档并输出。表达式是无状态的，只能用于计算当前聚合管道的文档，不能处理其它的文档。

聚合框架中常用的几个操作：

- $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- $match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
- $limit：用来限制MongoDB聚合管道返回的文档数。
- $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
- $group：将集合中的文档分组，可用于统计结果。
- $sort：将输入文档排序后输出。
- $geoNear：输出接近某一地理位置的有序文档。

### 管道操作符实例

1、$project实例

```ruby
db.article.aggregate(
    { $project : {
        title : 1 ,
        author : 1 ,
    }}
 );
```

这样的话结果中就只还有_id,tilte和author三个字段了，默认情况下_id字段是被包含的，如果要想不包含_id话可以这样:

```ruby
db.article.aggregate(
    { $project : {
        _id : 0 ,
        title : 1 ,
        author : 1
    }});
```

2.$match实例

```ruby
db.articles.aggregate( [
                        { $match : { score : { $gt : 70, $lte : 90 } } },
                        { $group: { _id: null, count: { $sum: 1 } } }
                       ] );
```

$match用于获取分数大于70小于或等于90记录，然后将符合条件的记录送到下一阶段$group管道操作符进行处理。

3.$skip实例

```ruby
db.article.aggregate(
    { $skip : 5 });
```

经过$skip管道操作符处理后，前五个文档被"过滤"掉。



## MongoDB 复制（副本集）

MongoDB复制是将数据同步在多个服务器上的过程。

复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性，并可以保证数据的安全性。

复制还允许您从硬件故障和服务中断中恢复数据。



#### 什么是复制?

- 保障数据的安全性
- 数据高可用性 (24*7)
- 灾难恢复
- 无需停机维护（如备份，重建索引，压缩）
- 分布式读取数据

#### MongoDB 复制原理

mongodb的复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。

mongodb各个节点常见的搭配方式为：一主一从、一主多从。主节点记录在其上的所有操作oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

![MongoDB复制结构图](http://www.runoob.com/wp-content/uploads/2013/12/replication.png)

#### 副本集特征

- N个节点的集群
- 任何节点可作为主节点
- 所有写入操作都在主节点上
- 自动故障转移
- 自动恢复



##### 读写分离：主节点负责写入，子节点负责读取，从而实现读写分离

##### 试验参考：https://blog.csdn.net/majinggogogo/article/details/51586409

![操作截图]({{"/assets/images/replSet.png" | absolute_url}})



## MongoDB 分片

### 分片

在MongoDB里存在另一种集群，可以满足MongoDB数据量大量增长的需求。

当MongoDB存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据。

### 为什么使用分片

- 复制所有的写入操作到主节点
- 延迟的敏感数据会在主节点查询
- 单个副本集限制在12个节点
- 当请求量巨大时会出现内存不足。
- 本地磁盘不足
- 垂直扩展价格昂贵

### MongoDB分片

下图展示了在MongoDB中使用分片集群结构分布：

![img](http://www.runoob.com/wp-content/uploads/2013/12/sharding.png)

上图中主要有如下所述三个主要组件：

- **Shard:** 用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组个一个replica set承担，防止主机单点故障
- **Config Server:** mongod实例，存储了整个 ClusterMetadata，其中包括 chunk信息。
- **Query Routers:** 前端路由，客户端由此接入，且让整个集群看上去像单一数据库，前端应用可以透明使用。



#### Mongos

   MongoDB分片的基本思想就是将集合切分成小块.这些块分散到若干片里面,每个片只负责总数据的一部分。应用程序不必知道哪片对应哪些数据，甚至不需要知道数据已经被拆分了，所以在分片之前要运行一个路由进程，进程名mongos，这个路由器知道所有数据的存放位置，所以应用可以连接它来正常发送请求.对应用来说，它仅知道连接了一个普通的mongod。路由器知道和片的对应关系,能够转发请求到正确的片上。如果请求有了回应，路由器将其收集起来回送给应用。

   在没有分片的时候，客户端连接mongod进程，分片时客户端会连接mongos进程。mongos对应用隐藏了分片的细节。从应用的角度看，分片和不分片没有区别。所以需要扩展的时候，不必修改应用程序的代码。

#### 健壮的片

生产环境中,每个片都应是副本集,这样单个服务器坏了,就不会导致整个片失效.用addshard命令就可以将副本集作为片添加,

添加时，只要指定副本集的名称和种子就行了.

如要添加副本集，其中包含一个服务器127.0.0.1:27020(还有别的服务器),就可以用下列命令将其添加到集群中

```ruby
$ mongo localhost:40000
> use admin
> db.runCommand({ addshard: 'rs0/localhost:27020,localhost:27021'})
> db.runCommand({ addshard: 'rs1/localhost:27030,localhost:27031'})
> db.runCommand({ enablesharding: 'test'})   # 设置分片数据库
> db.runCommand({ shardcollection: 'test.user', key: {name: 1}})
```

参考：https://www.cnblogs.com/clsn/p/8214345.html#auto_id_22



## MongoDB 备份(mongodump)与恢复(mongorestore)

#### 1. MongoDB数据备份

#### 语法

mongodump命令脚本语法如下：

```Ruby
> mongodump -h dbhost -d dbname -o dbdirectory
```

- **-h:** MongDB所在服务器地址，例如：127.0.0.1，当然也可以指定端口号：127.0.0.1:27017
- **-d:** 需要备份的数据库实例，例如：test
- **-o:** 备份的数据存放位置，例如：c:\data\dump，当然该目录需要提前建立，在备份完成后，系统自动在dump目录下建立一个test目录，这个目录里面存放该数据库实例的备份数据。

**导出文件格式为 BSON 和 JSON**

mongodump 命令可选参数列表如下所示：

| 语法                                       | 描述                | 实例                                       |
| ---------------------------------------- | ----------------- | ---------------------------------------- |
| mongodump --host HOST_NAME --port PORT_NUMBER | 该命令将备份所有MongoDB数据 | mongodump --host runoob.com --port 27017 |
| mongodump --dbpath DB_PATH --out BACKUP_DIRECTORY |                   | mongodump --dbpath /data/db/ --out /data/backup/ |
| mongodump --collection COLLECTION --db DB_NAME | 该命令将备份指定数据库的集合。   | mongodump --collection mycol --db test   |



### 2. MongoDB数据恢复

mongodb使用 mongorestore 命令来恢复备份的数据。

#### 语法

mongorestore命令脚本语法如下：

```ruby
> mongorestore -h <hostname><:port> -d dbname <path>
```

- **--host <:port>, -h <:port>:** MongoDB所在服务器地址，默认为： localhost:27017
- **--db, -d:** 需要恢复的数据库实例，例如：test，当然这个名称也可以和备份时候的不一样，比如test2
- **--drop:** 恢复的时候，先删除当前数据，然后恢复备份的数据。就是说，恢复后，备份后添加修改的数据都会被删除，慎用哦！
- **path:** mongorestore 最后的一个参数，设置备份数据所在位置，例如：c:\data\dump\test。你不能同时指定 <path> 和 --dir 选项，--dir也可以设置备份目录。
- **--dir:** 指定备份的目录你不能同时指定 <path> 和 --dir 选项。



# MongoDB 监控

### mongostat 命令

它会间隔固定时间获取mongodb的当前运行状态，并输出。

### mongotop 命令

mongotop提供了一个方法，用来跟踪一个MongoDB的实例，查看哪些大量的时间花费在读取和写入数据。 mongotop提供每个集合的水平的统计数据。默认情况下，mongotop返回值的每一秒

```ruby
mongotop <sleeptime> —locks
```

输出结果字段说明：

- **ns:** 包含数据库命名空间，后者结合了数据库名称和集合。
- **db:** 包含数据库的名称。名为 . 的数据库针对全局锁定，而非特定数据库。
- **total:** mongod花费的时间工作在这个命名空间提供总额。 
- **read:** 提供了大量的时间，这mongod花费在执行读操作，在此命名空间。 
- **write:** 提供这个命名空间进行写操作，这mongod花了大量的时间。
