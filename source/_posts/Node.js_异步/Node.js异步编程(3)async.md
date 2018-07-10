title: 三.async
date: 2014-02-13 14:45:23
tags: [Node.js_异步编程]
---

### async
---
async是一个流程控制库，它就像黑夜中的明灯照亮那陷入callback嵌套泥潭的人们。 这么说虽然有些夸张，但是async确实为我们带来了丰富的嵌套解决方案。

项目地址：

https://github.com/caolan/async

npm 安装
```
 npm install async
```
使用方法：
```
 var async = require('async');
```

### 2.serires(tasks, callback)
---
首先登场的是series函数，它的作用是串行执行，一个函数数组中的每个函数，每一个函数执行完成之后才能执行下一个函数，示例如下：
```
async.series({
    one: function(callback){
        callback(null, 1);
    },
    two: function(callback){
        callback(null, 2);
    }
},function(err, results) { 
});
```
series函数的第一个参数可以是一个数组也可以是一个JSON对象，参数类型不同，影响的是返回数据的格式，如示例中的参数为数组，返回的results应该是这样的'[1,2]'。

### 3.waterfall(tasks,[callback])
---
waterfall和series函数有很多相似之处，都是按顺序依次执行一组函数，不同之处是waterfall每个函数产生的值，都将传给下一个函数，而series则没有这个功能，示例如下：

```
async.waterfall([  
    function(callback){ 
      //task1 
      callback(null,1);       
    },function(data,callback){
      //task2 
      callback(null,2); 
    } 
],function(err,results){  
    console.log(results); 
});
```
另外需要注意的是waterfall的tasks参数只能是数组类型。

### 4.parallel(tasks,[callback])
---
parallel函数是并行执行多个函数，每个函数都是立即执行，不需要等待其它函数先执行。 传给最终callback的数组中的数据按照tasks中声明的顺序，而不是执行完成的顺序，示例如下：
```
async.parallel([
    function(callback){
        callback(null, 'one');
    },
    function(callback){
        callback(null, 'two');
    }
],
function(err, results){
});
```
tasks参数可以是一个数组或是json对象，和series函数一样，tasks参数类型不同，返回的results格式会不一样。

### 5.paralleLimit(tasks,limit,[callback])
---
parallelLimit函数和parallel类似，但是它多了一个参数limit。 limit参数限制任务只能同时并发一定数量，而不是无限制并发，示例如下：
```
async.parallelLimit([
    function(callback){
        callback(null, 'one');
    },
    function(callback){
        callback(null, 'two');
    }
],
2,
function(err, results){
});
```

### 6.whilst(test,fn,callback)
---
相当于while，但其中的异步调用将在完成后才会进行下一次循环。当你需要循环异步的操作的时候，它可以帮助你。示例如下：
```
var count = 0;
async.whilst(
    function () { return count < 5; },
    function (callback) {
        count++;
        setTimeout(callback, 1000);
    },
    function (err) {
    }
);
```
test参数是一个返回布尔值结果的函数，通过返回值来决定循环是否继续，作用等同于while循环停止的条件。

fn参数就是我们要异步执行的作业，每次fn执行完毕后才会进入下一次循环。


### 7.doWhilst
---
相当于do…while,较whilst而言，doWhilst交换了fn,test的参数位置，先执行一次循环，再做test判断。
```
var count = 0;
async.doWhilst(
    function (callback) {
        count++;
        setTimeout(callback, 1000);
    },
    function () { return count < 5; },
    function (err) { 
    }
);
```


### 8.until(test,fn,callback)
---
until与whilst正好相反，当test条件函数返回值为false时继续循环，与true时跳出。其它特性一致。示例如下：
```
var count = 5;
async.until(
    function () { return count < 0; },
    function (callback) {
        count--;
        setTimeout(callback, 1000);
    },
    function (err) {
    }
);
```

### 9.doUntil(fn,test,callback)
---
doUntil与doWhilst正好相反，当test为false时循环，与true时跳出。其它特性一致。

示例：
```
var count = 5;
async.doUntil(
    function (callback) {
        count--;
        setTimeout(callback, 1000);
    },
    function () { return count < 0; },
    function (err) { 
    }
);
```

### 10.forever(fn,errback)
---
forever函数比较特殊，它的功能是无论条件如何，函数都一直循环执行，只有出现程序执行的过程中出现错误时循环才会停止，callback才会被调用。

示例：
```
async.forever(
    function(next) {
    },
    function(err) {
    }
);
```

### 11.compose(fn1,fn2...)
---
使用compose可以创建一个异步函数的集合函数，将传入的多个异步函数包含在其中，当我们执行这个集合函数时，会依次执行每一个异步函数，每个函数会消费上一次函数的返回值。

我们可以使用compose把异步函数f、g、h，组合成f(g(h()))的形式，通过callback得到返回值，请看示例：
```
unction fn1(n, callback) {
    setTimeout(function () {
        callback(null, n + 1);
    }, 1000);
}
function fn2(n, callback) {
    setTimeout(function () {
        callback(null, n * 3);
    }, 1000);
}
var demo = async.compose(fn2, fn1);
demo(4, function (err, result) {
});
```

### 12.auto(tasks,[callback])
---
用来处理有依赖关系的多个任务的执行。示例如下：
```
async.auto({
    getData: function(callback){
        callback(null, 'data', 'converted to array');
    },
    makeFolder: function(callback){        
        callback(null, 'folder');
    },
    writeFile: ['getData', 'makeFolder', function(callback, results){        
        callback(null, 'filename');
    }],
    emailLink: ['writeFile', function(callback, results){
        callback(null, {'file':results.writeFile, 'email':'user@example.com'});
    }]
}, function(err, results) {
    console.log('err = ', err);
    console.log('results = ', results);
});
```
示例中writeFile依赖getData和makeFolder,emailLink依赖writeFile。

### 13.queue(worker,concurrency)
---
queue相当于一个加强版的parallel，主要是限制了worker数量，不再一次性全部执行。当worker数量不够用时，新加入的任务将会排队等候，直到有新的worker可用。

它有多个点可供回调，如无等候任务时(empty)、全部执行完时(drain)等。

示例：定义一个queue，其worker数量为2：

```
var q = async.queue(function(task, callback) {
    console.log('worker is processing task: ', task.name);
    callback();
}, 2);
q.push({name: 'foo'}, function (err) {
    console.log('finished processing foo');
});
q.push({name: 'bar'}, function (err) {
    console.log('finished processing bar');
});
```

当最后一个任务交给worker执行时，会调用empty函数:
```
q.empty = function() {
    console.log('no more tasks wating');
}
```

### 14.apply(function,arguments...)
---
apply是一个非常好用的函数，可以让我们给一个函数预绑定多个参数并生成一个可直接调用的新函数，简化代码。示例如下：

```
function(callback) { 
    test(3, callback); 
};
```

用apply改写:
```
async.apply(test, 3);
```

### 15.iterator(tasks)
---
将一组函数包装成为一个iterator，可通过next()得到以下一个函数为起点的新的iterator。该函数通常由async在内部使用，但如果需要时，也可在我们的代码中使用它。

```
var iter = async.iterator([
    function() { console.log('111') },
    function() { console.log('222') },
    function() { console.log('333') }
]);
iter();
```

直接调用()，会执行当前函数，并返回一个由下个函数为起点的新的iterator。调用next()，不会执行当前函数，直接返回由下个函数为起点的新iterator。

对于同一个iterator，多次调用next()，不会影响自己。如果只剩下一个元素，调用next()会返回null。

### 16.小结
---
async模块在流程控制方面给我们带来了比较全面的解决办法，下面我们来回顾一下都有哪几种方案：

串行控制： series、waterfall、compose;

并行控制：

parallel、parallelLimit、queue;

循环控制：
whilst、doWhilst、until、doUntil、forever;

其他控制：
apply、applyEach、iterator、auto;
