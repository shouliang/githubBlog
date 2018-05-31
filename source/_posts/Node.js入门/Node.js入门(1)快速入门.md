title: 一.快速入门
date: 2014-01-08 14:45:23
tags: [Node.js入门]
---

### 1.hello world
---
node使用javascript作为开发语言。没错，就是通常我们在前端页面里使用的javascript！如：```javascript
console.log('hello world');
```

### 2.回调函数
---
由于node是一个异步事件驱动的平台，所以在代码中我们经常需要使用回调函数。下面是回调函数应用的经典示例：	
```javascript
setTimeout(function(){
    console.log('callback is called');
},2000);
```

### 3.标准回调函数
---
node.js中回调函数格式是约定俗成的，它有两个参数，第一个参数为err，第二个参数为data，顾名思义，err是错误信息，data则是返回的数据，示例如下:
```javascript
function(err,data){
 
}
```

### 4.获取模块
---
为了支持快速开发，node平台上提供了大量的模块，封装了各自不同的功能，那么我们将如何调获取想要的模块呢， 在node中，我们可以使用require函数，具体语法如下：
```javascript
require("模块");
```
通过require函数我们就可以获取相应模块进而使用它的任意功能了。

### 5.使用模块
---
os模块可提供操作系统的一些基本信息，它的一些常用方法如下：
```javascript
var os = require("os");
var result = os.platform(); //查看操作系统平台
           //os.release(); 查看操作系统版本
           //os.type();  查看操作系统名称
           //os.arch();  查看操作系统CPU架构
console.log(result);
```
