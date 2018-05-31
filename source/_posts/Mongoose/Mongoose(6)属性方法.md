title: 六.属性方法
date: 2014-03-27 20:45:23
tags: [Mongoose]
---

### 1.ObjectId类型
---
#### ObjectId简述
存储在mongodb集合中的每个文档（document）都有一个默认的主键_id，这个主键名称是固定的，它可以是mongodb支持的任何数据类型，默认是ObjectId。该类型的值由系统自己生成，从某种意义上几乎不会重复，生成过程比较复杂，有兴趣的朋友可以查看源码。

使用过MySQL等关系型数据库的友友们都知道，主键都是设置成自增的。但在分布式环境下，这种方法就不可行了，会产生冲突。为此，MongoDB采用了一个称之为ObjectId的类型来做主键。ObjectId是一个12字节的 BSON 类型字符串。按照字节顺序，一次代表：

4字节：UNIX时间戳
3字节：表示运行MongoDB的机器
2字节：表示生成此_id的进程
3字节：由一个随机数开始的计数器生成的值
```
var mongoose = require('mongoose');
var tSchema = new mongoose.Schema({}); //默认_id:ObjectId类型
```
每一个文档都有一个特殊的键“_id”，这个键在文档所属的集合中是唯一的。

### 2.添加属性
---
#### Schema添加属性值
前面我们已经讲述了如何定义一个Schema并赋予某些属性值,那能不能先定义后添加属性呢，答案是可以的，如下所示：
```
var mongoose = require('mongoose');
var TempSchema = new mongoose.Schema;
TempSchema.add({ name: 'String', email: 'String', age: 'Number' });
```
### 3.实例方法
---
#### 实例方法
有的时候，我们创造的Schema不仅要为后面的Model和Entity提供公共的属性，还要提供公共的方法.那怎么在Schema下创建一个实例方法呢，请看示例：
```
var mongoose = require('mongoose');
var saySchema = new mongoose.Schema({name : String});
saySchema.method('say', function () {
  console.log('Trouble Is A Friend');
})
var say = mongoose.model('say', saySchema);
var lenka = new say();
lenka.say(); //Trouble Is A Friend
```

### 4.静态方法
---
#### Schema静态方法
接下来将讲述怎么为Schema创建静态方法。如下示例：
```
var mongoose = require("mongoose");
var db = mongoose.connect("mongodb://127.0.0.1:27017/test"); 
var TestSchema = new mongoose.Schema({
    name : { type:String },
    age  : { type:Number, default:0 },
    email: { type:String, default:"" },
    time : { type:Date, default:Date.now }
});
TestSchema.static('findByName', function (name, callback) {
    return this.find({ name: name }, callback);
});
var TestModel = db.model("test1", TestSchema );
TestModel.findByName('tom', function (err, docs) {
 //docs所有名字叫tom的文档结果集
});
```
### 5.追加方法
---
#### Schema追加方法
有时场景的需要，我们甚至可以为Schema模型追加方法。

为Schema模型追加speak方法，如下示例：
```
var mongoose = require("mongoose");
var db = mongoose.connect("mongodb://127.0.0.1:27017/test");
var TestSchema = new mongoose.Schema({
    name : { type:String },
    age  : { type:Number, default:0 },
    email: { type:String, default:"" },
    time : { type:Date, default:Date.now }
});
TestSchema.methods.speak = function(){
  console.log('我的名字叫'+this.name);
}
var TestModel = db.model('test1',TestSchema);
var TestEntity = new TestModel({name:'Lenka'});
TestEntity.speak();//我的名字叫Lenka
```