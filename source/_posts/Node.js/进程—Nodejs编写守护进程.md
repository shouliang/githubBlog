title: 进程—Nodejs编写守护进程

date: 2016-07-23 22:16:53

tags: [Node.js]
---
目前Nodejs编写一个守护进程非常简单，在6.3.1版本中已经存在非常方便的API，这些API可以帮助我们更方便的创建一个守护进程。本文仅在描述守护进程的创建方式，而不会对守护进程所要执行的任务做任何描述。

### 守护进程的启动方式

如果不在Nodejs环境中，我们如何创建守护进程？过程如下：

1. 创建一个进程A。
2. 在进程A中创建进程B，我们可以使用fork方式，或者其他方法。
3. 对进程B执行 `setsid` 方法。
4. 进程A退出，进程B由init进程接管。此时进程B为守护进程。

### Setsid详解

`setsid` 主要完成三件事：

1. 该进程变成一个新会话的会话领导。
2. 该进程变成一个新进程组的组长。
3. 该进程没有控制终端。

然而，Nodejs中并没有对 `setsid` 方法的直接封装，翻阅文档发现有一个地方是可以调用该方法的。

### Nodejs中启动子进程方法

借助 `clild_process` 中的 `spawn` 即可创建子进程，方法如下：

```javascript
var spawn = require('child_process').spawn;
var process = require('process');

var p = spawn('node',['b.js']);
console.log(process.pid, p.pid);
```

注意，这里只打印当前进程的PID和子进程的PID，同时为了观察效果，我并没有将父进程退出。

`b.js` 中代码很简单，打开一个资源，并不停的写入数据。

```javascript
var fs = require('fs');
var process = require('process');

fs.open("/Users/mebius/Desktop/log.txt",'w',function(err, fd){
	console.log(fd);
	while(true)
	{
		fs.write(fd,process.pid+"\n",function(){});
	}
});
```

运行后的效果如图：
![](http://7xq1il.com1.z0.glb.clouddn.com/fork01_180703.png)

我们来看以下 `top` 命令下的进程情况。
![](http://7xq1il.com1.z0.glb.clouddn.com/fork02_180703.png)

看一看到，此时父进程PID为17055，子进程的PPID为17055，PID为17056.

### Nodejs中Setsid的调用

到此为止，守护进程已经完成一半，下面要调用setsid方法，并且退出父进程。

代码修改如下：

```javascript
var spawn = require('child_process').spawn;
var process = require('process');

var p = spawn('node',['b.js'],{
        detached : true
    });
console.log(process.pid, p.pid);
process.exit(0);
```

在 `spawn` 的第三个参数中，可以设置 `detached` 属性，如果该属性为true，则会调用 `setsid` 方法。这样就满足我们对守护进程的要求。

在此运行命令。
![](http://7xq1il.com1.z0.glb.clouddn.com/fork03_180703.png)

查看 `top` 命令

![](http://7xq1il.com1.z0.glb.clouddn.com/fork04_180703.png)

可以看到，当前仅存在一个PID为17062的进程，这个进程就是我们要的守护进程。

> 由于每次运行PID都不同，所以此次子进程的PID于第一次不同。

### 总结

守护进程最重要的是稳定，如果守护进程挂掉，那么其管理的子进程都将变为孤儿进程，同时被init进程接管，这是我们不愿意看到的。于此同时，守护进程对于子进程的管理也是有非常多的发挥余地的，例如PM2中，将一个进程同时启动4次，达到CPU多核使用的目的（很有可能你的进程在同一核中运行），进程挂掉后自动重启等等，这些事情等着我们去造轮子。

普通的进程, 在用户退出终端之后就会直接关闭. 通过 & 启动到后台的进程, 之后会由于会（session组）被回收而终止进程. 守护进程是不依赖终端（tty）的进程, 不会因为用户退出终端而停止运行的进程.

总体来说，Nodejs启动守护进程方式比较简单，默认所暴露的API也屏蔽了很多系统级别API，使得大家使用上更加方便，但没有接触过Linux的人在理解上有一些复杂。推荐大家学习Nodejs的同时，多学习Linux系统调用的和系统内核的一些东西。

### 原文
https://ashan.org/archives/917