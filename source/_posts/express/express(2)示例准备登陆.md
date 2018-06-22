title: 二.准备登陆
date: 2014-05-07 15:45:23
tags: [express]
---

### 1.安装模板
---
我们要使用express框架实现一个简单的用户登陆功能，先准备一下相关资源。

在nodejs中使用express框架，它默认的是ejs和jade渲染模板，以ejs模板为例，讲述模板渲染网页模板的基础功能。

ejs模板安装方法
```
npm install ejs
```
1.目录下安装好了之后，如何调用呢，如下所示：
```
// 指定渲染模板文件的后缀名为ejs
app.set('view engine', 'ejs');
```
2.默认ejs模板只支持渲染以ejs为扩展名的文件，可能在使用的时候会觉得它的代码书写方式很不爽还是想用html的形式去书写，该怎么办呢，这时就得去修改模板引擎了，也就会用到express的engine函数。

3.engine注册模板引擎的函数，处理指定的后缀名文件。
```
// 修改模板文件的后缀名为html
app.set( 'view engine', 'html' );
// 运行ejs模块
app.engine( '.html', require( 'ejs' ).__express );
```
"__express"，ejs模块的一个公共属性，表示要渲染的文件扩展名。

### 2.静态资源
---
如果要在网页中加载静态文件（css、js、img），就需要另外指定一个存放静态文件的目录，当浏览器发出非HTML文件请求时，服务器端就会到这个目录下去寻找相关文件。

项目目录下添加一个存放静态文件的目录为public。

在public目录下在添加三个存放js、css、img的目录，相应取名为javascripts、stylesheets、images。

然后就可以把相关文件放到相应的目录下了。

比如，浏览器发出如下的样式表请求：
```javascript
<link href="/stylesheets/bootstrap.min.css" rel="stylesheet" media="screen">
```
服务器端就到public/stylesheets/目录中寻找bootstrap.min.css文件。

有了静态目录文件，我们还得在启动文件里告诉它这个静态文件路径，需要指定一下，如下所示：
```
app.use(express.static(require('path').join(__dirname, 'public')));
```
PS：express.static —— 指定静态文件的查找目录。

使用use函数调用中间件指定express静态访问目录，'public'就是我们我们新建的用来存放静态文件的总目录。

### 3.添加视图
---
下面我们就来添加网页模板了，项目中我们会新建一个目录用来单独存放模板文件，这里我们就统一放到根路径上了。

下面开始新建index.html、login.html、home.html三个页面。

1.index.html页面参考内容如下：
```
<div style="height:400px;width:550px;margin:50px auto;margin-left:auto;border:solid 1px;background: rgb(246, 246, 253);">
    <div style="margin-left: 35px;">
        # 首页
        <form action="#"  role="form" style="margin-top: 90px;margin-left: 60px;"> 
            # 欢迎进入首页！
            <div style="margin-top: 145px;">
                <input type="button" value="登 陆" />
            </div>
        </form>
    </div>
</div>
```
2.login.html页面参考内容如下：
```
...
    <title>用户登录</title>
    <meta charset="utf-8">
    <script src="http://code.jquery.com/jquery-2.1.1.min.js"></script>
... 
<div style="height:300px;width:350px;margin:100px auto;margin-left:auto;border:solid 1px;background: rgb(246, 246, 253);">
    <div style="width:200px;margin:auto;margin-top:50px;"> 
        # 用户登录
        <form action="#"  role="form" method="post" >
            <input id="username" type="text" name="username" style="margin: 20px 0px;" />
            <input id="password" type="password" name="password" />
            <div style="margin-top:30px;margin-left:125px;">
                <input type="button" value="登 陆" />
            </div>
        </form>
    </div>
</div>
```

3.home.html页面参考内容如下：
```
<div style="height:400px;width:550px;margin:50px auto;margin-left:auto;border:solid 1px;background: rgb(246, 246, 253);">
    <div style="margin-left: 45px;">
        # 主页
        <form action="#"  role="form" style="margin-top: 90px;">
            # 登陆成功！
            <div style="margin-top: 145px;">
                <input  type="button" value="退 出" />
            </div>
        </form>
    </div>
</div>
```
和静态文件一样，我们也要设置views存放的目录，如下：
```
// 设定views变量，意为视图存放的目录
app.set('views', __dirname);
```
有了网页模板和指定目录，下面就可以访问它们了。

### 4.渲染视图
---
我们要如何对网页模板进行访问呢，这就要用到res对象的render函数了。

render函数，对网页模板进行渲染。

格式：res.render(view, [locals], callback);

参数view就是模板的文件名callback用来处理返回的渲染后的字符串，options、callback可省略，在渲染模板时locals可为其模板传入变量值，在模板中就可以调用所传变量了。

比如渲染我们刚刚添加的index.html页面，我们就可以在app.js中写入如下内容：
```
ar express = require('express');
var app = express();
var path = require('path');
app.set('views', __dirname);
app.set( 'view engine', 'html' );
app.engine( '.html', require( 'ejs' ).__express );
app.get('/', function(req, res) {
    res.render('index');
});
app.listen(80);
```

### 5.url重定向
---
#### redirect基本用法
redirect方法允许网址的重定向，跳转到指定的url并且可以指定status，默认为302方式。

格式：res.redirect([status], url);

例1：使用一个完整的url跳转到一个完全不同的域名。
```
res.redirect("http://www.hubwiz.com");
```
例2：跳转指定页面，比如登陆页，如下：
```
res.redirect("login");
```
后面我们开始实现登陆功能，先试一下redirect重定向

