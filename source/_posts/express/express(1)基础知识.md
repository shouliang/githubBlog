title: 一.基础知识
date: 2014-05-04 14:45:23
tags: [express]
---

### 1.认识express
---
npm提供了大量的第三方模块，其中不乏许多Web框架，比如其中的一个轻量级的Web框架 ——— Express。

Express是一个简洁、灵活的node.js Web应用开发框架, 它提供一系列强大的功能，比如：模板解析、静态文件服务、中间件、路由控制等等,并且还可以使用插件或整合其他模块来帮助你创建各种 Web和移动设备应用,是目前最流行的基于Node.js的Web开发框架，并且支持Ejs、jade等多种模板，可以快速地搭建一个具有完整功能的网站。

好，下面开始吧！

1.NPM安装	
```javascript
npm install express
```
2.获取、引用
```javascript
var express = require('express');
var app = express();
```
通过变量“app”我们就可以调用express的各种方法了。

### 2.创建应用
---
认识了Express框架，我们开始创建我们的第一个express应用。

在我们的默认项目主文件app.js添加如下内容：
```javascript
var express = require('express');
var app = express();
app.get('/', function (request, response) {  
   response.send('Hello World!');
}); 
app.listen(80);
```

### 3.get请求
---
前面我们实现了一个简单的express应用，下面我们就开始具体讲述它的具体实现，首先我们先来学习Express的常用方法。

get方法 —— 根据请求路径来处理客户端发出的GET请求。

格式：app.get(path,function(request, response));

path为请求的路径，第二个参数为处理请求的回调函数，有两个参数分别是request和response，代表请求信息和响应信息。

如下示例：
```javascript
var express = require('express');
var app = express();
app.get('/', function(request, response) {
   response.send('Welcome to the homepage!');
});
app.get('/about', function(request, response) {
   response.send('Welcome to the about page!');
});
app.get("*", function(request, response) {
    response.send("404 error!");
});
app.listen(80); 
```
上面示例中，指定了about页面路径、根路径和所有路径的处理方法。并且在回调函数内部，使用HTTP回应的send方法，表示向浏览器发送一个字符串。

### 4.中间件
---
1.什么是中间件？

中间件(middleware)就是处理HTTP请求的函数，用来完成各种特定的任务，比如检查用户是否登录、分析数据、以及其他在需要最终将数据发送给用户之前完成的任务。 它最大的特点就是，一个中间件处理完，可以把相应数据再传递给下一个中间件。

2.一个不进行任何操作、只传递request对象的中间件，大概是这样：
```javascript
function Middleware(request, response, next) { 
   next();
}
```
上面代码的next为中间件的回调函数。如果它带有参数，则代表抛出一个错误，参数为错误文本。
```javascript
function Middleware(request, response, next) { 
   next('出错了！');
}
```
抛出错误以后，后面的中间件将不再执行，直到发现一个错误处理函数为止。如果没有调用next方法,后面注册的函数也是不会执行的。


### 5.all方法
---
#### all函数的基本用法
和get函数不同app.all()函数可以匹配所有的HTTP动词，也就是说它可以过滤所有路径的请求，如果使用all函数定义中间件，那么就相当于所有请求都必须先通过此该中间件。

格式：app.all(path,function(request, response));

如下所示，我们使用all函数在请求之前设置响应头属性。
```javascript
var express = require("express");
var app = express();
app.all("*", function(request, response, next) {
    response.writeHead(200, { "Content-Type": "text/html;charset=utf-8" });      //设置响应头属性值
    next();
});
app.get("/", function(request, response) {
    response.end("欢迎来到首页!");
});
app.get("/about", function(request, response) {
    response.end("欢迎来到about页面!");
});
app.get("*", function(request, response) {
    response.end("404 - 未找到!");
});
app.listen(80);
```
上面代码参数中的“*”表示对所有路径有效，这个方法在给特定前缀路径或者任意路径上处理时会特别有用，不管我们请求任何路径都会事先经过all函数。

### 6.use方法1
---
#### use基本用法1
use是express调用中间件的方法，它返回一个函数。

格式：app.use([path], function(request, response, next){});

可选参数path默认为"/"。

1.使用中间件
```javascript
app.use(express.static(path.join(__dirname, '/')));
```
如上，我们就使用use函数调用express中间件设定了静态文件目录的访问路径(这里假设为根路径)。

如何连续调用两个中间件呢，如下示例：
```javascript
var express = require('express');
var app = express();
app.use(function(request, response, next){
    console.log("method："+request.method+" ==== "+"url："+request.url);
    next();
});
app.use(function(request, response){
    response.writeHead(200, { "Content-Type": "text/html;charset=utf-8" });
    response.end('示例：连续调用两个中间件');
});
app.listen(80);
```
回调函数的next参数，表示接受其他中间件的调用，函数体中的next()，表示将请求数据传递给下一个中间件。

上面代码先调用第一个中间件，在控制台输出一行信息，然后通过next()，调用第二个中间件，输出HTTP回应。由于第二个中间件没有调用next方法，所以req对象就不再向后传递了。

### 7.use方法2
---
#### use基本用法2
use方法不仅可以调用中间件，还可以根据请求的网址，返回不同的网页内容，如下示例：
```javascript
var express = require("express");
var app = express();
app.use(function(request, response, next) {
   if(request.url == "/") {
      response.send("Welcome to the homepage!");
   }else {
      next();
   }
});
app.use(function(request, response, next) {
   if(request.url == "/about") {
     response.send("Welcome to the about page!");
   }else {
     next();
   }
});
app.use(function(request, response) {
  response.send("404 error!");
});
app.listen(80);
```
上面代码通过request.url属性，判断请求的网址，从而返回不同的内容。

### 8.回调函数
---
Express回调函数有两个参数，分别是request(简称req)和response(简称res)，request代表客户端发来的HTTP请求，response代表发向客户端的HTTP回应，这两个参数都是对象。示例如下:
```javascript
function(req, res) {
}
```
### 9.获取主机、路径名
---
如何使用req对象来处理客户端发来的HTTP请求。

1.req.host返回请求头里取的主机名(不包含端口号)。

2.req.path返回请求的URL的路径名。

如下示例：
```javascript
var express = require('express');
var app = express();
app.get("*", function(req, res) {
    console.log(req.path);
    res.send("req.host获取主机名，req.path获取请求路径名!");
});
app.listen(80);
```
### 10.Get请求 - query
---
query是一个可获取客户端get请求路径参数的对象属性，包含着被解析过的请求参数对象，默认为{}。
```javascript
var express = require('express');
var app = express();
app.get("*", function(req, res) {
    console.log(req.query.参数名);
    res.send("测试query属性!");
});
app.listen(80);
```
通过req.query获取get请求路径的对象参数值。

格式：req.query.参数名；请求路径如下示例：

例1： /search?n=Hanmeimei
```javascript
req.query.n  // "Hanmeimei"
```
例2： /shoes?order=desc&shoe[color]=blue&shoe[type]=converse
```javascript
req.query.order  // "desc"
req.query.shoe.color  // "blue"
req.query.shoe.type  // "converse"
```
### 11.Get请求 - param
---
和属性query一样，通过req.param我们也可以获取被解析过的请求参数对象的值。

格式：req.param("参数名")；请求路径如下示例：

例1： 获取请求根路径的参数值，如/?n=Lenka，方法如下：
```javascript
var express = require('express');
var app = express();
app.get("/", function(req, res) {
    console.log(req.param("n")); //Lenka
    res.send("使用req.param属性获取请求根路径的参数对象值!");
});
app.listen(80);
```
例2：我们也可以获取具有相应路由规则的请求对象，假设路由规则为 /user/:name/，请求路径/user/mike,如下：
```javascript
app.get("/user/:name/", function(req, res) {
    console.log(req.param("name")); //mike
    res.send("使用req.param属性获取具有路由规则的参数对象值!");
});
```
PS：所谓“路由”，就是指为不同的访问路径，指定不同的处理方法。

### 12.Get请求 - params
---
和param相似，但params是一个可以解析包含着有复杂命名路由规则的请求对象的属性。

格式：req.params.参数名；

例1. 如上课时请求根路径的例子，我们就可以这样获取，如下：
```javascript
var express = require('express');
var app = express();
app.get("/user/:name/", function(req, res) {
    console.log(req.params.name); //mike
    res.send("使用req.params属性获取具有路由规则的参数对象值!");
});
app.listen(80);
```
查看运行结果，和param属性功能是一样的，同样获取name参数值。

例2：当然我们也可以请求复杂的路由规则，如/user/:name/:id，假设请求地址为：/user/mike/123，如下：
```javascript
app.get("/user/:name/:id", function(req, res) {
    console.log(req.params.id); //"123"
    res.send("使用req.params属性复杂路由规则的参数对象值!");
});
```
对于请求地址具有路由规则的路径来说，属性params比param属性是不是又强大了那么一点点呢！

### 13.send
---
send()方法向浏览器发送一个响应信息，并可以智能处理不同类型的数据。格式如下：
```
res.send([body|status], [body]);
```
1.当参数为一个String时，Content-Type默认设置为"text/html"。
```
res.send('Hello World'); //Hello World
```
2.当参数为Array或Object时，Express会返回一个JSON。
```
res.send({ user: 'tobi' }); //{"user":"tobi"}
res.send([1,2,3]); //[1,2,3]
```
3.当参数为一个Number时，并且没有上面提到的任何一条在响应体里，Express会帮你设置一个响应体，比如：200会返回字符"OK"。
```
res.send(200); // OK
res.send(404); // Not Found
res.send(500); // Internal Server Error
```
send方法在输出响应时会自动进行一些设置，比如HEAD信息、HTTP缓存支持等等。
