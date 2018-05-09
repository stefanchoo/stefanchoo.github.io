---
layout: post
title: "MongoDB 进阶"
description: "NoSQL 文档数据库"
category : [SQL]
tags : [MongoDB, NoSQL]
---
{% include JB/setup %}

## MongoDB关系

#### 嵌入式关系

使用嵌入式方法，我们可以把用户地址嵌入到用户的文档中：

```javascript
 {
    "_id" : ObjectId("5af10a5dc1ff9494de25bf1e"),
    "contact" : "987654321",
    "dob" : "01-01-1991",
    "name" : "Tom Benzamin",
    "address" : [ 
        {
            "building" : "22 A, Indiana Apt",
            "pincode" : 123456.0,
            "city" : "Los Angeles",
            "state" : "California"
        }, 
        {
            "building" : "170 A, Acropolis Apt",
            "pincode" : 456789.0,
            "city" : "Chicago",
            "state" : "Illinois"
        }
    ]
}
```

这种方式的缺点是数据冗余比较大

#### 引用式关系

把用户数据文档和用户地址数据文档分开，通过引用文档的 **id** 字段来建立关系。

```Ruby
{
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin",
   "address_ids": [
      ObjectId("5af10c5bc1ff9494de25bf1f"),
      ObjectId("5af10c5bc1ff9494de25bf20")
   ]
}
```

```javascript
> var result = db.user.findOne({name:"Jackey"}, {address_ids:1, _id:0})
> db.address.find({_id:{$in:result["address_ids"]}}).pretty()
{
	"_id" : ObjectId("5af10c5bc1ff9494de25bf1f"),
	"building" : "22 A, Indiana Apt",
	"pincode" : 123456,
	"city" : "Los Angeles",
	"state" : "California"
}
{
	"_id" : ObjectId("5af10c5bc1ff9494de25bf20"),
	"building" : "170 A, Acropolis Apt",
	"pincode" : 456789,
	"city" : "Chicago",
	"state" : "Illinois"
}
```

## MongoDB查询分析

#### 使用 explain()

```javascript
> db.col.find({likes: {$gt:120}}, {title:1, _id:0}).explain()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "test.col",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"likes" : {
				"$gt" : 120
			}
		},
		"winningPlan" : {
			"stage" : "PROJECTION",
			"transformBy" : {
				"title" : 1,
				"_id" : 0
			},
			"inputStage" : {
				"stage" : "FETCH",
				"inputStage" : {
					"stage" : "IXSCAN",
					"keyPattern" : {
						"likes" : 1
					},
                  "indexName" : "likes_1",     # 使用的索引
					"isMultiKey" : false,
					"multiKeyPaths" : {
						"likes" : [ ]
					},
					"isUnique" : false,
					"isSparse" : false,
					"isPartial" : false,
					"indexVersion" : 2,
					"direction" : "forward",
					"indexBounds" : {
						"likes" : [
							"(120.0, inf.0]"
						]
					}
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "ChoodeMacBook-Pro.local",
		"port" : 27017,
		"version" : "3.6.4",
		"gitVersion" : "d0181a711f7e7f39e60b5aeb1dc7097bf6ae5856"
	},
	"ok" : 1
}
```

#### 使用 hint()

指定一个索引

```javascript
> db.col.find({likes: {$gt:120}, title:"Java 教程"}, {title:1, _id:0}).hint({likes:1}).explain()
```

## MongoDB原子操作

mongdb不支持事务，所以，在你的项目中应用时，要注意这点。无论什么设计，都不要要求mongodb保证数据的完整性。

但是mongodb提供了许多原子操作，比如文档的保存，修改，删除等，都是原子操作。

所谓原子操作就是要么这个文档保存到Mongodb，要么没有保存到Mongodb，不会出现查询到的文档没有保存完整的情况。

```javascript
book = {
          _id: 123456789,
          title: "MongoDB: The Definitive Guide",
          author: [ "Kristina Chodorow", "Mike Dirolf" ],
          published_date: ISODate("2010-09-24"),
          pages: 216,
          language: "English",
          publisher_id: "oreilly",
          available: 3,
          checkout: [ { by: "joe", date: ISODate("2012-10-15") } ]
        }
```

你可以使用 db.collection.findAndModify() 方法来判断书籍是否可结算并更新新的结算信息。

在同一个文档中嵌入的 available 和 checkout 字段来确保这些字段是同步更新的:

```javascript
db.books.findAndModify ( {
   query: {
            _id: 123456789,
            available: { $gt: 0 }
          },
   update: {
             $inc: { available: -1 },
             $push: { checkout: { by: "abc", date: new Date() } }
           }
} )
```



## 原子操作常用命令

#### $set

用来指定一个键并更新键值，若键不存在并创建。

```ruby
{ $set : { field : value } }
```

#### $unset 

用来删除一个键。

```ruby
{ $unset : { field : 1} }
```

#### $inc

$inc可以对文档的某个值为数字型（只能为满足要求的数字）的键进行增减的操作。

```ruby
{ $inc : { field : value } }
```

#### $push

用法：

```ruby
{ $push : { field : value } }
```

把value追加到field里面去，field一定要是数组类型才行，如果field不存在，会新增一个数组类型加进去。

#### $pushAll

同$push,只是一次可以追加多个值到一个数组字段内。

```Ruby
{ $pushAll : { field : value_array } }
```

#### $pull

从数组field内删除一个等于value值。

```ruby
{ $pull : { field : _value } }
```

#### $addToSet

增加一个值到数组内，而且只有当这个值不在数组内才增加。

#### $pop

删除数组的第一个或最后一个元素

```Ruby
{ $pop : { field : 1 } }
```

#### $rename

修改字段名称

```Ruby
{ $rename : { old_field_name : new_field_name } }
```

#### $bit

位操作，integer类型

```ruby
{$bit : { field : {and : 5}}}
```

#### 偏移操作符

```javascript
> t.find() { "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC", "comments" : [ { "by" : "joe", "votes" : 3 }, { "by" : "jane", "votes" : 7 } ] }
 
> t.update( {'comments.by':'joe'}, {$inc:{'comments.$.votes':1}}, false, true )
 
> t.find() { "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC", "comments" : [ { "by" : "joe", "votes" : 4 }, { "by" : "jane", "votes" : 7 } ] }
```



## MongoDB 高级索引

考虑以下文档集合（users ）:

```Json
{
   "address": {
      "city": "Los Angeles",
      "state": "California",
      "pincode": "123"
   },
   "tags": [
      "music",
      "cricket",
      "blogs"
   ],
   "name": "Tom Benzamin"
}
```

以上文档包含了 address 子文档和 tags 数组。

#### 索引数组字段

假设我们基于标签来检索用户，为此我们需要对集合中的数组 tags 建立索引。

在数组中创建索引，需要对数组中的每个字段依次建立索引。所以在我们为数组 tags 创建索引时，会为 music、cricket、blogs三个值建立单独的索引。

使用以下命令创建数组索引：

```javascript
> db.users.ensureIndex({"tags":1})
```

创建索引后，我们可以这样检索集合的 tags 字段：

```javascript
> db.users.find({tags:"cricket"})
```

为了验证我们使用使用了索引，可以使用 explain 命令：

```javascript
> db.users.find({tags:"cricket"}).explain()
```

以上命令执行结果中会显示已经使用的索引。



#### 索引子文档字段

假设我们需要通过city、state、pincode字段来检索文档，由于这些字段是子文档的字段，所以我们需要对子文档建立索引。

为子文档的三个字段创建索引，命令如下：

```javascript
> db.users.ensureIndex({"address.city":1,"address.state":1,"address.pincode":1})
```

一旦创建索引，我们可以使用子文档的字段来检索数据：

```javascript
> db.users.find({"address.city":"Los Angeles"})   
```

查询表达不一定遵循指定的索引的顺序，mongodb 会自动优化。所以上面创建的索引将支持以下查询：

```javascript
> db.users.find({"address.state":"California","address.city":"Los Angeles"}) 
```

同样支持以下查询：

```javascript
> db.users.find({"address.city":"LosAngeles","address.state":"California",
  "address.pincode":"123"})
```



## MongoDB 索引限制

#### 额外开销

每个索引占据一定的存储空间，在进行插入，更新和删除操作时也需要对索引进行操作。所以，如果你很少对集合进行读取操作，建议不使用索引。

#### 内存(RAM)使用

由于索引是存储在内存(RAM)中,你应该确保该索引的大小不超过内存的限制。

如果索引的大小大于内存的限制，MongoDB会删除一些索引，这将导致性能下降。

#### 查询限制

索引不能被以下的查询使用：

- 正则表达式及非操作符，如 $nin, $not, 等。
- 算术运算符，如 $mod, 等。
- $where 子句

所以，检测你的语句是否使用索引是一个好的习惯，可以用explain来查看。

#### 索引键限制

从2.6版本开始，如果现有的索引字段的值超过索引键的限制，MongoDB中不会创建索引。

#### 插入文档超过索引键限制

如果文档的索引字段值超过了索引键的限制，MongoDB不会将任何文档转换成索引的集合。与mongorestore和mongoimport工具类似。

#### 最大范围

- 集合中索引不能超过64个
- 索引名的长度不能超过128个字符 
- 一个复合索引最多可以有31个字段



## MongoDB Map Reduce

Map-Reduce是一种计算模型，简单的说就是将大批量的工作（数据）分解（MAP）执行，然后再将结果合并成最终结果（REDUCE）。

MongoDB提供的Map-Reduce非常灵活，对于大规模数据分析也相当实用。

#### MapReduce 命令

以下是MapReduce的基本语法：

```javascript
> db.collection.mapReduce(
   function() {emit(key,value);},  //map 函数
   function(key,values) {return reduceFunction},   //reduce 函数
   {
      out: collection,
      query: document,
      sort: document,
      limit: number
   }
)
```

使用 MapReduce 要实现两个函数 Map 函数和 Reduce 函数,Map 函数调用 emit(key, value), 遍历 collection 中所有的记录, 将 key 与 value 传递给 Reduce 函数进行处理。

Map 函数必须调用 emit(key, value) 返回键值对。

参数说明:

- **map**  映射函数 (生成键值对序列,作为 reduce 函数参数)。
- **reduce**  统计函数，reduce函数的任务就是将key-values变成key-value，也就是把values数组变成一个单一的值value。。
- **out**   统计结果存放集合 (不指定则使用临时集合,在客户端断开后自动删除)。
- **query**  一个筛选条件，只有满足条件的文档才会调用map函数。（query。limit，sort可以随意组合）
- **sort**  和limit结合的sort排序参数（也是在发往map函数前给文档排序），可以优化分组机制 
- **limit**  发往map函数的文档数量的上限（要是没有limit，单独使用sort的用处不大） 

以下实例在集合 orders 中查找 status:"A" 的数据，并根据 cust_id 来分组，并计算 amount 的总和。

![img](http://static.runoob.com/images/map-reduce.bakedsvg.svg)



#### 使用 MapReduce

考虑以下文档结构存储用户的文章，文档存储了用户的 user_name 和文章的 status 字段：

```javascript
> db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
> db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
> db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
> db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
> db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"disabled"
})
WriteResult({ "nInserted" : 1 })
> db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "runoob",
   "status":"disabled"
})
WriteResult({ "nInserted" : 1 })
> db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "runoob",
   "status":"disabled"
})
WriteResult({ "nInserted" : 1 })
> db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "runoob",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
```

现在，我们将在 posts 集合中使用 mapReduce 函数来选取已发布的文章(status:"active")，并通过user_name分组，计算每个用户的文章数：

```javascript
> db.posts.mapReduce( 
   function() { emit(this.user_name,1); }, 
   function(key, values) {return Array.sum(values)}, 
      {  
         query:{status:"active"},  
         out:"post_total" 
      }
)
```

以上 mapReduce 输出结果为：

```javascript
{
        "result" : "post_total",
        "timeMillis" : 23,
        "counts" : {
                "input" : 5,
                "emit" : 5,
                "reduce" : 1,
                "output" : 2
        },
        "ok" : 1
}
```

结果表明，共有 5 个符合查询条件（status:"active"）的文档， 在map函数中生成了 5 个键值对文档，最后使用reduce函数将相同的键值分为 2 组。

具体参数说明：

- **result** 储存结果的collection的名字,这是个临时集合，MapReduce的连接关闭后自动就被删除了。
- **timeMillis** 执行花费的时间，毫秒为单位
- **input** 满足条件被发送到map函数的文档个数
- **emit** 在map函数中emit被调用的次数，也就是所有集合中的数据总量
- **ouput** 结果集合中的文档个数**（count对调试非常有帮助）**
- **ok** 是否成功，成功为1
- **err** 如果失败，这里可以有失败原因，不过从经验上来看，原因比较模糊，作用不大

使用 find 操作符来查看 mapReduce 的查询结果：

```javascript
> db.posts.mapReduce( 
   function() { emit(this.user_name,1); }, 
   function(key, values) {return Array.sum(values)}, 
      {  
         query:{status:"active"},  
         out:"post_total" 
      }
).find()
```

以上查询显示如下结果，两个用户 tom 和 mark 有两个发布的文章:

```javascript
{ "_id" : "mark", "value" : 4 }
{ "_id" : "runoob", "value" : 1 }
```

用类似的方式，MapReduce可以被用来构建大型复杂的聚合查询。

Map函数和Reduce函数可以使用 JavaScript 来实现，使得MapReduce的使用非常灵活和强大。



## MongoDB 正则表达式

正则表达式是使用单个字符串来描述、匹配一系列符合某个句法规则的字符串。

许多程序设计语言都支持利用正则表达式进行字符串操作。

MongoDB 使用 **$regex** 操作符来设置匹配字符串的正则表达式。

MongoDB使用PCRE (Perl Compatible Regular Expression) 作为正则表达式语言。

不同于全文检索，我们使用正则表达式不需要做任何配置。

考虑以下 **posts** 集合的文档结构，该文档包含了文章内容和标签：

```Json
{
   "post_text": "enjoy the mongodb articles on runoob",
   "tags": [
      "mongodb",
      "runoob"
   ]
}
```

#### 使用正则表达式

以下命令使用正则表达式查找包含 runoob 字符串的文章：

```javascript
> db.posts.find({post_text:{$regex:"runoob"}})
```

以上查询也可以写为：

```javascript
> db.posts.find({post_text:/runoob/})
```

#### 不区分大小写的正则表达式

如果检索需要不区分大小写，我们可以设置 $options 为 $i。

以下命令将查找不区分大小写的字符串 runoob：

```javascript
> db.posts.find({post_text:{$regex:"runoob",$options:"$i"}})
```

集合中会返回所有包含字符串 runoob 的数据，且不区分大小写：

```javascript
{
   "_id" : ObjectId("53493d37d852429c10000004"),
   "post_text" : "hey! this is my post on  runoob", 
   "tags" : [ "runoob" ]
} 
```

#### 数组元素使用正则表达式

我们还可以在数组字段中使用正则表达式来查找内容。 这在标签的实现上非常有用，如果你需要查找包含以 run 开头的标签数据(ru 或 run 或 runoob)， 你可以使用以下代码：

```javascript
> db.posts.find({tags:{$regex:"run"}})
```

#### 优化正则表达式查询

- 如果你的文档中字段设置了索引，那么使用索引相比于正则表达式匹配查找所有的数据查询速度更快。
- 如果正则表达式是前缀表达式，所有匹配的数据将以指定的前缀字符串为开始。例如： 如果正则表达式为 **^tut **，查询语句将查找以 tut 为开头的字符串。

**这里面使用正则表达式有两点需要注意：**

正则表达式中使用变量。一定要使用eval将组合的字符串进行转换，不能直接将字符串拼接后传入给表达式。否则没有报错信息，只是结果为空！实例如下：

```javascript
var name=eval("/" + 变量值key +"/i"); 
```

以下是模糊查询包含title关键词, 且不区分大小写:

```javascript
title:eval("/"+title+"/i")    // 等同于 title:{$regex:title,$Option:"$i"}   
```



### MongoDB自动增长

MongoDB 没有像 SQL 一样有自动增长的功能， MongoDB 的 _id 是系统自动生成的12字节唯一标识。

可以使用编程的方式实现id自增长

#### 使用 counters 集合

创建 counters 集合，序列字段值可以实现自动长：

```shell
> db.createCollection("counters")
```

现在我们向 counters 集合中插入以下文档，使用 productid 作为 key:

```json
{
  "_id":"productid",
  "sequence_value": 0
}
```

sequence_value 字段是序列通过自动增长后的一个值。

使用以下命令插入 counters 集合的序列文档中：

```javascript
> db.counters.insert({_id:"productid",sequence_value:0})
```

#### 创建 Javascript 函数

现在，我们创建函数 getNextSequenceValue 来作为序列名的输入， 指定的序列会自动增长 1 并返回最新序列值。在本文的实例中序列名为 productid 。

```javascript
> function getNextSequenceValue(sequenceName){
   var sequenceDocument = db.counters.findAndModify(
      {
         query:{_id: sequenceName },
         update: {$inc:{sequence_value:1}},
         "new":true
      });
   return sequenceDocument.sequence_value;
}
```

#### 使用 Javascript 函数

接下来我们将使用 getNextSequenceValue 函数创建一个新的文档， 并设置文档 _id 自动为返回的序列值：

```javascript
> db.products.insert({
   "_id":getNextSequenceValue("productid"),
   "product_name":"Apple iPhone",
   "category":"mobiles"})

> db.products.insert({
   "_id":getNextSequenceValue("productid"),
   "product_name":"Samsung S3",
   "category":"mobiles"})
```

就如你所看到的，我们使用 getNextSequenceValue 函数来设置 _id 字段。

为了验证函数是否有效，我们可以使用以下命令读取文档：

```javascript
> db.products.find()
```

以上命令将返回以下结果，我们发现 _id 字段是自增长的：

```json
{ "_id" : 1, "product_name" : "Apple iPhone", "category" : "mobiles"}
{ "_id" : 2, "product_name" : "Samsung S3", "category" : "mobiles" }
```