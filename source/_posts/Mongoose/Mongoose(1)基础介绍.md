title: 一.基础知识
date: 2014-03-09 14:45:23
tags: [Mongoose]
---

### 1.认识Mongoose
---
什么是Mongoose呢，它于MongoDB又是什么关系呢，它可以用来做什么呢，介绍Mongoose之前，我们先简单了解一下MongoDB。

MongoDB是一个开源的NoSQL数据库，相比MySQL那样的关系型数据库，它更显得轻巧、灵活，非常适合在数据规模很大、事务性不强的场合下使用。同时它也是一个对象数据库，没有表、行等概念，也没有固定的模式和结构，所有的数据以文档的形式存储(文档，就是一个关联数组式的对象，它的内部由属性组成，一个属性对应的值可能是一个数、字符串、日期、数组，甚至是一个嵌套的文档。)，数据格式就是JSON。

介绍了MongoDB，我们下面就要认识Mongoose了。

1. Mongoose是什么？

Mongoose是MongoDB的一个对象模型工具，是基于node-mongodb-native开发的MongoDB nodejs驱动，可以在异步的环境下执行。同时它也是针对MongoDB操作的一个对象模型库，封装了MongoDB对文档的的一些增删改查等常用方法，让NodeJS操作Mongodb数据库变得更加灵活简单。

2. Mongoose能做什么？

Mongoose，因为封装了对MongoDB对文档操作的常用处理方法，让NodeJS操作Mongodb数据库变得easy、easy、So easy!


### 2.使用Mongoose
---
1. 安装mongoose：
```
npm install mongoose
```
2. 引用mongoose：
```
var mongoose = require("mongoose");
```
3. 使用"mongoose"连接数据库：
```
var db = mongoose.connect("mongodb://user:pass@localhost:port/database");
```
4. 执行下面代码检查默认数据库test，是否可以正常连接成功?
```javascript
var mongoose = require("mongoose");
var db = mongoose.connect("mongodb://127.0.0.1:27017/test");
db.connection.on("error", function (error) {
    console.log("数据库连接失败：" + error);
});
db.connection.on("open", function () {
    console.log("------数据库连接成功！------");
});
```

### 3.了解集合
---
MongoDB —— 是一个对象数据库，没有表、行等概念，也没有固定的模式和结构，所有的数据以Document(以下简称文档)的形式存储(Document，就是一个关联数组式的对象，它的内部由属性组成，一个属性对应的值可能是一个数、字符串、日期、数组，甚至是一个嵌套的文档。)，后面我们会学习如何创建文档并插入内容。

在MongoDB中，多个Document可以组成Collection(以下简称集合)，多个集合又可以组成数据库。

我们想要操作MongoDB数据，那就得先要具备上面所说的包含数据的“文档”，文档又是什么意思呢，请看如下介绍。

文档 —— 是MongoDB的核心概念，是键值对的一个有序集，在JavaScript里文档被表示成对象。同时它也是MongoDB中数据的基本单元，非常类似于关系型数据库管理系统中的行，但更具表现力。

集合 —— 由一组文档组成，如果将MongoDB中的一个文档比喻成关系型数据库中的一行，那么一个集合就相当于一张表。

如果我们要通过Mongoose去创建一个“集合”并对其进行增删改查，该怎么实现呢，到这里我们就要先了解Schema(数据属性模型)、Model、Entity了。

### 4.Schema
---
Schema —— 一种以文件形式存储的数据库模型骨架，无法直接通往数据库端，也就是说它不具备对数据库的操作能力，仅仅只是数据库模型在程序片段中的一种表现，可以说是数据属性模型(传统意义的表结构)，又或着是“集合”的模型骨架。

那如何去定义一个Schema呢，请看示例：
```javascript
var mongoose = require("mongoose");
var TestSchema = new mongoose.Schema({
    name : { type:String },//属性name,类型为String
    age  : { type:Number, default:0 },//属性age,类型为Number,默认为0
    time : { type:Date, default:Date.now },
    email: { type:String,default:''}
});
```
基本属性类型有：字符串、日期型、数值型、布尔型(Boolean)、null、数组、内嵌文档等。


### 5.Model
---
Model —— 由Schema构造生成的模型，除了Schema定义的数据库骨架以外，还具有数据库操作的行为，类似于管理数据库属性、行为的类。

如何通过Schema来创建Model呢，如下示例：
```
var db = mongoose.connect("mongodb://127.0.0.1:27017/test");
```
// 创建Model
```
var TestModel = db.model("test1", TestSchema);
```
test1：数据库中的集合名称,当我们对其添加数据时如果test1已经存在，则会保存到其目录下，如果未存在，则会创建test1集合，然后在保存数据。

拥有了Model，我们也就拥有了操作数据库的金钥匙。

如果你想对某个集合有所作为，那就交给Model模型来处理吧，创建一个Model模型，我们需要指定：1.集合名称，2.集合的Schema结构对象，满足这两个条件，我们就会拥有一个操作数据库的金钥匙。


### 6.Entity
---
Entity —— 由Model创建的实体，使用save方法保存数据，Model和Entity都有能影响数据库的操作，但Model比Entity更具操作性。

使用Model创建Entity，如下示例：
```
var TestEntity = new TestModel({
       name : "Lenka",
       age  : 36,
       email: "lenka@qq.com"
});
console.log(TestEntity.name); // Lenka
console.log(TestEntity.age); // 36
```
创建成功之后，Schema属性就变成了Model和Entity的公共属性了。

### 7.创建集合
---
下面是关于一些基础数据的定义
```javascript
var mongoose = require("mongoose");
var db = mongoose.connect("mongodb://127.0.0.1:27017/test");
var TestSchema = new mongoose.Schema({
    name : { type:String },
    age  : { type:Number, default:0 },
    email: { type:String },
    time : { type:Date, default:Date.now }
});
var TestModel = db.model("test1", TestSchema );
var TestEntity = new TestModel({
    name : "helloworld",
    age  : 28,
    email: "helloworld@qq.com"
});
TestEntity.save(function(error,doc){
  if(error){
     console.log("error :" + error);
  }else{
     console.log(doc);
  }
});
```
上面代码通过request.url属性，判断请求的网址，从而返回不同的内容。


### 8.小结
---
学习了如何通过Mongoose去创建一个数据库“集合”，还有定义“集合”的基本组成结构并使其具有相应的操作数据库能力。

简单回顾：

1. Schema：数据库集合的模型骨架，或者是数据属性模型传统意义的表结构。

2. Model ：通过Schema构造而成，除了具有Schema定义的数据库骨架以外，还可以具体的操作数据库。

3. Entity：通过Model创建的实体，它也可以操作数据库。