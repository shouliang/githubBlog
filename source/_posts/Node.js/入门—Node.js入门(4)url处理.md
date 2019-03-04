title: 入门—四.url处理
date: 2014-01-18 20:45:23
tags: [Node.js]
---

### 1.简介
---
#### url模块的基本用法
node.js为互联网而生，和url打交道是无法避免的了，url模块提供一些基础的url处理。

### 2.parse
---
#### parse函数的基础用法
parse函数的作用是解析url，返回一个json格式的数组，请看如下示例：

```javascript
var url = require('url');
url.parse('http://www.baidu.com');
```
运行结果：
```javascript
{ protocol: 'http:',
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search: null,
  query: null,
  pathname: 'www.baidu.com',
  path: 'www.baidu.com',
  href: 'http://www.baidu.com' 
}
```
### 3.条件解析
---
#### parse函数 —— 条件解析
parse函数的第二个参数是布尔类型，当参数为true时，会将查询条件也解析成json格式的对象。

```javascript
var url = require('url');
url.parse('http://www.baidu.com?page=1',true);
```
运行结果：

```javascript
{ protocol: 'http:',
  slashes: true,
  auth: null,
  host: 'www.baidu.com',
  port: null,
  hostname: 'www.baidu.com',
  hash: null,
  search: '?page=1',
  query: { page: '1' },
  pathname: '/',
  path: '/?page=1',
  href: 'http://www.baidu.com/?page=1' 
}
```
注意query值的不同

### 4.解析主机
---
#### parse函数 —— 解析主机
parse函数的第三个参数也是布尔类型的，当参数为true，解析时会将url的"//"和第一个"/"之间的部分解析为主机名，示例如下：

```javascript
var url = require('url');
url.parse('http://www.baidu.com/news',false,true);
```
运行结果：

```javascript
{ protocol: 'http:',
  slashes: true,
  auth: null,
  host: 'www.baidu.com',
  port: null,
  hostname: 'www.baidu.com',
  hash: null,
  search: null,
  query: null,
  pathname: '/news',
  path: '/news',
  href: 'http://www.baidu.com/news'
}
```
### 5.格式化
---
#### format函数的基础用法
format函数的作用与parse相反，它的参数是一个JSON对象，返回一个组装好的url地址，请看如下示例：
```javascript
var url = require('url');
url.format({
	protocol: 'http:',
	hostname:'www.baidu.com',
	port:'80',
	pathname :'/news',
	query:{page:1}
});
```
运行结果：
```javascript
http://www.baidu.com/news?page=1
```

### 6.reslove
---
#### resolve函数的基础用法
resolve函数的参数是两个路径，第一个路径是开始的路径或者说当前路径，第二个则是想要去往的路径，返回值是一个组装好的url，示例如下：
```javascript
var url = require('url');
url.resolve('http://example.com/', '/one')  // 'http://example.com/one'
url.resolve('http://example.com/one', '/two') // 'http://example.com/two'
```

