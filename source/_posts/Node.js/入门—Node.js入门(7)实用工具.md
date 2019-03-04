title: 入门—七.实用工具
date: 2014-01-26 20:45:23
tags: [Node.js]
---

### 1.简介
---
#### UTIL模块的基本介绍
util模块呢，是一个Node.js核心模块，提供常用函数的集合，用于弥补核心JavaScript的一些功能过于精简的不足。并且还提供了一系列常用工具，用来对数据的输出和验证。

### 2.转换字符串
#### inspect函数的基本用法
util.inspect(object,[showHidden],[depth],[colors])是一个将任意对象转换为字符串的函数，通常用于调试和错误输出。它至少接受一个参数object，即要转换的对象,我们来学习它的简单用法。使用语法如下：
```javascript
var util = require('util');
var result = util.inspect(Object);
console.log(result);
```
运行结果：
```javascript
[Function: Object]
```

### 3.字符串格式化
---
#### format函数的基本用法
format函数根据第一个参数，返回一个格式化字符串，第一个参数是一个可包含零个或多个占位符的字符串。每一个占位符被替换为与其对应的转换后的值，支持的占位符有："%s(字符串)"、"%d(数字<整型和浮点型>)"、"%j(JSON)"、"%(单独一个百分号则不作为一个参数)"。

1：如果占位符没有相对应的参数，占位符将不会被替换.如示例：

```javascript
var util = require('util');
var result = util.format('%s:%s', 'foo');
console.log(result);
```
运行结果：

```javascript
'foo:%s'
```

2：如果有多个参数占位符，额外的参数将会调用util.inspect()转换为字符串。这些字符串被连接在一起，并且以空格分隔。如示例：
```javascript
var util = require('util');
var result = util.format('%s:%s', 'foo', 'bar', 'baz');
console.log(result);
```
运行结果：

```javascript
'foo:bar baz'
```

3：如果第一个参数是一个非格式化字符串，则会把所有的参数转成字符串并以空格隔开拼接在一块，而且返回该字符串。如示例：
```javascript
var util = require('util');
var result = util.format(1, 2, 3);
console.log(result);
```
运行结果：

```javascript
'1 2 3'
```

### 4.数组验证
---
#### isArray函数的基本用法
isArray函数可以判断对象是否为数组类型，是则返回ture,否则为false。语法如下：

```javascript
var util = require('util');
var result = util.isArray([]);
console.log(result);
```
运行结果：

```javascript
true
```

### 5.日期验证
---
#### isDate函数的基本用法
isDate函数可以判断对象是否为日期类型，是则返回ture,否则返回false。语法如下：

例1：querystring.parse("字符串"，"分隔符"，"分配符")
```javascript
var util = require('util');
var result = util.isDate(new Date());
console.log(result);
```
运行结果：
```javascript
true
```
### 6.正则验证
---
#### isRegExp函数的基本用法

isRegExp函数可以判断对象是否为正则类型，是则返回ture,否则返回false。语法如下：
```javascript
var util = require('util');
var result = util.isRegExp(Object);
console.log(result);
```
请自行验证。

