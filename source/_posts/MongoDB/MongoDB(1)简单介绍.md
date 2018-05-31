title: 一.简单介绍
date: 2014-04-03 14:45:23
tags: [MongoDB]
---

### 1.什么是MongoDB
---
MongoDB是一个基于分布式文件存储的数据库，由C++语言编写，旨在为WEB应用提供可扩展的高性能数据存储解决方案。

MongoDB是一个高性能，开源，无模式的文档型数据库，官方给自己的定义是Key-value存储(高性能和高扩展)和传统RDBMS(丰富的查询和功能)之间的一座桥梁。

### 2.Document和BSON
---
MongoDB中保存的数据格式为BSON，如：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkbson.jpg)

MongoDB中数据的基本单元称为文档(Document)，它是MongoDB的核心概念，由多个键极其关联的值有序的放置在一起组成，数据库中它对应于关系型数据库的行。

数据在MongoDB中以BSON（Binary-JSON）文档的格式存储在磁盘上。

BSON（Binary Serialized Document Format）是一种类json的一种二进制形式的存储格式，简称Binary JSON，BSON和JSON一样，支持内嵌的文档对象和数组对象，但是BSON有JSON没有的一些数据类型，如Date和BinData类型。

BSON的优点是灵活性高，但它的缺点是空间利用率不是很理想，BSON有三个特点：轻量性、可遍历性、高效性。
