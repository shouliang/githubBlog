title: Node.js与Redis
date: 2015-05-09 14:45:23
tags: [Redis]
---
### 1.简介
---
Redis官方推荐的Node.js的Redis客户端可以选择的有node_redis[7] 和ioredis[8] ，相比而言前者发布时间较早，而后者的功能则更加丰富一些。从接口来看两者的使用方法大同小异。

### 2.安装
---
使用npm install ioredis命令安装最新版本的ioredis。

### 3.使用方法
---
首先加载ioredis模块：
```
var Redis = require('ioredis');
```
下面的代码将创建一个默认连接到地址127.0.0.1，端口6379的Redis连接：
```
var redis = new Redis();
```
也可以显式地指定需要连接的地址：
```
var redis = new Redis(6379, '127.0.0.1');
```
由于Node.js的异步特性，在处理返回值的时候与其他客户端差别较大。还是以GET/SET命令为例：
```
redis.set('foo', 'bar', function () {
    //此时 SET 命令执行完并返回结果，
    //因为这里并不关心 SET命令的结果，所以我们省略了回调函数的形参。
    redis.get('foo', function (error, fooValue) {
        //error 参数存储了命令执行时返回的错误信息，如果没有错误则返回 null。
        //回调函数的第二个参数存储的是命令执行的结果
        console.log(fooValue); // 'bar'
    });
});
```
  使用ioredis执行命令时需要传入回调函数（callback function）来获得返回值，当命令执行完返回结果后ioredis会调用该函数，并将命令的错误信息作为第一个参数、返回值作为第二个参数传递给该函数。同时ioredis还支持Promise形式的异步处理方式，如果省略最后一个回调函数，命令语句会返回一个Promise值，如：
```
redis.get('foo').then(function (fooValue) {
    //fooValue 即为键值
});
```

  Node.js的异步模型使得通过ioredis调用Redis命令的表现与Redis的底层管道协议十分相似：调用命令函数时（如redis.set()）并不会等待Redis返回命令执行结果，而是直接继续执行下一条语句，所以在Node.js中通过异步模型就能实现与管道类似的效果。上面的例子中我们并不需要SET命令的返回值，只要保证SET命令在GET命令前发出即可，所以完全不用等待SET命令返回结果后再执行GET命令。因此上面的代码可以改写成：
```
//不需要返回值时可以省略回调函数
redis.set('foo', 'bar');
redis.get('foo', function (error, fooValue) {
    console.log(fooValue); // 'bar'
});
```
  不过由于SET和GET并未真正使用Redis的管道协议发送，所以当有多个客户端同时向 Redis 发送命令时，上例中的两个命令之间可能会被插入其他命令，换句话说，GET命令得到的值未必是“bar”。
  虽然Node.js的异步特性给我们带来了相对更高的性能，然而另一方面使用Redis实现某个功能时我们经常需要读写若干个键，而且很多情况下都会依赖之前命令的返回结果。这时就会出现嵌套多重回调函数的情况，影响代码可读性。就像这样：
```
redis.get('people:2:home', function (error, home) {
    redis.hget('locations', home, function (error, address) {
        redis.exists('address:' + address, function (error, addressExists) {
            if (addressExists) {
                console.log('地址存在。');
            } else {
                redis.exists('backup.address:' + address, function (error, backupAddressExists) {
                    if (backupAddressExists) {
                        console.log('备用地址存在。');
                    } else {
                        console.log('地址不存在。');
                    }
                });
            }
        });
    });
});
```
上面的代码并不是极端的情况，相反在实际开发中经常会遇到这种多层嵌套。为了减少嵌套，可以考虑使用 Async 、Step等第三方模块。如上面的代码可以稍微修改后使用Async重写为：
```
async.waterfall([
    function (callback) {
        redis.get('people:2:home', callback);
    },
    function (home, callback) {
        redis.hget('locations', home, callback);
    },
    function (address, callback) {
        async.parallel([
            function (callback) {
                redis.exists('address:' + address, callback);
            },
            function (callback) {
               redis.exists('backup.address:' + address, callback);
            }], function (err, results) {
            if (results[0]) {
                console.log('地址存在。');
            } else if (results[1]) {
                console.log('备用地址存在。');
            } else {
                console.log('地址不存在。');
            }
        });
    }
]);
```
另外，可以使用co模块借助ES6的Generator特性来将ioredis的返回结果“串行化”：
```
var co = require('co');
co(function* () {
    var result = yield redis.get('foo');
    return result;
}).then(function (fooValue) {
    console.log(fooValue);
});
```

### 4.简便用法
---
1．HMSET/HGETALL
  ioredis同样支持在HMSET命令中使用对象作参数（对象的属性值只能是字符串），相应的HGETALL命令会返回一个对象。
2．事务
  事务的用法如下：
```
var multi = redis.multi();
multi.set('foo', 'bar');
multi.sadd('set', 'a');
mulit.exec(function (err, replies) {
    //replies 是一个数组，依次存放事务队列中命令的结果
    console.log(replies);
});
```
或者使用链式调用：
```
redis.multi()
    .set('foo', 'bar')
    .sadd('set', 'a')
    .exec(function (err, replies) {
        console.log(replies);
    });
```
3．“发布/订阅”模式
  Node.js 使用事件的方式实现“发布/订阅”模式。现在创建两个连接分别充当发布者和订阅者：
```
var pub = new Redis();
var sub = new Redis();
```
然后让sub订阅chat频道并在订阅成功后发送一条消息：
```
sub.subscribe('chat', function () {
    pub.publish('chat', 'hi!');
});
```
定义当接收到消息时要执行的回调函数：
```
sub.on('message', function (channel, message) {
    console.log('收到' + channel + '频道的消息：' + message);
});
```
运行后可以看到打印的结果：
```
$ node testpubsub.js
```
收到chat频道的消息：'hi!'
补充知识 在 ioredis 中建立连接的过程也是异步的，执行 redis = new Redis()后连接并没有立即建立完成。在连接建立完成前执行的命令会被加入到离线任务队列中，当连接建立成功后ioredis会按照加入的顺序依次执行离线任务队列中的命令。



