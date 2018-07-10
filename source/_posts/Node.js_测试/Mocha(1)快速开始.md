title: 一.快速开始
date: 2014-06-05 14:45:23
tags: [Node.js_测试]
---

### 1.什么是mocha
---
mocha 是一个功能丰富的javascript测试框架，可以运行在nodejs和浏览器环境，使异步测试变得简单有趣。mocha 串联运行测试，允许灵活和精确地报告结果，同时映射未捕获的异常用来纠正测试用例。

支持TDD/BDD 的 开发方式，结合 should.js/expect/chai/better-assert 断言库，能轻松构建各种风格的测试用例。

#### 特点
* 简单
* 灵活
* 有趣
#### 安装
通过npm全局安装： npm install -g mocha


### 2.第一个测试用例
---
我们首先来见识一下mocha最基本的测试用例是怎么的结构,如下。

测试用例：

//模块依赖
```
var assert = require("assert");
```

//断言条件
```
describe('Array', function(){
  describe('#indexOf()', function(){
    it('当值不存在时应该返回 -1', function(){
      assert.equal(-1, [1,2,3].indexOf(5));
      assert.equal(-1, [1,2,3].indexOf(0));
    });
  });
});
```
示例解析：测试用例首先需要引用断言模块，如上文中var assert = require('assert');，代码 assert.equal(-1, [1,2,3].indexOf(5)); 中使用的是assert.equal(actual, expected, [message]) 语法。作用等同于使用'=='进行相等判断。actual为实际值，expected 为期望值。message为返回的信息。

运行 Mocha：$ mocha


### 3.assert断言
---
断言（assert）指的是对代码行为的预期。一个测试用例内部，包含一个或多个断言（assert）。

断言会返回一个布尔值，表示代码行为是否符合预期。测试用例之中，只要有一个断言为false，这个测试用例就会失败，只有所有断言都为true，测试用例才会通过。

比如上节示例中的：

assert.equal(-1, [1,2,3].indexOf(5));

assert.equal(-1, [1,2,3].indexOf(0));

实际值（-1）和期望值（[1,2,3].indexOf(5)）是一样的，断言为true，所以这个测试用例成功了。

mocha 允许开发者使用任意的断言库，当这些断言库抛出了一个错误异常时，mocha将会捕获并进行相应处理。这意味着你可以利用如 should.js断言库、 Node.js 常规的 assert 模块或其它类似的断言代码库。以下是众所周知的适用于Node.js或浏览器的断言库：

* should.js
* expect.js
* chai.js
* better-assert
* assert：nodejs原生模块，在前文示例中我们有应用到。

### 4.chai.js断言库
---
Chai 是一个非常灵活的断言库，它可以让你使用如下三种主要断言方式的任何一种：

#### assert：

这是来自老派测试驱动开发的经典的assert方式。比如：

assert.equal(variable, "value");

#### expect：

这种链式的断言方式在行为驱动开发中最为常见。比如：

expect(variable).to.equal("value");

#### should：

这也是在测试驱动开发中比较常用的方式之一。举例：

variable.should.equal("value");


### 5.expect语法
---
expect 库应用是非常广泛的，它拥有很好的链式结构和仿自然语言的方法。通常写同一个断言会有几个方法，比如expect(response).to.be(true) 和 expect(response).equal(true)。以下列举了 expect 常用的主要方法：

* ok ：检查是否为真
* true：检查对象是否为真
* to.be、to：作为连接两个方法的链式方法
* not：链接一个否定的断言，如 expect(false).not.to.be(true)
* a/an：检查类型（也适用于数组类型）
* include/contain：检查数组或字符串是否包含某个元素
* below/above：检查是否大于或者小于某个限定值
mocha支持TDD/BDD 的 开发方式，结合 should.js、expect、chai、better-assert 断言库，能轻松构建各种风格的测试用例。这里面有两个知识点，一个是断言库，另一个是 TDD/BDD 。
