title: 初识单线程的Node.js
date: 2015-11-05 22:16:53

tags: [Node.js_事件]
---

### 前言

从Node.js进入人们的视野时，我们所知道的它就由这些关键字组成 **事件驱动、非阻塞I/O、高效、轻量**，它在官网中也是这么描述自己的。

Node.js® is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js uses an **event-driven**, **non-blocking I/O model** that makes it **lightweight** and **efficient**.

于是，会有下面的场景出现：
当我们刚开始接触它时，可能会好奇：

- **为什么在浏览器中运行的 Javascript 能与操作系统进行如此底层的交互？**
当我们在用它进行文件 I/O 和网络 I/O 的时候，发现方法都需要传入回调，是异步的：

- **那么这种异步，非阻塞的 I/O 是如何实现的？**
当我们习惯了用回调来处理 I/O，发现当需要顺序处理时，Callback Hell 出现了，于是有想到了同步的方法：

- **那么在异步为主的 Node.js，有同步的方法嘛？**
身为一个前端，你在使用时，发现它的异步处理是基于事件的，跟前端很相似：

- **那么它如何实现的这种事件驱动的处理方式呢？**
当我们慢慢写的多了，处理了大量 I/O 请求的时候，你会想：

- **Node.js 异步非阻塞的 I/O 就不会有瓶颈出现吗？**
之后你还会想：

- **Node.js 这么厉害，难道没有它不适合的事情吗？**
看到这些问题，是否有点头大，别急，带着这些问题我们来慢慢看这篇文章。

### Node.js 结构
从 Node.js 本身入手，先来看看 Node.js 的结构。
![](http://7xq1il.com1.z0.glb.clouddn.com/nodestandard.jpeg)

我们可以看到，Node.js 的结构大致分为三个层次：

 Node.js 标准库，这部分是由 Javascript 编写的，即我们使用过程中直接能调用的 API。在源码中的 [lib](https://github.com/nodejs/node/tree/master/lib) 目录下可以看到。

- Node bindings，这一层是 Javascript 与底层 C/C++ 能够沟通的关键，前者通过 bindings 调用后者，相互交换数据。实现在 [node.cc](https://github.com/nodejs/node/blob/master/src/node.cc)
- 这一层是支撑 Node.js 运行的关键，由 C/C++ 实现。
  - V8：Google 推出的 Javascript VM，也是 Node.js 为什么使用的是 Javascript 的关键，它为 Javascript 提供了在非浏览器端运行的环境，它的高效是 Node.js 之所以高效的原因之一。
  - Libuv：它为 Node.js 提供了跨平台，线程池，事件池，异步 I/O 等能力，是 Node.js 如此强大的关键。
  - C-ares：提供了异步处理 DNS 相关的能力。
  - http_parser、OpenSSL、zlib 等：提供包括 http 解析、SSL、数据压缩等其他的能力。

###  Libuv
Libuv 是 Node.js 关键的一个组成部分，它为上层的 Node.js 提供了统一的 API 调用，使其不用考虑平台差距，隐藏了底层实现。

具体它能做什么，官网的这张图体现的很好：

![](http://7xq1il.com1.z0.glb.clouddn.com/1807041754libuv.png)

可以看出，它是一个对开发者友好的工具集，包含定时器，非阻塞的网络 I/O，异步文件系统访问，子进程等功能。它封装了 Libev、Libeio 以及 IOCP，保证了跨平台的通用性。

我们只要先知道它本身是异步和事件驱动的，记住这点，下面的问题就有了答案，我们一一来看。

 与操作系统交互

举个简单的例子，我们想要打开一个文件，并进行一些操作，可以写下面这样一段代码：

```
var fs = require('fs');
fs.open('./test.txt', "w", function(err, fd) {
	//..do something
});
```

这段代码的调用过程大致可描述为：[lib/fs.js](https://github.com/nodejs/node/blob/master/lib/fs.js) → [src/node_file.cc](https://github.com/nodejs/node/blob/master/src/node_file.cc) → [uv_fs](https://github.com/nodejs/node/tree/master/deps/uv/src)

Node.js 深入浅出上的一幅图：
![](http://7xq1il.com1.z0.glb.clouddn.com/nodelib_180701.png)

具体来说，当我们调用 `fs.open` 时，Node.js 通过 `process.binding` 调用 C/C++ 层面的 `Open` 函数，然后通过它调用 Libuv 中的具体方法 `uv_fs_open`，最后执行的结果通过回调的方式传回，完成流程。在图中，可以看到平台判断的流程，需要说明的是，这一步是在编译的时候已经决定好的，并不是在运行时中。

总体来说，我们在 Javascript 中调用的方法，最终都会通过 `process.binding` 传递到 C/C++ 层面，最终由他们来执行真正的操作。Node.js 即这样与操作系统进行互动。

通过这个过程，我们可以发现，实际上，Node.js 虽然说是用的 Javascript，但只是在开发时使用 Javascript 的语法来编写程序。真正的执行过程还是由 V8 将 Javascript 解释，然后由 C/C++ 来执行真正的系统调用，所以并不需要过分担心 Javascript 执行效率的问题。可以看出，Node.js 并不是一门语言，而是一个 **平台**，这点一定要分清楚。

###  异步、非阻塞 I/O
通过上文，我们了解到，真正执行系统调用的其实是 Libuv。之前我们提到，Libuv 本身就是异步和事件驱动的，所以，当我们将 I/O 操作的请求传达给 Libuv 之后，Libuv 开启线程来执行这次 I/O 调用，并在执行完成后，传回给 Javascript 进行后续处理。

这里面的 I/O 包括文件 I/O 和 网络 I/O。两者的底层执行略有不同。从上面的 Libuv 官网的图中，我们可以看到，文件 I/O，DNS 等操作，都是依托线程池（Thread Pool）来实现的。而网络 I/O 这一大类，包括：TCP、UDP、TTY 等，是由 epoll、IOCP、kqueue 来具体实现的。

总结来说，一个异步 I/O 的大致流程如下：

- 发起 I/O 调用
  1. 用户通过 Javascript 代码调用 Node 核心模块，将参数和回调函数传入到核心模块；
  2. Node 核心模块会将传入的参数和回调函数封装成一个请求对象；
  3. 将这个请求对象推入到 I/O 线程池等待执行；
  4. Javascript 发起的异步调用结束，Javascript 线程继续执行后续操作。
- 执行回调
  1. I/O 操作完成后，会将结果储存到请求对象的 result 属性上，并发出操作完成的通知；
  2. 每次事件循环时会检查是否有完成的 I/O 操作，如果有就将请求对象加入到 I/O 观察者队列中，之后当做事件处理；
  3. 处理 I/O 观察者事件时，会取出之前封装在请求对象中的回调函数，执行这个回调函数，并将 result 当参数，以完成 Javascript 回调的目的。

![](http://7xq1il.com1.z0.glb.clouddn.com/updateloop_180701.png)

这里面涉及到了 Libuv 本身的一个设计理念，事件循环（Event Loop），它是一个类似于 `while true` 的无限循环，其核心函数是 `uv_run`，下文会用到。

从这里，我们可以看到，我们其实对 Node.js 的单线程一直有个误会。事实上，它的单线程指的是自身 Javascript 运行环境的单线程，Node.js 并没有给 Javascript 执行时创建新线程的能力，最终的实际操作，还是通过 Libuv 以及它的事件循环来执行的。这也就是为什么 Javascript 一个单线程的语言，能在 Node.js 里面实现异步操作的原因，两者并不冲突。

###  事件驱动
说到，事件驱动，对于前端来说，并不陌生。事件，是一个在 GUI 开发时很常用的一个概念，常见的有鼠标事件，键盘事件等等。在异步的多种实现中，事件是一种比较容易理解和实现的方式。

说到事件，一定会想到回调，当我们写了一大堆事件处理函数后，Libuv 如何来执行这些回调呢？这就提到了我们之前说到的 `uv_run`，先看一张它的执行流程图：
![](http://7xq1il.com1.z0.glb.clouddn.com/loopalive_180701.png)

在 `uv_run` 函数中，会维护一系列的监视器：

```
typedef struct uv_loop_s uv_loop_t;
typedef struct uv_err_s uv_err_t;
typedef struct uv_handle_s uv_handle_t;
typedef struct uv_stream_s uv_stream_t;
typedef struct uv_tcp_s uv_tcp_t;
typedef struct uv_udp_s uv_udp_t;
typedef struct uv_pipe_s uv_pipe_t;
typedef struct uv_tty_s uv_tty_t;
typedef struct uv_poll_s uv_poll_t;
typedef struct uv_timer_s uv_timer_t;
typedef struct uv_prepare_s uv_prepare_t;
typedef struct uv_check_s uv_check_t;
typedef struct uv_idle_s uv_idle_t;
typedef struct uv_async_s uv_async_t;
typedef struct uv_process_s uv_process_t;
typedef struct uv_fs_event_s uv_fs_event_t;
typedef struct uv_fs_poll_s uv_fs_poll_t;
typedef struct uv_signal_s uv_signal_t;
```

这些监视器都有对应着一种异步操作，它们通过 `uv_TYPE_start`，来注册事件监听以及相应的回调。

在 `uv_run` 执行过程中，它会不断的检查这些队列中是或有 `pending` 状态的事件，有则触发，而且它在这里只会执行一个回调，避免在多个回调调用时发生竞争关系，因为 Javascript 是单线程的，无法处理这种情况。

上面的图中，对 I/O 操作的事件驱动，表达的比较清楚。除了我们常提到的 I/O 操作，图中还表述了一种情况，timer（定时器）。它与其他两者不同之处在于，它没有单独开立新的线程，而是在事件循环中直接完成的。

事件循环除了维护那些观察者队列，还维护了一个 `time` 字段，在初始化时会被赋值为0，每次循环都会更新这个值。所有与时间相关的操作，都会和这个值进行比较，来决定是否执行。

在图中，与 timer 相关的过程如下：

1. 更新当前循环的 time 字段，即当前循环下的“现在”；
2. 检查循环中是否还有需要处理的任务（handlers/requests），如果没有就不必循环了，即是否 alive。
3. 检查注册过的 timer，如果某一个 timer 中指定的时间落后于当前时间了，说明该 timer 已到期，于是执行其对应的回调函数；
4. 执行一次 I/O polling（即阻塞住线程，等待 I/O 事件发生），如果在下一个 timer 到期时还没有任何 I/O 完成，则停止等待，执行下一个 timer 的回调。如果发生了 I/O 事件，则执行对应的回调；由于执行回调的时间里可能又有 timer 到期了，这里要再次检查 timer 并执行回调。
Node.js 会一直调用 `uv_run` 直到到循环不在 alive。

###  同步方法
虽然 Node.js 是以异步为主要模式的，但我们在实际开发中，难免会有一些情况是有时序性的，如果由异步来写，就会写出很丑的 Callback Hell，如下：

```
db.query('select nickname from users where id="12"', function() {
	db.query('select * from xxx where id="12"', function() {
		db.query('select * from xxx where id="12"', function() {
			db.query('select * from xxx where id="12"', function() {
				//...	
			});
		});
	});
});
```

这个时候如果有同步方法，就会方便很多。这一点，Node.js 的开发者也想到了，目前大部分的异步操作函数，都存在其对应的同步版本，只需要在其名称后面加上 `Sync` 即可，不用传入回调。

```
var file = fs.readFileSync('/test.txt', {"encoding": "utf-8});
```

这写方法还是比较好用的，执行 shell 命令，读取文件等都比较方便。不过，体验不太好的一点就是这种调用的错误收集，它不会像回调函数那样，在第一参数中传入错误信息，它会将错误直接抛出，你需要使用 `try...catch` 来获取，如下：

```
var data;
try {
  data = fs.readFileSync('/test.txt');
} catch (e) {
	if (e.code == 'ENOENT') {
		//...
	}
 	//...
}
```

至于这些方法如何实现的，我们下回再论。

###  一些可能的瓶颈
首先，文件的 I/O 方面，用户代码的运行，事件循环的通知等，是通过 Libuv 维护的线程池来进行操作的，它会运行全部的文件系统操作。既然这样，我们抛开硬盘的影响，对于严谨的 C/C++ 来说，这个线程池一定是有大小限制的。官方默认给出的大小是 **4**。当然是可以改变的。在启动时，通过设置 `UV_THREADPOOL_SIZE` 来改变这个值即可。不过，最大也只能是 **128**，因为这个是涉及到内存占用的。

这个线程池对于所有的事件循环是共享的。当一个函数要使用线程池的时候（比如调用 `uv_queue_work`），Libuv 会预先分配并初始化 `UV_THREADPOOL_SIZE` 所允许的线程出来。而**128** 占用的内存大约是 1MB，如果设置的太高，当使用线程池频繁时，会因为内存占用过多而降低线程的性能。[具体说明](https://github.com/libuv/libuv/blob/master/docs/src/threadpool.rst);

对于网络 I/O 方面，以 Linux 系统下来说，网络 I/O 采用的是 epoll 这个异步模型。它的优点是采用了事件回调的方式，大大降低了文件描述符的创建（Linux下什么都是文件）。

在每次调用 `epoll_wait` 时，实际返回的是就绪描述符的数量，根据这个值，去 epoll 指定的数组里面取对应数量的描述符，是一种 **内存映射** 的方式，减少了文件描述符的复制开销。

上面提到的 epoll 指定的数组，它的大小即可监听的数量大小，它在不同的系统下，有不同的默认值，可见这里 [epoll create](https://github.com/nodejs/node/blob/master/deps/uv/src/unix/linux-syscalls.c#L80)。

有了大小的限制，还远不够，为了保证运行的稳定，防止你在调用 epoll 函数时，指针越界，导致内存泄漏。还会用到另外一个值 `maxevents`，它是 `epoll_wait` 所能处理的最大数量，在调用 `epoll_wait` 时可以指定。一般情况下小于创建时（epoll_create）的数组大小，当然，也可以设置的比 size 大，不过应该没什么用。可以想到如果就绪的事件很多，超过了 `maxevents`，那么超出的事件就要等待前面的事件处理完成，才可以继续，可能会导致效率的下降。

在这种情况下，你可能会担心事件会丢失。其实，是不会丢失的，它会通过 `ep_collect_ready_items` 将这些事件保存在一个队列中，在下一个 `epoll_wait` 再进行通知。

###  Node.js 不适合做什么
虽然看起来，Node.js 可以做很多事情，并且拥有很高的性能。比如做聊天室，搭建 Blog 等等，这些 I/O 密集型的应用，是比较适合 Node.js 的。

但是，有一种类型的应用，可能 Node.js 处理起来会比较吃力，那就是 CPU 密集型的应用。前文提到，Libuv 通过事件循环来处理异步的事件，这是存在于 Node.js 主线程的机制。通过这个机制，所有的 I/O 操作，底层API的调用都变成了异步的。但用户的 Javascript 代码是运行在主线程中的，如果这部分代码运行耗时很长，就会导致事件循环被阻塞。因为，它对于事件的处理，都是按照队列顺序的，所以如果其中的任何一个事务/事件本身没有完成，那么其他的回调、监听器、超时、nextTick() 都得不到运行的机会，被阻塞的事件循环没有机会去处理它们。这样下去，轻则效率降低，重则运行停滞。

比如我们常见的模板渲染，压缩，解压缩，加/解密等操作，都是 Node.js 的软肋，所以使用的时候要考虑到这方面。

###  总结
- Node.js 通过 `libuv` 来处理与操作系统的交互，并且因此具备了异步、非阻塞、事件驱动的能力。
- Node.js 实际上是 Javascript 执行线程的单线程，真正的的 I/O 操作，底层 API 调用都是通过多线程执行的。
- CPU 密集型的任务是 Node.js 的软肋。

### 原文
http://taobaofed.org/blog/2015/10/29/deep-into-node-1/