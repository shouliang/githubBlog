title: 三.简单查询
date: 2014-03-14 14:45:23
tags: [Mongoose]
---

### 1.简介
---
查询就是返回一个集合中的文档的子集，Mongoose 模型提供了find、findOne、和findById方法用于文档查询。

这里先添加一些测试数据。
```
TestModel.create([
                  { name:"test1", age:20 },
                  { name:"test2", age:30 },
                  { name:"test3", age:24 },
                  { name:"test4", age:18 },
                  { name:"test5", age:60 },
                  { name:"test6", age:50, email:"t6@qq.com" },
                  { name:"test7", age:40, email:"t7@163.com" },
                  { name:"test8", age:27 },
                  { name:"test9", age:27, email:"t9@yeah.net" },
                  { name:"test10",age:65 }
                 ], function(error,docs) {
    if(error) {
        console.log(error);
    } else {
        console.log('save ok');
    }
});
```

### 2.find过滤查询
---
了解和掌握它的过滤功能，怎么个过滤法呢，请看如下介绍。

1.属性过滤 find(Conditions,field,callback);

field省略或为Null，则返回所有属性。
```
//返回只包含一个键值name、age的所有记录
Model.find({},{name:1, age:1, _id:0}，function(err,docs){
   //docs 查询结果集
})
```
说明：我们只需要把显示的属性设置为大于零的数就可以，当然1是最好理解的，_id是默认返回，如果不要显示加上("_id":0)，但是，对其他不需要显示的属性且不是_id，如果设置为0的话将会抛异常或查询无果。

### 3.findOne查询
---
与find相同，但只返回单个文档，也就说当查询到即一个符合条件的数据时，将停止继续查询，并返回查询结果。

1.单条数据 findOne(Conditions,callback);
```
TestModel.findOne({ age: 27}, function (err, doc){
   // 查询符合age等于27的第一条数据
   // doc是查询结果
});
```
findOne方法，只返回第一个符合条件的文档数据。

### 4.findById
---
与findOne相同，但它只接收文档的_id作为参数，返回单个文档。

1.单条数据 findById(_id, callback);
```
TestModel.findById('obj._id', function (err, doc){
 //doc 查询结果文档
});	
```
同样是单条数据，findById和findOne还是有些区别的。


### 5.小结
---
简单回顾：

1. find过滤查询 ：find查询时我们可以过滤返回结果所显示的属性个数。

2. findOne查询 ：只返回符合条件的首条文档数据。

3. findById查询：根据文档_id来查询文档。

