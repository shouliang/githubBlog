title: 当我们谈论 cluster 时我们在谈论什么

date: 2015-11-08 22:16:53

tags: [Node.js_进程]
---

Node.js 诞生之初就遭到不少这样的吐槽，当然这些都早已不是问题了。

1. 可靠性低。
2. 单进程，单线程，只支持单核 CPU，不能充分的利用多核 CPU 服务器。一旦这个进程崩掉，那么整个 web 服务就崩掉了。

回想以前用 php 开发 web 服务器的时候，每个 request 都在单独的线程中处理，即使某一个请求发生很严重的错误也不会影响到其它请求。Node.js 会在一个线程中处理大量请求，如果处理某个请求时产生一个没有被捕获到的异常将导致整个进程的退出，已经接收到的其它连接全部都无法处理，对一个 web 服务器来说，这绝对是致命的灾难。

应用部署到多核服务器时，为了充分利用多核 CPU 资源一般启动多个 Node.js 进程提供服务，这时就会使用到 Node.js 内置的 cluster 模块了。相信大多数的 Node.js 开发者可能都没有直接使用到 cluster，cluster 模块对 child_process 模块提供了一层封装，可以说是为了发挥服务器多核优势而量身定做的。简单的一个 fork，不需要开发者修改任何的应用代码便能够实现多进程部署。当下最热门的带有负载均衡功能的 Node.js 应用进程管理器 pm2 便是最好的一个例子，开发的时候完全不需要关注多进程场景，剩余的一切都交给 pm2 处理，与开发者的应用代码完美分离。

```
pm2 start app.js
```

pm2 确实非常强大，但本文并不讲解 pm2 的工作原理，而是从更底层的进程通信讲起，为大家揭秘使用 Node.js 开发 web 应用时，使用 cluster 模块实现多进程部署的原理。

## fork

说到多进程当然少不了 fork ,在 un*x 系统中，fork 函数为用户提供最底层的多进程实现。

> fork() creates a new process by duplicating the calling process. The new process is referred to as the child process. The calling process is referred to as the parent process.
>
> The child process and the parent process run in separate memory spaces. At the time of fork() both memory spaces have the same content. Memory writes, file mappings (mmap(2)), and unmappings (munmap(2)) performed by one of the processes do not affect the other.

本文中要讲解的 fork 是 cluster 模块中非常重要的一个方法，当然了，底层也是依赖上面提到的 fork 函数实现。 多个子进程便是通过在master进程中不断的调用 cluster.fork 方法构造出来。下面的结构图大家应该非常熟悉了。

![](http://7xq1il.com1.z0.glb.clouddn.com/cluster01_180703.png)

上面的图非常粗糙， 并没有告诉我们 master 与 worker 到底是如何分工协作的。Node.js 在这块做过比较大的改动，下面就细细的剖析开来。

### 多进程监听同一端口

最初的 Node.js 多进程模型就是这样实现的，master 进程创建 socket，绑定到某个地址以及端口后，自身不调用 listen 来监听连接以及 accept 连接，而是将该 socket 的 fd 传递到 fork 出来的 worker 进程，worker 接收到 fd 后再调用 listen，accept 新的连接。但实际一个新到来的连接最终只能被某一个 worker 进程 accpet 再做处理，至于是哪个 worker 能够 accept 到，开发者完全无法预知以及干预。这势必就导致了当一个新连接到来时，多个 worker 进程会产生竞争，最终由胜出的 worker 获取连接。

![](http://7xq1il.com1.z0.glb.clouddn.com/cluster02_180703.png)
为了进一步加深对这种模型的理解，我编写了一个非常简单的 demo。

master 进程

```
const net = require('net');
const fork = require('child_process').fork;

var handle = net._createServerHandle('0.0.0.0', 3000);

for(var i=0;i<4;i++) {
   fork('./worker').send({}, handle);
}
```

worker 进程

```
const net = require('net');
process.on('message', function(m, handle) {
  start(handle);
});

var buf = 'hello nodejs';
var res = ['HTTP/1.1 200 OK','content-length:'+buf.length].join('\r\n')+'\r\n\r\n'+buf;

function start(server) {
    server.listen();
    server.onconnection = function(err,handle) {
        console.log('got a connection on worker, pid = %d', process.pid);
        var socket = new net.Socket({
            handle: handle
        });
        socket.readable = socket.writable = true;
        socket.end(res);
    }
}
```

保存后直接运行 `node master.js` 启动服务器，在另一个终端多次运行 `ab -n10000 -c100 http://127.0.0.1:3000/`

各个 worker 进程统计到的请求数分别为

```
worker 63999  got 14561 connections
worker 64000  got 8329  connections
worker 64001  got 2356  connections
worker 64002  got 4885  connections
```

相信到这里大家也应该知道这种多进程模型比较明显的问题了

- 多个进程之间会竞争 accpet 一个连接，产生惊群现象，效率比较低。
- 由于无法控制一个新的连接由哪个进程来处理，必然导致各 worker 进程之间的负载非常不均衡。

这其实就是著名的”惊群”现象。

简单说来，多线程/多进程等待同一个 socket 事件，当这个事件发生时，这些线程/进程被同时唤醒，就是惊群。可以想见，效率很低下，许多进程被内核重新调度唤醒，同时去响应这一个事件，当然只有一个进程能处理事件成功，其他的进程在处理该事件失败后重新休眠（也有其他选择）。这种性能浪费现象就是惊群。

惊群通常发生在 server 上，当父进程绑定一个端口监听 socket，然后 fork 出多个子进程，子进程们开始循环处理（比如 accept）这个 socket。每当用户发起一个 TCP 连接时，多个子进程同时被唤醒，然后其中一个子进程 accept 新连接成功，余者皆失败，重新休眠。

### nginx proxy

现代的 web 服务器一般都会在应用服务器外面再添加一层负载均衡，比如目前使用最广泛的 nginx。
利用 nginx 强大的反向代理功能，可以启动多个独立的 node 进程，分别绑定不同的端口，最后由nginx 接收请求然后进行分配。

```
http { 
  upstream cluster { 
      server 127.0.0.1:3000; 
      server 127.0.0.1:3001; 
      server 127.0.0.1:3002; 
      server 127.0.0.1:3003; 
  } 
  server { 
       listen 80; 
       server_name www.domain.com; 
       location / { 
            proxy_pass http://cluster;
       } 
  }
}
```

这种方式就将负载均衡的任务完全交给了 nginx 处理，并且 nginx 本身也相当擅长。再加一个守护进程负责各个 node 进程的稳定性，这种方案也勉强行得通。但也有比较大的局限性，比如想增加或者减少一个进程时还得再去改下 nginx 的配置。该方案与 nginx 耦合度太高，实际项目中并不经常使用。

## 小结

说了这么多，一直在讲解 Node.js 多进程部署时遇到的各种问题。小伙伴们肯定会有非常多的疑问。实际的 Node.js 项目中我们到底是如何利用多进程的呢，并且如何保障各个 worker 进程的稳定性。如何利用 cluster 模块 fork 子进程，父子进程间又是如何实现通信的呢？

下篇将为大家一一揭晓，敬请期待！

[上篇文章](http://taobaofed.org/blog/2015/11/03/nodejs-cluster/)讲解了 Node.js 中多进程部署时遇到的各种问题，那么实际的线上项目中到底是如何利用多进程，如何保障各个 worker 进程稳定性的呢，又是如何利用 cluster 模块 fork 子进程，父子进程间又是如何实现通信的呢？本篇就来一一揭晓。

## 负载均衡

回忆一下上篇中提到的最初 Node.js 多进程模型，多个进程绑定同一端口，相互竞争 accpet 新到来的连接。由于无法控制一个新的连接由哪个进程来处理，导致各 worker 进程之间的负载非常不均衡。

于是后面就出现了基于 round-robin 算法的另一种模型。主要思路是 master 进程创建 socket，绑定地址以及端口后再进行监听。该 socket 的 fd 不传递到各个 worker 进程。当 master 进程获取到新的连接时，再决定将 accept 到的客户端连接分发给指定的 worker 处理。这里使用了**指定**, 所以如何传递以及传递给哪个 worker 完全是可控的。round-robin 只是其中的某种算法而已，当然可以换成其他的。

![](http://7xq1il.com1.z0.glb.clouddn.com/cluster03_180703.png)

同样基于这种模型也给出一个简单的 demo。

master 进程

```
const net = require('net');
const fork = require('child_process').fork;

var workers = [];
for (var i = 0; i < 4; i++) {
   workers.push(fork('./worker'));
}

var handle = net._createServerHandle('0.0.0.0', 3000);
handle.listen();
handle.onconnection = function (err,handle) {
    var worker = workers.pop();
    worker.send({},handle);
    workers.unshift(worker);
}
```

woker 进程

```
const net = require('net');
process.on('message', function (m, handle) {
  start(handle);
});

var buf = 'hello Node.js';
var res = ['HTTP/1.1 200 OK','content-length:'+buf.length].join('\r\n')+'\r\n\r\n'+buf;

function start(handle) {
    console.log('got a connection on worker, pid = %d', process.pid);
    var socket = new net.Socket({
        handle: handle
    });
    socket.readable = socket.writable = true;
    socket.end(res);
}
```

由于只有 master 进程接收客户端连接，并且能够按照特定的算法进行分发， 很好的解决了上篇中提到的由于竞争导致各 worker 进程负载不均衡的硬伤。

## 优雅退出

上篇文章开头提到 Node.js 被吐槽稳定性差，进程发生未捕获到的异常就会退出。实际项目中由于各种原因，不可避免最后上线时还是存在各种 bug 以及异常，最终进程退出。

当进程异常退出时，有可能该进程上还有很多未处理完的请求，简单粗暴的使进程直接退出必然导致所有的请求都会丢失，给用户带来非常糟的体验，这就非常需要一个进程优雅退出的方案。

给 process 对象添加 uncaughtException 事件绑定能够避免发生异常时进程直接退出。在回调函数里调用当前运行 server 对象的 close 方法，停止接收新的连接。同时告知 master 进程该 worker 进程即将退出，可以 fork 新的 worker 了。

接着在几秒中之后差不多所有请求都已经处理完毕后，该进程主动退出，其中 timeout 可以根据实际业务场景进行设置。

```
setTimeout(function () {
  process.exit(1);
}, timeout)
```

这里面有一个小的细节处理，在关闭服务器之前，后续新接收的 request 全部关闭 keep-alive 特性，通知客户端不需要与该服务器保持 socket 连接了。

```
server.on('request', function (req, res) {
    req.shouldKeepAlive = false;
    res.shouldKeepAlive = false;
    if (!res._header) {
        res.setHeader('Connection', 'close');
    }
});
```

第三方 `graceful` 模块专门来处理这种场景的，感兴趣的同学可以阅读下源码。

## 进程守护

master 进程除了负责接收新的连接，分发给各 worker 进程处理之外，还得像天使一样默默地守护着这些 worker 进程，保障整个应用的稳定性。一旦某个 worker 进程异常退出就 fork 一个新的子进程顶替上去。

这一切 cluster 模块都已经好处理了，当某个 worker 进程发生异常退出或者与 master 进程失去联系（disconnected）时，master 进程都会收到相应的事件通知。

```
cluster.on('exit', function () {
    clsuter.fork();
});

cluster.on('disconnect', function () {
    clsuter.fork();
});
```

推荐使用第三方模块 recluster 和 cfork，已经处理的很成熟了。

这样一来整个应用的稳定性重任就落在 master 进程上了，所以一定不要给 master 太多其它的任务，百分百保证它的健壮性，一旦 master 进程挂掉你的应用也就玩完了。

## IPC

master 进程能够接收连接进行分发，同时守护 worker 进程，这一切都离不开进程间的通信。
讲了这么多，终于到最核心的地方了，要用多进程模型就一定会涉及到 IPC（进程间通信）了。Node.js 中 IPC 都是在父子进程之间进行，按有无发送 fd 分为 2 种方式。

### 发送 fd

当进程间需要发生文件描述符 fd 时，libuv 底层采用消息队列来实现 IPC。master 进程接收到客户端连接分发给 worker 进程处理时就用到了进程间 fd 的传递。

### 不发送 fd

这种情况父子进程之间只是发送简单的字符串，并且它们之间的通信是双向的。master 与 worker 间的消息传递便是这种方式。虽然 pipe 能够满足父子进程间的消息传递，但由于 pipe 是半双工的，也就是说必须得创建 2 个 pipe 才可以实现双向的通信，这无疑使得程序逻辑更复杂。

libuv 底层采用 socketpair 来实现全双工的进程通信，父进程 fork 子进程之前会调用 socketpair 创建 2 个 fd，下面是一个最简单的也最原始的利用 socketpair 来实现父子进程间双向通信的 demo。

```
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <errno.h>
#include <sys/socket.h>
#include <stdlib.h>
#define BUF_SIZE 100

int main () {
    int s[2];
    int w,r;
    char * buf = (char*)calloc(1 , BUF_SIZE);
    pid_t pid;
    
    if (socketpair(AF_UNIX,SOCK_STREAM,0,s) == -1 ) {
        printf("create unnamed socket pair failed:%s\n", strerror(errno));
        exit(-1);
    }
    
    if ((pid = fork()) > 0) {
        printf("Parent process's pid is %d\n",getpid());
        close(s[1]);
        char *messageToChild = "a message to child  process!";
        if ((w = write(s[0] , messageToChild , strlen(messageToChild) ) ) == -1) {
            printf("Write socket error:%s\n",strerror(errno));
            exit(-1);
        }
        sleep(1);
        if ( (r = read(s[0], buf , BUF_SIZE )) == -1) {
          printf("Pid %d read from socket error:%s\n",getpid() , strerror(errno) );
          exit(-1);
        }
        printf("Pid %d read string : %s \n",getpid(),buf);
    } else if (pid == 0) {
         printf("Fork child process successed\n");
         printf("Child process's pid is :%d\n",getpid());
         close(s[0]);
         char *messageToParent = "a message to parent process!";
         if ((w = write(s[1] , messageToParent , strlen(messageToParent))) == -1 ) {
             printf("Write socket error:%s\n",strerror(errno));
             exit(-1);
         }
         sleep(1);
         if ((r = read(s[1], buf , BUF_SIZE )) == -1) {
             printf("Pid %d read from socket error:%s\n", getpid() , strerror(errno) );
             exit(-1);
         }
         printf("Pid %d read string : %s \n",getpid(),buf); 
     } else {
        printf("Fork failed:%s\n",strerror(errno));
        exit(-1);
    }
    exit(0);
}
```

保存为 socketpair.c 后运行 `gcc socketpair.c -o socket && ./socket` 输出

```
Parent process's pid is 52853
Fork child process successed
Child process's pid is :52854
Pid 52854 read string : a message to child  process! 
Pid 52853 read string : a message to parent process!
```

### Node.js 中的 IPC

上面从 libuv 底层方面讲解了父子进程间双向通信的原理，在上层 Node.js 中又是如何实现的呢，让我们来一探究竟。

Node.js 中父进程调用 fork 产生子进程时，会事先构造一个 pipe 用于进程通信，

```
new process.binding('pipe_wrap').Pipe(true);
```

构造出的 pipe 最初还是关闭的状态，或者说底层还并没有创建一个真实的 pipe，直至调用到 libuv 底层的`uv_spawn`, 利用 socketpair 创建的全双工通信管道绑定到最初 Node.js 层创建的 pipe 上。

管道此时已经真实的存在了，父进程保留对一端的操作，通过环境变量将管道的另一端文件描述符 fd 传递到子进程。

```
options.envPairs.push('NODE_CHANNEL_FD=' + ipcFd);
```

子进程启动后通过环境变量拿到 fd

```
var fd = parseInt(process.env.NODE_CHANNEL_FD, 10);
```

并将 fd 绑定到一个新构造的 pipe 上

```
var p = new Pipe(true);
p.open(fd);
```

于是父子进程间用于双向通信的所有基础设施都已经准备好了。说了这么多可能还是不太明白吧？ 没关系，我们还是来写一个简单的 demo 感受下。

Node.js 构造出的 pipe 被存储在进程的`_channel`属性上

master.js

```
const WriteWrap = process.binding('stream_wrap').WriteWrap;
var cp = require('child_process');

var worker = cp.fork(__dirname + '/worker.js');
var channel = worker._channel;

channel.onread = function (len, buf, handle) {
    if (buf) {
        console.log(buf.toString())
        channel.close()
    } else {
        channel.close()
        console.log('channel closed');
    }
}

var message = { hello: 'worker',  pid: process.pid };
var req = new WriteWrap();
var string = JSON.stringify(message) + '\n';
channel.writeUtf8String(req, string, null);
```

worker.js

```
const WriteWrap = process.binding('stream_wrap').WriteWrap;
const channel = process._channel;

channel.ref();
channel.onread = function (len, buf, handle) {
    if (buf) {
        console.log(buf.toString())
    }else{
        process._channel.close()
        console.log('channel closed');
    }
}

var message = { hello: 'master',  pid: process.pid };
var req = new WriteWrap();
var string = JSON.stringify(message) + '\n';
channel.writeUtf8String(req, string, null);
```

运行`node master.js` 输出

```
{"hello":"worker","pid":58731}
{"hello":"master","pid":58732}
channel closed
```

### 进程失联

在多进程服务器中，为了保障整个 web 应用的稳定性，master 进程需要监控 worker 进程的 exit 以及 disconnect 事件，收到相应事件通知后重启 worker 进程。

exit 事件不用说，disconnect 事件可能很多人就不太明白了。还记得上面讲到的进程优雅退出吗，当捕获到未处理异常时，进程不立即退出，但是会立刻通知 master 进程重新 fork 新的进程，而不是等该进程主动退出后再 fork。具体的做法就是调用 worker进程的 disconnect 方法，从而关闭父子进程用于通信的 channel ，此时父子进程之间失去了联系，此时master 进程会触发 disconnect 事件，fork 一个新的 worker进程。

下面是一个触发`disconnect`事件的简单 demo

master.js

```
const WriteWrap = process.binding('stream_wrap').WriteWrap;
const net = require('net');
const fork = require('child_process').fork;

var workers = [];
for (var i = 0; i < 4; i++) {
     var worker = fork(__dirname + '/worker.js');
     worker.on('disconnect', function () {
         console.log('[%s] worker %s is disconnected', process.pid, worker.pid);
     });
     workers.push(worker);
}

var handle = net._createServerHandle('0.0.0.0', 3000);
handle.listen();
handle.onconnection = function (err,handle) {
    var worker = workers.pop();
    var channel = worker._channel;
    var req = new WriteWrap();
    channel.writeUtf8String(req, 'dispatch handle', handle);
    workers.unshift(worker);
}
```

worker.js

```
const net = require('net');
const WriteWrap = process.binding('stream_wrap').WriteWrap;
const channel = process._channel;
var buf = 'hello Node.js';
var res = ['HTTP/1.1 200 OK','content-length:' + buf.length].join('\r\n') + '\r\n\r\n' + buf;

channel.ref(); //防止进程退出
channel.onread = function (len, buf, handle) {
    console.log('[%s] worker %s got a connection', process.pid, process.pid);
    var socket = new net.Socket({
        handle: handle
    });
    socket.readable = socket.writable = true;
    socket.end(res);
    console.log('[%s] worker %s is going to disconnect', process.pid, process.pid);
    channel.close();
}
```

运行`node master.js`启动服务器后，在另一个终端执行多次`curl http://127.0.0.1:3000`，下面是输出的内容

```
[63240] worker 63240 got a connection
[63240] worker 63240 is going to disconnect
[63236] worker 63240 is disconnected
```

## 最简单的负载均衡 server

回到前面讲的 round-robin 多进程服务器模型，用于通信的 channel 除了可以发送简单的字符串数据外，还可以发送文件描述符，

```
channel.writeUtf8String(req, string, null);
```

最后一个参数便是要传递的 fd。round-robin 多进程服务器模型的核心也正式依赖于这个特性。 在上面的 demo 基础上，我们再稍微加工一下，还原在 Node.js 中最原始的处理。

master.js

```
const WriteWrap = process.binding('stream_wrap').WriteWrap;
const net = require('net');
const fork = require('child_process').fork;

var workers = [];
for(var i = 0; i < 4; i++) {
    workers.push(fork(__dirname + '/worker.js'));
}

var handle = net._createServerHandle('0.0.0.0', 3000);
handle.listen();
handle.onconnection = function (err,handle) {
    var worker = workers.pop();
    var channel = worker._channel;
    var req = new WriteWrap();
    channel.writeUtf8String(req, 'dispatch handle', handle);
    workers.unshift(worker);
}
```

worker.js

```
const net = require('net');
const WriteWrap = process.binding('stream_wrap').WriteWrap;
const channel = process._channel;
var buf = 'hello Node.js';
var res = ['HTTP/1.1 200 OK', 'content-length:' + buf.length].join('\r\n') + '\r\n\r\n' + buf;

channel.ref();
channel.onread = function (len, buf, handle) {
    var socket = new net.Socket({
        handle: handle
    });
    socket.readable = socket.writable = true;
    socket.end(res);
}
```

运行 `node master.js`， 一个简单的多进程 Node.js web 服务器便跑起来了。

## 小结

到此整个 Node.js 的多进程服务器模型，以及底层进程间通信原理就讲完了，也为大家揭开了 cluster 的神秘面纱， 相信大家对 cluster 有了更深刻的认识。祝大家 Node.js 的开发旅途上玩得更愉快！

### 原文
http://taobaofed.org/blog/2015/11/03/nodejs-cluster/
http://taobaofed.org/blog/2015/11/10/nodejs-cluster-2/
