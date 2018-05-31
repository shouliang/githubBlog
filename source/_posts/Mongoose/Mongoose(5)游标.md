title: 五.游标
date: 2014-03-22 14:45:23
tags: [Mongoose]
---

### 1.简介
---
数据库使用游标返回find的执行结果。客户端对游标的实现通常能够对最终结果进行有效的控制。可以限制结果的数量，略过部分结果，根据任意键按任意顺序的组合对结果进行各种排序，或者是执行其他一些强的操作。

最常用的查询选项就是限制返回结果的数量(limit函数)、忽略一点数量的结果(skip函数)以及排序(sort函数)。所有这些选项一点要在查询被发送到服务器之前指定。

### 2.限制返回数量
---
#### limit函数的基本用法
在查询操作中，有时数据量会很大，这时我们就需要对返回结果的数量进行限制，那么我们就可以使用limit函数，通过它来限制结果数量。

1.限制数量：find(Conditions,fields,options,callback);
```
Model.find({},null,{limit:20},function(err,docs){
    console.log(docs);
});
```
如果匹配的结果不到20个，则返回匹配数量的结果，也就是说limit函数指定的是上限而非下限。

### 3.跳过结果数量
---
#### skip函数的基本用法
skip函数和limit类似，都是对返回结果数量进行操作，不同的是skip函数的功能是略过指定数量的匹配结果，返回余下的查询结果。如下示例：

1.跳过数量：find(Conditions,fields,options,callback);
```
Model.find({},null,{skip:4},function(err,docs){
    console.log(docs);
});
```
如果查询结果数量中少于4个的话，则不会返回任何结果。

### 4.排序
---
#### sort函数的基本用法
sort函数可以将查询结果数据进行排序操作，该函数的参数是一个或多个键/值对，键代表要排序的键名，值代表排序的方向，1是升序，-1是降序。

1.结果排序：find(Conditions,fields,options,callback);
```
Model.find({},null,{sort:{age:-1}},function(err,docs){
  //查询所有数据，并按照age降序顺序返回数据docs
});
```
sort函数可根据用户自定义条件有选择性的来进行排序显示数据结果。

### 5.小结
---
简单回顾：

1. limit函数：限制返回结果的数量。

2. skip函数：略过指定的返回结果数量。

3. sort函数：对返回结果进行有效排序。
