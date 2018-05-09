---
layout: post
title: "MongoDB 简介"
description: "NoSQL 文档数据库"
category : [SQL]
tags : [MongoDB, NoSQL]
---
{% include JB/setup %}

## MongoDB是什么？

一个NoSQL数据库，是NoSQL中的一个分支：文档数据库。

一个基于分布式文件存储的数据库。由C++语言编写，旨在为WEB应用提供可扩展的高性能数据存储方案。

一个介于关系数据库和非关系数据库之间的产品，是非关系数据库中功能最丰富，最像关系数据库的。

在高负载的情况下，添加更多的节点，可以保证服务器性能。

## MongoDB的主要特点

- MongoDB是一个面向文档存储的数据库，安装、操作简单
- 可以设置任何属性的索引（如：firstname="Sameer", Address="8 Gandhi Road"）来实现更快的排序
- 可以通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性
- 如果负载增加（需要更多的存储空间和更强的处理能力），可以分布在计算机网络中的其他节点上，这就是所谓的分片
- 支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组
- 使用update()命令可以实现替换完成的文档（数据）或者一些指定的数据字段
- Map/Reduce主要是用来对数据进行批量处理和聚合操作
- Map和Reduce Map函数调用emit(key,value)遍历集合中所有的记录，将key与value传给Reduce函数进行处理
- Map函数和Reduce函数是使用Javascript编写的，并可以通过db.runCommand或mapreduce命令来执行MapReduce操作
- GridFS是MongoDB中的一个内置功能，可以用于存放大量小文件
- 允许在服务端执行脚本，可以用Javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可
- 支持各种编程语言:RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言

## NoSQL的应用场景是什么？

摘至：作者 接灰的电子产品 链接 https://www.jianshu.com/p/fe3b9532b852

假设说我们现在要构建一个论坛，用户可以发布帖子（帖子内容包括文本、视频、音频和图片等）。那么我们可以画出一个下图的表关系结构。

![论坛ER简略图]({{"/assets/images/post.png" | absolute_url}})

这种情况下如果我们需要展示帖子的文字，以及关联的图片、音频、视频、用户评论、赞和用户信息的话，我们需要关联8张表来取得自己想要的数据。如果我们有这样的帖子列表，随着用户的滚动动态加载，同时需要监听是否有新内容产生，这样的一个任务我们需要太多这种复杂的查询了。

NoSQL解决这类问题的思路是，干脆抛弃传统的表结构，直接存储包含一个帖子所有内容的数据，像下面这样。

```json
{
	"id":"5894a12f-dae1-5ab0-5761-1371ba4f703e",
	"title":"2017年的Spring发展方向","date":"2017-01-21",
	"body":"这篇文章主要探讨如何利用Spring Boot集成NoSQL",
	"createdBy":User,
	"images":["http://dev.local/myfirstimage.png","http://dev.local/mysecondimage.png"],
	"videos":[
 		{"url":"http://dev.local/myfirstvideo.mp4", "title":"The first video"},
		{"url":"http://dev.local/mysecondvideo.mp4", "title":"The second video"} 
	],
	"audios":[ 
		{"url":"http://dev.local/myfirstaudio.mp3", "title":"The first audio"}, 
		{"url":"http://dev.local/mysecondaudio.mp3", "title":"The second audio"} 
	]
}
```

NoSQL一般情况下是没有Schema这个概念的，这给开发带来了较大的自由度。因为在关系型数据库中，一旦Schema确定，以后更改Schema，维护Schema是很麻烦的一件事。但反过来说Schema对于维护数据的完整性是非常必要的。

一般来说，如果你在做一个Web、物联网等类型的项目，你应该考虑使用NoSQL。如果你要面对的是一个对数据的完整性、事务处理等有严格要求的环境（如财务系统），则应该考虑关系型数据库。