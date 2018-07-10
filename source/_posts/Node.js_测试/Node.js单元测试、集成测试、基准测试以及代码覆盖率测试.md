title: Node.js单元测试、集成测试、基准测试以及代码覆盖率测试
date: 2017-06-05 14:45:23

tags: [Node.js_测试]
---

### 黑盒测试
黑盒测试 (Black-box Testing), 测试应用程序的功能, 而不是其内部结构或运作. 测试者不需了解代码、内部结构等, 只需知道什么是应用应该做的事, 即当键入特定的输入, 可得到一定的输出. 测试者通过选择`有效输入`和`无效输入`来验证是否正确的输出. 此测试方法可适合大部分的软件测试, 例如集成测试 (Integration Testing) 以及系统测试 (System Testing).

### 白盒测试
白盒测试 (White-box Testing) 测试应用程序的内部结构或运作, 而不是测试应用程序的功能 (即黑盒测试). 在白盒测试时, 以编程语言的角度来设计测试案例. 白盒测试可以应用于单元测试 (Unit Testing)、集成测试 (Integration Testing) 和系统的软件测试流程, 可测试在集成过程中每一单元之间的路径, 或者主系统跟子系统中的测试.

## 单元测试
单元测试，又称模块测试，针对程序中的最小执行单元进行正确性测试。常见的开发模式包括 TDD 和 BDD 两类。

TDD（Test-driven development，测试驱动开发），先编写测试用例，然后针对测试用例开发模块，当测试用例不足时，补充测试用例；当模块无法通过测试时，持续更新模块代码，直到完全通过测试用例。其开发核心围绕测试用例展开，即测试用例的完整性决定了开发模块的健壮性和正确性，这容易由边界条件引发单元测试覆盖度不够的问题。

BDD（Behavior-driven development，行为驱动开发），用语义化的编程语言开发紧贴业务需求的测试用例，继而驱动相关模块的开发。

[AVA](https://github.com/sindresorhus/ava) 是 JavaScript 生态中最新潮的测试框架，其内置了 Babel，可以直接使用 ES6 语法，具有轻量高效、并发执行、强制隔离等优点，安装方法：

```javascript
npm install --save-dev ava
```

设置 `package.json` 中的 `scripts` 字段：

```javascript
{
    "scripts": {
        "test": "ava",
        "test:watch": "ava --watch"
    }
}
```

运行：

```javascript
npm test

# or
npm test:watch
```

下面是一个基本的测试代码：

```javascript
import test from 'ava';

const fibonacci = (n) => {
    if (n === 0 || n === 1) {
        return n;
    }
    return fibonacci(n - 1) + fibonacci(n - 2);
}

test('Test Fibonacci(0)', t => {
    t.is(fibonacci(0), 0);
});

test('Test Fibonacci(1)', t => {
    t.is(fibonacci(1), 1);
});

// HOOK CALLS
test.before('Before', t => {
    console.log('before');
});

test.after('After', t => {
    console.log('after');
});

test.beforeEach('BeforeEach', t => {
    console.log('   beforeEach');
});

test.afterEach('AfterEach', t => {
    console.log('   afterEach');
});
```

在上面的代码中，我们首先引入了 AVA 模块，然后创建了待测试的 `fibonacci` 函数，接下来是两个测试用例，最后是四个钩子方法：before() / after() / beforeEach() / afterEach()。

AVA 提供了一下修饰方法来指定测试的执行方式：

- `skip()`，跳过添加了 `skip()` 的测试用例

- `only()`，只执行添加了 `only()` 的测试用例

- `todo()`，占位标识符，表示将来需要添加的测试用例

- `serial()`，串行执行测试用例，默认情况下 AVA 会以并行的方式执行测试用例

```javascript
  test('Test Fibonacci(0)', t => {
      t.is(fibonacci(0), 0);
  });
```

在上面代码回调函数中的 `t`，称为断言执行对象，该对象包含以下方法：

- `t.end()`，结束测试，只在 `test.cb()` 中有效

- `t.plan(count)`，指定执行次数

- `t.pass([message])`，测试通过

- `t.fail([message])`，测试失败

- `t.ok(value, [message])`，断言 `value` 的值为真值

- `t.notOK(value, [message])`，断言 `value` 的值为假值

- `t.true(value, [message])`，断言 `value` 的值为 `true`

- `t.false(value, [message])`，断言 `value` 的值为 `false`

- `t.is(value, expected, [message])`，断言 `value === expected`

- `t.not(value, expected, [message])`，断言 `value !== expected`

- `t.same(value, expected, [message])`，断言 `value` 和 `expected` 深度相等

- `t.notSame(value, expected, [message])`，断言 `value` 和 `expected` 深度不等

- `t.throws(function | promise, [error, [message]])`，断言 `function` 抛出异常或 `promise`reject 错误

- `t.notThrows(function | promise, [message])`，断言 `function` 不会异常或 `promise` resolve

- `t.regex(contents, regex, [message])`，断言 `contents` 匹配 `regex`

- `t.ifError(error, [message])`，断言 `error` 是假值

### Mock
Mock 主要用于单元测试中. 当一个测试的对象可能依赖其他 (也许复杂/多个) 的对象. 为了确保其行为不受其他对象的影响, 你可以通过模拟其他对象的行为来隔离你要测试的对象.

当你要测试的单元依赖了一些很难纳入单元测试的情况时 (例如要测试的单元依赖数据库/文件操作/第三方服务 等情况的返回时), 使用 mock 是非常有用的. 简而言之, Mock 是模拟其他依赖的 behaviour.

### 常见测试工具
- [Mocha](https://github.com/mochajs/mocha)
- [ava](https://github.com/avajs/ava)
- [Jest](https://github.com/facebook/jest)

## 集成测试
相对于专注微观模块的单元测试，集成测试是从宏观整体的角度发现问题，所以也称为组装测试和联合测试。[Travis CI](https://travis-ci.org/) 是一款优秀的持续集成工具，可以监听 Github 项目的更新，便于开源软件的集成测试。使用 Travis CI 需要在项目的根目录下创建 `.travis.yml` 配置文件（以 Node.js 为例）：

```javascript
language: node_js

node_js:
    - "6"
    - "5"

before_script:

script:
    - npm test
    - node benchmark/index.js

after_script:
```

默认情况下，Travis CI 会自动安装依赖并执行 `npm test` 命令，通过 `script` 字段可以自定义需要执行的命令，其完整的生命周期包括：

1. Install `apt addons`
2. `before_install`
3. `install`
4. `before_script`
5. `script`
6. `after_success` or `after_failure`
7. OPTIONAL `before_deploy`
8. OPTIONAL `deploy`
9. OPTIONAL `after_deploy`
10. `after_script`

## 基准测试
基准测试使用严谨的测试方法、测试工具或测试系统评估目标模块的性能，常用于观测软硬件环境发生变化后的性能表现，其结果具有可复现性。在 Node.js 环境中最常用的基准测试工具是 [Benchmark.js](https://benchmarkjs.com/docs)，安装方式：
```javascript
npm install --save-dev benchmark 
```
基本示例：
```javascript
const Benchmark = require('benchmark');
const suite = new Benchmark.Suite;

suite.add('RegExp#test', function() {
    /o/.test('Hello World!');
})
.add('String#indexOf', function() {
    'Hello World!'.indexOf('o') > -1;
})
.on('cycle', function(event) {
    console.log(String(event.target));
})
.on('complete', function() {
    console.log('Fastest is ' + this.filter('fastest').map('name'));
})
// run async
.run({ 'async': true });
```
你可以将同一个功能的不同实现基于同一个标准来比较不同实现的速度, 从而得到最优解.

黑盒级别的基准测试, 则推荐 [Apache ab](https://httpd.apache.org/docs/2.4/programs/ab.html) 以及 [wrk](https://github.com/wg/wrk) 等。:
## 代码覆盖率

  测试覆盖率 (Test Coverage) 是指代码中各项逻辑被测试覆盖到的比率, 比如 90% 的覆盖率, 是指代码中 90% 的情况都被测试覆盖到了.

  覆盖率通常由四个维度贡献:

- 行覆盖率 (line coverage) 是否每一行都执行了？

- 函数覆盖率 (function coverage) 是否每个函数都调用了？

- 分支覆盖率 (branch coverage) 是否每个if代码块都执行了？

- 语句覆盖率 (statement coverage) 是否每个语句都执行了？

常用的测试覆盖率框架 [istanbul](https://github.com/gotwarlost/istanbul).

覆盖率工具根据测试用例覆盖的代码行数和分支数来判断模块的完整性。AVA 推荐使用 `nyc` 测试代码覆盖率，安装 nyc：

```javascript
npm install nyc --save-dev
```
修改 `.gitignore` 忽略相关文件：
```javascript
node_modules
coverage
.nyc_output
```
修改 `package.json` 中的 `test` 字段：
```javascript
{
    "scripts": {
        "test": "nyc ava"
    }
}
```
执行 `npm test`，得到：
```
➜  test-in-action (master) ✔ npm test

> test-in-action@1.0.0 test /Users/sean/Desktop/test-in-action
> nyc ava

   2 passed
----------|----------|----------|----------|----------|----------------|
File      |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
----------|----------|----------|----------|----------|----------------|
----------|----------|----------|----------|----------|----------------|
All files |      100 |      100 |      100 |      100 |                |
----------|----------|----------|----------|----------|----------------|

```
## 压力测试
压力测试 (Stress testing), 是保证系统稳定性的一种测试方法. 通过预估系统所需要承载的 QPS, TPS 等指标, 然后通过如 [Jmeter](http://jmeter.apache.org/) 等压测工具模拟相应的请求情况, 来验证当前应能能否达到目标.

对于比较重要, 流量较高或者后期业务量会持续增长的系统, 进行压力测试是保证项目品质的重要环节. 常见的如负载是否均衡, 带宽是否合理, 以及磁盘 IO 网络 IO 等问题都可以通过比较极限的压力测试暴露出来.

## Assert
断言 (Assert) 是快速判断并对不符合预期的情况进行报错的模块. 是将:

```
if (condition) {
  throw new Error('Sth wrong');
}
```
写成:

```
assert(!condition, 'Sth wrong');
```
等等情况的一种简化. 并且提供了丰富了 `equal` 判断, 对于对象类型也有深度/严格判断等情况支持.

Node.js 中内置的 `assert` 模块也是属于断言模块的一种, 但是官方在文档中有注明, 该内置模块主要是用于内置代码编写时的基本断言需求, 并不是一个通用的断言库 (**not intended to be used as a general purpose assertion library**)

### 常见断言工具
- [Chai](https://github.com/chaijs/chai)
- [should.js](https://github.com/shouldjs/should.js)

## 参考
http://ourjs.com/detail/5738493888feaf2d031d24fa
https://elemefe.github.io/node-interview/#/sections/zh-cn/test?id=mock