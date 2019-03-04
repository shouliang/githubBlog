title: 入门—六.字符串转换
date: 2014-01-23 20:45:23
tags: [Node.js]
---

### 1.简介
---
#### Query String模块的基本用法
Query String模块用于实现URL参数字符串与参数对象之间的互相转换，提供了"stringify"、"parse"等一些实用函数来针对字符串进行处理，通过序列化和反序列化，来更好的应对实际开发中的条件需求，对于逻辑的处理也提供了很好的帮助。

### 2.序列化
---
#### stringify函数的基础用法
stringify函数的作用就是序列化对象，也就是说将对象类型转换成一个字符串类型（默认的分割符（"&"）和分配符（"="））。

例1：querystring.stringify("对象")

```javascript
var querystring= require('querystring');
var result = querystring.stringify({foo:'bar',cool:['xux', 'yys']});
console.log(result);
```
运行结果：
```javascript
foo=bar&cool=xux&cool=yys
```
### 3.序列化<多参数>
---
#### stringify函数的多参数用法
stringify函数的多参数用法，上节我们知道了对象被序列化为字符串之后默认是通过分割符（"&"）和分配符（"="）组成的，那可不可以改变呢，这节我们就来了解一下，是否可以自己去定义组合结果，看下面的小例子

例1：querystring.stringify("对象"，"分隔符"，"分配符")

```javascript
var querystring = require('querystring');
var result = querystring.stringify({foo:'bar',cool:['xux', 'yys']},'*','$');
console.log(result);
```
运行结果：

```javascript
'foo$bar*cool$xux*cool$yys'
```

### 4.反序列化
---
#### parse函数的基本用法
接下来就来学习反序列化函数——parse函数，parse函数的作用就是反序列化字符串（默认是由"="、"&"拼接而成），转换得到一个对象类型。如下示例：

例1：querystring.parse("字符串")

```javascript
var querystring = require('querystring');
var result = querystring.parse('foo=bar&cool=xux&cool=yys');
console.log(result);
```
运行结果：

```javascript
{ foo: 'bar', cool: ['xux', 'yys']}
```

### 5.反序列化<多参数>
---
#### parse函数的多参数用法
和上节stringify函数的多参数用法不同的是，parse函数可以根据用户所自定义的分割符、分配符来反序列化字符串，从而得到相应的对象结果.如下示例：

例1：querystring.parse("字符串"，"分隔符"，"分配符")
```javascript
var querystring = require('querystring');
var result = querystring.parse('foo@bar$cool@xux$cool@yys','@','$');
console.log(result);
```
运行结果：
```javascript
{ foo: '', bar: 'cool', xux: 'cool', yys: '' }
```

