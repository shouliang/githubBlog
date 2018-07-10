title: 四.为项目开发一个BDD测试
date: 2014-06-13 19:45:23
tags: [Node.js_测试]
---

### 1.辅助模块
---
我们进行单元测试，一般都需要组合几个工具来来使用的。下面我们开始介绍：

chai断言库
chai 断言库支持BDD 的 expect/should 语法风格 和TDD的 assert 语法风格。

superagent
在用Node做Web开发的时候，模拟HTTP请求时必不可少的。这也就引出了superagent这个模块，它是一个模拟http请求的库。它作用是简化发起请求的库。

项目的package.json 代码如下：
```
{
  "name": "mocha-test",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app.js",
    "test": "mocha test"
  },
  "devDependencies": {
    "superagent": "1.4.0",
    "chai": "3.4.0"
  }
}
```

### 2.项目描述
---
首先我们创建一个app.js文件。内容如下：
```
var http = require('http'),
    PORT = 3000; 
function onRequest(request, response) {
    console.log("Request received.");
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
}
var server = http.createServer(onRequest);
server.listen(PORT);
```
描述：为了让项目尽可能的简单，我们没有用到任何的框架。只是创建了一个http服务器监听了3000端口。

### 3.项目重构
---
我们的app.js需要最外部暴露两个方法。所以要对我们右边的项目进行修改。

需要修改的代码：
```
var server = http.createServer(onRequest);
server.listen(PORT);
```
重构为：
```
var server = http.createServer(onRequest);
var boot = function () {
    server.listen(PORT, function () {
        console.info('Express server listening on port ' + PORT);
    });
};
var shutdown = function () {
    server.close();
};
if (require.main === module) {
    boot();
} else {
    console.info('Running app as a module');
    exports.boot = boot;
    exports.shutdown = shutdown;
}
```
重构描述：现在我们把启动服务和关闭服务分别进行了封装并且对外进行了暴露。


### 4.测试用例
---
现在，我们创建一个名字为 tests 的测试文件夹，并创建一个index.js的文件。测试用例前需要启动服务器，结束后关闭服务。这个时候就用到了前面暴露的 boot() 和 shutdown() 方法。

示例如下：
```
var boot = require('../app').boot,
    shutdown = require('../app').shutdown,
    request = require('superagent'),
    expect = require('chai').expect;
 
describe('server', function () {
    before(function () {
        boot();
    });
    describe('index', function () {
        it('should respond to GET', function (done) {
            request
                .get('http://localhost:3000')
                .end(function (err, res) {
                    expect(res.status).to.equal(200);
                    done();
                });
        });
    });
    after(function () {
        shutdown();
    });
});
```

运行命令：mocha tests