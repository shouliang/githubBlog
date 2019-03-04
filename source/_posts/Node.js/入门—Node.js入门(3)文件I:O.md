title: 入门—三.文件I/O
date: 2014-01-14 17:45:23
tags: [Node.js]
---

### 1.简介
---
#### fs模块的基本用法
开发中我们经常会有文件I/O的需求，node.js中提供一个名为fs的模块来支持I/O操作，fs模块的文件I/O是对标准POSIX函数的简单封装。

### 2.写入文件
---
#### writeFile函数的基本用法
文件I/O，写入是必修课之一。fs模块提供writeFile函数，可以异步的将数据写入一个文件, 如果文件已经存在则会被替换。用法如下：

例：fs.writeFile(filename, data, callback)
```javascript
var fs= require("fs");
fs.writeFile('test.txt', 'Hello Node', function (err) {
   if (err) throw err;
   console.log('Saved successfully'); //文件被保存
});
```
数据参数可以是string或者是Buffer,编码格式参数可选，默认为"utf8"，回调函数只有一个参数err。

### 3.追加文件
---
#### appendFile函数的基本用法
writeFile函数虽然可以写入文件，但是如果文件已经存在，我们只是想添加一部分内容，它就不能满足我们的需求了，很幸运，fs模块中还有appendFile函数，它可以将新的内容追加到已有的文件中，如果文件不存在，则会创建一个新的文件。使用方法如下：

例：fs.appendFile(文件名,数据,编码,回调函数(err));
```javascript
var fs= require("fs"); 
fs.appendFile('test.txt', 'data to append', function (err) {
   if (err) throw err; 
    //数据被添加到文件的尾部
    console.log('The "data to append" was appended to file!'); 
});
```
编码格式默认为"utf8"

### 4.是否存在
---
#### exists函数的基本用法
如何检查一个文件是否存在呢？我想exists函数可以帮助你，用法如下：

例：fs.exists(文件，回调函数(exists));

exists的回调函数只有一个参数，类型为布尔型，通过它来表示文件是否存在。
```javascript
var fs= require("fs");
fs.exists('/etc/passwd', function (exists) {
  console.log(exists ? "存在" : "不存在!");
});
```

### 5.修改名称
---
#### rename函数的基本用法
修改文件名称是我们经常会遇见的事情，rename函数提供修改名称服务：
```javascript
var fs= require("fs");
fs.rename(旧文件，新文件，回调函数(err){
   if (err) throw err;
   console.log('Successful modification,');
});
```

### 6.移动文件
---
移动文件也是我们经常会遇见的，可是fs没有专门移动文件的函数，但是我们可以通过rename函数来达到移动文件的目的，示例如下。
```javascript
var fs = require('fs');
fs.rename(oldPath,newPath,function (err) {
   if (err) throw err;
   console.log('renamed complete');
});
```

### 7.读取文件
---
#### readFile函数的基本用法
读取文件是最常用到的功能之一，使用fs模块读取文件语法如下：

例：fs.readFile(文件,编码,回调函数);

```javascript
var fs = require('fs');
fs.readFile(文件名, function (err, data) {
  if (err) throw err;
  console.log(data);
});
```
回调函数里面的data,就是读取的文件内容。

### 8.删除文件
---
#### unlink函数的基本用法
面对一堆垃圾的文件总是有想删除的冲动，我有强迫症？你才有呢。
好在有unlink函数，终于得救了，示例如下：
例：fs.unlink(文件,回调函数(err));
```javascript
var fs = require('fs');
fs.unlink(文件, function(err) {
  if (err) throw err;
  console.log('successfully deleted');
});
```

### 9.创建目录
---
#### mkdir函数的基本用法
除了针对文件的操作，目录的创建、删除也经常遇到的，下面我们来看看node.js中如何创建目录：
```javascript
fs.mkdir(路径，权限，回调函数(err));
```
参数

1.路径：新创建的目录。
2.权限：可选参数，只在linux下有效，表示目录的权限，默认为0777，表示文件所有者、文件所有者所在的组的用户、所有用户，都有权限进行读、写、执行的操作。
3.回调函数：当发生错误时，错误信息会传递给回调函数的err参数。

### 10.删除目录
---
#### rmdir函数的基本用法
删除目录也是必不可少的功能，rmdir函数可以删除指定的目录：

例：fs.rmdir(路径，回调函数(err));
```javascript
var fs = require('fs'); 
fs.rmdir(path, function(err) {
  if (err) throw err;
  console.log('ok');
});
```

### 11.读取目录
---
#### readdir函数的基本用法
如果要读取目录下所有的文件应该怎么办呢？readdir函数可以读取到指定目录下所有的文件，示例如下：
```javascript
var fs = require('fs');
fs.readdir(目录,回调函数(err,files));
```
回调函数 (callback) 接受两个参数 (err, files) 其中 files 是一个存储目录中所包含的文件名称的数组，数组中不包括 '.' 和 '..'。

### 12.小结
---
文件I/O是最基本的操作，应该熟悉掌握。

fs模块不但提供异步的文件操作，还提供相应的同步操作方法，需要指出的是，nodejs采用异步I/O正是为了避免I/O时的等待时间，提高CPU的利用率，所以在选择使用异步或同步方法的时候需要权衡取舍。

