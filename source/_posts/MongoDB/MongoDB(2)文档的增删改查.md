title: 二.文档的增删改查
date: 2014-04-06 15:45:23
tags: [MongoDB]
---

### 1.简单插入文档
---
在数据库中，数据插入是最基本的操作，在MongoDB使用db.collection.insert(document)语句来插入文档，如下图：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkinsert1.png)

document是文档数据，collection是存放文档数据的集合。

例如：所有用户的信息存放在users集合中，每个用户的信息为一个user文档，插入数据:
```
db.users.insert(user);
```
如果collection存在，document会添加到collection目录下， 如果collection不存在，数据库会先创建collection，然后再保存document。

### 2.批量插入文档
---
nsert语句不但可以插入单个文档，还可以一次性插入多个文档。

插入多个文档时，insert命令的参数为一个数组，数组元素为BSON格式的文档。

多个文档可以放在一个数组内，一次插入多条数据，例如：
```
db.users.insert([{name:"tommy"},{name:"xiaoming"}])
```
文档批量插入非常方便，但是使用批量插入时也有一些问题需要注意，因为BSON格式的限制，一次插入的数据量不能超过16M，在一个insert命令中插入多条数据时，MongoDB不保证完全成功或完全失败。

### 3.查询与投影
---
在MongoDB中，查询指向特定的文档集合，查询设定条件，指明MongoDB需要返回的文档；查询也可以包含一个投影，指定返回的字段。

如下图，在查询过程指定了一个查询条件和一个排序修饰。

![](http://7xq1il.com1.z0.glb.clouddn.com/mkquery.jpg)

在关系型数据库中，投影指的是对列的筛选，类似的，在MongoDB中，投影指的是对出现在结果集中的对象属性的筛选。

### 4.find()方法
---
MongoDB中查询检索数据时使用find命令，使用方法如下：

语法：
db.collection.find(criteria,projection);
参数：
criteria – 查询条件，文档类型，可选。

projection– 返回的字段，文档类型，可选,若需返回所有字段，则忽略此参数。

find命令两个可选参数，criteria为查询条件，projection为返回的字段，如果不传入条件数据库会返回该集合的所有文档。

如下图简单示例：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkfind.jpg)

### 5.修改文档 - update命令
---
update命令可以更新指定文档的特定字段值，也可以替换整个文档，如果更新操作会增加文档大小，MongoDB将重新分配空间并重新定位。

语法：
db.collection.update(query,update,{upsert:boolean,multi:boolean});
参数：
query：查询条件，文档，和find中的查询条件写法一致。

update：修改内容，文档。

upsert(可选)：如果值为true，那么当集合中没有匹配文档时，创建文档。默认false。

multi(可选)：如果值为true，那么将更新全部符合条件的文档，否则仅更新一个文档，默认false。

如下示例：将users集合中所有符合条件"age>18"文档的status字段更新为"A"。

![](http://7xq1il.com1.z0.glb.clouddn.com/mkupdate.jpg)

### 6.修改文档 - save命令
---
save命令可以更新或插入一个新文档，与update命令不同的是，save只能对一个文档进行操作。

语法：
```
db.collection.save();
```
参数：
document：新的文档；

### 7.删除文档 - remove命令
---
需要删除文档时使用remove命令，删除文档可以清理掉不需要的数据，释放存储空间，提升检索效率，但是错误的删除会是一场灾难，因此在执行数据删除操作时需要非常的谨慎！

语法：
```
    db.collection.remove(
        query,
        justOne
     )
```
参数：
query：BSON类型，删除文档的条件。

justOne：布尔类型，true：只删除一个文档，false：默认值，删除所有符合条件的文档。

下面是一个是将users集合中所有status="D"的文档删除操作，对比一下MongoDB和传统SQL数据库删除的操作，看看有哪些不同之处。

与传统SQL比较 - 删除文档

MongoDB

![](http://7xq1il.com1.z0.glb.clouddn.com/mkremove.jpg)

传统SQL

![](http://7xq1il.com1.z0.glb.clouddn.com/mkremove-sql.jpg)
