title: Mongoose四.高级查询
date: 2014-03-19 8:45:23
tags: [Node.js]
---

### 1.条件查询
---
接下来我们来看一些在查询时会用的常用操作符，通过操作符的使用，我们就可对数据进行更细致性的查询，一起来看一下吧。

"$lt"(小于)，"$lte"(小于等于),"$gt"(大于)，"$gte"(大于等于)，"$ne"(不等于)，"$in"(可单值和多个值的匹配)，"$or"(查询多个键值的任意给定值)，"$exists"(表示是否存在的意思)"$all"。

### 2.大、小于
---
#### $gt、$lt简述
查询时我们经常会碰到要根据某些字段进行条件筛选查询，比如说Number类型，怎么办呢，我们就可以使用$gt(>)、$lt(<)、$lte(<=)、$gte(>=)操作符进行排除性的查询，如下示例：
```
Model.find({"age":{"$gt":18}},function(error,docs){
   //查询所有nage大于18的数据
});
Model.find({"age":{"$lt":60}},function(error,docs){
   //查询所有nage小于60的数据
});
Model.find({"age":{"$gt":18,"$lt":60}},function(error,docs){
   //查询所有nage大于18小于60的数据
});
```
如果我们要对类似age字段的数据进行筛选，使用$gt、$lt是不是很方便快捷呢！

### 3.不匹配
---
#### $ne简述
$ne(!=)操作符的含义相当于不等于、不包含，查询时我们可通过它进行条件判定，具体使用方法如下：
```
Model.find({ age:{ $ne:24}},function(error,docs){
    //查询age不等于24的所有数据
});
Model.find({name:{$ne:"tom"},age:{$gte:18}},function(error,docs){
  //查询name不等于tom、age>=18的所有数据
});
```
$ne可以匹配单个值，也可以匹配不同类型的值。

### 4.匹配
---
#### $in简述
和$ne操作符相反，$in相当于包含、等于，查询时查找包含于指定字段条件的数据。具体使用方法如下：
```
Model.find({ age:{ $in: 20}},function(error,docs){
   //查询age等于20的所有数据
});
Model.find({ age:{$in:[20,30]}},function(error,docs){
  //可以把多个值组织成一个数组
}); 
```
$in和$ne除了条件判定不同，用法是不是很相似呀！


### 5.或者
---
#### $or
$or操作符，可以查询多个键值的任意给定值，只要满足其中一个就可返回，用于存在多个条件判定的情况下使用，如下示例：

```
Model.find({"$or":[{"name":"yaya"},{"age":28}]},function(error,docs){
  //查询name为yaya或age为28的全部文档
});
```

### 6.存在
---
#### $exists
$exists操作符，可用于判断某些关键字段是否存在来进行条件查询。如下示例：

``` 
Model.find({name: {$exists: true}},function(error,docs){
  //查询所有存在name属性的文档
});
Model.find({telephone: {$exists: false}},function(error,docs){
  //查询所有不存在telephone属性的文档
});
```
### 7.小结
---
简单回顾：

1. $gt(>),$lt(<),$lte(<=),$gte(>=)操作符：针对Number类型的查询具体超强的排除性。

2. $ne(!=)操作符：相当于不等于、不包含，查询时可根据单个或多个属性进行结果排除。

3. $in操作符：和$ne操作符用法相同，但意义相反。

4. $or操作符：可查询多个条件，只要满足其中一个就可返回结果值。

5. $exists操作符：主要用于判断某些属性是否存在。