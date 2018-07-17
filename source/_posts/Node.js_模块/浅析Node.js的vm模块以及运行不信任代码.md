title: 浅析Node.js的vm模块以及运行不信任代码
date: 2018-03-07 14:45:23
tags: [Node.js_模块]
---

在一些系统中，我们希望给用户提供插入自定义逻辑的能力，除了 `RPC` 和 `REST` 之外，运行客户提供的代码也是比较常用的方法，好处是可以极大地减少在网络上的耗时。JavaScript 是一种非常流行而且容易上手的语言，因此，让用户用 JavaScript 来写自定义逻辑是一个不错的选择。下面我们介绍 [Node.js](https://nodejs.org/en/) 提供的 [vm](https://nodejs.org/dist/latest-v7.x/docs/api/vm.html) 模块以及分析用它来运行不信任代码可能遇到的问题。

### vm模块

vm 模块是 Node.js 内置的核心模块，它能让我们编译 JavaScript 代码和在指定的环境中运行。请看下面例子：

```javascript
const util = require('util');
const vm = require('vm');

// 1. 创建一个 vm.Script 实例, 编译要执行的代码
const script = new vm.Script('globalVar += 1; anotherGlobalVar = 1; ');
// 2. 用于绑定到 context 的对象
const sandbox = {globalVar: 1};
// 3. 创建一个 context, 并且把 sandbox 这个对象绑定到这个环境, 作为全局对象
const contextifiedSandbox = vm.createContext(sandbox);
// 4. 运行上面编译的代码, context 是 contextifiedSandbox
const result = script.runInContext(contextifiedSandbox);

console.log(`sandbox === contextifiedSandbox ? ${sandbox === contextifiedSandbox}`);
// sandbox === contextifiedSandbox ? true
console.log(`sandbox: ${util.inspect(sandbox)}`);
// sandbox: { globalVar: 2, anotherGlobalVar: 1 }
console.log(`result: ${util.inspect(result)}`);
// result: 1
```

`vm.Script` 是一个类，用于创建代码实例，后面可以多次运行。

`vm.createContext(sandbox)` 用于 "contextify" 一个对象，根据 [ECMAScript 2015 语言规范](http://www.ecma-international.org/ecma-262/6.0/)，代码的执行需要一个 [execution context](http://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts)。这里的 "contextify"，就是把传进去的对象与 V8 的一个新的 [context](https://github.com/v8/v8/wiki/Embedder) 进行关联。这里所说的关联，我的理解是，这个 "contextified" 对象的属性将会成为那个 context 的全局属性，同时，在 context 下运行代码时产生的全局属性也会成为这个 "contextified" 对象的属性。

`script.runInContext(contextifiedSandbox)` 就是使代码在 `contextifiedSandbox` 这个 context 中运行，从上面的输出可以看到，代码运行后，`contextifiedSandbox` 里面的属性的值已经被改变了，运行结果是最后一个表达式的值。

除了上面几个接口之外，vm 模块还有一些更便捷的接口，例如 `vm.runInContext(code, contextifiedSandbox[, options])`，`vm.runInNewContext(code[, sandbox][, options])`等，详细可看[文档](https://nodejs.org/api/vm.html)。

### 外层如何得到代码运行结果

我们用 vm 运行代码的时候很可能需要得到一些结果，从上面的例子中可以看到，我们可以通过把结果作为最后一个表达式的值传给外层，或者作为`context` 的属性给外层使用，这在同步代码里没有问题，但是假如结果需要依赖里面的异步操作呢？这时，我们可以通过在 `context` 里放一个回调函数。 下面是例子：

```javascript
const util = require('util');
const vm = require('vm');

const sandbox = {globalVar: 1, setTimeout: setTimeout, cb: function(result) {
    console.log(result);
}};
vm.createContext(sandbox);
const script = new vm.Script(`
    setTimeout(function(){
        globalVar++;
        cb("async result");
    }, 1000);
`,{});
script.runInContext(sandbox);
console.log(`globalVar: ${sandbox.globalVar}`);
// globalVar: 1
// async result
```

### 代码运行时间限制

`script.runInContext(contextifiedSandbox[, options])` 方法有一个 `timeout` 选项可以设定代码的运行时间，如果超过时间就会抛出错误，请看下面例子：　

```javascript
const util = require('util');
const vm = require('vm');
const sandbox = {};
const contextifiedSandbox = vm.createContext(sandbox);
const script = new vm.Script('while(true){}');
const result = script.runInContext(contextifiedSandbox, {timeout: 1000});
// const result = script.runInContext(contextifiedSandbox, {timeout: 1000});
//                       ^
// Error: Script execution timed out.
```

再试试异步代码，

```javascript
const util = require('util');
const vm = require('vm');

const sandbox = {globalVar: 1, setTimeout: setTimeout, cb: function(result) {
    console.log(result);
}};
vm.createContext(sandbox);
const script = new vm.Script(`
    setTimeout(function(){
        globalVar++;
        cb("async result");
    }, 1000);
    globalVar;
`,{});
const result = script.runInContext(sandbox, {timeout: 500});
console.log(`result: ${result}`);
// result: 1
// async result
```

没有错误抛出，也就是说，这个选项并不能限制异步代码的运行时间，那应该怎么去限制所有代码的执行时间呢，目前好像没有接口终止 vm 代码的运行，如果有异步代码长时间不结束，很容易造成内存泄露，目前可行的方案是使用子进程去运行代码，如果超过限定时间还没有结果，就杀掉该子进程，另外，使用子进程还可以更方便地对内存等资源进行限制。

### 定制 context 与安全问题

在一个全新的 [V8 context](https://github.com/v8/v8/wiki/Embedder) 里运行代码，里面包含了语言规范规定的内置的一些函数和对象，如果我们想要一些语言规范之外的功能或者模块，我们需要把相应对象放到与这个 context 关联的对象里，例如在上面例子中的这句代码：

```javascript
const sandbox = {globalVar: 1, setTimeout: setTimeout, cb: function(result) {
    console.log(result);
}};
```

`setTimeout` 不是语言规范规定的内置函数， context 本身不提供，所以我们需要通过关联的对象传进去。

然而，当我们把一些模块功能提供给 context 的时候，也同时带入了更多的安全隐患，请看下面来自例子：

```javascript
const util = require('util');
const vm = require('vm');

const sandbox = {};
vm.createContext(sandbox);
const script = new vm.Script(`
    // sandbox 的 constructor 是外层的 Object 类
    // Object 类的 constructor 是外层的 Function 类
    const OutFunction = this.constructor.constructor;
    // 于是, 利用外层的 Function 构造一个函数就可以得到外层的全局 this
    const OutThis = (OutFunction('return this;'))();
    // 得到 require
    const require = OutThis.process.mainModule.require;
    // 试试
    require('fs');
`,{});
const result = script.runInContext(sandbox);
console.log(result === require('fs'));
// true
```

显然，定制 context 的时候，任何一个传进去的对象或者函数都可能带来上面的问题，安全问题真的有很多工作需要做。

Github 上有一些开源的模块用于运行不信任代码，例如 [sandbox](https://github.com/gf3/sandbox)，[vm2](https://github.com/patriksimek/vm2)，[jailed](https://github.com/asvd/jailed)等。查看这些项目的 issue 可以发现，sandbox 和 jailed 都可以用类似上面的方法突破限制，而 vm2 对这方面做了防护，其它方面也做了更多的安全工作，相对安全些。

生产中可以考虑在子进程中运行 vm2， 然后增加更低层的安全限制， 例如限制进程的权限和使用 [cgroups](https://wiki.archlinux.org/index.php/cgroups) 进行 IO，内存等资源限制，这里不详细讨论。

### 总结

本文通过几个例子介绍了 Node.js 的 vm 模块以及使用 vm 模块运行不信任代码可能遇到的问题，并且对安全问题给出了一些建议。

### 原文
https://segmentfault.com/a/1190000008284054