title: 模块—Node.js 中的循环依赖
date: 2015-06-07 14:45:23

tags: [Node.js]
---
我们在写node的时候有可能会遇到循环依赖的情况，什么是循环依赖，怎么避免或解决循环依赖问题？

先看一段官网给出的循环依赖的代码:

`a.js`:

```
console.log('a starting'); 
exports.done = false;
var b = require('./b.js'); // ---> 1
console.log('in a, b.done = %j', b.done);
exports.done = true;
console.log('a done') // ---> 4
```

`b.js`:

```
console.log('b starting'); 
exports.done = false;
var a = require('./a.js');  // ---> 2
// console.log(a);  ---> {done:false}
console.log('in b, a.done = %j', a.done); // ---> 3
exports.done = true;
console.log('b done');
```

`main.js`:

```
console.log('main starting'); 
var a = require('./a.js'); // --> 0
var b = require('./b.js');
console.log('in main, a.done=%j, b.done=%j', a.done, b.done);
```

如果我们启动 `main.js` 会出现什么情况？ 在 `a.js` 中加载 `b.js`，然后在`b.js`中加载 `a.js`，然后再在 `a.js`中加载 `b.js` 吗？这样就会造成循环依赖死循环。

让我们执行看看：

```
$ node main.js

main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done=true, b.done=true
```

可以看到程序并没有陷入死循环，从上面的执行结果可以看到 `main.js` 中先`require`了 `a.js` ，`a.js` 中执行完了`console`和`export.done=fasle`之后，转而去加载`b.js`，待`b.js`被load完之后，再返回`a.js`中执行完剩下的代码。

我在官网的代码基础上增加了一些注释，基本 load 顺序就是按照这个`0-->1-->2-->3-->4`的顺序去执行的，然后在第二步下面我打印出了`require('./a')`的结果，可以看到是`{done:false}`，可以猜测在`b.js`中`require('./a')`的结果是`a.js`中已经执行到的`exports`出的值。

上面所说的还只是基于结果基础上的猜测，没有什么说服力，为了验证我的猜测是正确的，我把 Node 的源码稍微翻看了一些，C++ 的代码看不懂没关系，能看懂 JS 的部分就可以了，下面就是 Node 源码的分析（主要是 module 的分析， [Node 源码在此](https://github.com/nodejs/node)）：

将会分析的主要源码：

1. node/src/node.js
2. node/lib/module.js

## 启动 $ node main.js

C++ 的代码我看不懂，总而言之，在我查了资料之后知道当我们在`shell`中输入`node main.js`之后，会先执行 `node/src/node.cc`，然后会执行 `node/src/node.js`， 所以C++代码不分析，从分析 `node/src/node.js` 开始（只会分析和主题相关的代码）。

## node.js 源码分析

`node.js`文件主要结构为

```
(function(process) {

    this.global = this
    
    function startup() {
      ...
    }
    
    startup()

})
```

这种闭包代码很常见，从名字可以看出，此处为启动文件。接下来看看 startup 函数中有一大块条件语句，我删除大多数无关代码，如下：

```
if (process.argv[1]) {
     // ...
    
    var Module = NativeModule.require('module');
    Module.runMain();
}
```

我把无关的代码基本都删除了。可以看到这段代码主要做的事是先通过 Native 引入`module`模块，执行 `Module.runMain()`。

很多人都知道 `require` 核心代码，如 require('path')，不需要写全路径，Node 是怎样做到的呢？

> Node 采用了 V8 附带的 js2c.py 工具，将所有内置的 JavasSript 代码( src/node.js 和 lib/*.js) 转成 c++ 里面的数组生成 node_navtives.h 头文件。 
> 在这个过程中， JavasSript 以字符串的形式存储在 node 命名空间中， 是不可直接执行的。
> 在启动 Node 进程时， JavaScript 代码直接加载进内存中。
>
> Node 在启动时，会生成一个全局变量 process， 并提供 binding() 方法来协助加载内建模块。

上面大段介绍基本引自朴老师的「深入浅出 Node.js」。大概理解就是在启动命令的时候，Node 会把 `node.js` 和 `lib/*.js` 的内容都放到 `process` 中传入当前闭包中，我们在当前函数就可以通过`process.binding('natives')`取出来放到 _source 中，如下代码所示：

```
  function NativeModule(id) {
    this.filename = id + '.js';
    this.id = id;
    this.exports = {};
    this.loaded = false;
  }

  NativeModule._source = process.binding('natives');
  NativeModule._cache = {};
```

接下来看看`NativeModule.require`做了哪些事情：

```
  NativeModule.require = function(id) {
    if (id == 'native_module') {
      return NativeModule;
    }

    var cached = NativeModule.getCached(id);
    if (cached) {
      return cached.exports;
    }

    var nativeModule = new NativeModule(id);

    nativeModule.cache();
    nativeModule.compile();

    return nativeModule.exports;
  };
```

这上面的代码表明内建模块被缓存，就直接返回内建模块的`exports`，如果没有的话，就生成一个核心模块的实例，然后先把模块根据id来`cache`，然后调用`nativeModule.compile`接口编译源文件：

```
  NativeModule.getSource = function(id) {
    return NativeModule._source[id];
  };

  NativeModule.wrap = function(script) {
    return NativeModule.wrapper[0] + script + NativeModule.wrapper[1];
  };

  NativeModule.wrapper = [
    '(function (exports, require, module, __filename, __dirname) {\n',
    '\n});'
  ];

  NativeModule.prototype.compile = function() {
    var source = NativeModule.getSource(this.id);
    source = NativeModule.wrap(source);

    var fn = runInThisContext(source, {
      filename: this.filename,
      lineOffset: -1
    });
    fn(this.exports, NativeModule.require, this, this.filename);

    this.loaded = true;
  };

  NativeModule.prototype.cache = function() {
    NativeModule._cache[this.id] = this;
  };
```

cache 是把实例根据 id 放到 _cache 对象中。先从 _source 中取出对应id的源文件字符串，包上一层
`(function (exports, require, module, __filename, __dirname) {\n','\n});`。比如`main.js`最终变成如下JS代码的字符串:

```
(function (exports, require, module, __filename, __dirname) {
 // 如果是main.js
     console.log('main starting'); 
    var a = require('./a.js'); // --> 0
    var b = require('./b.js');
    console.log('in main, a.done=%j, b.done=%j', a.done, b.done);
})
```

`runInThisContext`是将被包装后的源字符串转成可执行函数，（`runInThisContext`来自`contextify `模块），`runInThisContext`的作用，类似`eval`，再执行这个被`eval`后的函数，就算被 load 完成了，最后把 load 设为 true。

可以看到`fn`的实参为 `this.exports; NativeModule.require; this; this.filename;`。

所以`require('module')`的作用是加载`/lib/module.js`文件。让我们再回到 startup 函数，加载完 module.js，紧接着运行 `Module.runMain()`方法。（估计有人忘了前面的startup函数是干嘛的，我再放一次，省得再拉回去了）

```
if (process.argv[1]) {
     // ...
    
    var Module = NativeModule.require('module');
    Module.runMain();
}
```

## module.js源码分析

上面走完了`NatvieModule`的加载代码。再看看`module.js`是怎样加载用户使用的文件的。

```
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  if (parent && parent.children) {
    parent.children.push(this);
  }

  this.filename = null;
  this.loaded = false;
  this.children = [];
}
module.exports = Module;

Module._cache = {};
Module._pathCache = {};
Module._extensions = {};
var modulePaths = [];
Module.globalPaths = [];

Module.wrapper = NativeModule.wrapper;
Module.wrap = NativeModule.wrap;
```

这是`Module`的构造函数，`Module.wrapper`和`Module.wrap`，是由`NativeModule`赋值来的，`Module._cache`是个空对象，存放所有被 load 后的模块 id。

在`node.js`文件的 startup 函数中，最后一步走到`Module.runMain()`:

```
Module.runMain = function() {
  // Load the main module--the command line argument.
  Module._load(process.argv[1], null, true);
  // Handle any nextTicks added in the first tick of the program
  process._tickCallback();
};
```

在`runMain`方法中调用了`_load`方法:

```
Module._load = function(request, parent, isMain) {
  var filename = Module._resolveFilename(request, parent);
  var cachedModule = Module._cache[filename];
  
  if (cachedModule) {
    return cachedModule.exports;
  }

  var module = new Module(filename, parent);
  Module._cache[filename] = module;

  module.load(filename);
  
  return module.exports;
};
```

上述代码照例我删除了一些不是很相关的代码，从剩下的代码可以看出`_load`函数的主要干了两件事（还有一件加载NativeModule的代码被我删掉了）:

1. 先判断当前的源文件有没有被加载过，如果 _cache 对象中存在，直接返回 _cache 中的exports对象
2. 如果没有被加载过，新建这个源文件的 module 的实例，并存放到 _cache 中，然后调用 load 方法。

```
Module.prototype.load = function(filename) {
  this.filename = filename;
  this.paths = Module._nodeModulePaths(path.dirname(filename));

  var extension = path.extname(filename) || '.js';
  if (!Module._extensions[extension]) extension = '.js';
  Module._extensions[extension](this, filename);
  this.loaded = true;
};
```

在`load`方法中判断源文件的扩展名是什么，默认是`'.js'`，（我这里也只分析后缀是 `.js` 的情况），然后调用 `Module._extensions[extension]()` 方法，并传入 this 和 filename；当`extension`是`'.js'`的时候， 调用`Module._extensions['.js']()` 方法。

```
// Native extension for .js
Module._extensions['.js'] = function(module, filename) {
  var content = fs.readFileSync(filename, 'utf8');
  module._compile(internalModule.stripBOM(content), filename);
};
```

这个方法是读到源文件的字符串后，调用`module._compile`方法。

```
Module.prototype._compile = function(content, filename) {

  var self = this;

  function require(path) {
    return self.require(path);
  }

  var dirname = path.dirname(filename);
  // create wrapper function
  var wrapper = Module.wrap(content);

  var compiledWrapper = runInThisContext(wrapper,
                                      { filename: filename, lineOffset: -1 });

  var args = [self.exports, require, self, filename, dirname];
  return compiledWrapper.apply(self.exports, args);
};
```

其实跟`NativeModule`的`_complie`做的事情差不多。先把源文件`content`包装一层`(function (exports, require, module, __filename, __dirname) {\n','\n});`， 然后通过 `runInThisContext `把字符串转成可执行的函数，最后把 
`self.exports, require, self, filename, dirname` 这几个实参传入可执行函数中。

`require` 方法为:

```
Module.prototype.require = function(path) {
  return Module._load(path, this);
};
```

## 循环依赖的时候为什么不会无限循环引用

所谓的循环依赖就是在两个不同的文件中互相应用了对方。假设按照最上面官网给出的例子中，

在 `main.js` 中:

1. `require('./a.js')`；此时会调用 `self.require()`,
   然后会走到`module._load`，在`_load`中会判断`./a.js`是否被load过，当然运行到这里，`./a.js`还没被 load 过，所以会走完整个load流程，直到`_compile`。
2. 运行`./a.js`，运行到 `exports.done = false` 的时候，给 esports 增加了一个属性。此时的 `exports={done: false}`。
3. 运行`require('./b.js')`，同 第 1 步。
4. 运行`./b.js`，到`require('./a.js')`。此时走到`_load`函数的时候发现`./a.js`已经被load过了，所以会直接从`_cache`中返回。所以此时`./a.js`还没有运行完，`exports = {done.false}`，那么返回的结果就是 `in b, a.done = false`;
5. `./b.js`全部运行完毕，回到`./a.js`中，继续向下运行，此时的`./b.js`的 `exports={done:true}`， 结果自然是`in main, a.done=true, b.done=true`

### 原文
https://segmentfault.com/a/1190000004151411