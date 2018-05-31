title: 二.增删改查
date: 2014-03-12 10:45:23
tags: [Mongoose]
---

### 1.查询
---
查询分很多种类型，如条件查询，过滤查询等等，先看最基本的find查询。
1.find查询： obj.find(查询条件,callback);
```
Model.find({},function(error,docs){
   //若没有向find传递参数，默认的是显示所有文档
});
Model.find({ "age": 28 }, function (error, docs) {
  if(error){
    console.log("error :" + error);
  }else{
    console.log(docs); //docs: age为28的所有文档
  }
}); 
```

### 2.Model保存
---
Model提供了一个create方法来对数据进行保存。下面我们来看一下示例：

1. Model.create(文档数据, callback))
```
Model.create({ name:"model_create", age:26}, function(error,doc){
    if(error) {
        console.log(error);
    } else {
        console.log(doc);
    }
});
```
### 3.Entity保存
---
entity的保存方法。如下示例：

1. Entity.save(文档数据, callback))
```
var Entity = new Model({name:"entity_save",age: 27});
Entity.save(function(error,doc) {
    if(error) {
        console.log(error);
    } else {
        console.log(doc);
    }
});
```
model调用的是create方法，entity调用的是save方法。

### 4.更新数据
---
数据更新！

1.示例：obj.update(查询条件,更新对象,callback);
```
var conditions = {name : 'test_update'};
var update = {$set : { age : 16 }};
TestModel.update(conditions, update, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('Update success!');
    }
});
```
更新后find一下，此时数据已经修改成功了！

### 5.删除数据
---
有了数据的保存、更新，就差删除了。

1.示例：obj.remove(查询条件,callback);
```
var conditions = { name: 'tom' };
TestModel.remove(conditions, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('Delete success!');
    }
});
```
和update类似吧，有了remove方法我们就可以进行删除操作了。

### 6.小结
---
简单回顾：

1. 查询：find查询返回符合条件一个、多个或者空数组文档结果。

2. 保存：model调用create方法，entity调用的save方法。

3. 更新：obj.update(查询条件,更新对象,callback)，根据条件更新相关数据。

4. 删除：obj.remove(查询条件,callback)，根据条件删除相关数据。

